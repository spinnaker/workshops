# Lab 4: Notifications

Previous Lab: [Lab 3](/lab-3.md)

Jump to [TOC](/README.md)

---

**Goal:** Understand how to receive notifications when something happens in Spinnaker.

In this lab, we'll add email notifications to our deployment pipelines. There are several other types of notifications that you can add (including slack).

You can add notifications at three different levels in Spinnaker: stage level, pipeline level, and application level.

**Add a stage level notification**

1. Click `Configure` on your `deploy to dev` pipeline

1. Select your deploy stage and click the checkbox that says `send notifications for this stage`

1. Click `+Add Notification Preference` and configure a new email notification for when the stage starts. Enter a custom message and your email address. Select `Notify when` the stage is starting

1. Start your pipeline

1. Check your email. You should see a notification from Spinnaker. If you don't see it, check your spam!

**Add a pipeline level notification**

1. Remove the stage level notification you added in step 1

1. Configure the same notification, but this time at the pipeline level under the `Configuration` section of your pipeline

1. Re-run the pipeline

**Add an application level notification**

1. Remove that pipeline level notification

1. Now set an application level configuration under the `Config` tab of your application

**Add hoc notifications**

Also notice that you can set an ad-hoc notifications when you start a pipeline manually by checking `Notify me when the pipeline completes`

Next Lab: [Lab 5](/lab-5.md)

**Resources:**

* Events and Notifications Guide: [https://www.spinnaker.io/setup/features/notifications/](https://www.spinnaker.io/setup/features/notifications/)



