## EKS Anywhere - Running EKS Cluster on your own infrastructure

- [What is EKS Anywhere?](#what-is-eks-anywhere)
- [What does EKS Anywhere solve?](#what-does-eks-anywhere-solve)
- [Announcement](#announcement)
- [First Look 👀](#first-look-)
  - [Installing EKS Anywhere (Local)](#installing-eks-anywhere-local)
  - [Manage on-premise EKS Anywhere Cluster from AWS EKS Console](#manage-on-premise-eks-anywhere-cluster-from-aws-eks-console)
  - [Setup GitOps in EKS Anywhere Cluster](#setup-gitops-in-eks-anywhere-cluster)
- [Conclusion](#conclusion)

## What is EKS Anywhere?

Amazon EKS Anywhere is a new deployment option for Amazon EKS that enables you to easily create and operate Kubernetes clusters on-premises with your virtual machines. It brings a consistent AWS management experience to your data center, building on the strengths of Amazon EKS Distro, the same distribution of Kubernetes that powers EKS on AWS. Its goal is to include complete lifecycle management of multiple Kubernetes clusters capable of operating entirely independently of any AWS services.

## What does EKS Anywhere solve?

The central selling point of EKS Anywhere is that it allows customers to use Amazon’s Kubernetes software within private data centers. It makes EKS Anywhere handy in situations where businesses already have a large on-premises infrastructure that they want to modernize the cloud, keep the data in their private network due to legal reasons, or where they believe their total cost of ownership will be lower if they host workloads on their servers instead of in the public cloud.

So, the use cases the EKS Anywhere solves are

1. Hybrid Cloud Consistency
2. Disconnected Environment
3. Application Modernization
4. Data sovereignty

## Announcement

ECS Anywhere and EKS Anywhere — two new versions of AWS' managed containers and managed Kubernetes services announcements are made at AWS re: Invent 2020. In [May 2021, ECS Anywhere was generally available](https://aws.amazon.com/about-aws/whats-new/2021/05/amazon-elastic-container-service-anywhere-is-now-generally-available/), and today (8th September 2021 [the EKS Anywhere is available](https://aws.amazon.com/blogs/aws/amazon-eks-anywhere-now-generally-available-to-create-and-manage-kubernetes-clusters-on-premises/) for general use.

%[https://twitter.com/ksivamuthu/status/1435678851435311110]

## First Look 👀

I planned to explore AWS EKS with the three features I'm excited about.

- Installation methods. (Local and/or typical Production on-premise environments (VMware)
- Manage on-premise EKS Anywhere Cluster from AWS EKS Console
- Setup GitOps in EKS Anywhere Cluster

In this blog post, I'm capturing the steps I followed and my thoughts on them.

### Installing EKS Anywhere (Local)

The [eksctl](https://github.com/weaveworks/eksctl) tool is my favorite tool for creating the EKS cluster in AWS. To install "EKS Anywhere" eksctl got a plugin called `eksctl-anywhere` . This will let you create a cluster in multiple providers for local development or production workloads.

- I tried macOS first. It got stuck at "Creating new bootstrap cluster" ending up a lot of containers created and exiting. Maybe, it's an issue with my macOS resource requirements. (16GB RAM, 4GB assigned for docker). Later I switched to Linux VM (2 Core, 8GB Ram) created in Azure.

To install in macOS using home brew.

```bash
brew install aws/tap/eks-anywhere
```

or install plugin using the following command in Linux.

```bash
export EKSA_RELEASE="0.5.0" OS="$(uname -s | tr A-Z a-z)"
curl "https://anywhere-assets.eks.amazonaws.com/releases/eks-a/1/artifacts/eks-a/v${EKSA_RELEASE}/${OS}/eksctl-anywhere-v${EKSA_RELEASE}-${OS}-amd64.tar.gz" \
    --silent --location \
    | tar xz ./eksctl-anywhere
sudo mv ./eksctl-anywhere /usr/local/bin/
```

Now, we are going to install the EKS Anywhere in the local environment using the docker provider.

```bash
eksctl anywhere generate clusterconfig siva-dev --provider docker > siva-dev.yaml
eksctl anywhere create cluster -f siva-dev.yaml
```

The cluster is created successfully in a Linux VM.

```bash
root@eks-anywhere-vm:/home/azureuser# eksctl anywhere create cluster -f siva-dev.yaml
Performing setup and validations
Warning: The docker infrastructure provider is meant for local development and testing only
✅ Docker Provider setup is valid
Creating new bootstrap cluster
Installing cluster-api providers on bootstrap cluster
Provider specific setup
Creating new workload cluster
Installing networking on workload cluster
Installing storage class on workload cluster
Installing cluster-api providers on workload cluster
Moving cluster management from bootstrap to workload cluster
Installing EKS-A custom components (CRD and controller) on workload cluster
Creating EKS-A CRDs instances on workload cluster
Installing AddonManager and GitOps Toolkit on workload cluster
GitOps field not specified, bootstrap flux skipped
Writing cluster config file
Deleting bootstrap cluster
🎉 Cluster created!
```

Install test workloads.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.0.0/docs/examples/echoservice/echoserver-namespace.yaml &&\
> kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.0.0/docs/examples/echoservice/echoserver-service.yaml &&\
> kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.0.0/docs/examples/echoservice/echoserver-deployment.yaml
```

and verify the resources are created.

```bash
root@eks-anywhere-vm:/home/azureuser# k get all -n echoserver
NAME                              READY   STATUS    RESTARTS   AGE
pod/echoserver-7dd65ffdd6-jzl4f   1/1     Running   0          3m20s

NAME                 TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/echoserver   NodePort   10.137.108.237   <none>        80:30795/TCP   3m21s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/echoserver   1/1     1            1           3m20s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/echoserver-7dd65ffdd6   1         1         1       3m20s
```

**Installing EKS Anywhere (VMWare)**

I didn't try this out in VMWare yet. You can follow through with the documentation [here](https://anywhere.eks.amazonaws.com/docs/getting-started/production-environment/). The bare metal support is planned to release in 2022.

### Manage on-premise EKS Anywhere Cluster from AWS EKS Console

The eksctl and kubectl tools are helping us to manage the on-premise EKS Anywhere cluster. Now let's see how to connect this on-premise cluster in the AWS EKS console and manage it from there as a single pane of a dashboard of all of your EKS clusters in AWS EKS and on-premise EKS Anywhere. For this purpose, The AWS EKS Connector lets you connect your EKS Anywhere cluster to the AWS EKS console, where you can see your EKS Anywhere cluster, its configuration, workloads, and their status. EKS Connector is a software agent that can be deployed on your EKS Anywhere cluster, enabling the cluster to register with the EKS console.

Create the required IAM roles and policies to register a cluster.  I followed the documentation [here](https://docs.aws.amazon.com/eks/latest/userguide/eks-connector.html).  Create a service-linked role - Don't miss this step. I got stuck in this without any idea why EKS Anywhere Cluster is not registering.

```bash
aws iam create-service-linked-role --aws-service-name eks-connector.amazonaws.com
```

Register the EKS cluster by selecting EKS Anywhere Provider and IAM role. It will give you the yaml file of the ***eks-connector-agent*** to install in the EKS anywhere cluster. And if you apply t[he necessary permissions to the IAM for managing the cluster](https://docs.aws.amazon.com/eks/latest/userguide/connector-grant-access.html), all set. 

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1631152082138/d_j5QtwtO.png)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1631152105679/HNMCEGNnH.png)

Now you can manage the workloads, configuration of the EKS anywhere cluster from the AWS EKS console.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1631152138798/B1tbPstZC.png)

### Setup GitOps in EKS Anywhere Cluster

EKS Anywhere supports a [GitOps](https://www.weave.works/technologies/gitops/) workflow for the management of your cluster. This is another feature I'm very much interested in - to manage the cluster configuration and workloads from the version-controlled source repo in Git.

When you create a cluster with GitOps enabled, EKS Anywhere will automatically commit your cluster configuration to the provided GitHub repository and install a GitOps toolkit on your cluster which watches that committed configuration file.  Once a change is detected by the GitOps controller running in your cluster, the cluster will be adjusted to match the committed configuration file.

Add the gitops configuration as below in the YAML file you generated above.  There is no way to modify the existing cluster from eksctl anywhere. So I deleted the cluster and recreated it with gitops configuration.

```yaml
apiVersion: anywhere.eks.amazonaws.com/v1alpha1
kind: Cluster
metadata:
  name: siva-dev
spec:
	...
  gitopsRef:
    kind: GitOpsConfig
    name: siva-dev-gitops
---
apiVersion: anywhere.eks.amazonaws.com/v1alpha1
kind: DockerDatacenterConfig
metadata:
  name: siva-dev
spec: {}

---
apiVersion: anywhere.eks.amazonaws.com/v1alpha1
kind: GitOpsConfig
metadata:
  name: siva-dev-gitops
spec:
  flux:
    github:
      personal: true
      repository: eks-anywhere-gitops
      owner: ksivamuthu
```

The repo is set and the flux is installed in the cluster pointing to the repo we configured. The changes in the configuration you pushed to the git branch will be monitored by the GitOps controller. GitOps Controller pulls the changes and apply the changes in the cluster - configuration and workloads. 

```docker
root@eks-anywhere-vm:/home/azureuser# k get pods -n flux-system
NAME                                       READY   STATUS    RESTARTS   AGE
helm-controller-6d875c9745-8np9b           1/1     Running   0          8m9s
kustomize-controller-74c85f9944-8f48m      1/1     Running   0          8m9s
notification-controller-7c59756d9d-krtrc   1/1     Running   0          8m9s
source-controller-65dcfdf7f7-twds7         1/1     Running   0          8m9s
```

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1631152739634/2Mhy6pTsu.png)

## Conclusion

With Amazon EKS Anywhere, customers can now create and run Kubernetes clusters on-premises using VMware Inc.’s vSphere. It comes with an installable software package to create and operate Kubernetes, plus automation tools for cluster lifecycle support. I'm so interested to explore the possibilities of EKS Anywhere at the customer's on-premise environments on modernization of legacy workloads.  Let me know your thoughts in the comments below.

I'm Siva - working as Sr. Software Architect at [Computer Enterprises Inc](https://www.ceiamerica.com) from Orlando. I'm an AWS Community builder, Auth0 Ambassador and I am going to write a lot about Cloud, Containers, IoT, and Devops. If you are interested in any of that, make sure to follow me if you haven’t already. Please follow me [@ksivamuthu](https://www.twitter.com/ksivamuthu) Twitter or check out my blogs at https://blog.sivamuthukumar.com