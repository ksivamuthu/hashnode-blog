# GitHub Actions Scheduled Workflows: Understanding the Limitations and Possible Solutions

GitHub Actions is a powerful tool that allows developers to automate their workflow, but when it comes to scheduling workflows, it may not always work as expected. Many users have reported that their scheduled workflows are not triggering at the scheduled time, and in some cases, the delay can be as long as an hour. In this blog post, we will explore the limitations of GitHub Actions scheduled workflows, the reasons for these delays, and what can be done to ensure that your workflows are executed on time. We will also discuss various alternative solutions that can be used to trigger the workflows manually, guaranteeing the execution of your production tasks.

### Scenario: Actions not running at the scheduled time.

I've recently written a GitHub action that generates reports by running scripts and sending an email of the report every Tuesday at 8 AM EST.

E.g.,

```yaml
name: Generate Report

on:
  schedule:
    - cron: "0 13 * * 2" # UTC

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
    # Reduced the code for brevity
    - name: Generate report and Send email
      run: |
        generate-report.sh 
        send-email.sh
```

It took less than a minute to generate a report. I was expecting the email to be received at least by 8.05 AM. But it's not started. It took 30min to 1 hour to get the new workflow run queued.

One of the main reasons for delays in scheduled workflows is the fact that GitHub runs workflows on shared runners. There is no guarantee that the workflow will run at the exact scheduled time. When you set up a workflow schedule, you request GitHub to schedule it. However, many factors can affect the execution of the workflow, such as system load, queue size, and other workflows competing for resources.

This means that the resources available to run workflows are not dedicated to a specific user or repository and are subject to the demands of other users and workflows. As a result, there may be delays in the execution of a scheduled workflow, as it may need to wait for resources to become available.

## Solution: Use external schedulers & Workflow Dispatch

One possible solution to overcome these limitations is manually triggering the workflow using an external scheduler. GitHub Actions supports the workflow\_dispatch trigger, allowing you to trigger a workflow manually. This means you can use a third-party cron scheduling service, like [IFTTT](https://ifttt.com/), [Cronhub](https://cronhub.io/), [Cronitor](https://cronitor.io/), etc., to request the GitHub API to trigger the workflow. By doing so, you can ensure that the workflow is executed at the desired time, regardless of the state of the shared runners or the cron schedule.

Add the workflow\_dispatch to trigger the GitHub Actions manually.

```yaml
name: Generate Report

on:
 workflow_dispatch:

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
    # Reduced the code for brevity
    - name: Generate report and Send email
      run: |
        generate-report.sh 
        send-email.sh
```

You could trigger the workflow using an HTTP call, for e.g,

```bash
curl -X POST -H "Accept: application/vnd.github+json" -H "Authorization: token YOUR_ACCESS_TOKEN" https://api.github.com/repos/OWNER/REPO/actions/workflows/WORKFLOW_ID/dispatches
```

This way, your workflow will be executed at the exact scheduled time, without any delays, regardless of the state of shared runners.

### Conclusion

In summary, while GitHub Actions is a powerful tool for automating your workflow, its scheduled workflows may not always run at the exact scheduled time. This can be due to the shared runner's nature, where the system load, queue size and other workflows can affect the execution of your workflow. To ensure that your production tasks are executed on time, it is recommended to use an external scheduler to manually trigger the workflow. This way, you can guarantee the execution of your workflow and avoid delays.

I'm Siva - Director, DevOps & Principal Architect at [**Computer Enterprises Inc**](https://www.ceiamerica.com/) from Orlando. I'm an AWS Community builder. I write blogs and tutorials about Cloud, Containers, IoT, and DevOps. If you are interested, please follow me [@ ksivamuthu](https://twitter.com/ksivamuthu)on Twitter or check out my blogs at [**sivamuthukumar.com**](http://sivamuthukumar.com)