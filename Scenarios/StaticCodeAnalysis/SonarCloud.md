# SonarCloud

SonarCloud offers static code analysis with their cloud-based software as a service product. It is easy to set up a CI/CD pipeline with PR validation with SonarCloud in Azure DevOps.

SonarCloud recommends that the workflow uses pull requests to control changes, which can be configured to validate the code.

![SonarCloud PR validation](images/SonarCloudPRvalidation.png =700x500)

## Setup

This documentation will help you to configure SonarCloud with Pull Request validation (the recommended setup).

* [Integrate Visual Studio Team Services with SonarCloud](https://docs.microsoft.com/en-us/labs/devops/sonarcloudlab/)

* If you also want to enable code coverage reporting, you will need to review and implement the guidance in this [article](https://writeabout.net/2019/04/27/net-core-code-coverage-done-right/).

### Tips and Recommendataions

### Eliminate False Positives

SonarCloud has a feature to report false positives. It also has a feature to bulk report on the false positives.
The false positive report is inherited by the master after the PR is merged. One issue of this workflow is that
only the Administrator role can report the false positives.

While a developer can see the false positive on the pull request, they cannot themselves report a false positive on the PR.
This means a developer would need to ask the Administrator to report false positives; this is possible room for future improvement.
One idea is to create a bot to manage Pull Requests. Please refer to the bot strategy in the next chapter.

### Work Item Integration

SonarCloud doesn't have a feature of creating work items automatically; instead, they recommend the use of the PR validation workflow.
This is the reason why the SonarCloud task in Azure DevOps never fails. However, if a PR is submitted and fails validation, that
PR should be linked with a Work Item.

With that idea in mind, we recommmend creation of a Pull Request bot that integrates the creation of work items. We can track the PR creation with the
service hook of Azure DevOps; once the developer kicks off a PR, the bot polls the status of the Pull Request. If the developer wants to create a work item for a specific code smell/bug,
then they can add a comment like '@workitem' then a work item ID created by the bot with the contents of the code smell/bug. A similar strategy can be used for reporting false positives.

### Fail on CI without Pull Request

SonarCloud recommends us to use PR validation. However, you might want to fail the CI by Quality Gates.
In this case, you can use [SonarQube build breaker](https://marketplace.visualstudio.com/items?itemName=SimondeLang.sonar-buildbreaker)

### Mono Project Scanning

In the case of a Mono project, we advise the developers migrate it to a .NET Core project. SonarCloud currently supports .NETCore/Standard and .NET Framework. There are [older versions](https://github.com/SonarSource/sonar-scanner-msbuild/releases?after=4.1.1.1164)
of the scanner, which support building a Mono project. However, this is not supported with the latest version of SonarCloud.
More information on this can be found:
[MSBuild 15 on macOS with Mono does not work with SonarQube MSBuild Scanner](https://github.com/Microsoft/msbuild/issues/1956)
