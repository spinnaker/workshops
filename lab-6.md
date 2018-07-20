# Lab 6: Advanced Pipeline JSON

Previous Lab: [Lab 5](/lab-5.md)

Jump to [TOC](/README.md)

---

**Goal:** take advantage of some of the other features Spinnaker has to offer

**Pipeline Json**

You can edit Spinnaker pipelines via the UI, or via the `JSON` of the pipeline. Let's play with the `JSON` version.

1. Create a new pipeline, called `expression`

1. Add a wait stage

1. To the right of the stage name and type, select `Edit stage as JSON`

1. Change `"waitTime"` to 10 (10 seconds)

1. Click `Update Stage`

You'll notice the UI has updated the value to 10 seconds.

You can use Wait stages to easily play around with pipeline flow without actually making any infrastructure changes. It's quite useful.

**Conditional On Expression**

Sometimes, you want to conditionally run certain stages in your pipeline. You can control that with the `Conditional on Expression` checkbox under `Execution Options` on a specific stage.

1. For the wait stage you just created, check the `Conditional On Expression` checkbox

1. Type `false` in the expression box that pops up

1. Save and run your pipeline

Notice how the output looks different. What's the status of the wait stage? Is that the behavior you expected?

**Pipeline Parameters**

You can parameterize pipelines in Spinnaker. This is useful if you need to pass parameters to a Jenkins job, or if you want to do different things in your pipeline based on input \(like conditionally skip a stage\).

Let's add a parameter to our `expression` pipeline.

1. Click on the `Configuration` stage in the pipeline

1. In the `Parameters` section, add a parameter

1. Name it `runStage`, in the description put `Should we run the wait stage?`, and for the default value put `true`
    
    1. You can leave the "Label" field blank, it provides a optional different user-facing name for the parameter

1. Save your pipeline and run it

We don't do anything with the parameter yet, so nothing will change

**Expressions**

We can use the expressions to skip the wait stage based on the parameter we just added. To do that, we use the Spring Expression Language (SpEL). Expressions are a powerful feature in Spinnaker that let you use a placeholder value in your pipeline configuration that gets parsed at runtime. Expressions can read values from any part of an execution `JSON` document, allowing stages to be dynamically configured based on output from other stages, or even other pipelines.

1. Expand the `Details` section of the last run of our `expression` pipeline, then click on the `Source` link in the lower right corner. This loads the `JSON` document representing the pipeline execution. (Note, if your browser doesn't format `JSON` for easy reading, it helps to install an extension such as JSON Lite for Chrome.)

1. Scroll through the `JSON` and note how the `runStage` parameter and its value appears in the execution.

1. Take a look at the [expression documentation](https://www.spinnaker.io/guides/user/pipeline/expressions/) ([https://www.spinnaker.io/guides/user/pipeline/expressions/](https://www.spinnaker.io/guides/user/pipeline/expressions/))

1. Find the section about turning a stage on and off, and use those instructions to turn your wait stage on and off with the parameter you just added

1. Run your pipeline again with both options

**Check Precondition Stage**

The Check Preconditions stage is useful for branching logic. You can control whether a branch in a pipeline should be executed based on some condition \(either an expression or an infrastructure condition\).

1. Add a Check Preconditions stage to your pipeline as the first stage that runs
    
    1. Having trouble getting the stages to execute in the correct order? Ordering is configured by editing the `Depends On` configuration for each stage

1. Select `Add precondition`
    
    1. Change the `Check` to `Expression`
    
    1. For now, set the value to `${ 'parameters.runStage' } == 'false'` \(now, either the wait stage will run or this branch will run\)
    
    1. Uncheck `Fail Pipeline`, because we don't want the pipeline to fail if this branch does not get evaluated

1. Under `Execution Options`, for `If stage fails`, check `halt this branch of the pipeline`. This allows us to stop the branch but have the pipeline still succeed

1. Add a wait stage that runs after the check preconditions stage, and set the wait time for 5 seconds

1. Run the pipeline with both `true` and `false` as the values, and see which branch gets executed

1. Return to the `Check Preconditions` stage configuration and edit the precondition by clicking the small edit button next to the expression.
    
    1. Check `Fail Pipeline`

1. Run the pipeline again with `true` as the input, and notice how the execution of this pipeline now fails

When would this stage be useful in deploying a real application? What conditions could you check that would be helpful?

---

Next Lab: [Lab 7](/lab-7.md)

---
