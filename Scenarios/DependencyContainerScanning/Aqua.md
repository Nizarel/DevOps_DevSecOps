# Aqua Security installation and configuration

This article explains, how to configure Aqua Security Tools on Azure Kubernetes Service with a scanning container in Azure Container Registry and use an Azure DevOps pipeline to scan containers during build operation. Enforcers can also be configured to scan deployed environments to insure that vulnerable containers are addressed that previously had no issues reported.

## Prerequistes

* A running instance of Azure Kubernetes Services
* An Azure Container Registry
* An Azure DevOps Project
* Azure CLI (latest version installed) / Cloudshell

## Install Aqua on Kubernetes

You can find a link to the official product documentation below. Of the many ways to get Aqua installed in your environment, one of the simplest ways to accomplish this is with a helm chart and the process is detailed in the link below:

* [Kubernetes - Aqua](https://docs.aquasec.com/docs/std-deployment-kubernetes)

PS. Login may be required in order to access the official documentation.

### Get the credentials of the k8s cluster

In order to manage your cluster using the kubectl command line tools you will need to retrieve the relevant cluster's access credentials. Start up a console window and issue the commands below:
```
$ az login
$ az aks get-credentials -n <YOUR CLUSTER NAME> -g <RESOURCE GROUP NAME>
```

### Create a security namespace

Create a separate security namespace by issuing the command below:

```
$ kubectl create namespace aqua-security
```

### Create a secret to access the Aqua Registry

NOTE: This is not for accessing your private registry. This will be used for the vendor registry.

Create a secret by issuing the comand below: (You will need the vendor issued username/password to login https://my.aquasec.com.)

```
$ kubectl create secret docker-registry aqua-registry --docker-server=registry.aquasec.com --docker-username=<Aquq Username>  --docker-password=<Aquq Password> --docker-email=no@email.com -n aqua-security
```

### Create a Service account 

```
kubectl create -n aqua-security -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: aqua
imagePullSecrets:
- name: aqua-registry
EOF

```

### Create a secret for AquaDB 

```
kubectl create secret generic aqua-db --from-literal=password=<DB_PASSWORD> -n aqua-security
```

### Create a YAML file 

The aqua deployments consists of an Aqua Server, Database (Aqua uses a PostgreSQL database to store scanning log.) and Gateway Server. Create a deployment yaml file by copying the yaml represented below:

```
apiVersion: v1
kind: Service
metadata:
  name: aqua-db
  labels:
    app: aqua-db
spec:
  ports:
    - port: 5432
  selector:
    app: aqua-db
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: aqua-db
spec:
  template:
    metadata:
      labels:
        app: aqua-db
      name: aqua-db
    spec:
      serviceAccount: aqua
      containers:
      - name: aqua-db
        image: registry.aquasec.com/database:4.0
        env:
          - name: POSTGRES_PASSWORD
            valueFrom:
              secretKeyRef:
                name: aqua-db
                key: password
        volumeMounts:
          - mountPath: /var/lib/postgresql/data
            name: postgres-db
        ports:
        - containerPort: 5432
      volumes:
        - name: postgres-db
          hostPath:
            path: /var/lib/aqua/db
---
apiVersion: v1
kind: Service
metadata:
  name: aqua-gateway
  labels:
    app: aqua-gateway
spec:
  ports:
    - port: 3622
  selector:
    app: aqua-gateway
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: aqua-gateway
spec:
  template:
    metadata:
      labels:
        app: aqua-gateway
      name: aqua-gateway
    spec:
      serviceAccount: aqua      
      containers:
      - name: aqua-gateway
        image: registry.aquasec.com/gateway:4.0
        env:
          - name: SCALOCK_GATEWAY_PUBLIC_IP
            value: aqua-gateway
          - name: SCALOCK_DBUSER
            value: "postgres"
          - name: SCALOCK_DBPASSWORD
            valueFrom: 
              secretKeyRef:
                name: aqua-db
                key: password
          - name: SCALOCK_DBNAME
            value: "scalock"
          - name: SCALOCK_DBHOST
            value: aqua-db
          - name: SCALOCK_DBPORT
            value: "5432"
          - name: SCALOCK_AUDIT_DBUSER
            value: "postgres"
          - name: SCALOCK_AUDIT_DBPASSWORD
            valueFrom: 
              secretKeyRef:
                name: aqua-db
                key: password
          - name: SCALOCK_AUDIT_DBNAME
            value: "slk_audit"
          - name: SCALOCK_AUDIT_DBHOST
            value: aqua-db
          - name: SCALOCK_AUDIT_DBPORT
            value: "5432"
        ports:
        - containerPort: 3622
---
apiVersion: v1
kind: Service
metadata:
  name: aqua-web
  labels:
    app: aqua-web
spec:
  ports:
    - port: 443
      protocol: TCP
      targetPort: 8443
      name: aqua-web-ssl
    - port: 8080
      protocol: TCP
      targetPort: 8080
      name: aqua-web

  selector:
    app: aqua-web
  type: LoadBalancer    
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: aqua-web
spec:
  template:
    metadata:
      labels:
        app: aqua-web
      name: aqua-web
    spec:
      serviceAccount: aqua
      containers:
      - name: aqua-web
        image: registry.aquasec.com/console:4.0
        env:
          - name: SCALOCK_DBUSER
            value: "postgres"
          - name: SCALOCK_DBPASSWORD
            valueFrom:
              secretKeyRef:
                name: aqua-db
                key: password
          - name: SCALOCK_DBNAME
            value: "scalock"
          - name: SCALOCK_DBHOST
            value: aqua-db
          - name: SCALOCK_DBPORT
            value: "5432"
          - name: SCALOCK_AUDIT_DBUSER
            value: "postgres"
          - name: SCALOCK_AUDIT_DBPASSWORD
            valueFrom: 
              secretKeyRef:
                name: aqua-db
                key: password
          - name: SCALOCK_AUDIT_DBNAME
            value: "slk_audit"
          - name: SCALOCK_AUDIT_DBHOST
            value: aqua-db
          - name: SCALOCK_AUDIT_DBPORT
            value: "5432"
        volumeMounts:
          - mountPath: /var/run/docker.sock
            name: docker-socket-mount
        ports:
        - containerPort: 8080
      volumes:
        - name: docker-socket-mount
          hostPath:
            path: /var/run/docker.sock
```

Once create the YAML file, apply it. 

```
kubectl create -f aqua-server.yaml -n aqua-security
```

## Configure the Aqua Security Dashboard

After the deployment, You can see the aqua-web external ip-address. For the purpose of this document the IP address will bre represented as  `13.66.214.195`, please change the address to the IP Address for your environment. 

```
$ kubectl get service aqua-web -n aqua-security
NAME       TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)                        AGE
aqua-web   LoadBalancer   10.0.94.99   13.66.214.195   443:30029/TCP,8080:30734/TCP   1m
```

To view the administrator dashboard, from a browser window,navigate to  `http://13.66.214.195:8080` and provide the license token for your environment as depicted below.

![Token input](images/token.png)

This token can be retrieved from https://my.aquasec.com and pasted in the area provided on this screen. 

![my.aquqsec.com token](images/token2.png)

The Aqua dashboard will be displayed which will provide you with configuration options for your instance.

### Configure Azure Container Registry integration

If you are interested in integrating the scanner with your existing ACR so that you can scan existing containers you can refer to the link below:

  [Image Vulnerability Scanning in Azure Container Registry](https://blog.aquasec.com/image-vulnerability-scanning-in-azure-container-registry)

In the Aqua left menu select:
  System > integration 

![Azure Container Registry integration](images/aqua-integration.png)

Input the value of the Container Registry, click Save changes > Test Connection. 

Then to scan the images in the registry, click `ADD IMAGES`  and scan.

![Add Images](images/images.png)

### View Scan Results

After a scan has been concluded. You will see a collection of results alerting you to vulnerabilities found, sensitive information identified and whether or not malware was detected. Some samples are depicted below:

![dashboard](images/risk.PNG)
![Scan images](images/scan-images.PNG)
![Resource](images/Resources.png)
![Sensitive Data](images/sensitive.png)
![Vulnerability](images/vulnerability.png)

## Configure the Aqua Task in a CI pipeline in Azure DevOps

Navigate to the address below or search for Aqua security in the Azure DevOps Marketplace.  

 [Container Security](https://marketplace.visualstudio.com/items?itemName=aquasec.aquasec)

Once you have installed the extension, create a pipeline with a Linux based hosted agent. e.g. Hosted Ubuntu 1604.  

The steps that supports this workflow are detailed below:

* Build a target image 
* Login to the Aqua Security registry
* Pull the scanner image
* Scan the target image

![CI overview](images/CI.png)

### Build a target image

Build your docker image in this task. Don't forget to add a tag to uniquely identify the result of a pipeline execution.  Provide configuration information for your Container Registry by supplying the address to DockerHub or your private Azure Container Registry. 

![Docker build](images/docker-build.png)

If you have not pre-configured the service connection to your container registry, click manage link and configure the host,username and password.

![Service Connection (Azure Container Registry)](images/ContainerRegistrySettings.png)

![Azure Portal (Azure Container Registry)](images/containerRegistry.png)

### Login to the Aqua Security registry

Supply the log in information for the Aqua Security registry

![Docker login](images/docker-login.png)

 If you have not pre-configured the service connection to the Aqua Security registry, click the "Manage" link and follow the configuration instructions, remembering to provide the username/password that you use to log into https://my.aquqsec.com.

 ![Aqua registry](images/ContainerRegistrySettingsForAqua.png)

### Pull the scanner image 

Download the scanner image from Aqua registry. (The Aqua Security task will use this image to scan the target image for vulnerabilities).

![Download the scanner](images/docker-pull.png)

### Scan the target image using the Aqua Security Scanning Task

Please note that the default configuration of the Docker task will append the Azure Container Registry hostname as a prefix to the image name. It is also prudent to insure that an a unique image tag is provided. It is therefore important to match this when configuring the Aqua Security Scanning Task.

![Image scan](images/docker-aqua-scan.png)

If you have not pre-configured the service connection to the Aqua Management Consol, click "Manage" and provide the configuration information for your Aqua management consol.

![Aqua management console connection](images/aqua-management-console-connection-settings.png)

## Security Vulnerability Reports. 

### Pipeline Build Output 

Once the pipeline has executed, a new tab will be visible names "Aqua Security Report" This tab should provide vulnerability information for the image scanned directly in the pipeline and as a result there is no need to navigate to the Aqua Dashboard to view. It should be noted that this information is also stored in the AquaSec Database in your k8s cluster. The image below depicts a sample pipeline report.

![Pipeline report](images/pipeline-report.png)

### Dashboard report

To view a history of reports, produced by the tool. Navigate to the AquaSec Portal.

Go to Images > CI/CD Scans. Here you will be able to inspect reports produced as a result of image scans within your build pipeline.

![Dashboard report](images/dashboard-report.png)

## Windows Container Scanning with CI on a Self-Hosted agent

In order to perform scans on Windows container images in a self hosted environment a few additional configuration steps are require.

TO save time during build executions, install the scanner-cli on the self-hosted agent in advance.  You can download the scanner-cli using the link below. 
Download the scanner-cli from scanner-cli (Windows).

### Download and install scanner-cli

* [Version 3.5.0.3 (2019 Jan 14)](https://docs.aquasec.com/v3.5/docs/version-3503)

You will need to provide the installer with the URL of the Aqua DashBoard(on k8s), the Username and password of the Aqua Server when prompted.This can be automated with msiexec if you prefer.

### Integratinq with the Azure DevOps Pipeline 

Configure the Task in the same way that it is configured for Linux image scans, expect in the case of the settings depicted below. Bear in mind that you do not need to pull the scanner docker since it may already be installed.
 
 ![Aqua Security Task](images/scanning.png)

* [Azure DevOps (Microsoft VSTS) Integration](https://docs.aquasec.com/docs/azure-devops-integration)
* [Scanner-CLI](https://docs.aquasec.com/v3.5/docs/command-line)
