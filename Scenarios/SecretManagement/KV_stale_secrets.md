# Find stale secrets within Azure Keyvault

Using Azure CLI, and the logging feature in azure Keyvault you can track/report secrets that have been stale within your vault.

Azure Keyvault uses Azure storage account service to store the logs of every event executed on a keyvault. The logs are stored as blobs where every log blob contains the logs of the events executed on the key vault for one hour. 

## Azure Keyvault logging setup :

1- create or use an exisiting azure key vault instance.

2- create or use an existing azure storage account for the logs.

3- set the storage account to be used for the keyvault logs using the following powershell code sample.

```Powershell
# Get the storage account
$sa=Get-AzureRmStorageAccount -ResourceGroupName "RG01" -AccountName "YOURSTORAGEACCOUNTNAME"

# Get the keyvault
$kv = Get-AzKeyVault -VaultName 'YOURVAULTNAME'

# Enable logging, you can set a retention policy for your logs using -RetentionEnabled, and  -RetentionInDays params
Set-AzDiagnosticSetting -ResourceId $kv.ResourceId -StorageAccountId $sa.Id -Enabled $true -Category AuditEvent -RetentionEnabled $true -RetentionInDays 180

```

## Report Azure keyvault stale secrets :

Now that you have the logging enabled on your keyvault. you can run the following bash script to report the stale secrets that have not been used in the last X days

```Bash
#!/bin/bash
#KV Stale Secrets Script 
#Requires jq and az cli
usage(){
        echo "***Find stale secrets in an azure keyVault through logging***"
        echo "Usage: ./KV_Stale_secrets.sh <storageAccountName> <keyvaultName> <azSubscriptionId> <daysOldLimit>"
		echo "Expects 3 arguments: storageAccountName, azure storage account name, key vault name, az SubscriptionId, and the no of days limit from last use"
}
ReportKVStaleSecrets(){
	
	#set the azure subscription
	echo "set default azure subscrition to:"$azSubscriptionId
	azSub=$(az account set -s $azSubscriptionId)
	
  	#Get all azure audit blobs in the storage account
	echo "Get the list of logs blobs in the storage account"
	blobs=$(az storage blob list -c insights-logs-auditevent --account-name $storageAccountName)
	
	#Get the list of vault secrets
	echo "Get the list of vault secrets"
	kvsecretslist=($(echo $(az keyvault secret list --vault-name $keyvaultName) | jq -r '.[].id' ))
	
	declare -A kvsecretslastuselist
	for ((idx=0; idx<${#kvsecretslist[@]}; ++idx)); do
			secret=${kvsecretslist[idx]}
			kvsecretslastuselist+=([$secret]=$(date -d 2000-01-01T01:00:00.5444810Z- +%F))		
	done
	
	#Get the list of azure blobs names
	blobnameslist=($(echo $blobs | jq -r '.[].name' ))
	
	#loop on every blob
	echo "download every blob in logs, read the contents and check the secrets events dates"
	for blobname in ${blobnameslist[*]}
	do
		
		echo "dowload log file "$blobname
		#download the blob it self sing the cmd
		blobfilename="blobfile"
		az storage blob download -f $blobfilename -c insights-logs-auditevent --account-name $storageAccountName -n $blobname
		
		#open the file to explor the events contents
		echo "open the file to explor the log contents, and check the secrets history"
		bloboperationslist=$(cat $blobfilename| jq '.')	
		operationsnames=($(echo $bloboperationslist| jq -r '.operationName')) 
		
		operationsids=($(echo $bloboperationslist| jq -r '.properties.id'))
		operationstime=($(echo $bloboperationslist| jq -r '.time'))
		
		
		for ((idx=0; idx<${#operationsnames[@]}; ++idx)); do
			if [ ${operationsnames[idx]} == 'SecretSet' ] || [ ${operationsnames[idx]} == 'SecretGet'  ]
			then
				
				#check if the secret id has version combined
				secretvalueurl=${operationsids[idx]}				
				vaultsecretname=${secretvalueurl##*secrets}
				charcount=$(echo $vaultsecretname | tr -cd '/' | wc -c)
								
				
				#remove the version, if found, from the secret ID
				if [  $charcount  -le 1 ]
				then
					secrettrimmedid=${operationsids[idx]}
					
				else
					secrettrimmedid=${operationsids[idx]%/*}
				fi
				
				after=$(date -d ${operationstime[idx]} +%s)		
				before=$(date -d ${kvsecretslastuselist[$secrettrimmedid]}  +%s)
				
				
				if [ $after > $before ] 
				then
					kvsecretslastuselist[$secrettrimmedid]=$(date -d ${operationstime[idx]} +%F)
				fi
			fi
		done		
	done
	
	#Print secrets last use
	for ((idx=0; idx<${#kvsecretslist[@]}; ++idx)); do
			secret=${kvsecretslist[idx]}
			daysdiff=$(( ($(date +%s) - $(date -d ${kvsecretslastuselist[$secret]} +%s) )/(60*60*24) ))
			
			if [ $daysdiff -ge $daysOldLimit ]
			then
				echo "stale secret: ["$secret"]" ", last time used : " "${kvsecretslastuselist[$secret]}" 	
			fi
	done
}
${kvsecretslastuselist[$secret]}
storageAccountName=$1
keyvaultName=$2
azSubscriptionId=$3
daysOldLimit=$4

#check arguments
[[ $# < 4 ]] && { usage && exit 1; } || ReportKVStaleSecrets "$@"

```

**Rerernences:**

- [Azure key-vault-logging](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-logging)

- [Manage Keyvault using Azure CLI](https://docs.microsoft.com/en-us/cli/azure/keyvault?view=azure-cli-latest)

- [Manage storage account blob using Azure CLI](https://docs.microsoft.com/en-us/cli/azure/storage/blob?view=azure-cli-latest)  
