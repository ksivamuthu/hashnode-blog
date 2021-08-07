## EKS VPC CNI Addon - Support for high pod density in node

In this blog post, let's see how the IP Address prefix assignment functions enable EKS to support more pods per node. 

By default, the number of IP addresses available to assign to pods is based on the number of IP addresses assigned to Elastic network interfaces and the number of network interfaces attached to your Amazon EC2 node. The  [Amazon VPC CNI add-on (v1.9.0 or later) ](https://github.com/aws/amazon-vpc-cni-k8s/releases/tag/v1.9.0) can be configured to assign /28 (16 IP addresses) IP address prefixes, instead of assigning individual IP addresses to network interfaces. 

The CNI v1.9.0 release will support higher pod density per node and also reduces the number of EC2 calls to create and attach more ENIs by leveraging the recent EC2 feature -  [Assigning prefixes to Amazon EC2 network interfaces](https://aws.amazon.com/about-aws/whats-new/2021/07/amazon-virtual-private-cloud-vpc-customers-can-assign-ip-prefixes-ec2-instances/). A (/28) prefix will replace each secondary IP. IPAMD will derive a (/32) IP from these prefixes for pod IP allocation.

### Prerequisites

1. New or Existing EKS Cluster. You can follow the guidelines documented  [here](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html) to create an EKS cluster using the portal or eksctl.

2. Amazon VPC CNI (v1.9.0 or later) add-on added to your cluster. To add or update the add-on on your cluster, see  [Managing the Amazon VPC CNI add-on](https://docs.aws.amazon.com/eks/latest/userguide/managing-vpc-cni.html).  If you want to update the version in the existing cluster, run the below command.
```bash
eksctl update addon \
    --name vpc-cni \
    --version <1.9.x-eksbuild.y> \
    --cluster <my-cluster> \
    --force
```
3.  [Amazon EC2 Nitro instances ](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html#ec2-nitro-instances) for node groups in the cluster.

### How to enable the feature?

To enable the IP prefix assignment feature, run the below command.
```bash
kubectl set env daemonset aws-node \
 -n kube-system ENABLE_PREFIX_DELEGATION=true
```

### Calculate the number of maximum pods per node.

Download the max pods per node calculator script and change the access mode.

```bash
curl -o max-pods-calculator.sh https://raw.githubusercontent.com/awslabs/amazon-eks-ami/master/files/max-pods-calculator.sh
chmod +x max-pods-calculator.sh
```
Calculate the max pods using the max pods calculator script. You can pass the instance type and CNI version to calculate the max pods per node.

```bash
./max-pods-calculator.sh \
--instance-type <m5.large> \
--cni-version <1.9.x-eksbuild.y> \
--cni-prefix-delegation-enabled
```
You can take a look at the max pods calculator script. Without enabling the prefix delegation feature or non-nitro instance types, the calculation of the pods are 
**ENI * (# of IPv4 per ENI - 1) + 2. **

The calculation changed into **ENI * ((# of IPv4 per ENI - 1) * 16) + 2** where 16 is IPS_PER_PREFIX. The pod density per node is significantly more than before.

### Create Node Groups with EC2 Nitro Amazon Linux 2 Instance Type

Create one of the following node groups with at least one Amazon EC2 Nitro Amazon Linux 2 instance type. See the list of  [Nitro Amazon Linux 2 Instance Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html#ec2-nitro-instances).

```bash
eksctl create nodegroup \
  --cluster <my-cluster> \
  --region <us-east-1> \
  --name <my-nodegroup> \
  --node-type <m5.large> \
  --managed \
  --max-pods-per-node <110> -- Count from the max pods calculator
```

Once your nodes are deployed, you can run the `kubectl describe` command in the node to see the max pods in the allocatable.

## Conclusion

The advantages of using EKS are huge. When you move into complex and big architectures, EKS can end up consuming a lot of IPs. One of the most discussed EKS concerns is solved using the IP Address prefixes feature with VPC CNI version 1.9 or later. I'm looking forward to experimenting more on the EKS cluster to see how it behaves with cluster auto-scalers.