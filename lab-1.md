# Lab 1: Spinnaker and the Cloud

**Goal:** This Exercise introduces the concepts of Applications, Clusters and Server Groups and how they are represented in Spinnaker. Explore how to see what actions you've taken in Spinnaker.

Spinnaker organizes application resources into applications, clusters and server groups. Server groups have the following naming convention: `application-stack-detail-sequenceNumber`. As new versions of a new server group are added, the sequence number is incremented. Let's see how that looks in Spinnaker:

**Create an application**

This application will contain all the infrastructure you'll create in this lab

1. In the Application screen, select `Actions` -> `Create Application`

1. Fill out the application creation form. Enter `<yourname>` for the application name  (no spaces, only alphanumeric characters)

1. Enter your email address

1. Leave everything else as is

1. Click `Create`

You'll be dropped into the `Infrastructure` tab. This is blank, because you haven't launched any infrastructure. Visit each tab - note that in the `Tasks` tab, you'll see a task for creating your application. Explore the `Config` tab - are any sections confusing? Can you see the information you just filled out at the top?

**Launch your first server group**

Now we will explore deploying infrastructure with Spinnaker

1. You should be in the `Infrastructure` view of your application.

1. Select `+Create Server Group`

1. Enter `dev` as your stack name

1. Type `workshop` in the container search field, then select the `workshop-demo:red` image

1. Scroll down to the Container section

1. Set `Pull Policy` to `ALWAYS`

1. Expand the Probes subsection

1. Click "Enable Readiness Probe" to add a readiness probe. Set the Initial Delay field value to `10`, leaving the other defaults in place

1. Click `Create`

It'll take a bit of time for your container to appear. You can follow along in the `Tasks` tab.
When the container is deployed, you will see a blue or green chicklet in the `Infrastructure` tab in `Clusters`. Click on it, and explore the right sidebar full of information.
Click on the long white box that the chicklet is a part of. This is the server group. Note how the right sidebar changes to contain information about the server group.

**Cloning a server group to a new stack**

This operation is usually used to create a new cluster with similar settings as the current cluster.

1. Select the server group that was just created by clicking on the white box

1. In the sidebar, select `Server Group Actions` -> `Clone`

1. In the dialog, change the stack field to `devclone`

1. Click `Create`

1. Note how the two clusters, each with one server group, appear in the screen.

**Cloning a server group to the same stack**

This is used to deploy a new version of software, or to update the container/vm settings on the same version.

1. Select the first server group you created \(stack`dev)` and clone again, this time leaving the stack name as `dev`. Keep an eye on the version number.

1. This new server group will have the same container as the existing version. To change that, select a different image.

1. Click `Create`

1. Note how the version number changes.

Notice that your `dev` cluster now has 2 server groups in it, each with a different version.

**Inspect the Tasks view**

Navigate to the `Tasks` tab. You'll see a record for each action that you took

1. Click on the most recent \(or top\) task to expand it.

1. Notice the steps that Spinnaker took to complete each action.

1. Which step took the longest?

Bonus: in the lower righthand corner of the expanded task, you'll see a `source` link. Click that, and it'll open the `JSON` representation of this tasks. This is what Spinnaker uses internally, and contains more information than the UI. Search for `authentication` to see your user block. Search for `trigger` to see that this task was manually triggered. Search for `imageId` to see the container that was deployed.

**Resources:**

* Asgard: Web-based Cloud Management and Deployment - [https://medium.com/netflix-techblog/asgard-web-based-cloud-management-and-deployment-2c9fc4e4d3a1](https://medium.com/netflix-techblog/asgard-web-based-cloud-management-and-deployment-2c9fc4e4d3a1)
* Netflix's OSS Frigga Library - [https://github.com/Netflix/frigga](https://github.com/Netflix/frigga)
* Moniker - [https://github.com/spinnaker/moniker](https://github.com/spinnaker/moniker)



