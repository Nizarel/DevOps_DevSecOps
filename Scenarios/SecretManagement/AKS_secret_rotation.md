# Rotate Service principal account for deployed AKS

This bash script uses Azure CLI to create a new service principal account to be used in a deployed AKS instead of the current service principal.

This script provides secret rotation mechanism that can be part of a defined security policy.

You can use an existing certificate stored in azure key vault to create the service priciple account by using the following command.

## Update AKS credentials by creating new service principal account

```Bash
newSP=$(az ad sp create-for-rbac --keyvault MyVault --cert CertName)
```

Before you run the script ensure the machine has Azure CLI version 2.0.65 or later installed and configured.

```Bash
#!/bin/bash
#AKS Service Priciple rotation Script 
#Requires jq and az cli
usage(){
        echo "***Rotate AKS Service Priciple credentials***"
        echo "Usage: ./SP_rotation.sh <rgName> <aksClusterName><azSubscriptionId><yearsValidfor>"
		echo "Expects 2 arguments: rgName, aksClusterName, azSubscriptionId AKS cluster resource group and cluster name, the azure subscription id, and number of years new service principal account is valid for"
}

RotateAKSCred(){
	#set the azure subscription
	echo "set default azure subscrition to:"$azSubscriptionId
	azSub=$(az account set -s $azSubscriptionId)
				
	#create new SP account
	echo "Create a new service principal account..."
	newSP=$(az ad sp create-for-rbac --skip-assignment --years $yearsValidfor)
	
	if [ -z "$newSP" ]
	then
		echo "Service principal creation failed"
		exit 1
	fi
   
	spAppId=$(echo $newSP | jq -r .appId)
	spName=$(echo $newSP | jq -r .name)
	spPassword=$(echo $newSP | jq -r .password)
		
	echo "New service principal created, Id:"$spAppId" and Name:"$spName
	
	#update the existing AKS with new SP
	echo "Update AKS:"$aksClusterName" to use new service principal:"$spName
	aksUpdated=$(az aks update-credentials --resource-group $rgName -n $aksClusterName --reset-service-principal --service-principal $spAppId --service-principal $spAppId --client-secret $spPassword)
	echo $aksUpdated
}

rgName=$1
aksClusterName=$2
azSubscriptionId=$3
yearsValidfor=$4
#check arguments
[[ $# < 4 ]] && { usage && exit 1; } || RotateAKSCred "$@"
```
## Update AKS credentials by rotating credentials of assigned service pricipal

```Bash
#!/bin/bash
#AKS Service Priciple rotation Script 
#Requires jq and az cli
usage(){
        echo "***Rotate AKS Service Priciple credentials by updating existing service principal***"
        echo "Usage: ./SP_rotation.sh <rgName> <aksClusterName><azSubscriptionId><expiryDate>"
		echo "Expects 2 arguments: rgName, aksClusterName, azSubscriptionId AKS cluster resource group, cluster name, the azure subscription id, and desired new expiry date"
}

RotateAKSCred(){
	#set the azure subscription
	echo "set default azure subscrition to:"$azSubscriptionId
	azSub=$(az account set -s $azSubscriptionId)
	
	
	echo "Get the service principal assigned to AKS"
	spAppId=$(az aks show --resource-group $rgName --name $aksClusterName --query servicePrincipalProfile.clientId -o tsv)
	
	if [ -z "$spAppId" ]
	then
		echo "Failed to get assigned AKS service principal"
		exit 1
	fi
	
	echo "Reset credentials for AKS service principle account:"$spAppId
	newSP=$(az ad sp credential reset --name $spAppId --end-date $expiryDate)
	
	if [ -z "$newSP" ]
	then
		echo "Failed to reset credentials for Service principle:"$spAppId
		exit 1
	fi
	
	echo "list all sp credentials"
	spcredList=$(az ad sp credential list --id $spAppId)
	echo $spcredList
	
	spPassword=$(echo $newSP | jq -r .password)		
	echo "Service principal accountId:"$spAppId" credentials reset to new expiry date:"$expiryDate

	echo "Update AKS:"$aksClusterName" to use new service principal:"$spAppId" credentials"
	aksUpdated=$(az aks update-credentials --resource-group $rgName -n $aksClusterName --reset-service-principal --service-principal $spAppId --client-secret $spPassword)
	echo $aksUpdated
}

rgName=$1
aksClusterName=$2
azSubscriptionId=$3
expiryDate=$4
#check arguments
[[ $# < 4 ]] && { usage && exit 1; } || RotateAKSCred "$@"
```
Refernences:

-[Manage azure active directory service principals in AZ CLI](https://docs.microsoft.com/en-us/cli/azure/ad/sp?view=azure-cli-latest)

-[Update service principal for AKS](https://docs.microsoft.com/en-us/azure/aks/update-credentials#update-aks-cluster-with-new-credentials)
