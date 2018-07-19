# Lab 2: Infrastructure Management

Previous Lab: [Lab 1](/lab-1.md)

**Goal:** Understand how to use Spinnaker to manage and view your cloud infrastructure.

In this section, we will adjust the infrastructure we created in the first section, and create new infrastructure.

## Manual Operations
Sometimes, you'd like to manually scale up a server group to contain more instances.

**Resize server group to have more instances**

1. Select the `dev` server group you created in Lab 1

1. Click on `Server Group Actions` -> `Resize`

1. Change desired size to `2` instances

1. Click `Submit`

Soon, you'll see a new instance launching.

**Replace all the pods in the server group** 

1. Next to the page heading, where it says `Clusters` , click on the `Edit multiple` button. Enable `with details`

1. Click on the checkbox next to `instance` in your server group. Take a note of the launch times of the pods.

1. On the sidebar, select `Actions` -> `terminate`

1. Enter the validation code and click on `Terminate 2 instances`

1. You should see the pods terminate and new ones launch to replace them. The new pods will have a new launch time. Inspect the launch time by clicking on an instance, and looking at the righthand sidebar `Instance Information` section.

1. Click on the `Tasks` tab. You should see a record of all these manual actions.

**Creating a load balancer**

Load balancers in Spinnaker represent services in Kubernetes. Let's create a load balancer so we can access our pod.

1. Click on `Load Balancers` on the top right navigation bar

1. Click on `Create Load Balancer` button

1. Under stack, enter `dev`, to indicate that this load balancer will connect to the instances in the `dev` cluster

1. Under advanced settings, enter `LoadBalancer` as the `type`

1. Click `create`

1. In a couple of minutes, you should see a load balancer named `<yourname>-dev`
It might take a couple of more minutes for your load balancer to get a dynamic IP (`Ingress`)

**Attaching your load balancer**

You can only attach a load balancer to newly created infrastructure.

1. Navigate to the clusters view and select the server group you created in lab 1.

1. Select `Server Group Actions` -> `Clone`

1. Under the `Load Balancers` heading, select the load balancer you just added \( you can type the name \)

1. Click `Create`

1. You should now see a small icon next to the cluster indicating that it has a load balancer attached. It is the same icon as next to the `Load Balancer` heading

To make sure everything works, click on the small load balancer icon, it should show you the details of the load balancer. There should be a number under `ingress` on the sidebar. This is the IP to use to access your load balancer. Open that IP in a new tab. It should take you to a page that says `hello` with a colorful background. If the IP link completes as `https`you won't be able to access the container. Change it to `http` and refresh.

Now, you have all the basics for a simple deployment. Your application, a simple webpage that says `hello`, is deployed and under a load balancer. You can access it from a single IP address. Traffic is balanced between the containers in the server group.

Next, we will explore how to set up a pipeline that will deploy a new version of this application without downtime, and without these manual clone steps.

**Cleaning Up**

Delete every server group that **does not** have a load balancer attached. Delete each server group by selecting the server group, clicking on `Server Group Actions`, and then click on Destroy.

At the end of this clean up you should be left with one server group that has a load balancer attached \(this will be the most recently created server group\).

Next Lab: [Lab 3](/lab-3.md)
