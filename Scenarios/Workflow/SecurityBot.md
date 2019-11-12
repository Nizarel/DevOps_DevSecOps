# Security Bot

Security Bot is a reference implementation of the DevSecOps Pull Request centered validation, operation system.

To see a demonstration of the Security Bot. Click on the image below: 

[![SecurityBot demo](https://img.youtube.com/vi/_c-dvMDlnsY/0.jpg)](https://www.youtube.com/watch?v=_c-dvMDlnsY)

## Issues to solve

What we found throughout various DevSecOps engagements that needed to be addressed:

* The results of scans that we performed were scattered across multiple third party sites/portals.
* There were varying ways that we could suppress false positives, but not all of then were intuitive and in some cases individuals needed elevated permissions to address issues.
* How could create work items in our work item tracking system by itegrating the report results and linking items accordingly?

This reference solution aims to solve all of the above problems.

## General Idea

The general idea of the Security Bot is, through Pull Request Validation, we can scan source code and find several issues produced by several 3rd party services.
This bot will decorate the pull request with all of the issues surfaced as pull request comments that will need to be resolved in order for the merge to be approved. It centralizes the issue/s on a Pull Request so that a developer can handle all of the issues in one place. You can also invoke a command by just replying to a comment. e.g. suppressing false-positives or creating work items by replying "suppress false positive" or "create work item".
This simplifies access requirements and can help developers as they are now able to handle all of the surfaced issues in one place.

![Overview of security bot usage flow](images/SecurityBotOverview.png)

## Reference implementation

We developed a reference implementation on GitHub.

* [TsuyoshiUshio/SecurityBot](https://github.com/TsuyoshiUshio/SecurityBot)

This bot is implemented by C# Azure Functions. You can deploy it to Azure with one click from the repository. It is deployed as a Consumption plan which is pay-per-use.
It only requires a Storage Account with Security tools access tokens. For more details, you can take a look at the repo. The bot has a good extension mechanism. It uses Providers and therefore it is easy to extend by creating a new provider for the tool that you would like to support.

### Installation

TO install the Bot follow the links to the [GitHub project page](https://github.com/TsuyoshiUshio/SecurityBot) and click the "Deploy to Azure" button.

### Current support status

The current implementation supports three providers. We make use of GitHub repositories for source control. The scanner implementation makes use of a SonarCloud instance, and for Work Item tracking , we have opted to use Azure DevOps. You can however add your preferred provider/s without changing the main code.

![Support status from high-level](images/CurrentSupport.png)

### Architecture

The Security Bot saves state to a storage account using [Durable Entity](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-entities).

### Sample Pipeline demonstrating how to decorate a PR.

[SonarCloudWithGitHub](https://dev.azure.com/csedevops/DevSecOps/_apps/hub/ms.vss-ciworkflow.build-ci-hub?_a=edit-build-definition&id=69)
