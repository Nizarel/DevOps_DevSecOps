# Serucity Bot

Security Bot is a reference implementation of the DevSecOps Pull Request centered validation, operation system. 

[![SecurityBot demo](https://img.youtube.com/vi/_c-dvMDlnsY/0.jpg)](https://www.youtube.com/watch?v=_c-dvMDlnsY)


## Issue to solve 

What we found through DevSecOps engagement is 

* Scan result is scattered on each 3rd party scanning portal
* How to suppress false-positives?
* How can we create a work item? 

This reference solution solve all of these problem. 

## General Idea 

The general idea of the Security Bot is, through the Pull Request Validation, we can scan source code and find several issues by several 3rd party services. 
This bot decorate pull request with all of these issues as a pull request comments. It centerlize the issue on a Pull Request so that Developer can handle oll 
of issues in one place. Also, you can invoke a command just reply a comment. for example, suppress false-positive, or create work item. 
It help developers and handle all of the security issue in one place. 



## Reference implementation

We develop a reference implementation on the GitHub. 

* [TsuyoshiUshio/SecurityBot](https://github.com/TsuyoshiUshio/SecurityBot)

This bot is implemented by C# Azure Functions. You can deploy it to Azure with one click from the repository. It is deployed as a Consumption plan which is pay-par-use.
It only requires Storage Account with Serucity tools access tokens. For more details, you can see the repo. The bot has a good extension mechanism. It has Providers to which we can easy to extend as creating a provider.

### Install

Go to the GitHub project page then click Deploy to Azure button. 

### Current support status

Currently, we implement three providers. For repository, we can use GitHub, For scanner, we can use Sonar Cloud, and WorkItem, We can use Azure DevOps. However, you can add provider without chaning the main code. 

### Architecture 

The Security Bot save the state to a storage account using [Durable Entity]().

Sample Pipeline to decorate PR. 
* [SonarCloudWithGitHub](https://dev.azure.com/csedevops/DevSecOps/_apps/hub/ms.vss-ciworkflow.build-ci-hub?_a=edit-build-definition&id=69)
