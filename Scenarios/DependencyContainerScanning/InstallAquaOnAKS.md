# Install Aqua on Kubernetes

This article explains, how to configure Aqua Security Tools on Azure Kubernetes Service with a scanning container in Azure Container Registry and use an Azure DevOps pipeline to scan containers during build operation. Enforcers can also be configured to scan deployed environments to assure that vulnerable containers are addressed that previously had no issues reported.

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

The aqua deployments consists of an Aqua Server, Database (Aqua uses a PostgreSQL database to store scanning log.) and Gateway Server. Create a deployment yaml file by copying the yaml represented below:

```Yaml
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
