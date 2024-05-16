# Exploring Gitops using ArgoCD
This repository contains the complete guide for deploying a web-app using Argocd on a Kubernetes cluster on a linux machine


### Setup
First make sure you have Docker installed on your local machine.
Follow the official documentation for installation https://docs.docker.com/engine/install/

### Step 1
Create a virtual kubernetes cluster. This documentation includes creating one using kind.

```bash
# install kind on linux

[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# create a kind cluster
kind create cluster
```
for any difficulties follow the official documentation
https://kind.sigs.k8s.io/docs/user/quick-start/

### Step 2
once you have a kubernetes cluster running install kubectl to interact with the cluster

```bash
sudo snap install kubectl --classic
```

### Step 3 
installing argocd on your kubernetes cluster
Official Documentation https://argo-cd.readthedocs.io/en/stable/getting_started/

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
installing argocd cli 

```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

Change the argocd-server service type to LoadBalancer

```bash 
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Kubectl port-forwarding can also be used to connect to the API server without exposing the service.

```bash 
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

you can now access API-server at https://localhost:8080

for login credentials
Username: admin
for password type in the following command to access
```bash
argocd admin initial-password -n argocd
```


### Step 4 

setting up argocd rollouts

```bash 
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

## Implementing argo-rollouts

Make sure you argo-rollouts installed then get kubectl with argo-rollouts plugin installed using

```bash
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x ./kubectl-argo-rollouts-linux-amd64
sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
```

Create a rollout specification deployment and yaml file.
Follow the official docs https://argoproj.github.io/argo-rollouts/getting-started/

Then run the command to deploy the initial rollout and service

```bash
kubectl apply -f <your github repo id path to rollout.yaml file>
kubectl apply -f <your github repo id path to service.yaml file>
```

Performing the update

```bash
kubectl argo rollouts set image <app name> \
  <name>=<updated image version>

# example command
kubectl argo rollouts set image rollouts-demo \
  rollouts-demo=argoproj/rollouts-demo:yellow
```

Monitoring the resources

```bash
kubectl argo rollouts get rollout <name> --watch
```

## Clearing the resources 

### Deleting the app deployed on kubernetes using argocd
you can delete the app deployed on the cluster simply using the ui provided by argocd or using argocd cli.
For more info https://argo-cd.readthedocs.io/en/stable

### Deleting the argocd deployment using kubectl

You can delete the namespace argocd created while deploying argocd which will in-turn delete all the resources(pods,services, deployments) associated with it, deleting the argocd deployments

```bash
kubectl delete namespace argocd
```

### Deleting the argo-rollouts extension.

```bash
kubectl delete namespace argo-rollouts
```

## Challenges you may face 

Installing kind doesn't work as directed alternative I found was using GO. Installing kind using go is simple and straightforward

Install go https://go.dev/doc/install

Then type in 
```bash
go install sigs.k8s.io/kind@v0.23.0
```











