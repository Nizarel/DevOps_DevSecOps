# DevSecOps Workflow

This article shows the best practices of the DevSecOps workflow. 

## Shift left

We used to have a security testing on the staging phase. The idea of shift left is moving the security testing eariler stage of development to get feedback more quickly. 

![Shift Left](images/ShiftLeft.png =800x400)

We recommend to security test to shif left. If you have all security testing on the Local Development phase, it might be ideal. Especially for Credential, SSN, CreditCard number scanning. It protect to push your code with sensitive data. However, it depends on the support of the Service/Product. 
If you are available, we recomment to use it. Generally, speaking, we recommend to start the security testing on Pull Request review. 

## Security Testing with Pull Request

The reason why we recommend pull request validation for security testing is 

* It reduce the number of issue in one validation
* Quality Gate integrated with CI
* It protect us to inject the vulnerablity to master branch
* As a developer, we want to see several vulnerablity reports in one place
* Want to suppress false positives (Optional)

For the pull request model, some products support that it just report the delta for the issue. 
It helps to developers to focus on what they change and fix it. Also we can specify a quality gate for the pull request. The quality gate start a CI pipeline and validate the code. Unless developers pass the gate, 
the PR is never get merged. Pull request also help to developers to notice all vulnerablity in one place. 

![Overview](images/Overview.png =1000x500)

For the advanced scenario, we can use PR Bot to suppress false positives and create advanced work item integration. You can refer the PR bot strategy on the other document.

## WorkFlow patterns

![WorkFlowType](images/WorkFlowType.png =800x400)


### Serial Flow 

If you have a flow that you need to execute one by one, use the Serial folow pattern.

### Parallel Flow

If you don't need to execute the serially, you can use the parrllel flow. It help to reduce the execution time of CI.

### Enforce Policy

If you want to inject specific task for all pipeline on your organization or project, you can use this strategy. 
Please refer the [Enforce policy](../EnforceOrgSecurityPolicy/README.md) Documentation. 

# Configration 

## Serial Flow configuration 

If you have Job A, job B, Job C, you need to configure the dependency and condition. 
Job B depends on Job A, Job C depends on JobB, also you can configure the configuration of Job B and C as "Even if a previous job has failed. 

![SerialFlow](images/SerialFlow.png =800x400)

## Parallel Flow configuration 

For the parallel, you don't need to specify any dependency. Jobs work pallarelly by default.

## Fail if the quality gate has failed. 

Scanners have a quality gate. Usually, each task has a future of fail if it doesn't reach the quality gate, however, some of them don't have the future for that. 
SonarCloud it the one. In case of Sonar Cloud, you can use a task. 

* [SonarCloud build breaker](https://marketplace.visualstudio.com/items?itemName=SimondeLang.sonarcloud-buildbreaker) 

You can find it for SonarQube as well. For the configuration of the Sonar Cloud pipeline and PR validation, please refer [This]() page. 

## Create Work Item 

If you want to create a work item when the scan is failed, you can use Create Work Item task. 

* [Create Work Item](https://marketplace.visualstudio.com/items?itemName=mspremier.CreateWorkItem)

You can find a sample configuration for the task. For more detail of the configuration, please refer the link above. 

```
variables:
  AssignTo: 'Tsuyoshi Ushio <tsushi@microsoft.com>'

steps:
- task: CreateWorkItem@1
  displayName: 'Create work item'
  inputs:
    teamProject: DevSecOps
    workItemType: Bug
    title: 'Fossa Scan Failed: $(System.PullRequest.PullRequestId)'
    assignedTo: '$(AssignTo)'
    areaPath: 'DevSecOps\Scenarios'
    iterationPath: 'd7f2e7e9-377e-4264-8962-814b21f7fcb7@currentIteration'
    fieldMappings: 'System Info=Fossa Task failed. For more information please refer to this <a href="$(System.TeamFoundationCollectionUri)$(System.TeamProject)/_build/results?buildId=$(Build.BuildId)">link</a>'
    associate: true
    linkPR: true
    preventDuplicates: true
    keyFields: System.Title
    createOutputs: true
    outputVariables: 'CWI.Id=System.Id'
  condition: failed()
```

This task suppress to create a duplicate work item by setting `keyFields` and preventDuplicates. In this example, this task compare the Title and if it is the same, then this task doesn't create a new work item. 
The title incluce the pull request id. 

`condition` should be `failed` or `Only when a previous task has failed.` 

One more tips is please specify the `outputVariables` as `CWI.Id=System.Id`. It enable us to get the WorkItem.Id on the subsequnt task. 


**NOTE:** Currently, this task has a bug that fails when there is a work item that has the same Title name. We have a branch to fix it. However, it is going to be merged soon. 

## Create a comment to link 

[Create WorkItem task](https://dev.azure.com/csedevops/DevSecOps/_git/CreatePRCommentTask?path=%2FREADME.md&version=GBfeature%2Fsimplecomment&_a=preview)

Since this is an alpha version for internal use, we don't push it to the market place. You can use to follow this. 

### isntall tfx command 

Go to [Node CLI for Azure DevOps](https://github.com/Microsoft/tfs-cli) and install the cli. 

### Login your AzureDevOps organization 

Get the Personal Access Token of your Azure DevOps, then login it using the cli. 

```
tfx login -t {Your personal access token} -u https://{your organization name}.visualstudio.com/DefaultCollection
```

### Upload task 

```
cd Task
tsc
(update the task.json version)
tfx build tasks --task-path .\Task\
```

## Workaround 

Currently, Create WorkItem doesn't expose the workItem.Id if there is already created. To avoid this issue, create a powershell task with this inline script. 

```
if ($env:CWI_Id -eq $null) {
    Write-Host "##vso[task.setvariable variable=SkipCreatePRComment]True"
}
```

Then configure the Control Option > Custom Condition 

```
and(failed(), ne(variables['SkipCreatePRComment'], 'True'))
```


