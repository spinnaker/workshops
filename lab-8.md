# Lab 8: Managed Pipeline Templates

Previous Lab: [Lab 7](/lab-7.md)

Jump to [TOC](/README.md)

---

**Goal:** Understand how to use managed pipeline templates

In this lab we are going to use managed pipeline templates. We will start with a simple pipeline (one 'wait' stage) and then move on to a deploy pipeline.

**Create a New Pipeline**

1. Click the `+` to create a new pipeline

1. Name the pipeline `wait template`

1. Select the `Template` radio button in the `Create From` dialog

1. Under `Template Source` make sure that `Managed Templates` is selected, and in the dropdown choose `Simple wait template`

1. This template has a variable for you to configure - it is the time \(in seconds\) the wait stage will wait. Input any positive value \(like 30\) and proceed

1. Notice that the UI looks different - you can't edit pipelines defined from managed pipeline templates in the UI. But, you can see the stages in the UI. Click on wait stage \(called `wait1`\) to see the value you set in the parameter

1. Click on the `Configure Template` button on the bottom of the screen. You can change the values you set for template parameters here. Change the wait time if you'd like, then save and run the pipeline

1. The source for the template is at [https://github.com/spinnaker/pipeline-templates/blob/master/workshop/wait-template.yml](https://github.com/spinnaker/pipeline-templates/blob/master/workshop/wait-template.yml). It's also in the MPT Reference at the end of this lab book

1. Take a look at the `variables` section in the template. That defines the parameter you filled out

1. Take a look at the `stages` section. That is how you define the wait stage. To use the value of a variable in the template you use jinja syntax: `"{{ waitTime }}"` injects the value of the `waitTime` parameter into the stage config.

Note that Spinnaker is the source of truth for these pipeline templates - there isn't a dependency on source control. When you make an update to a template, you must re-submit it to Spinnaker.

This is a simple example of how to use a managed pipeline template templates. This specific pipeline is not very useful though. Next, we will take a look at a deploy pipeline.

**Create a MPT Deploy Pipeline**

1. Create a new pipeline and call it `deploy MPT`

1. Just as above, select `Template` and choose the `deploy to account` template and continue

1. This template defines three variables with default values. Take a look at the `?` next to the parameter name to see what each represents. We're going to have this pipeline be our dev workflow, so leave account and stack the same. Change the tag of the image we're deploying if you want

1. Save the pipeline and refresh to see the `triggers` pop up successfully (known bug, will be fixed shortly).

1. The source for this template is at [https://github.com/spinnaker/pipeline-templates/blob/master/workshop/deploy-template.yml](https://github.com/spinnaker/pipeline-templates/blob/master/workshop/deploy-template.yml). It's also in the MPT Reference at the end of this lab book

1. Check out how the variables are defined and used. Can you find the variable that isn't defined but is used? That's something that's built into MPT.

1. Run the pipeline, it should have similar behavior to your dev pipeline. Select any tag in the manual trigger - it doesn't mater which as it won't be used.

This is a simple deploy pipeline. You might use this if you wanted to give teams in your organization a guide on how to deploy - let them specify the account and stack, but control that they're using a specific strategy or specific container settings. You could also use something like this if you manage many services and want to define all pipelines in a very similar way.

It's also worth noting that specifying the image to deploy in a variable and injecting that does not deploy the trigger tag. In this example the same container is deployed ever time (the tag we defined in the parameter while setting up the pipeline). To exactly replicate the tag resolved at runtime behavior that we have in our `deploy to dev` pipeline you need configure the deploy stage accordingly in the pipeline template.

This pipeline was created by converting a pipeline created in the UI to an MPT using [roer \(https://github.com/spinnaker/roer\)](https://github.com/spinnaker/roer). This is an easy way to get started with MPT, but not a great long term strategy as `roer` has no more active development.

**Extend that Deploy Pipeline to add a Manual Judgment Stage**

1. Create a new pipeline and call it `deploy MPT manual judgment`

1. Just as above, select `Template` and choose the `deploy with manual judgment` template and continue

1. Change `deployStack` to `prod` and change the color. Also, create your own manual judgment message that will pop up during the manual judgment stage

1. Save the pipeline and refresh to see the `triggers` pop up successfully \(known bug, will be fixed shortly\)

1. See in the UI that there is a new stage - a manual judgment stage - before your pipeline starts. This allows your pipeline to be triggered by an automated trigger but have a user intervene before the deploy happens. As a team, we have all our prod deploys gated by a manual judgment

1. Go to the `Pipelines` screen and run the pipeline

1. Approve the manual judgment stage and see how your pipeline deploys your new prod cluster

1. The source for this template is at [https://github.com/spinnaker/pipeline-templates/blob/master/workshop/extended-deploy-template.yml](https://github.com/spinnaker/pipeline-templates/blob/master/workshop/extended-deploy-template.yml). It's also in the MPT Reference at the end of this lab book

1. Take a look at the template and notice the `source: spinnaker://template-deploy`. This tells the template that it has a parent template to inherit from \(the parent template is the `deploy to account` template\)

1. In this template we also are injecting a stage before all stages in the parent template. We do that by adding the below block to the stage config:
`inject:`
`first: true`

This example shows how you can customize a base template for a pipeline. This is useful when you're building a more specific pipeline off of a base pipeline.

There are plenty of other MPT features. Take a look at the [docs \(https://github.com/spinnaker/dcd-spec/blob/master/PIPELINE\_TEMPLATES.md\)](https://github.com/spinnaker/dcd-spec/blob/master/PIPELINE_TEMPLATES.md) for a more complete guide on all the features.

**Bonus:** you can create your own template as a gist on [github](https://gist.github.com/) ([https://gist.github.com/](https://gist.github.com/%29\), and use it to create a new template. Try to make a new template that either has both a deploy and manual judgment stage, or make a template that extends from the base deploy template but has different behavior.

After your gist is publicly available, get the link for the `raw` version (visible by clicking on the gist then clicking the "Raw" button at the top right of the file). Enter that link into the box that appears when you select the `Template Source` : `URL` radio button in the create new pipeline dialog.

Next Lab: [Clean up](clean-up.md)