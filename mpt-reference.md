# MPT Reference

Jump to [TOC](/README.md)

---


Detailed docs https://github.com/spinnaker/dcd-spec

[Repo](https://github.com/spinnaker/pipeline-templates/tree/master/workshop) with below templates.

**Simple Wait Template**

```
schema: "1"        
id: wait-template
metadata:
  name: Simple wait template
  description: Extensible root template
  owner: sample@sample.com
  scopes:
    - global
variables:         
- name: waitTime
  description: The time a wait stage should pause
  type: int
stages:
- id: wait1
  type: wait
  config:
    waitTime: "{{ waitTime }}"  # Variables can be used anywhere
```

---

**Deploy to Account Template**

```
schema: "1"
id: template-deploy
metadata:
  description: Deploy to account template
  name: deploy to account
  owner: sample@sample.com
  scopes: 
    - global
protect: false
configuration:
  concurrentExecutions:
    limitConcurrent: true
  triggers:
  - account: dockerhub
    enabled: true
    name: dockertrigger
    organization: spinnaker
    registry: index.docker.io
    repository: spinnaker/workshop-demo
    type: docker
variables:
- name: deployAccount
  description: The account where the application will be deployed to
  defaultValue: workshop
- name: deployStack
  description: The stack to deploy to
  defaultValue: dev
- name: deployTag
  description: The tag of the docker image to deploy
  example: Pick from blue, orange, purple, red, or white
  defaultValue: purple
stages:
- config:
    clusters:
    - account: "{{ deployAccount }}"
      application: "{{ application }}"
      cloudProvider: kubernetes
      containers:
      - args: []
        command: []
        envFrom: []
        envVars: []
        imageDescription:
          account: dockerhub
          imageId: "index.docker.io/spinnaker/workshop-demo:{{ deployTag }}"
          registry: index.docker.io
          repository: spinnaker/workshop-demo
          tag: "{{ deployTag }}"
        imagePullPolicy: ALWAYS
        limits: {}
        name: spinnaker-workshop-demo
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          handler:
            execAction:
              commands: []
            httpGetAction:
              path: /
              port: 80
              uriScheme: HTTP
            tcpSocketAction:
              port: 80
            type: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        requests: {}
        volumeMounts: []
      dnsPolicy: ClusterFirst
      events: []
      initContainers: []
      interestingHealthProviderNames:
      - KubernetesContainer
      - KubernetesPod
      loadBalancers:
      - "{{ application }}-{{ deployStack }}"
      namespace: default
      nodeSelector: {}
      podAnnotations: {}
      provider: kubernetes
      region: default
      replicaSetAnnotations: {}
      securityGroups: []
      stack: "{{ deployStack }}"
      strategy: highlander
      targetSize: 1
      terminationGracePeriodSeconds: 30
      tolerations: []
      volumeSources: []
  id: deploy1
  inheritanceControl: {}
  inject: {}
  name: Deploy
  type: deploy
```

---

**Deploy with Manual Judgment Template**

```
schema: "1"        
id: manual-judge-deploy-template
source: spinnaker://template-deploy
metadata:
  name: deploy with manual judgment
  description: deploy to an account with a manual judgment before hand
  owner: sample@sample.com
  scopes:
    - global
variables:         
- name: judgeMessage
  description: User message for judgment
  defaultValue: Please judge me. Do you want to continue?
stages:
- id: judgy
  type: manualJudgment
  config:
    instructions: "{{ judgeMessage }}"
  inject: 
    first: true
```



