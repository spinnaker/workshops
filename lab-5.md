#
Lab 5: Deployment Safeguards

Previous Lab: [Lab 4](/lab-4.md)

Jump to [TOC](/README.md)

---

**Goal:** Understand some of the features built into Spinnaker for safer deployments

In this lab, we're going to look at some manual and automated safeguards you can add to Spinnaker.

**Manual Rollback**

One of the more powerful features in Spinnaker is the ability to easily roll back a bad deployment in one step. Spinnaker can automatically roll back to a previous version if the version is still deployed but disabled.

1. Navigate to the clusters view and select your active `prod` server group

1. Select `Server Group Actions`->; `Rollback`

1. Under `Restore To`, Select a previously disabled server group

1. Click `Submit`

Check out the `Tasks` tab to see the specific steps Spinnaker took to roll back the cluster

### **Pipeline Safeguards**

**Adding a manual judgment**

Manual judgment is useful when you need to control authorization to specific accounts, or you want a human to judge whether or not something should proceed. You can put a custom message in the manual judgment text to give humans context to make decisions

1. Go to your `deploy to prod pipeline` and click on `configure`

1. Click on `Configuration` (the first graph node), then `Add Stage` to add a `manual judgment stage`
Clicking on `Configuration` ensures the new stage is added at the beginning.

1. Enter some text for the instructions for this manual judgment stage

1. Ensure `Propagate Authentication` is checked

1. Make the stage run before the find image stage

    1. On manual judgment stage, ensure there are no stages in the `Depends On` list

    1. In the find image stage, change the depends on field to add the `Manual Judgment` stage

1. Run your pipeline

    1. In the manual judgment, select `stop`. See what happens

    1. Run your pipeline again, this time selecting `continue` as your manual judgment option

**Add a deployment window**

Deployment windows are really useful if you have automated triggers but want a pipeline to only run at specific times (like, only deploy to production during the workday).

1. Go to your `deploy to dev` pipeline

1. Select the `deploy` stage

1. Under the `Execution Options` header, check the `Restrict execution to specific time windows`

    1. Click on `Add execution window`
    
    1. Select a time of day that is 5 hours from now
    
    1. Save your pipeline
    
    1. Go back to the executions screen and run your `deploy to dev` pipeline
    
    1. See how the execution window stops at the deploy stage
    
    1. Mouse over the stage and click on `Skip remaining window` to start the pipeline immediately

1. Click on the blue bar \(your running execution\) to see the steps the pipeline is taking

1. Notice the `Restrict execution during time window` step that is already complete

Next Lab: [Lab 6](/lab-6.md)

**Resources:**

1. Safe Deployments Code Lab: [https://www.spinnaker.io/guides/tutorials/codelabs/safe-deployments](https://www.spinnaker.io/guides/tutorials/codelabs/safe-deployments/)

2. Safer Deployment blog post: [https://blog.spinnaker.io/can-i-push-that-building-safer-low-risk-deployments-with-spinnaker-a27290847ac4](https://blog.spinnaker.io/can-i-push-that-building-safer-low-risk-deployments-with-spinnaker-a27290847ac4)

3. Spinnaker Summit Videos

* Kayenta: Automated Canary Analysis - [https://www.youtube.com/watch?v=LqquUa-IT28](https://www.youtube.com/watch?v=LqquUa-IT28)
* Canary Analysis at Netflix - [https://www.youtube.com/watch?v=XB\_En3bsUwM](https://www.youtube.com/watch?v=XB_En3bsUwM)
* Automated Canary Analysis Panel - [https://www.youtube.com/watch?v=JeAV1NBmU-Y](https://www.youtube.com/watch?v=JeAV1NBmU-Y)



