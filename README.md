# QA POC

### Pre-Requisites



### Commands

#### Create Cluster

```shell
eksctl create cluster -f stab-qa-poc.yaml
```

#### Update Kube Config

```shell
aws eks update-kubeconfig --name stab-qa-poc
```

#### Check kubectl Connection and Pods Running

```shell
kubectl get pods --all-namespaces
```

#### Install Argocd to Cluster

```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

#### Check Argo Pods are running

```shell
kubectl get pods -n argocd
```

#### Forward localhost:8080 to Argo Server

```shell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

#### Retrieve Initial Password

```shell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

#### Navigate to UI at localhost (proceed past http warning)

> [localhost:8080](http://localhost:8080)

#### Login Using Default Details

 - user = admin

 - password (from above)

> :warning: **Update your password immediately**

#### Create qa Namespace

```shell
kubectl create namespace qa
```

#### Deploy qa App of Apps

- This will look for templates in the qa directory to deploy

```shell
kubectl apply --filename apps.yaml
```

##### Delete Cluster

```shell
eksctl delete cluster -n stab-qa-poc
```