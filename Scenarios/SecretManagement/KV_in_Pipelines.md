# Secret management with Azure KeyVault

Use Key Vault to safeguard and manage cryptographic keys and secrets used by cloud applications and services.

## Azure Keyvault integration with Azure Devops

### Integration by using Variable groups

Manual integration with an existing keyvault, as apart of the pipeline setup.

1- create an azure keyvault instance and set the keys/secrets.

2- create a new variable group in azdo pipeline, enable "Link secrets from an azure keyvault as variables", then provide the subscription and keyvault name to link.

3- Ensure the Azure service connection has at least Get and List management permissions on the vault for secrets.  NOTE: You can authorize access to the key vault from Azure Devops if you are linking the key vault to a variable group.  If you are only using the key vault task, you need to set get/list permissions.

4- In the variable groups page, choose + Add to select specific secrets from your vault that will be mapped to this variable group.

**Notes:**

1- Only the secret names are mapped to the variable group, not the secret values. The latest version of the secret's value is fetched from the vault and used in the pipeline linked to the variable group during the run.

2- Changes made to existing secrets values in the key vault will available automatically to all the pipelines in which the variable group is used.

3- Newly added, or deleted secrets to the vault are not automatically reflected in the the associated variable groups. The secrets included in the variable group must be explicitly updated in order for the pipelines using the variable group to execute correctly.

4- Variable groups linked to KV can be linked in the build/release pipelines.

5- You can control users, and thier access roles to the linked variable group, from the security tab in the variable group.

Refernences:

- [Key vault with varibale groups](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=yaml)
- [Secure your keyvault](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-secure-your-key-vault)

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

References:

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
