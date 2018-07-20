# Setup with hal + GKE (7/2018)

Jump to [TOC](/README.md)

---

This doc details the steps we took to set up the infrastructure for the workshop given in August 2018. 
There might be steps missing but it is here to serve as a reference for some one trying to replicate this setup.

We used `hal` and GKE

Project: qcon-2017-workshop

Kub cluster name: oscon2018

Change the keystore passwords if you are following this.

## GKE k8s setup notes:

Legacy Auth must be enabled (it isn’t by default) when creating the cluster for halyard to work with it. If not initially enabled, you may need to delete and recreate the cluster, enabling it after the fact doesn’t work reliably as of gke 1.9.7-3.

CLI to create a new cluster (once gcloud is setup below) using:
* 9 “n1-standard-8” instances in a single zone

The resulting cluster has 72 cores + 270Gb ram.

```bash
gcloud beta container \
  --project "qcon-2017-workshop" \
  clusters create "oscon2018" --zone "us-central1-a" \
  --username "admin" --cluster-version "1.9.7-gke.3" \
  --machine-type "n1-standard-8" --image-type "UBUNTU" \
  --disk-type "pd-standard" --disk-size "100" \
  --scopes "https://www.googleapis.com/auth/compute","https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" \
  --num-nodes "9" --enable-cloud-logging \
  --enable-cloud-monitoring --network "default" \
  --subnetwork "default" --enable-legacy-authorization \
  --addons HorizontalPodAutoscaling,HttpLoadBalancing,KubernetesDashboard \
  --no-enable-autoupgrade --no-enable-autorepair
```

## Prerequisites:
K8s cluster in google cloud
Docker
Docker Hub credentials

We’re going to create a spinnaker deployment on kubernetes
( I did this with ubuntu 16 running on parallels desktop for mac, doing so because some of the commands in the guide don’t really work with the docker version of HAL )

## Setup Hal

```bash
curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/debian/InstallHalyard.sh

sudo bash InstallHalyard.sh

#Enter: parallels when asked about providing a non root user

hal -v

sudo update-halyard
```


