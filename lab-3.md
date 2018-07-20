# Lab 3: Deployment Pipelines

Previous Lab: [Lab 2](/lab-2.md)

Jump to [TOC](/README.md)

---

**Goal:** Understand the different parts of a Spinnaker deployment pipeline, automate deploying a new version of your application.

Creating a dev Deployment pipeline from docker registry

Go to the clusters view and make sure there are no server groups without a load balancer configured. Remove any server group without a load balancer by clicking on it and selecting `Destroy` on the sidebar. This probably includes your `devclone` cluster and early versions of your `dev` cluster.

**Creating a Deployment Pipeline**

1. Click on the `Pipelines` tab for your application

1. Click `Configure a new pipeline`

1. Enter `deploy to dev` as your pipeline name

1. Leave `Create From` set to `pipeline`

1. Click `create`

**Adding a Trigger**

Triggers allow you to run pipelines off of events. We want our pipeline to run every time a new Docker tag is pushed to the Docker registry, and deploy that new image to our `dev` cluster, so we will use a Docker trigger.

1. On the `Configuration` section of the pipeline, under `Automated Triggers`, click `Add Trigger`

1. Select `Docker Registry` as the type

1. Select `spinnaker` as the organization for your trigger

1. Select `spinnaker/workshop-demo` as your image

1. Leave the `Tag` field blank. This will trigger on all _new_ tags, but will not trigger on a re-push of an existing tag \(like `latest`\)

**Adding a Deploy Stage**

When the pipeline is run, we want it to deploy a new server group with the new version of the container. We tell Spinnaker what to deploy using the Deploy stage.

1. Under the green dot labeled "Configuration" at the top of the page, click `Add stage`

1. Select `Deploy` as the stage type

1. Under Deploy Configuration, click `Add server group`. This is where we configure the container settings.

1. Select your `dev` pipeline as the template, then click `Use this template`. We don't want to copy all the options by hand, so we start using our existing deployment.

1. Under containers, remove the existing image selected by clicking on the garbage bin.

1. Under containers, select the option with`Images from Triggers (Tag resolved at runtime)`

1. Under strategy, select `highlander` \(see the Vocabulary Reference at the end of this guide for an explanation of the different strategies\)

1. Scroll down to the `Container` section

    1. Set `Pull Policy` to `Always`  

    1. Under `Ports`: make sure the `Container Port` is `80`

    1. Under `Probes`: make sure there is a readiness probe \(click `Enable Readiness Probe` \) configured with a 10 second `Initial Delay` and a port of `80`

1. Click the blue `Add` button at the bottom of the modal

1. Click `Save Changes` at the bottom of the page to save your pipeline

**Running the Pipeline**

1. Click on the blue arrow next to the name of the pipeline to go return to the `Pipelines` tab, or click on the `Pipelines` tab in the navigation bar.

1. Click on `Start manual execution` on the right side of the blue bar that your new pipeline is in.

1. Select a different color, represented by a different tag name, as the Tag in the drop down

1. Click `Run`. This will start the pipeline

    1. You'll see a blue bar appear that shows you what your pipeline is doing. If it doesn't appear, try clicking the arrow next to the pipeline to display it. Click on that blue bar to see the steps in the pipeline.

1. Click on the clusters view. You should see a spinning ball next to the clusters being worked on

In the next section, we'll create a "deploy to production" pipeline that reads from our development cluster. This captures the typical "promote an image that has been running in dev" flow. This pipeline _won't_ run after every commit.

**Creating a Prod load balancer**

1. Before we continue, we need to add a new load balancer. These steps are the same as in Lab 2. Create a Load Balancer, except that instead of putting it in the `dev` stack, we'll put it in `prod.`

1. Click `Load Balancers` on the top right navigation bar.

1. Click `Create Load Balancer`. Select `LoadBalancer` as the `Type` under Advanced Options.

1. Click `Create`

**Create a new pipeline**

1. Click on the `Pipelines` tab. Click on the `+ Create` button and create a new pipeline called `deploy to prod`

1. Leave `Create From` set to `pipeline`

**Add a Find Image Stage**

We want to run this pipeline manually, so we are not going to include a trigger. Instead, the first stage will find the image we want to deploy.

1. Click `Add stage`

1. Set stage type to `Find Image from Cluster`

1. Select `default` for the Namespaces

1. Click `Toggle for list of existing clusters` and select your dev cluster

1. Set `Server Group Selection` to `Newest`

**Add a Deploy Stage**

In this stage, we're going to take the image that we found in the dev cluster and deploy it to a new prod cluster.

1. Click `Add stage`

1. Set stage type to `Deploy`

1. Click `Add server group`

1. Under `Copy configuration from`, select you dev cluster, then click `Use this template`

1. Change the stack of the pipeline from `dev` to `prod`

1. Next to Containers, Delete the selected image under containers by clicking on the garbage bin.

1. Add the option under `Find Image Results` in from the containers drop down.

1. For `Strategy`, select `Red/Black`. In the options for that strategy, set `Maximum number of server groups to leave` to `2`.
Notice that this is a different strategy than the `deploy to dev` pipeline. For our prod pipeline we want the previous server group to hang around in case we need to quickly roll back. We don't want many disabled previous versions hanging around, so we set the max value to leave to equal 2.

1. Under the `Load Balancers` heading, remove the `dev` load balancer and add the `prod` load balancer you created in step 1.

1. Under the `Container` heading,

    1. `Pull Policy` set to `ALWAYS`

    1. Under `Ports` make sure the container port is set to 80

    1. Under `Probes` a readiness probe on port 80 with a 10 second `Initial Delay`, just like before.

1. Save your Pipeline

**Manually Run your Pipeline**

1. Click on the blue arrow next to the name of the pipeline to return to the executions view

1. Click on start pipeline and select `deploy to prod`

**Check out your "Prod" app**

1. Click on the load balancer icon on your prod server group

1. Copy the ingress link to a new tab to view your app

**Manually Run your Pipeline a Second Time**

1. Observe the difference in the strategy you chose: `Red/Black` leaves the previous server group up but disabled.

1. You should have two 'prod' server groups - one should be green \(healthy\) and one should be greyed out \(disabled\)

---

Next Lab: [Lab 4](/lab-4.md)

---

**Resources:**

* Kubernetes Source to Prod Code lab - [https://www.spinnaker.io/guides/tutorials/codelabs/kubernetes-source-to-prod/](https://www.spinnaker.io/guides/tutorials/codelabs/kubernetes-source-to-prod/)
* Google Source to Prod video. This shows the pipelines involved in a VM deployment. In particular, there is a bake stage that bakes an virtual image for deployment with a library called Rosco - [https://www.spinnaker.io/guides/tutorials/videos/\#google-source-to-production-codelab-walk-through](https://www.spinnaker.io/guides/tutorials/videos/#google-source-to-production-codelab-walk-through)



