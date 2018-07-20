# Lab 7: Canaries

Previous Lab: [Lab 6](/lab-6.md)

Jump to [TOC](/README.md)

---

In this lab we are going to go through the canary process. We've already launched the infrastructure (application `canarylab`) for you and it has been running (and publishing metrics) for a while. In this lab we will create the canary config and pipelines in our existing application but we will pull metrics from the clusters that are already running in the `canarylab` application. This allows us to do a retrospective analysis of the metrics they've been publishing for the last few hours instead of waiting for the containers to launch and publish metrics for a reasonable amount of time.

We have two versions of the docker image for canaries. These versions emit metrics to Stackdriver. Kayenta is configured to understand these Stackdriver metrics with a little help from the Canary config.

| Canary Run | Docker Tag |
| :--- | :--- |
| AA Canary | canarylab-aa-\* |
| AB Canary | canarylab-ab-\* |

**Setup**

Canaries need to be enabled in your application. To do that, click on your application `CONFIG` on the top right. Scroll down to `Features`, click the checkbox to enable `Canary`, and hit `Save`. Refresh the page. You should see a new top level menu called `Delivery` (it replaced `Pipelines`). The Delivery tab now contains pipelines as well as the canary info.

**Background**

We've modified the container used in the previous labs to emit stackdriver metrics. There are three metrics that it emits, named as follows:

* custom.googleapis.com/workshop/canary/request/errors
* custom.googleapis.com/workshop/canary/request/random1
* custom.googleapis.com/workshop/canary/request/random2

The values of these metrics change based on the name of the server group deployed. The full code is on github and linked in the Canary Reference section.

**Create Canary Config**

Canary configs tell Kayenta which metrics to look at and what indicates success and failure. We're going to create a canary config in our application to look at the three metrics show above.

An example config is in the application `canarylab`, and is called "MultipleMetrics". Feel free to reference that config if you get stuck.

1. Navigate to the `Delivery` -> `Canary Configs` tab.

1. Create a new configuration by clicking on `+ Add configuration`

1. Name the config `<your-app-name>-multipleMetrics-config`

1. We need to tell the canary config how to parse our metric. In the `Filter Templates` section \(below `Metrics`\) click `Add Template`
    
    1. Name that template `filterOnServerGroupName`
    
    1. In the template box, add `metric.labels.servergroup="${scope}"`
    An explanation: The canary stage has baseline and canary fields, which will be populated via expressions with servergroup names. That defines the value of `${scope}`. The application we’re canarying tags its stackdriver metrics with the key “servergroup” containing the name of the spinnaker server group the application is running in. A filter template is how we tell Kayenta how to read metrics that are specific to the server groups being evaluated.
    
    1. Hit `OK` to save

1. Now we will add a metric that uses the template we just created. Under `Metrics` click `Add Metric`
    
    1. Call this metric `Errors`
    
    1. For `Filter Template` select the template you just created
    
    1. For `Metric Type` select `custom.googleapis.com/workshop/canary/request/errors`
    1. Hit `OK`

1. Repeat the above steps with two more metrics:
    
    1. Name: `Random1` , Metric Type: `custom.googleapis.com/workshop/canary/request/random1`
    
    1. Name: `Random2`, Metric Type: `custom.googleapis.com/workshop/canary/request/random2`

1. Scroll to the bottom of the config and take a look at the `Scoring` section. In the `Metric Group Weights` box set `Group 1` to have a value of 100. This means that metric group 1 \(the only group we have\) is weighted at 100%, and is the only thing that affects the score.

1. Save the config

This canary config can now be used in canary pipelines.

**Create an AA Canary Pipeline**

The canary config is used in a canary stage. We've already deployed the infrastructure for you, so we will be doing a retroactive canary for the past three hours on this deployed infrastructure.

First we will set up an AA Canary to evaluate our metric. We expect this canary to get a perfect score since our metric should reliably indicate a change, and therefor not show a difference when measured against itself.

The pipeline we are modeling this on is in the `canarylab` application and is called `Canary AA`

1. Create a new pipeline in your application called `Canary AA`

1. Add a `Find Image From Cluster` stage and call it `FindBaseline`. Note that you must match this name exactly for the expressions to work below.

    1. For `Account` ensure `workshop` is selected
    
    1. For `Namespaces`check `production`
    
    1. For `Cluster` type `canarylab-aa-baseline`
    
    1. For `Server Group Selection` select `Newest`
    
    1. Check `Only consider enabled Server Groups`

