# Injecting Policy using Azure Devops Pipeline Decorators

## What are Pipeline Decorators

Pipeline decorators allow you to add steps to the beginning and/or end of every job. This is different than adding steps to a single definition because it applies to all pipelines in an organization.

A good example of such a task would be running a malware detector on every pipeline to make sure that dependencies do not contain vulnerabilities or scanning for credentials in code to insure that those secrets do not make their way into the wrong hands. For the extra time that these task add to pipeline execution they prove to be well worth ir

## When to run decorators

Decorators can be run either before or after a regular pipeline. The contributions section of the vss-extension.json file needs to contain the following for pre-processing: 
 
"targets": [
                "ms.azure-pipelines-agent-job.pre-job-tasks"
            ]

and conversely for post-processing:

"targets": [
                "ms.azure-pipelines-agent-job.post-job-tasks"
            ] .



For more information about [creating and publishing Azure DevOps extensions](https://docs.microsoft.com/en-us/azure/devops/extend/develop/add-build-task?toc=%2Fazure%2Fdevops%2Fextend%2Ftoc.json&bc=%2Fazure%2Fdevops%2Fextend%2Fbreadcrumb%2Ftoc.json&view=azure-devops) and  creating a [pipeline decorator](https://docs.microsoft.com/en-us/azure/devops/extend/develop/add-pipeline-decorator?view=azure-devops) follow the aforementioned hyperlinks.

An example that demonstrates how to inject a credential scanning and reporting into each pipeline can be found in the [CSE DevSecOps Repo](https://dev.azure.com/csedevops/DevSecOps/_git/SecOps_PipelineDecorator)
 

 ## Installation

 To [install](https://docs.microsoft.com/en-us/azure/devops/marketplace/install-extension?view=azure-devops) an Azure DevOps Extension to your Azure DevOps Organisation. You will need to be an organization admin or you may have to request the assistance of one.

 ## Output of the Pipeline Decorator
![Sample Output](./images/1_decoratorOutput.png)
