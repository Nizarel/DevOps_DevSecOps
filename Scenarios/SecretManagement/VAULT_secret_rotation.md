# Kubernetes Secret Rotation Strategies

Secret rotation provide us to automate the process of reflecting secret change to your applications. However, we need to additional technique to 
reflect the change to your pod on kubernetes. 
This topic provide an overview and how to implement three strategies. 

# KeyVault Flex Volume with Restarting 

[KeyVault FlexVolume](https://github.com/Azure/kubernetes-keyvault-flexvol) provide us an integration Azure KeyVault with Kubernetes. Secrets, keys, and certificates in a key management system become a volume accessible to pods. 
However, once it is mounted, the secret will not reflect the change of Key Vault. Once KeyVautl Change happnes, we need to restart the pod. 

![](images/flexVolume.png =950x500)

## Setup the KeyVault Flex Volume

Follow the instruction [here](https://github.com/Azure/kubernetes-keyvault-flexvol). 

## Azure DevOps Secret Rotation with Restart

[Azure DevOps](https://azure.microsoft.com/en-us/services/devops/) provide us the feature [Scheduled Trigger](https://docs.microsoft.com/en-us/azure/devops/pipelines/build/triggers?view=azure-devops&tabs=yaml#scheduled-triggers).
Execute [KeyVault Secret Rotation](KV_secret_rotation.md) script on the Azure DevOps as a Pipeline. Once the rotation is finished, execute the command of rotation. 
For example, If you have a deployment, you can restart the pod, one by one using this command. You restart all pods on the `nginx-deployment` deployment. 

```
 kubectl rollout restart deployment/nginx-deployment
```

By reloading the pods, the change has been reflected to your pod. 

![](images/flexVolumeWithAzDO.png =950x500)

# Notifications for Azure KeyVault (Preview)

Azure KeyVault has a preview feature [Notifications for Azure keyVault](https://keyvaultdocs.azurewebsites.net/KeyVault/Notifications/OnBoarding.html). It is designed to allow users to be notified when the status of a secret stored in key vault has changed.
Currently, the link is available MSFT only. You need to submit the request to use this preview feature. 

Once the secret has been changed, you can get the notification through the EventGrid. Then you can restart the pod with Azure Functions for example. 

![](images/KeyVaultNotification.png =950x500)

# HashiCrop Vault Secret Rotation

HashCorp provides [Vault](https://www.vaultproject.io). The [Consul Template](https://learn.hashicorp.com/consul/developer-configuration/consul-template) watch the EoL of a secret of Vault, 
dinamically update the secret with automatic vault token renewal and caching with [Vault Agent](https://www.vaultproject.io/docs/agent/). You can deploy Vault with [helm chart](https://www.hashicorp.com/blog/announcing-the-vault-helm-chart).  Also, you can learn how to setup Vault Secret Rotation with 
AKS in [Vault Agent with Kubernetes](https://learn.hashicorp.com/vault/identity-access-management/vault-agent-k8s#azure-kubernetes-service-cluster).

See this diagram. It shows the architecture of the secret rotation. 

1. Vault Agent Talk to Vault fetch the token
2. Vault Agent generate a vault-token on the Volume
3. Consul Template fetch secrets using vault-token
4. Consul Template generates secret on the Volume

Then Target Container can use the secret by mounting the Volume. Since the Consul Template is daemon, if the secret reaches TTL, it periodically fetch 
the new Secret to the Vault. 


To update the secret with ttl, vault command. It will set the secret and expire in 30 sec. After the 30 sec, if you update the secret using same command with changing secret, 
you will find the secret on the pod is updated after 30 sec.

```
vault kv put secret/myapp/config username='appuser' password='suP4rsec(et!' ttl='30s'
```

You can find the tips when you deploy HashiCorp Vault with AKS integration, I wrote a blog post to it. 
See more detail on the blog. 

![](images/Vault.png =950x500)

# Vault Mirror (TODO)

HashiCorp Vault provide dynamic secret rotation feature. If you want have with KeyVault, We can create the same tools like Vault Agent and Consul Template. 
However, the easiest way for using KeyVault might be developing a small daemon called `Vault Mirror`. It watches the KeyVault once change happens, it mirrors the secret on HashiCorp Vault. 
I'd like to provide a sample implementation of Vault Miller in the near future. It will use polling model or KeyVault notification feature. 

![](images/VaultMirror.png =950x500)

# How to choose your strategy?

This article introduce four strategies. Flex Volume solution requires Pod restarting. In case of `Flex Volume with Rotation` strategy, in your secret rotation script, 
you need to include restart command. It might required REST API call or Client Library of Kubernetes. `Flex Volume with KeyVault Notification` strategy enable to restart it by Webhook of KeyVault which the secret changes.
`HashiCorp Vault` strategy provide a dynamic rotation and very good tools when customer want onPrem solution. `Vault Miller` strategy is good when you want to centerize the secret on KeyVault however, 
want to use HashiCorp vault tools on Kubernetes. 

![](images/ProsCons.png =950x500)



