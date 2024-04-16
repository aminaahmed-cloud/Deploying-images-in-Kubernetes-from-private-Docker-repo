# Deploying-images-in-Kubernetes-from-private-Docker-repo

## Overview
This guide demonstrates how to deploy an application from a private Docker registry to a Kubernetes cluster. The setup assumes the usage of an AWS Elastic Container Registry (ECR) for the private Docker repository and a local Minikube Kubernetes cluster.

### My Setup
- Private Docker repo hosted on AWS ECR
- Simple Node.js app called `my-app`
- Local Minikube setup 

## Main Points

1. Create Secret Component
2. Create Deployment Component

---

### 1. Create Secret Component

Create a secret component that contains access token credentials to the private repository, allowing Docker to pull the image inside the cluster.

**- Docker login: create a `config.json` file for the secret.**

```bash
aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin 211125390603.dkr.ecr.eu-west-2.amazonaws.com

cat .docker/config.json

```
Execute aws ecr get-login-password and copy the outcome to get the authentication token to log into AWS in the command line

```bash
minikube ssh
```

Then, login into the AWS private repo from Minikube, not from your computer.

```bash
docker login --username AWS -p [TOKEN] 211125390603.dkr.ecr.eu-west-2.amazonaws.com
```

Use the file .docker/config.json to create the Secret Component.

```bash
minikube cp minikube:/home/docker/.docker/config.json /users/ahmed/.docker/config.json
```

<img src="https://i.imgur.com/GZxCSVS.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


<img src="https://i.imgur.com/e44M2EM.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

**- Create docker-secret.yaml.**

```yaml

apiVersion: v1
kind: Secret
metadata:
  name: my-registry-key
data:
  .dockerconfigjson: [ENCODED_TOKEN]
type: kubernetes.io/dockerconfigjson

```
Apply the Secret Component.

```bash
kubectl create secret generic my-registry-key \
--from-file=.dockerconfigjson=/Users/ahmed/.docker/config.json \
--type=kubernetes.io/dockerconfigjson
```

**OR**

```bash
kubectl create secret docker-registry my-registry-key-two \
--docker-server=https://211125390603.dkr.ecr.eu-west-2.amazonaws.com \
--docker-username=AWS \
--docker-password=[ENCODED_PASSWORD]
```

<img src="https://i.imgur.com/xSIiALI.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


---

### 2. Create Deployment Component

**Create the deployment component for your app.**

file my-app-deployment.yaml.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-two
  labels:
    app: my-app-two
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app-two
  template:
    metadata:
      labels:
        app: my-app-two
    spec:
      imagePullSecrets:
      - name: my-registry-key-two
      containers:
      - name: my-app-two
        image: 211125390603.dkr.ecr.eu-west-2.amazonaws.com/my-app:1.0
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
```

Apply the deployment.

```bash
kubectl apply -f my-app-deployment.yaml
```

<img src="https://i.imgur.com/EsRqvug.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
