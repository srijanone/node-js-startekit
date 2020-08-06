# Introduction

NodeJS Starterkit provides developers an ability to get a containered nodejs application on local systems and helps them to deploy to to Kubernetes with minimal or no understanding of Kubernetes. The starterkit is build using Docker and Kubernetes best practices and hence developers can just write application logic without worrying about Docker and Kubernetes complexities.

# Local Development using Docker and docker-compose
Node Js Starterkit can be used to set up a containerized node web applications on your local system using docker-compose.

## Prerequisites  
* Install [Docker](https://docs.docker.com/get-docker/)
* Install [docker-composer](https://docs.docker.com/compose/install/)

## Download Starterkit

### Using git
```bash
git clone git@github.com:srijanone/node-js-startekit.git
```

### Using wget
```bash
wget https://github.com/srijanone/node-js-startekit/archive/master.zip
unzip master.zip
```

## Start the application
```bash
cd node-js-startekit
docker-compose up -d
```
Upon executing the commands above check if the respective services are up & running

```bash
docker-compose ps
      Name                     Command               State           Ports         
-----------------------------------------------------------------------------------
nodejs-2_backend    docker-entrypoint.sh npm start   Up      0.0.0.0:3000->3000/tcp
nodejs-2_postgres   docker-entrypoint.sh postgres    Up      0.0.0.0:5432->5432/tcp
$ curl localhost:3000
{"message":"Welcome to the beginning of nothingness."}% 
```

Check whether the application is accessible via browser, open localhost:3000 in your favorite browser
All the default enviroment variables are managed in .env shipped with the startekit, change them as per your requirement. 

----
**NOTE**
The values in .env files are for local development only. Never push these to git repositories.
Start building your application.
----

## Installing npm packages
```bash
docker-compose run --rm node npm i -S <package>@<release>
```

## Stop the application
```
docker-compose down
```

# Deploying the nodejs application to Kubernetes(PKS)

## Prerequisites  
* Install [helm](https://helm.sh/docs/intro/install/)
* Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

----
**NOTE**
This tutorial is only to get familiar with Kubernetes and is not recommended for production enviroment which should have well build CI/CD process.
----

## Set up the application locally

## Deploy the application to Kubernetes

### Prerequisite
* A docker repository is created on harbor.

### Authenticate to harbor

```bash
export DOCKERKUSER=my-user
export DOCKERPASS=my-password
docker login -u $DOCKERUSER -p $DOCKERPASS harbor.pks.aws.foo.com/srijan/my-node-app
```

----
**NOTE**
Please replace DOCKERUSER and DOCKERPASS with your harbor username and password
----

### Build the image locally

```bash
export REPOSITORY_URL=harbor.pks.aws.foo.com/srijan/my-node-app
docker build -t $REPOSITORY_URL:v1 .
```

### Push the image to harbor
```bash
docker push $REPOSITORY_URL:v1
```

### Login to PKS
```
export USERNAME=my-user
export PASSWORD=my-password
pks login -a api.pks.aws.yogendra.me -u $USERNAME -p $PASSWORD -k
```
Check the cluster to which you have access
```bash
pks clusters
PKS Version     Name     k8s Version  Plan Name  UUID                                  Status     Action
1.8.0-build.16  sandbox  1.17.5       small      0a405ecc-1034-47e7-a550-10781a846a07  succeeded  UPGRADE
1.8.0-build.16  devops   1.17.5       small      75d2b8d3-6a0e-4305-8586-b18c7abc95b3  succeeded  UPGRADE
```

Generate configuration file for the required cluster
```
pks get-credentials devops > devops.yaml
```
Check if cluster is accessible
```
kubectl cluster-info
Kubernetes master is running at https://devops-k8s.aws.yogendra.me.:8443
CoreDNS is running at https://devops-k8s.aws.yogendra.me.:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

### Create a namespace
```
kubectl create ns my-app
```

### Deploy docker image to Kubernetes cluster

The starterkit ships with pre-baked helm chart to deploy the application to Kubernetes. Execute the command below to deploy your docker image to PKS
cluster.
```
helm install my-release --set node.image.repository harbor.pks.aws.yogendra.me/srijan/my-app --set node.image.tag v1 ./charts/node --namespace my-app
```

Helm chart shipped with the starterkit has following parameters
## Parameters

The following table lists the configurable parameters of the node chart and their default values.

| Parameter                               | Description                                                                 | Default                                                 |
| --------------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------- |
| `global.imageRegistry`                  | Global Docker image registry                                                | `docker.io`                                             |
| `node.registry`                         | Node image registry                                                         | `nil`                                                   |
| `node.repository`                       | Node image name                                                             | `srijanlabs/node:demo`                                 |
| `node.pullPolicy`                       | Node image pull policy                                                      | `IfNotPresent`                                          |
| `node.extraEnv`                         | Node container environment variables                                        | `nill`                                                  |
| `node.command`                          | Node container entry point                                                  | from image                                              |
| `node.arg`                              | Node container arguments                                                    | from image                                              |
| `node.port`                             | Node container listing port                                                 | 9000                                                    |
| `nameOverride`                          | String to partially override node.fullname template                         | `nil`                                                   |
| `fullnameOverride`                      | String to fully override node.fullname template                             | `nil`                                                   |
| `applicationKind`                       | Deployment or ReplicaSet                                                    | `Deployment`                                            |
| `replicas`                              | Number of replicas for the application                                      | `1`                                                     |
| `extraEnv`                              | Any extra environment variables to be pass to the pods                      | `{}`                                                    |
| `affinity`                              | Map of node/pod affinities                                                  | `{}` (The value is evaluated as a template)             |
| `nodeSelector`                          | node labels for pod assignment                                              | `{}` (The value is evaluated as a template)             |
| `tolerations`                           | Tolerations for pod assignment                                              | `[]` (The value is evaluated as a template)             |
| `securityContext.enabled`               | Enable security context                                                     | `true`                                                  |
| `securityContext.fsGroup`               | Group ID for the container                                                  | `1001`                                                  |
| `securityContext.runAsUser`             | User ID for the container                                                   | `1001`                                                  |
| `resources`                             | Resource requests and limits                                                | `{}`                                                    |
| `service.type`                          | Kubernetes Service type                                                     | `NodePort`                                             |
| `service.port`                          | Kubernetes Service port                                                     | `80`                                                    |
| `service.annotations`                   | Annotations for the Service                                                 | {}                                                      |
| `service.loadBalancerIP`                | LoadBalancer IP if Service type is `LoadBalancer`                           | `nil`                                                   |
| `service.nodePort`                      | nodePort if Service type is `LoadBalancer` or `nodePort`                    | `nil`                                                   |
| `ingress.enabled`                       | Enable ingress controller resource                                          | `false`                                                 |
| `ingress.hosts[0].name`                 | Hostname to your node installation                                          | `node.local`                                            |
| `ingress.hosts[0].path`                 | Path within the url structure                                               | `/`                                                     |
| `ingress.hosts[0].tls`                  | Utilize TLS backend in ingress                                              | `false`                                                 |
| `ingress.hosts[0].certManager`          | Add annotations for cert-manager                                            | `false`                                                 |
| `ingress.hosts[0].tlsSecret`            | TLS Secret (certificates)                                                   | `node.local-tls-secret`                                 |
| `ingress.hosts[0].annotations`          | Annotations for this host's ingress record                                  | `[]`                                                    |
| `ingress.secrets[0].name`               | TLS Secret Name                                                             | `nil`                                                   |
| `ingress.secrets[0].certificate`        | TLS Secret Certificate                                                      | `nil`                                                   |
| `ingress.secrets[0].key`                | TLS Secret Key                                                              | `nil`                                                   |


Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

Example:-
```console
$ helm install my-release \
  --set replicas=2 \
    ./node
```

Check if application is running
```
kubectl get po -n my-app
NAME                            READY   STATUS    RESTARTS   AGE
my-app-v011-nk7mf             1/1     Running   0          114m
```
```
kubectl get svc -n my-app
NAME                  TYPE           CLUSTER-IP       EXTERNAL-IP                                                                    PORT(S)        AGE
my-app              LoadBalancer   10.100.200.128   a69e152e54f124a58abd2972f94589d5-2041595907.ap-southeast-1.elb.amazonaws.com   80:31841/TCP   17d
```

In case the external IP shows <PENDING> it implies that the cluster does not provide a Loadbalancer, in that case you can build a proxy to the concerned service

```
kubectl port-forward  -n my-app svc/my-app 8080:80
Forwarding from 127.0.0.1:8080 -> 3000
Forwarding from [::1]:8080 -> 3000
Handling connection for 8080
```

Access the application
```
curl localhost:8080
```
