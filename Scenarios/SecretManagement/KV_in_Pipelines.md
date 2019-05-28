# Secret management with Azure KeyVault

- [Secret management with Azure KeyVault](#secret-management-with-azure-keyvault)
  - [Azure Keyvault integration with Azure Devops](#azure-keyvault-integration-with-azure-devops)
    - [Integration by using Variable groups](#integration-by-using-variable-groups)
    - [Integration by using Azure KeyVault task](#integration-by-using-azure-keyvault-task)
  - [Azure Keyvault integration with Jenkins](#azure-keyvault-integration-with-jenkins)
  - [Design/split secrets across environments for AZDO with KV](#designsplit-secrets-across-environments-for-azdo-with-kv)
    - [One KV to store credentails for all environments (Dev/QA/Prod)](#one-kv-to-store-credentails-for-all-environments-devqaprod)
    - [One KV to store credentails Per environment (Dev/QA/Prod)](#one-kv-to-store-credentails-per-environment-devqaprod)

## Azure Keyvault integration with Azure Devops

### Integration by using Variable groups

Manual integration with an existing keyvault, as apart of the pipeline setup.

1- create an azure keyvault instance and set the keys/secrets.

2- create a new variable group in azdo pipeline, enable "Link secrets from an azure keyvault as variables", then provide the subcsrition and keyvault name to link.

3- Ensure the Azure service connection has at least Get and List management permissions on the vault for secrets.

4- In the variable groups page, choose + Add to select specific secrets from your vault that will be mapped to this variable group.

**Notes:**

1- Only the secret names are mapped to the variable group, not the secret values. The latest version of the secret's value is fetched from the vault and used in the pipeline linked to the variable group during the run.

2- Changes made to existing secrets values in the key vault will available automatically to all the pipelines in which the variable group is used.

3- Newly added, or deleted secrets to the vault are not automatically reflected in the the associated variable groups. The secrets included in the variable group must be explicitly updated in order for the pipelines using the variable group to execute correctly.

4- Variable groups linked to KV can be linked in the build/release pipelines.

5- You can control users, and thier access roles to the linked variable group, from the security tab in the variable group.

Refernences:
- [Key vault with varibale groups](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=yaml)


### Integration by using Azure KeyVault task

The task can be used to fetch the latest values of all or a subset of secrets from the vault, and set them as variables that can be used in subsequent tasks of a pipeline.

1- create an azure keyvault instance and set the keys/secrets.

2- Ensure the Azure service connection has at least Get and List management permissions on the vault for secrets.

3- Add "Azure Key Vault" task to the build/relesa pipeline, configure and authorize the connection to your KV istance.

4- Select secrets to use in the secret filter field or leave * to fetch all secrets from the selected key vault.

5- the task generates variables with the same name as in KV that can be used by subsequent tasks. use $(KVSecretname) to reference the creaeted variable.

6- Handeling certificate encryption/decryption sample [Here](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/azure-key-vault?view=azure-devops)

**Notes:**

1- Newly added, or deleted secrets to the vault are automatically reflected in created task variablesif you chose '*' in secret filter field in task setup.

Refernences:
- [Key vault task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/azure-key-vault?view=azure-devops)


## Azure Keyvault integration with Jenkins
TO be investigated

## Design/split secrets across environments for AZDO with KV

### One KV to store credentails for all environments (Dev/QA/Prod)

- Use variable groups to split variables based on env ( 1 group per env)
- Link secrets used for the same env to the same variable group ( e.g all QA secrests linked to the QA variable group)
- Link each group to its corresponding phase in release pipelines, or in build pipeline. (e.g dev variable group linked to build pipeline while QA linked to the QA stage in the relase pipeline)

### One KV to store credentails Per environment (Dev/QA/Prod)

Gives more control on who case see, use the linked variable groups.

- You can still have multiple groups as above reading from the same variable group (e.g Prod2, prod 2, prod 3) can all read from the production key vault as they are linked to "relase to Prod" phase in the release pipeline 
