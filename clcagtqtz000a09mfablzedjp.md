# Boost Your Enterprise Security with GitHub Actions and the OSSF Score Card

In this blog post, we will cover how you can use the OSSF Scorecard to assess the security score of your repository and report the results in GitHub Advanced Security - Overview. This will allow you to assess the security scores of your repositories across your organization, providing a comprehensive view of your repository’s security posture.

The Open Source Security Foundation (OSSF) Scorecard is a tool that helps assess the security measures taken in a repository. It does this by performing a series of checks, each of which is assigned a score of 0-10. By using the OSSF Scorecard, you can assess the risks that dependencies introduce, understand specific areas where you can improve the security posture of your project, and make informed decisions about accepting those risks or evaluating alternative solutions.

%[https://github.com/ossf/scorecard] 

Some of the checks included in the OSSF Scorecard are:

* Branch-Protection: This check assesses whether the project uses Branch Protection to ensure that code is reviewed before it is merged.
    
* CI-Tests: This check looks for the presence of tests that are run in CI, such as GitHub Actions.
    
* Code-Review: This check assesses whether the project requires code review before code is merged.
    
* Dangerous-Workflow: This check looks for the use of dangerous coding patterns in GitHub Action workflows, which can pose a security risk.
    
* Dependency-Update-Tool: This check looks for the use of tools to help update dependencies, which can help keep the project up-to-date and secure.
    
* License: This check assesses whether the project has a declared license, which is important for ensuring legal compliance.
    
* Maintained: This check looks for projects that are at least 90 days old and are actively maintained.
    
* Pinned Dependencies: This check looks for the declaration and pinning of dependencies, which can help ensure that the project is using stable and secure versions of those dependencies.
    
* Packaging: This check looks for the building and publishing of official packages from CI/CD, such as GitHub Publishing.
    
* SAST: This check looks for the use of static code analysis tools such as CodeQL or other SAST tools such as SonarCloud, which can help identify vulnerabilities in the code.
    
* Security Policy: This check looks for the presence of a security policy, which can help outline the steps taken to ensure the security of the project.
    
* Signed-Releases: This check looks for the use of cryptographic signing for releases, which can help ensure the authenticity and integrity of those releases.
    
* Token-Permissions: This check assesses whether the project declares GitHub workflow tokens as read-only, which can help prevent unauthorized access to sensitive information.
    
* Vulnerabilities: This check looks for unfixed vulnerabilities in the project and uses the OSV service to assess the risk level.
    
* Webhooks: This check looks for the presence of tokens in webhooks that are used to authenticate the origins of requests. This can help prevent unauthorized access to the project.
    

You can use GitHub Actions to automate the process of updating an OSSF scorecard for a project. To do this, you will need to set up a workflow that runs the OSSF Scorecard and then parses and displays the results.

In this tutorial, we will walk through the process of using GitHub Actions to automate the process of updating an OSSF scorecard for a project. We will cover the following steps:

1. Setting up a GitHub repository for your project.
    
2. Creating a workflow file to automate OSSF scorecard updates
    
3. Running security scans and updating dependencies with GitHub Actions
    
4. Viewing and updating your OSSF scorecard
    

### Setting up a GitHub repository for your project

If you don't already have a GitHub repository, the first step is to create one. To do this, log in to your GitHub account and click the "New repository" button. Give your repository a name and description, and choose whether you want it to be public or private.

### Creating a workflow file to automate OSSF Scorecard updates

Next, we will create a workflow file to define the steps performed by our GitHub Action.

* You can set up this directly or navigate to the “Security” tab → Setup Code scanning (or Add more scanning tools).
    
* Choose the “OSSF Scorecards supply-chain security analysis” from the list and set up the workflow. It will create a workflow file like the one below.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1672400684900/722f9744-7585-4603-bf98-48841a8e4eac.png align="center")
    
