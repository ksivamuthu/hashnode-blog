## AWS EKS Connector - Manage all your Kubernetes Clusters in one place

[Amazon EKS Connector](https://docs.aws.amazon.com/eks/latest/userguide/eks-connector.html) is now generally available. With EKS Connector, you can now extend the EKS console to view your Kubernetes clusters outside AWS. What !?! Yes, now we can use the EKS console to visualize the Kubernetes clusters, both EKS and non-EKS clusters running in.

- On-premises Kubernetes Cluster
- Self-managed Clusters running on AWS EC2 instances
- Clusters from other cloud providers

In today's blog, we will see how to connect the kubernetes clusters from my home lab and Azure / GCP Cloud providers along with EKS using eksctl. Once connected, we can see the cluster details, configurations, and workloads in one place on the EKS console.

## Kubernetes Clusters

**Home Lab**

I have the home lab kubernetes cluster running on my mini pc that's running Linux. It was previously running on Raspberry Pi, and later I switched the lab to mini pc. I often flash Raspberry Pi for my IoT experiments; there are no apparent reasons. You can set up the kubernetes cluster on Raspberry Pi if you wish. I'm using K3s - lightweight kubernetes distribution designed for running production workloads in resource-constrained, IoT edge devices.

**Azure Kubernetes Service**

The Amazon EKS connector can also connect the Kubernetes clusters running on other cloud providers. I've set up the AKS (Azure Kubernetes Service) for this blog in the Azure environment.

**EKS**

I've EKS cluster along with the other clusters too. Amazon Elastic Kubernetes Service (Amazon EKS) is a managed container service to run and scale Kubernetes applications in AWS infrastructure.

## Registering a Cluster

`eksctl` simplifies registering non-EKS clusters by creating the required AWS resources and generating Kubernetes manifests for EKS Connector to apply to the external cluster.

To register or connect a non-EKS Kubernetes cluster, run

```bash
➜ eksctl register cluster --name siva-home-lab --provider rancher
```

The supported providers are : EKS_ANYWHERE, ANTHOS, GKE, AKS, OPENSHIFT, TANZU, RANCHER, EC2, OTHER

When registering, eksctl creates three yaml files.

- eks-connector.yaml - Deploys EKS connector agent
- eks-connector-clusterrole.yaml - Cluster Role of cluster
- eks-connector-console-dashboard-full-access-group.yaml - Console Dashboard Full Access

```bash
➜ eksctl register cluster --name siva-home-lab --provider rancher
2022-01-03 21:30:07 [ℹ]  creating IAM role "eksctl-20220103213007309381"
2022-01-03 21:30:17 [ℹ]  registered cluster "siva-home-lab" successfully
2022-01-03 21:30:17 [ℹ]  wrote file eks-connector.yaml to /Users/ksivamuthu/personal
2022-01-03 21:30:17 [ℹ]  wrote file eks-connector-clusterrole.yaml to /Users/ksivamuthu/personal
2022-01-03 21:30:17 [ℹ]  wrote file eks-connector-console-dashboard-full-access-group.yaml to /Users/ksivamuthu/personal
2022-01-03 21:30:17 [!]  note: "eks-connector-clusterrole.yaml" and "eks-connector-console-dashboard-full-access-group.yaml" give full EKS Console access to IAM identity "arn:aws:iam::495775103319:user/siva", edit if required; read https://docs.aws.amazon.com/eks/latest/userguide/connector-grant-access.html for more info
2022-01-03 21:30:17 [ℹ]  run `kubectl apply -f eks-connector.yaml,eks-connector-clusterrole.yaml,eks-connector-console-dashboard-full-access-group.yaml` before 07 Jan 22 02:30 UTC to connect the cluster
```

Apply the generated YAMLs in your cluster.

```bash
➜ kubectl apply -f eks-connector.yaml, \
			eks-connector-clusterrole.yaml, \
			eks-connector-console-dashboard-full-access-group.yaml
```

## Dashboard - Workloads, Configuration & Tags

Once the cluster is registered and yamls are applied, you can view your non-EKS cluster in EKS Console.

![cluster](https://cdn.hashnode.com/res/hashnode/image/upload/v1641267529416/tVDE5aMnu.png)

You can see the workloads, configuration and manage tags of the non-eks clusters.
![cluster](https://cdn.hashnode.com/res/hashnode/image/upload/v1641267531267/jmEK2OyAa.png)
Let’s register the AKS cluster by repeating the registration steps.
![workloads](https://cdn.hashnode.com/res/hashnode/image/upload/v1641267533106/QM7ZH8lOj.png)

## Deregistering a cluster

To deregister cluster, you can use eksctl command.

```bash
➜ eksctl deregister cluster --name siva-home-lab --region us-east-1
```

It deregisters the cluster and we can delete the API server objects using kubectl

```bash
➜ eksctl deregister cluster --name siva-home-lab --region us-east-1
2022-01-03 21:56:39 [ℹ]  deleting IAM role "eksctl-20220103213007309381"
2022-01-03 21:56:39 [ℹ]  unregistered cluster "siva-home-lab" successfully
2022-01-03 21:56:39 [ℹ]  run `kubectl delete -f eks-connector.yaml,eks-connector-clusterrole.yaml,eks-connector-console-dashboard-full-access-group.yaml` on your cluster to remove EKS Connector resources
```

## Conclusion

EKS Anywhere and EKS Connector are part of a clear play for businesses embracing hybrid cloud and private infrastructure setups. EKS Connector supports self-managed clusters on EC2, EKS Anywhere clusters running on-premises, and other Kubernetes clusters running outside of AWS to the EKS console. It’s great tool providing single pane of dashboard managing all of your Kubernetes cluster including EKS & Non EKS.

I'm Siva - working as Sr. Software Architect at [Computer Enterprises Inc](https://www.ceiamerica.com) from Orlando. I'm an AWS Community builder, Auth0 Ambassador and I am going to write a lot about Cloud, Containers, IoT, and Devops. If you are interested in any of that, make sure to follow me if you haven’t already. Please follow me [@ksivamuthu](https://www.twitter.com/ksivamuthu) Twitter or check out my blogs at https://sivamuthukumar.com

