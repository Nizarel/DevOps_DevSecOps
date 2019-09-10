# Rotate Service principal account for deployed AKS

This bash script uses Azure CLI to create a new service principal account to be used in a deployed [AKS](https://docs.microsoft.com/en-us/azure/aks/) cluster instead of the current service principal.

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
	echo "Usage: ./SP_rotation.sh <rgName> <aksClusterName> <azSubscriptionId> <yearsValidFor>"
	echo ""
	echo "Expects 4 arguments:"
	echo "    rgName = AKS cluster resource group name"
	echo "    aksClusterName = name of AKS cluster"
	echo "    azSubscriptionId = the azure subscription id"
	echo "    yearsValidFor = number of years new service principal account is valid for"
}

RotateAKSCred(){
	#set the azure subscription
	echo "Setting default azure subscrition to: "$azSubscriptionId
	azSub=$(az account set -s $azSubscriptionId)

	#create new SP account
	echo "Create a new service principal account."
	newSP=$(az ad sp create-for-rbac --skip-assignment --years $yearsValidFor)

	if [ -z "$newSP" ]
	then
		echo "Service principal creation failed."
		exit 1
	fi

	spAppId=$(echo $newSP | jq -r .appId)
	spName=$(echo $newSP | jq -r .name)
	spPassword=$(echo $newSP | jq -r .password)

	echo "New service principal created, Id: "$spAppId" and Name: "$spName

	#update the existing AKS with new SP
	echo "Update AKS Cluster: "$aksClusterName" to use new service principal: "$spName
	aksUpdated=$(az aks update-credentials --resource-group $rgName -n $aksClusterName --reset-service-principal --service-principal $spAppId --service-principal $spAppId --client-secret $spPassword)
	echo $aksUpdated
}

rgName=$1
aksClusterName=$2
azSubscriptionId=$3
yearsValidFor=$4
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
	echo "Usage: ./rotate_existing_SP.sh <rgName> <aksClusterName> <azSubscriptionId> <yearsValidFor>"
	echo ""
	echo "Expects 4 arguments:"
	echo "    rgName = AKS cluster resource group name"
	echo "    aksClusterName = name of AKS cluster"
	echo "    azSubscriptionId = the azure subscription id"
	echo "    yearsValidFor = number of years new service principal account is valid for"
}

RotateAKSCred(){
	#set the azure subscription
	echo "set default azure subscrition to: "$azSubscriptionId
	azSub=$(az account set -s $azSubscriptionId)


	echo "Get the service principal assigned to AKS."
	spAppId=$(az aks show --resource-group $rgName --name $aksClusterName --query servicePrincipalProfile.clientId -o tsv)

	if [ -z "$spAppId" ]
	then
		echo "Failed to get assigned AKS service principal."
		exit 1
	fi

	echo "Reset credentials for AKS service principle account: "$spAppId
	newSP=$(az ad sp credential reset --name $spAppId --years $yearsValidFor)

	if [ -z "$newSP" ]
	then
		echo "Failed to reset credentials for Service principle: "$spAppId
		exit 1
	fi

	echo "list all sp credentials"
	spcredList=$(az ad sp credential list --id $spAppId)
	echo $spcredList

	spPassword=$(echo $newSP | jq -r .password)
	echo "Service principal accountId: "$spAppId" credentials reset to expire in "$yearsValidFor" years"

	echo "Update AKS:"$aksClusterName" to use new service principal:"$spAppId" credentials"
	aksUpdated=$(az aks update-credentials --resource-group $rgName -n $aksClusterName --reset-service-principal --service-principal $spAppId --client-secret $spPassword)
	echo $aksUpdated
}

rgName=$1
aksClusterName=$2
azSubscriptionId=$3
yearsValidFor=$4
#check arguments
[[ $# < 4 ]] && { usage && exit 1; } || RotateAKSCred "$@"
```

## Refernences:

- [Manage azure active directory service principals in AZ CLI](https://docs.microsoft.com/en-us/cli/azure/ad/sp?view=azure-cli-latest)

- [Update service principal for AKS](https://docs.microsoft.com/en-us/azure/aks/update-credentials#update-aks-cluster-with-new-credentials)
