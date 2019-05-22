# Sonarcloud 

Sonarcloud offers Static code analysis with SaaS offering. You can easy to set up the CI/CD pipleline and PR validation. 
Sonarcloud recommends to have a workflow with Validating the code by Pull Requests. 

# Setup 

You can follow this documentation. You can configure the Sonarcloud with PR validation which is recommended. 

* [Integrate Visual Studio Team Services with SonarCloud](https://docs.microsoft.com/en-us/labs/devops/sonarcloudlab/)

# Tips and recommendataions

## Eliminate false positive 

SonarCloud have a feature to report failse positive. It also have a feature of bulk report for the false positive. 
The report of the false positive is inherited to the master after the PR merged. One of the issue of the workflow of the 
false positive report is, only the Administrator report the false positives. 
  
You can see the False positive on your PR, however, you can't report from the false positive on the PR. Also you need to ask 
Administrator to report false postivies. We might have a room to improvement of the reporting False positve. 
One of the idea is creating a bot to the Pull Request. Please refer the bot strategy on the next chapter. 

## Work Item integration 

SonarCloud doesn't have a feature of creating work item automatically. The reason is, SonarCloud recommend to use PR validation workflow. 
That is why the SonarCloud task of the Azure DevOps never fails. However PR validation fails. PR should be linked with WorkItem already. 

However, if you want it, I recommmend to have a Pull Request bot for creating work item integration. We can track the PR creation with 
Service hook of Azure DevOps, once start the PR, the bot poll the status of the Pull Request. If you want to create a work item for a specific code smell/bug, 
then you can comment like '@workitem' then a work item created by bot with the contents of the code smell/bug. 
If you need this solution, I'd happy to develop a PoC. The same strategy can be used for reporting false postiives.   

## Fail on CI without Pull Request

SonarCloud recommends us to use PR validation. However, you might want to fail the CI by Quality Gates. 
In this case, you need create a custom PowerShell task after the SonarCloud Tasks. This is a sample code for it. It check the API of SonarCloud, 
then if it against the quality gate, it will fail. 

```
$token = [System.Text.Encoding]::UTF8.GetBytes($env:SONAR_TOKEN_ENV_VAR + ":")
$base64 = [System.Convert]::ToBase64String($token)
 
$basicAuth = [string]::Format("Basic {0}", $base64)
$headers = @{ Authorization = $basicAuth }
 
$result = Invoke-RestMethod -Method Get -Uri http://sonarcloud.io/api/qualitygates/project_status?projectKey=TsuyoshiUshio_VulnerableApp -Headers $headers
$result | ConvertTo-Json | Write-Host
 
if ($result.projectStatus.status -eq "OK") {
Write-Host "Quality Gate Succeeded"
}else{
Write-Error "Quality Gate Failed"
exit 1
}
```

This is the sample implementation. 
https://dev.azure.com/csedevops/devsecopshack/_apps/hub/ms.vss-ciworkflow.build-ci-hub?_a=edit-build-definition&id=46


## Mono project scanning

If you have a mono project, I recommend to move to the .NetCore project. SonarCloud support .NetCore/Standard and .NetFramework currently. 
If you really want to build the mono project, you can do it with [old version](https://github.com/SonarSource/sonar-scanner-msbuild/releases?after=4.1.1.1164). However it is no longer supported for the latest version and 
include the binary to the repo. Then execute it using mono. 

* [MSBuild 15 on macOS with Mono does not work with SonarQube MSBuild Scanner](https://github.com/Microsoft/msbuild/issues/1956)


