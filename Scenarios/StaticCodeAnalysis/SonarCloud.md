# SonarCloud

SonarCloud offers static code analysis with their cloud-based software as a service product. A CI pipeline for Azure DevOps can be found [here](../../pipelines/StaticCodeAnalysis/SonarCloud.yml).

SonarCloud recommends that the workflow uses pull requests to control changes, which can be configured to validate the code.

![SonarCloud PR validation](images/SonarCloudPRvalidation.png =700x500)

## Setup

This documentation will help you to configure SonarCloud with Pull Request validation (the recommended setup).

* [Integrate Visual Studio Team Services with SonarCloud](https://docs.microsoft.com/en-us/labs/devops/sonarcloudlab/)

* If you also want to enable code coverage reporting, you will need to review and implement the guidance in this [article](https://writeabout.net/2019/04/27/net-core-code-coverage-done-right/).

## Tips and Recommendataions

### Configure options in pipeline

Sonar cloud and sonar qube allow for configuration of the executing task to tune the execution.  This can have a dramatic impact on code smells detected and make it much simpler to get good feedback from the tooling.  In addition, areas like code coverage require some additional configuration in order to get the correct reporting.  You can find information on all available options here:

* [Analysis Parameters](https://sonarcloud.io/documentation/analysis/analysis-parameters/)
* [Test Coverage and Execution](https://sonarcloud.io/documentation/analysis/coverage/)

Example - In the YAML below we configure: 

* The path to opencover formatted test coverage report
* Exclude the lib directory since we have dependency scanning to scan our dependencies
* Set verbose output to true (This is optional but handy when first working with Sonarcloud)
* branch parameter changes [how sonar cloud does scans on feature branches](https://sonarcloud.io/documentation/branches/overview/).

``` YAML
    extraProperties: |
      sonar.cs.opencover.reportsPaths=$(Build.SourcesDirectory)/coverage/coverage.opencover.xml
      sonar.exclusions=src/Web/wwwroot/lib/**/*
      sonar.verbose=true
      sonar.branch.name=$(Build.SourceBranchName)
```

### Setting up Quality Gate

Sonar Cloud has a feature called Quality Gates. You can configure a threshold to fail the PR if it doesn't meet the quality you configured.  To configure the quality gate, go to your organization in SonarCloud > Quality Gate. Pick the condition that meets the goals of your organization. We recommend you test your pipeline for both success failure conditions to ensure it is working as expected.

![Quality Gate](images/qualitygate.png =800x500)

Then go to your organization > project > Administrator > Quality Gate to pick the one you have configured.

![Quality Gate Configuration](images/qualitygateconfig.png =800x500)

For more details see [Quality Gates](https://sonarcloud.io/documentation/user-guide/quality-gates/).

### Eliminate False-Positives

SonarCloud has a feature to report false-positives. It also has a feature to bulk report on the false-positives.
The false-positive report is inherited by the master after the PR is merged. One issue of this workflow is that
only the Administrator role can report the false positives.

The other way to suppress false positive is just add `//NOSONAR` comment on your code. For more detail, refer to [FAQ](https://sonarcloud.io/documentation/faq/).

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