* Here is an example workflow file that runs a security scorecard scan and uploads the findings in Code scanning alerts whenever code is pushed to the `main` branch:
    
    ```yaml
    
    name: Scorecards supply-chain security
    on:
      branch_protection_rule:
      schedule:
        - cron: '30 8 * * 3'
      push:
        branches: [ "main" ]
    
    permissions: read-all
    
    jobs:
      analysis:
        name: Scorecards analysis
        runs-on: ubuntu-latest
        permissions:
          # Needed to upload the results to code-scanning dashboard.
          security-events: write
          # Uncomment the permissions below if installing in a private repository.
          contents: read
          actions: read
    
        steps:
          - name: "Checkout code"
            uses: actions/checkout@93ea575cb5d8a053eaa0ac8fa3b40d7e05a33cc8 # v3.1.0
            with:
              persist-credentials: false
    
          - name: "Run analysis"
            uses: ossf/scorecard-action@99c53751e09b9529366343771cc321ec74e9bd3d # v2.0.6
            with:
              results_file: results.sarif
              results_format: sarif
    	        repo_token: ${{ secrets.SCORECARD_TOKEN }}
    
              # Public repositories:
              #   - Publish results to OpenSSF REST API for easy access by consumers
              #   - Allows the repository to include the Scorecard badge.
              #   - See https://github.com/ossf/scorecard-action#publishing-results.
              # For private repositories:
              #   - `publish_results` will always be set to `false`, regardless
              #     of the value entered here.
              publish_results: false
    
          # Upload the results as artifacts (optional). Commenting out will disable uploads of run results in SARIF
          # format to the repository Actions tab.
          - name: "Upload artifact"
            uses: actions/upload-artifact@3cea5372237819ed00197afe530f5a7ea3e805c8 # v3.1.0
            with:
              name: SARIF file
              path: results.sarif
              retention-days: 5
    
          # Upload the results to GitHub's code scanning dashboard.
          - name: "Upload to code-scanning"
            uses: github/codeql-action/upload-sarif@807578363a7869ca324a79039e6db9c843e0e100 # v2.1.27
            with:
              sarif_file: results.sarif
    ```
    

In this example, the `on` block defines the trigger for the workflow, which is a push event to the `main` branch. The `jobs` block defines the steps that are performed by the workflow. In this case, the workflow has an `analysis` job that runs on an Ubuntu virtual machine. The job has three steps:

* `actions/checkout@<sha>`: This action checks out the code from the repository to the virtual machine.
    
* `ossf/scorecard-action@<sha>`: This action runs an OSSF Scorecard section on the repo to identify any misconfigurations based on the OSSF rules.
    
* `actions/upload-artifact@<sha>:`: This action runs an upload artifact - the sarif file result from the scorecard action
    
* `actions/upload-artifact@<sha>:`: This action uploads OSSF Scorecard results to the code scanning alerts to the dashboard.
    

### Code scanning Alerts

Using the OSSF Score card in conjunction with code scanning alerts can be a powerful way to ensure the security of your projects in the enterprise. By integrating the OSSF Scorecard with code scanning alerts, you can receive notifications whenever a check fails or a score falls below a certain threshold. This can help you stay informed about the security of your repository and take prompt action to fix any identified issues.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1672400711828/cfcb2525-5c81-4639-a14c-3b91a5657694.png align="center")

You can see the details of the check and the remediation steps to fix it.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1672400724227/e5116188-9a2b-483a-9f6e-f743f201ad07.png align="center")

To view code scanning alerts of all your repositories in the organization, you can go to the "Security" section. Monitoring code scanning alerts is an important part of maintaining the security of your repositories in the enterprise. Regularly reviewing and addressing any identified vulnerabilities can help ensure that your projects are secure and protect your organization against potential vulnerabilities. Here is the screenshot of the sample Security Risk of your organization.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1672400740454/840d6728-03eb-4207-9c9f-75a5c401bcbb.png align="center")

### Best Practices

It is important to note that the OSSF Scorecard is just one tool that can help you assess the security of your repositories. It is recommended that you use a combination of tools and best practices to ensure the security of your projects in the enterprise. Some other best practices to consider include the following:

* Regularly updating dependencies and keeping them up-to-date.
    
* Using secure coding practices and following best practices for secure development.
    
* Regularly scanning your code for vulnerabilities using tools such as CodeQL or other SAST tools.
    
* Implementing branch protection and code review processes to ensure that code is reviewed before it is merged.
    
* Creating and maintaining a security policy that outlines the steps taken to ensure the security of the project.
    
* Cryptographically signing releases to ensure the authenticity and integrity of those releases.
    

By following these best practices and using tools like the OSSF Scorecard, you can help improve the security of your projects in the enterprise and protect your organization against potential vulnerabilities.

### Conclusion

Using the OSSF Scorecard, you can get a comprehensive view of the security measures taken in your repository and identify areas for improvement. You can also use the scorecard in conjunction with code scanning alerts to receive notifications when potential vulnerabilities or security issues are discovered, allowing you to take prompt action to fix those issues.

Overall, the OSSF Scorecard is a valuable tool for assessing and improving the security of your projects in the enterprise. By using it regularly, you can ensure that your projects are secure and well-maintained, and you can demonstrate to your customers and stakeholders that you are committed to maintaining a high level of security.

I'm Siva - Director, DevOps & Principal Architect at [**Computer Enterprises Inc**](https://www.ceiamerica.com/) from Orlando. I'm an AWS Community builder. I write blogs and tutorials about Cloud, Containers, IoT, and DevOps. If you are interested, please follow me @[@ksivamuthu](@ksivamuthu) on Twitter or check out my blogs at [**sivamuthukumar.com**](http://sivamuthukumar.com)