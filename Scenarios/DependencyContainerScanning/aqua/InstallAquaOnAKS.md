# Install Aqua on Kubernetes

This article explains, how to configure Aqua Security Tools on Azure Kubernetes Service (AKS) with a scanning container in Azure Container Registry and use an Azure DevOps pipeline to scan containers during build operation. Enforcers can also be configured to scan deployed environments to assure that vulnerable containers are addressed that previously had no issues reported.

## Prerequistes

* A running instance of Azure Kubernetes Services
* An Azure Container Registry
* An Azure DevOps Project
* Azure CLI (latest version installed) / Cloudshell

You can find a link to the official product documentation below. Of the many ways to get Aqua installed in your environment, one of the simplest ways to accomplish this is with a helm chart using the process that is detailed in the link below:

* [Kubernetes - Aqua](https://docs.aquasec.com/docs/std-deployment-kubernetes)

_Note:_ Login is required to access the official documentation.

## Get the credentials of the k8s cluster

In order to manage your cluster using the kubectl command line tools you will need to retrieve the relevant cluster's access credentials. Start up a console window and issue the commands below:

``` Bash
az login
az aks get-credentials -n <YOUR CLUSTER NAME> -g <RESOURCE GROUP NAME>
```

## Create a security namespace

Create a separate security namespace by issuing the command below:

``` Bash
kubectl create namespace aqua-security
```

## Create a secret to access the Aqua Registry

NOTE: This is not for accessing your private registry. This will be used for the vendor registry.

Create a secret by issuing the comand below: (You will need the vendor issued username/password to login <https://my.aquasec.com>).

``` Bash
kubectl create secret docker-registry aqua-registry --docker-server=registry.aquasec.com --docker-username=<Aquq Username>  --docker-password=<Aquq Password> --docker-email=no@email.com -n aqua-security
```

## Create a Service account

``` Yaml
kubectl create -n aqua-security -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: aqua
imagePullSecrets:
- name: aqua-registry
EOF

```

## Create a secret for AquaDB

```Bash
kubectl create secret generic aqua-db --from-literal=password=<DB_PASSWORD> -n aqua-security
```

## Create a YAML file

The aqua deployments consists of an Aqua Server, Database (Aqua uses a PostgreSQL database to store scanning log.) and Gateway Server all of which can be installed using [this yaml](./aqua.yaml)

Once the YAML file is create go ahead and apply it.

``` Bash
kubectl create -f aqua-server.yaml -n aqua-security
```

## Configure the Aqua Security Dashboard

After the deployment you can see the aqua-web external ip-address. For the purposes of this document the IP address will be represented as  `13.66.214.195`. **Important:** change the address to the IP Address for your environment!

``` Bash
kubectl get service aqua-web -n aqua-security
NAME       TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)                        AGE
aqua-web   LoadBalancer   10.0.94.99   13.66.214.195   443:30029/TCP,8080:30734/TCP   1m
```
