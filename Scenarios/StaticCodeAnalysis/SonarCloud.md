# SonarCloud

SonarCloud offers static code analysis with their cloud-based software as a service product. It is easy to set up a CI/CD pipeline with PR validation with SonarCloud in Azure DevOps.

SonarCloud recommends that the workflow uses pull requests to control changes, which can be configured to validate the code.

![SonarCloud PR validation](images/SonarCloudPRvalidation.png =700x500)

## Setup

This documentation will help you to configure SonarCloud with Pull Request validation (the recommended setup).

* [Integrate Visual Studio Team Services with SonarCloud](https://docs.microsoft.com/en-us/labs/devops/sonarcloudlab/)

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
In this case, you need to create a custom PowerShell task **after** the SonarCloud tasks. For example, the following checks the SonarCloud API to query
the quality gate associated with a specific project key, and fail accordingly:

```powershell
$token = [System.Text.Encoding]::UTF8.GetBytes($env:SONAR_TOKEN_ENV_VAR + ":")
$base64 = [System.Convert]::ToBase64String($token)

$basicAuth = [string]::Format("Basic {0}", $base64)
$headers = @{ Authorization = $basicAuth }

$result = Invoke-RestMethod -Method Get -Uri http://sonarcloud.io/api/qualitygates/project_status?projectKey=MY_PROJECT_KEY_GOES_HERE -Headers $headers
$result | ConvertTo-Json | Write-Host

if ($result.projectStatus.status -eq "OK") {
    Write-Host "Quality Gate Succeeded"
} else {
    Write-Error "Quality Gate Failed"
    exit 1
}
```

A sample implementation can be found at
<https://dev.azure.com/csedevops/devsecopshack/_apps/hub/ms.vss-ciworkflow.build-ci-hub?_a=edit-build-definition&id=46>

### Mono Project Scanning

In the case of a Mono project, we advise the developers migrate it to a .NET Core project. SonarCloud currently supports .NETCore/Standard and .NET Framework. There are [older versions](https://github.com/SonarSource/sonar-scanner-msbuild/releases?after=4.1.1.1164)
of the scanner, which support building a Mono project. However, this is not supported with the latest version of SonarCloud.
More information on this can be found:
[MSBuild 15 on macOS with Mono does not work with SonarQube MSBuild Scanner](https://github.com/Microsoft/msbuild/issues/1956)
