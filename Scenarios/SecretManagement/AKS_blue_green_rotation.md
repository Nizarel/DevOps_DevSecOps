# Azure Kubernetes Service Secret Rotation with Blue Green Rotation

In this article, you can learn how to rotate secret without downtime for rotating storage account connection string. 


We introduce Secret Rotation with [Automatic secret rotation wth Azure Keyvault](./KV_secret_rotation.md). We use Azure Automation for that scenario. However, for an AKS scenario, Azure DevOps might be a best option to execute shell since Azure Automation doesn't support `kubectl` command. We use [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) to create the bash script. It work well with `kubectl` command.

## Create ConfigMap

This configmap is used for which connection string you are using. 

```
```

## Create Azure DevOps pipeline
  Create Azure DevOps pipeline with Linux Based Agent. This pipline do ing following step. 

  1- Get current connection string from ConfigMap

  2- ReGenerate the other connection string

  3- Update KeyVault secret with the other connection string

  4- Update ConfigMap 

  5- Restart Pods

Storage Account 

`kubectl rollout restart {deployment name}` will restart the pod one by one. `kubectl rollout status {deployment name}` will wait until the rollout is finished. 


_azure_pipeline.yml_

```yaml
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- master

pool:
  vvmImage: 'ubuntu-latest'
  
variables:
  Namespace: 'ketopal'
  StorageAccount: 'hackstorage01'
  ResourceGroup: 'hack-runbooks-rg'
  ConnectionStringKeyVaultSecret: 'StorageAccountConnectionString'
  KeyVaultName: 'hack-runbooks-kv'
  ConfigMapName: 'hack-keyvault-secrets'
  SubscriptionServiceEndpoint: 'Nebbia - Partner Network (1979771a-9163-4750-8947-e6dbe596a8d7)'
  KubernetesServiceConnection: 'prod2-nebbia-aks'
  KubernetesDeployment: 'deployment/ketopal-api'

steps:
- task: KubectlInstaller@0
  displayName: 'Install Kubectl latest'
- task: AzureCLI@1
  displayName: 'Azure CLI '
  inputs:
    azureSubscription: '$(SubscriptionServiceEndpoint)'
    scriptLocation: inlineScript
    inlineScript: |
     currentKeyId=$(kubectl get configmap $(ConfigMapName) -n $(Namespace) -o json | jq '.data.CurrentIdentityStorageAccountKeyId')
     
     keyName="secondary"
     if [ "$currentKeyId" == "secondary" ]; then
         echo "regenerating primary key"
         keyName="primary"
     else 
        echo "regenerating secondary key"
     fi
     
     # regen the key
     az storage account keys renew --account-name $(StorageAccount) --key $keyName -g $(ResourceGroup)
     
     # update the secret in key vault
     echo "updating the secret in keyvault"
     
     # show connection string
     connectionString=$(az storage account show-connection-string -n $(StorageAccount) -g $(ResourceGroup) --key $keyName | jq '.connectionString')
     
     # update the key vault 
     az keyvault secret set -n $(ConnectionStringKeyVaultSecret) --value $connectionString --vault-name $(KeyVaultName)
     
     echo "Setting the pipeline variable with $keyName"

     echo "##vso[task.setvariable variable=NewStorageAccountKeyName;]$keyName"
- bash: |
    echo "Keyname is $(NewStorageAccountKeyName)"

    cat <<EOF > configmap.yml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: $(ConfigMapName)
      namespace: $(Namespace)
    data:
      CurrentAccountKeyId: $(NewStorageAccountKeyName)
    EOF

    cat configmap.yml
- task: Kubernetes@1
  displayName: "kubectl create configmap"
  inputs: 
    namespace: $(Namespace)
    kubernetesServiceEndpoint: $(KubernetesServiceConnection)
    command: apply
    arguments: -f configmap.yml --record
- task: Kubernetes@1
  displayName: 'kubectl rollout restart'
  inputs:
    kubernetesServiceEndpoint: $(KubernetesServiceConnection)
    namespace: $(Namespace)
    command: rollout
    arguments: 'restart $(KubernetesDeployment)'
- task: Kubernetes@1
  displayName: 'kubectl rollout status'
  inputs:
    kubernetesServiceEndpoint: $(KubernetesServiceConnection)
    namespace: $(Namespace)
    command: rollout
    arguments: 'status $(KubernetesDeployment)'
    checkLatest: true

```


