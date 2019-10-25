# Sample Pipelines

This folder provides you with sample yml pipelines for every Secops you are about to implement within the DevSecOps OpenHack Lite. Navigate through each scenario folder and check out the sample piplines generated and ready to use.

On the root of this folder, there are two yml files to help you get started with build and test the 'eShopOnWeb' web application.

The [eShopOnWeb-CI.yml](./eShopOnWeb-CI.yml) is an ASP based yml CI template to build the eShopOnWeb application and run the tests. It is triggered on changes to master branch.

The [eShopOnWeb-Docker-CI.yml](./eShopOnWeb-Docker-CI.yml) is a docker based CI yml CI to build and push the eShopOnWeb application to an Azure Container Registry Instance of your choice. It is triggered on changes to master branch.

## Notes

All the samples are configureable,, Hence you might need to check the 'Variables' section in a template and replace the variables with your own values.

## References

- [YAML schema reference](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=example)

- [Build pipeline triggers](https://docs.microsoft.com/en-us/azure/devops/pipelines/build/triggers?tabs=yaml&view=azure-devops#ci-triggers)