## Setup Gcloud 
( see https://cloud.google.com/kubernetes-engine/docs/quickstart )

Create environment variable for correct distribution

```bash
export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"
```

Add the Cloud SDK distribution URI as a package source

```bash
echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
```

Import the Google Cloud Platform public key

```bash
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Update the package list and install the Cloud SDK

```bash
sudo apt-get update && sudo apt-get install google-cloud-sdk

gcloud init
```

Login w/ your google credentials in the browser

Select your project ( qcon-2017-workshop )

## Setup Kubectl

```bash
sudo apt get install kubectl

gcloud container clusters get-credentials oscon2018 --zone us-central1-a

kubectl config current-context

#Should see something like gke_qcon-2017-workshop_us-central1-a_oscon2018
```

## Add Kubernetes V1 Provider
Docker Registry

Use any user or service account.

```bash
hal config provider docker-registry enable

hal config provider docker-registry account add dockerhub --address index.docker.io --repositories spinnaker/workshop-demo --username tomaslin --password
```

Provider ( lab uses v1 provider because v2 is manifest based so basically huge text input box )

```bash
kubectl get namespace

gcloud config set container/use_client_certificate True

export CLOUDSDK_CONTAINER_USE_CLIENT_CERTIFICATE=True

kubectl create -f https://kubernetes.io/examples/admin/namespace-prod.json 

kubectl create -f https://k8s.io/examples/admin/namespace-dev.json 

hal config provider kubernetes enable

hal config provider kubernetes account add workshop --docker-registries dockerhub --context $(kubectl config current-context)
```

## Add Persistence Bucket

Get service account ( you need to have owner’s permission in gcp )

```bash
SERVICE_ACCOUNT_NAME=spinnaker-gcs-vel-account
SERVICE_ACCOUNT_DEST=~/.gcp/gcs-account.json

gcloud iam service-accounts create \
    $SERVICE_ACCOUNT_NAME \
    --display-name $SERVICE_ACCOUNT_NAME

SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:$SERVICE_ACCOUNT_NAME" \
    --format='value(email)')

PROJECT=$(gcloud info --format='value(config.project)')

gcloud projects add-iam-policy-binding $PROJECT \
    --role roles/storage.admin --member serviceAccount:$SA_EMAIL

mkdir -p $(dirname $SERVICE_ACCOUNT_DEST)

gcloud iam service-accounts keys create $SERVICE_ACCOUNT_DEST \
    --iam-account $SA_EMAIL
```

## Create Bucket

```bash
PROJECT=$(gcloud info --format='value(config.project)')

s# see https://cloud.google.com/storage/docs/bucket-locations

BUCKET_LOCATION=us

hal config storage gcs edit --project $PROJECT \
    --bucket-location $BUCKET_LOCATION \
    --json-path $SERVICE_ACCOUNT_DEST \
    --bucket oscon2018

hal config storage edit --type gcs
```

## Setup SSL

Anywhere a passphrase was used, set it to nosecrets

Note, the java keystore/trustore (required by gate) requires a passphrase, but the apache style cert setup used by deck does not.
Generate a *.spinnaker.tk certificate via Let’s Encrypt (current keys expire in 10/2018)

Install certbot - see https://certbot.eff.org/lets-encrypt/ubuntuxenial-apache

Generate a wildcard cert for *.spinnaker.tk:

```bash
sudo certbot certonly -d \*.spinnaker.tk \
  --manual --server https://acme-v02.api.letsencrypt.org/directory \
  --preferred-challenges dns
```
Under something like /etc/letsencrypt/live/spinnaker.tk (only readable by root) you’ll get the following files which you’ll need: fullchain.pem, privkey.pem, chain.pem. Copy these to a folder readable by hal, i.e. ~/spin-ssl

```bash
cd ~/spin-ssl # where above files were copied
```

## Gate Java TrustStore/KeyStore setup

```bash
cat fullchain.pem privkey.pem > wildcard.pem

openssl pkcs12 -export -out wildcard.pkcs12 \
  -in wildcard.pem -name spinnaker \
  -password pass:nosecrets

keytool -v -importkeystore -srckeystore wildcard.pkcs12 \
  -destkeystore wildcard.jks -deststoretype JKS

keytool -trustcacerts -keystore wildcard.jks \
  -importcert -file chain1.pem

KEYSTORE_PATH=~/spin-ssl/wildcard.jks

hal config security api ssl edit --key-alias spinnaker \
  --keystore $KEYSTORE_PATH --keystore-password \
  --keystore-type jks --truststore $KEYSTORE_PATH \
  --truststore-password --truststore-type jks

hal config security api ssl enable
```

## Deck SSL Setup (easier!)

```bash
cp privkey.pem wildcard.key

cp fullchain.pem wildcard.crt

SERVER_CERT=wildcard.crt

SERVER_KEY=wildcard.key

hal config security ui ssl edit --ssl-certificate-file $SERVER_CERT --ssl-certificate-key-file $SERVER_KEY

hal config security ui ssl enable
```

Note: a tls config issue made here could prevent either deck or gate from starting once hal deploy apply is run, check their logs if so - it’ll be clear if a startup issue is related
Further reference: https://www.spinnaker.io/setup/security/authentication/ssl/ (only addresses use of a self-signed cert however)

## Setup Domain and api key

Create 2 static ips and follow instructions here basically

https://www.spinnaker.io/setup/quickstart/halyard-gke-public/

For domains, I set up a free domain at spinnaker.tk
https://my.freenom.com/clientarea.php?action=domains

## Enable Oauth
Follow this: https://www.spinnaker.io/setup/quickstart/halyard-gke-public/

Note that when you set up domain you need to not provide anything so 

```bash
hal config security authn oauth2 edit --provider google \
    --client-id your-client-id \
    --client-secret your-client-secret

hal config security authn oauth2 enable
```

## Add Email

Create file in .hal/profiles/echo-local.yml

```yaml
spinnaker:
  baseUrl: http://lab.spinnaker.tk
 mail:
  enabled: true
  from: youremail
spring:
  mail:
    host: smtp.gmail.com
    username: youremail
    password: yourpass
    properties:
      mail:
        smtp:
           auth: true
          ssl:
            enable: true
          socketFactory:
            port: 465
            class: javax.net.ssl.SSLSocketFactory
            fallback: false
```

Edit setting.js for deck
???

## Enable MPT

```bash
hal config features edit --pipeline-templates true

#Deploy Spinnaker
hal version list 

hal config version edit --version 1.8.0

hal config deploy edit --account-name workshop \
  --type distributed

hal deploy apply   

#Enable stackdriver metrics
hal config metric-stores stackdriver edit --project qcon-2017-workshop

hal config metric-stores stackdriver enable
```

## Edit K8S services to enable external TLS access, bound to our static IP’s

```bash
kubectl edit svc spin-deck -n spinnaker
# change port to 443 (or 80 if TLS wasn’t setup above), change type: from ClusterIP to LoadBalancer, add line “loadBalancerIP: $UI_ADDRESS” bellow type, where $UI_ADDRESS is the static IP setup earlier and assigned to lab.spinnaker.tk.

kubectl edit svc spin-gate -n spinnaker

# same changes as for deck, but set loadBalancerIP: to the static IP created earlier and assigned to api.spinnaker.tk.
#See also: https://www.spinnaker.io/setup/quickstart/halyard-gke-public/ 

hal deploy apply 
# after this stage, you should actually be able to login to https://lab.spinnaker.tk/ with google auth and TLS should be correctly validated in your browser
```

## Kayenta Configuration

```bash
hal config canary google account add my-google-account \
  --project qcon-2017-workshop \
  --bucket qcon-2017-workshop-kayenta \
  --bucket-location us-central1 \
  --json-path /home/asher/.gcp/gcs-account.json

hal config canary google edit --gcs-enabled=true --stackdriver-enabled=true

hal config canary enable


hal config canary edit --default-metrics-store stackdriver

hal config canary edit --default-metrics-account my-google-account

hal config canary google enable

hal deploy apply
```

Backup Configuration

## Cleanup
To remove access for users simply remove the redirect link in the api token to api.spinnaker.tk. Should also delete all the static ip addresses and tear down the kube cluster
