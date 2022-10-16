# QA POC

---

### Pre-Requisites

#### [Install eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

```shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

#### [Install aws cli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)

```shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

#### Authenticate with AWS

```shell

```

---

### Setup and Tear-Down

#### [Create Cluster](https://eksctl.io/introduction/)

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

#### Create qa Namespace

```shell
kubectl create namespace qa
```

##### Delete Cluster (as required)

```shell
eksctl delete cluster -n stab-qa-poc
```

---

### ArgoCD

#### [Install Argocd to Cluster](https://argo-cd.readthedocs.io/en/stable/getting_started/)

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

---

### Deploy Applications

#### App of Apps - qa

- This will look for templates in the qa directory to deploy

```shell
kubectl apply --filename apps.yaml
```
