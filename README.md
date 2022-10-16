# QA POC

Estimated Time To Complete: 60 mins

---

These instructions will:
- Launch a cluster in Amazon Elastic Kubernetes Service (EKS):
- Run a proof of concept (POC) Quality Assurance (qa) environment
- Deploy the applications defined in qa directory of this repo using GitOps principles
- Immediately update the environment with any code changes to the main branch of this repository

The [react-poc](https://github.com/aaronbushell1984/react-poc) repository holds the react starter app poc deployed by this repository

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

#### [Configure AWS credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)

- Follow guidance at link above and using eksctl video resource below if not already authenticated.

```shell
aws configure
```

> :warning: WARNING: **Provisioning services on AWS will charge** :warning:
> - Approximately $10 for two days at time of writing
> - Close any unneeded services
> - Gain authorisation from bill payer

---

### Setup and Tear-Down

#### [Create Cluster](https://eksctl.io/introduction/)

- Uses a pre-defined file rather than command for speed and consistency:

```shell
eksctl create cluster -f stab-qa-poc.yaml
```

#### Update Kube Config

- Update local config for aws connection

```shell
aws eks update-kubeconfig --name stab-qa-poc
```

#### Check kubectl Connection and Pods Running

- Resolve any issues relating to installation and setup of tools if this command is not successful:

```shell
kubectl get pods --all-namespaces
```

#### Create qa Namespace

```shell
kubectl create namespace qa
```

##### Delete Cluster (as required)

- Remove the cluster after completing all other steps and as required in between breaks
- Setup will need to be completed again

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

- Check pods reach a running state before continuing:

```shell
kubectl get pods -n argocd -w
```

#### Forward localhost:8080 to Argo Server

- Expose Argo CD server running in cluster at on port 443 to your local at port 8080:

```shell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

#### Retrieve Initial Password

- Copy the password for later:

```shell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

#### Navigate to UI at localhost (proceed past http warning)

> [localhost:8080](http://localhost:8080)

#### Login Using Default Details

 - user = admin

 - password (from above)

> :warning: WARNING: **Update your password immediately by accessing Userinfo and then Update Password** :warning:

---

### Deploy Applications

#### App of Apps for QA Environment

- The App of Apps (apps.yaml) manages other applications (currently only react-poc)
- Other applications may be in other repositories
- Argo CD will monitor this repository for changes and deploy applications defined here automatically

```shell
kubectl apply --filename apps.yaml
```

### Watch in [Argo](https://localhost:8080/)

![image](https://user-images.githubusercontent.com/67497207/196037467-4577f99b-700d-4f93-89ba-934ae9f384ae.png)

### Watch in [AWS](https://eu-west-2.console.aws.amazon.com/)

![image](https://user-images.githubusercontent.com/67497207/196037652-a1d2977b-55d5-4bb5-859b-214d750ea8b0.png)

#### Access Application Locally

- Forward local port 1337 to the react-poc app listening on port 80 in the container
- This is convenience for poc and should be defined as an ingress later

```shell
kubectl port-forward <name-of-deployment> -n qa 1337:80
```

> :information_source: INFORMATION: **The name-of-deployment is available in both UI's** :information_source:

#### Get Name Of Deployment

- If name is required quickly from the command line you can run:

```shell
kubectl get pods --show-labels -n qa
```

### See POC running!

- Navigate to: [localhost:1337](http://localhost:1337/)

![image](https://user-images.githubusercontent.com/67497207/196037781-8941248b-418a-4886-a947-064231143812.png)

---

### Resources

#### Videos

- [TechWorld with Nana - eksctl](https://www.youtube.com/watch?v=p6xDCz00TxU&t=4s) - Starts at 4:46
- [Cloud Quick Labs - ArgoCD on AWS EKS](https://www.youtube.com/watch?v=PJTEDKOyAxo&t=1215s)
- [DevOPs Toolkit - GitOps](https://www.youtube.com/watch?v=vpWQeoaiRM4)

#### Guides

- [Official eksctl Documentation](https://eksctl.io/introduction/)
- [Getting started with Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)
- [Enabling IAM user and role access to your cluster](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html)
- [Argo CD - Declarative GitOps CD for Kubernetes](https://argo-cd.readthedocs.io/en/stable/)