1. Repeat step 2 and call this second `Find Image From Cluster` stage `FindCanary` .Note that you must match this name exactly for the expressions to work below. Make sure both extend from `Configuration` so that the run in parallel. To do this, make sure that neither stage has anything in the `Depends On` box.
    
    1. Change the `Cluster` for this stage to `canarylab-aa-canary`

1. Create a new stage with type `Canary Analysis`.
   This stage should depend on the previous two stages. In the `Depends On` configuration for the stage, add both `FindBaseline` and `FindCanary`

    1. For `Analysis Type`, select `Retrospective`

    2. For `Config Name` select the config you just created

    3. For `Interval`, select 15 minutes

    4. Now comes the expression fun. Under `Metrics Scope` we are going to use expressions to correctly fill out the `Baseline` and `Canary` fields

        1. Set `Baseline` to equal `${ #stage('FindBaseline')['context']['artifacts'][0]['metadata']['sourceServerGroup'] }`

        1. Set `Baseline Location` to equal `us-central1`

        1. Set `Canary` to equal `${ #stage('FindCanary')['context']['artifacts'][0]['metadata']['sourceServerGroup'] }`

        1. Set `Canary Location` to equal `us-central1`

        1. Set `Step` to 60

        1. Use [http://www.timestampgenerator.com/](http://www.timestampgenerator.com/) to generate ISO timestamps for now and three hours ago, and set `Start Time` to three hours ago and `End Time` to now. ISO-8601 standard format looks like `2018-07-12T20:28:29+00:00` - however for these two fields to work correctly we need to replace the `+00:00` with `Z`. Yes, UX improvements are coming. A sample usable date time string looks like: `2018-07-12T20:28:29Z`

        1. For `Resource Type` select `gce_instance`

    1. Under `Advanced Settings` set both Metrics Account and Storage Account to `my-google-account`

1. Save the pipeline

**Do the Canary**

Run the pipeline.

You'll see this pipeline run really quickly, because it is evaluating metrics that were already emitted over the past 3 hours. In each Find Image task you'll see that the image is the same: `index.docker.io/spinnaker/workshop-demo:canarylab-AA`

In the canary analysis details you'll see the score - a 100. That means that our AA canary does not have a difference when compared to itself. That is what we want to see.

1. Click on `Canary Reports`. The first entry in the table is the canary you just ran

1. Click on `Report` , the last field in the row. This will show you the detailed canary report

1. There's a graph for each of the metrics we're evaluating, click on each and look at the metrics graph

1. You can see that the metrics were incredibly similar - this gave us the score of 100.

If you were canarying a new version of your software, you'd want to deploy the infrastructure, run the canary stage, and then clean up your infrastructure in the same pipeline.

**AB Canary **

Now that we think our metrics are an ok judge \(extremely similar/ passing score in an AA\), we will do an AB canary.

For this canary we will do a "real time" canary - we will gather metrics every _interval_. This means that the canary will take longer to run. We're going to do a very short run in this lab, but normally you'd run a canary over a couple of hours.

1. Create a new pipeline called `Canary AB` and set `Copy From` to be the pipeline you just created, "Canary AA"

1. In the `FindBaseline` stage, update the `Cluster` to contain `canarylab-ab-baseline`

1. In the `FindCanary` stage, update the `Cluster` to contain `canarylab-ab-canary`

1. In the `Canary Analysis` stage:

    1. Update `Analysis Type` to be `Real Time`

    1. Set `Interval` to 5 minutes

    1. Set `Lifetime` to 6 minutes

1. Save and run the pipeline.

After 6 minutes, you'll see the score. Once the pipeline has finished, you can quick link to the canary report from the stage detail by clicking the bar graph icon.

**Notes**

In this example, we've looked at three metrics. One of the metrics had a large difference (the error metric). You can see the canary score of `66` shows that two metrics passed while one metric failed.

The difference for this metric was pretty drastic - it would be easy to tell with your eyes that there was a difference. In most cases, the difference is not this drastic. This is where Kayenta comes in - Kayenta can tell you with confidence that there is a difference in the metrics.

If you wanted this one metric to fail the whole canary with a score of `0`, you can update your canary config to have that behavior. In the `Metrics` section of the canary config edit your metric (`Errors`) and check the box for `Fail the canary if this metric fails`. However, this makes the score of the canary less obvious.

---

Next Lab: [Lab 8](/lab-8.md)

---

**Extra Resources**

For an overview of why Netflix uses canaries, check out this [blog post](https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69). \( [https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69](https://medium.com/netflix-techblog/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69) \)

For detailed documentation, check out the [documentation about Canaries](https://www.spinnaker.io/guides/user/canary/). \([https://www.spinnaker.io/guides/user/canary](https://www.spinnaker.io/guides/user/canary)\) Some general points are below.

