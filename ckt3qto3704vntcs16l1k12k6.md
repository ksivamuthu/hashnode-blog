## AWS Load Balancer Controller on EKS Cluster

Hello all, Let's see how to enable AWS Load Balancer Controller on EKS Cluster to integrate the AWS Application / Network Load Balancers. This is a guide to provision the use cases & solutions of the AWS Load Balancer integration with Kubernetes Cluster.

  - [Kubernetes Networking](#kubernetes-networking)
  - [Application Load Balancer](#application-load-balancer)
  - [AWS Load Balancer Controller on EKS Cluster](#aws-load-balancer-controller-on-eks-cluster-1)
    - [Creating an EKS Cluster](#creating-an-eks-cluster)
    - [Install the AWS Load Balancer Controller](#install-the-aws-load-balancer-controller)
    - [Deploy the Ingress](#deploy-the-ingress)
  - [Patterns](#patterns)
    - [Application Load Balancer - Traffic Routing](#application-load-balancer---traffic-routing)
    - [Ingress Group - Multiple Ingress Resources Together](#ingress-group---multiple-ingress-resources-together)
    - [Custom Domain & SSL Certificate](#custom-domain--ssl-certificate)
    - [HTTPS Redirect](#https-redirect)
    - [Access Control](#access-control)
    - [Custom Attributes & Addons](#custom-attributes--addons)
  - [Conclusion](#conclusion)

## Kubernetes Networking

We will start with a high-level overview of Kubernetes Networking. Kubernetes has all the components you need to deploy an application - like load balancer, ingress/egress gateways, network security policy, traffic routing within the cluster, mutual TLS authentication, etc. Kubernetes has the ability to layer these components and combine them to make a network that supports various scenarios in the organizations.  

[Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/#ingress-v1-networking-k8s-io) exposes HTTP and HTTPS routes from outside the cluster to [services](https://kubernetes.io/docs/concepts/services-networking/service/) within the cluster. Traffic routing is controlled by rules defined on the Ingress resource. An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS and offer name-based virtual hosting. An [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers) is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic. 

## Application Load Balancer

An Application Load Balancer functions at the application layer, the seventh layer of the Open Systems Interconnection (OSI) model. After the load balancer receives a request, it evaluates the listener rules in priority order to determine which rule to apply, and then selects a target from the target group for the rule action. You can configure listener rules to route requests to different target groups based on the content of the application traffic. Routing is performed independently for each target group, even when a target is registered with multiple target groups.

You can add and remove targets from your load balancer as your needs change, without disrupting the overall flow of requests to your application. Elastic Load Balancing scales your load balancer as traffic to your application changes over time. Elastic Load Balancing can scale to the vast majority of workloads automatically.

ALB supports multiple features including host or path-based routing, TLS (Transport Layer Security) termination, WebSockets, HTTP/2, AWS WAF (Web Application Firewall) integration, integrated access logs, and health checks.

## AWS Load Balancer Controller on EKS Cluster

The AWS Load Balancer Controllers manages AWS Elastic Load Balancers for a Kubernetes Cluster. When you install the AWS Load Balancer Controller, the controller dynamically provisions

- An AWS Application Load Balancer (ALB) when you create a Kubernetes Ingress
- An AWS Network Load Balancer (NLB) when you create a Kubernetes Service of type Load Balancer.
    - You can configure either of the target types - instance targets and IP targets.
  
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630635442585/1MNC0w-Fy.png)

Pic: [https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/](https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/)

1. When the Ingress resource is created in kubernetes API, the alb-ingress-controller observes the changes made.
2. The alb-ingress-controller creates the AWS Application Load Balancer based on the annotations added in the ingress resource.
3. The target groups are created for each backend specified in the ingress resource.
4. The Application Load Balancer URL is accessed with the path or query params.
5. Based on the Rules configured in the Ingress resource, the request is redirected to a specific target group and reaching Pod service using ClusterIP or NodePort

### Creating an EKS Cluster

Let's get started with an EKS cluster. You can deploy the EKS Cluster using AWS Console or the eksctl tool. In this blog post, we are going to create an EKS Cluster using the eksctl tool.

```bash
➜ eksctl create cluster -f cluster.yaml
```

The cluster yaml configuration is as below.

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: app-lb-demo
  region: us-east-1

iam:
  withOIDC: true   
  serviceAccounts:
    - metadata:
        name: aws-load-balancer
        namespace: kube-system
      wellKnownPolicies:
        awsLoadBalancerController: true  

managedNodeGroups:
  - name: app-lb-demo-ng
    instanceType: t3.medium
    minSize: 1
    maxSize: 2
```

### Install the AWS Load Balancer Controller

- Add the EKS chart repo to helm

    ```bash
    helm repo add eks https://aws.github.io/eks-charts
    ```

- Install the AWS Load Balancer Controller CRDs - Ingress Class Params and Target Group Bindings

    ```bash
    kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"
    ```

- Install the helm chart by passing the serviceAccount.create=false adn serviceAccount.name=aws-load-balancer-controller

    ```bash
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<cluster-name> --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer
    ```

### Deploy the Ingress

The AWS Application Load Balancer (ALB) will not be created until you create an ingress object. Now we will deploy the sample deployment file and expose using the Ingress object.

Note: This blog uses Kubernetes 1.21+ latest and the ingress objects are updated. If you are looking at the ALB controller documentation, you may see few differences over ingress objects.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.0.0/docs/examples/echoservice/echoserver-namespace.yaml &&\
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.0.0/docs/examples/echoservice/echoserver-service.yaml &&\
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.0.0/docs/examples/echoservice/echoserver-deployment.yaml
```

```bash
namespace/echoserver created
service/echoserver created
deployment.apps/echoserver created
```

## Patterns

### Application Load Balancer - Traffic Routing

In this ingress object, we are setting an internet-facing Application Load Balancer with added resource tags. In the rules, if the path has prefix echo, the backend service is echoserver and the port number is 80.

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echoserver
  namespace: echoserver
  annotations:
    **kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/tags: Environment=dev,Team=app**
spec:
  rules:
    - host: "*.amazonaws.com"
      http:
        paths:
          - path: /echo
            pathType: Prefix
            backend:
              service:
                name: echoserver
                port:
                  number: 80
```

Verify the ingress is created and ALB is provisioned. Once the ALB is provisioned, you will get the ALB address.

```bash
➜  flux-demo  k get ingress -A
NAMESPACE    NAME         CLASS    HOSTS             ADDRESS                                                                   PORTS   AGE
echoserver   echoserver   <none>   *.amazonaws.com   k8s-echoserv-echoserv-xxxxxxxxxxxxxxxxxxxxx.us-east-1.elb.amazonaws.com   80      10s
```

Make the request to the URL and the path to verify whether you get the expected response.

```bash
➜  flux-demo  curl http://k8s-echoserv-echoserv-xxxxxxxxxxxxxxxxxxxxx.us-east-1.elb.amazonaws.com/echo -i

HTTP/1.1 200 OK
Date: Thu, 02 Sep 2021 02:41:57 GMT
Content-Type: text/plain
Transfer-Encoding: chunked
Connection: keep-alive
Server: nginx/1.10.0
```

### Ingress Group - Multiple Ingress Resources Together

IngressGroup feature enables you to group multiple Ingress resources together. The controller will automatically merge Ingress rules for all Ingresses within IngressGroup and support them with a single ALB. In addition, most annotations defined on a Ingress only apply to the paths defined by that Ingress.

First, let's create another ingress resource and see whether it creates a separate application load balancer.

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echoserver-1
  namespace: echoserver
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/tags: Environment=dev,Team=app
spec:
  rules:
    - host: "*.amazonaws.com"
      http:
        paths:
          **- path: /echo-test**
            pathType: Prefix
            backend:
              service:
                name: echoserver
                port:
                  number: 80
```

Verify the ingress is created and you can see another ALB get provisioned. So now we got two ALBs with different addresses.

```bash
➜  flux-demo  k get ingress -A
NAMESPACE    NAME           CLASS    HOSTS             ADDRESS                                                                   PORTS   AGE
echoserver   echoserver     <none>   *.amazonaws.com   k8s-echoserv-echoserv-64b6592087-xxxxxxxxx.us-east-1.elb.amazonaws.com   80      28m
echoserver   echoserver-1   <none>   *.amazonaws.com   k8s-echoserv-echoserv-6c350fcfc9-xxxxxxxxx.us-east-1.elb.amazonaws.com    80      2m39s
```

Let's add the `[alb.ingress.kubernetes.io/group.name](http://alb.ingress.kubernetes.io/group.name)` for both ingress objects with the same value

```yaml
annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/tags: Environment=dev,Team=app
    alb.ingress.kubernetes.io/group.name: alb-demo-group
```

Once you re-apply the ingress, (delete and recreate - for a quick check), you can see both ingresses has the same ALB address. 

```bash
➜  flux-demo  k get ingress -A
NAMESPACE    NAME           CLASS    HOSTS             ADDRESS                                                             PORTS   AGE
echoserver   echoserver     <none>   *.amazonaws.com   k8s-albdemogroup-5364dc26a8-xxxxxxxxx.us-east-1.elb.amazonaws.com   80      5s
echoserver   echoserver-1   <none>   *.amazonaws.com   k8s-albdemogroup-5364dc26a8-xxxxxxxxx.us-east-1.elb.amazonaws.com   80      5s
```

The rules will be merged and you can apply the order of the rules in the group to configure in the Application Load Balancer Controller.

### Custom Domain & SSL Certificate

Now, let's bind the custom domain to the ALB and SSL Certificates. There are two ways you can bind the SSL certificate

1. Setting up the certificate from the annotation - [alb.ingress.kubernetes.io/certificate-arn](http://alb.ingress.kubernetes.io/certificate-arn) specifies the ARN of one or more certificates managed by AWS Certificate Manager

    ```yaml
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-west-2:xxxxx:certificate/xxxxxxx
    ```

2. You can set up the auto-discovery of the SSL certificate using the domain name.

    TLS certificates for ALB Listeners can be automatically discovered with hostnames from Ingress resources if the `[alb.ingress.kubernetes.io/certificate-arn](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/ingress/annotations/#certificate-arn)` annotation is not specified.

    The controller will attempt to discover TLS certificates from the `tls` field in Ingress and `host` field in Ingress rules.

In this example, we will set up the auto-discovery of the SSL certificate.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echoserver
  namespace: echoserver
  annotations:
    **kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/tags: Environment=dev,Team=app**
spec:
  rules:
    - host: "dev.sivamuthukumar.com"
      http:
        paths:
          - path: /echo
            pathType: Prefix
            backend:
              service:
                name: echoserver
                port:
                  number: 80
```

Execute the curl command. 

```yaml
➜  flux-demo  curl https://dev.spectaflare.com/echo -i
HTTP/1.1 200 OK
```

### HTTPS Redirect

The URL with HTTP protocol should redirect to HTTPS protocol. We'll use the [alb.ingress.kubernetes.io/actions.${action-name}](http://alb.ingress.kubernetes.io/actions.$%7Baction-name%7D) annotation to setup an ingress to redirect http traffic into https

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echoserver
  namespace: echoserver
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/tags: Environment=dev,Team=app
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
spec:
  rules:
    - host: "dev.sivamuthukumar.com"
      http:
        paths:
          - path: /echo
            pathType: Prefix
            backend:
              service:
                name: echoserver
                port:
                  number: 80
```

### Access Control

Access control of the Load Balancer can be controlled with the following annotations.

```yaml
alb.ingress.kubernetes.io/scheme: internal # To enable the internal load balancers
alb.ingress.kubernetes.io/inbound-cidrs: 10.0.0.0/24 # Inbound CIDRs from your network or VPC
alb.ingress.kubernetes.io/security-groups: sg-xxxx, sg-yyyy # Security groups you want to attach the load balancer
```

### Custom Attributes & Addons

Custom attributes to Load Balancers can be controlled with the annotations - [`alb.ingress.kubernetes.io/load-balancer-attributes`](http://alb.ingress.kubernetes.io/load-balancer-attributes:)

For e.g To enable HTTP2 in the Application Load Balancer

```yaml
alb.ingress.kubernetes.io/load-balancer-attributes: routing.http2.enabled=true
```

Check the other attributes you can add it in annotations [here](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/ingress/annotations/#custom-attributes).

You can attach WAF or enable AWS Shield to the load balancer by adding the below annotations in the Ingress object.

```yaml
alb.ingress.kubernetes.io/wafv2-acl-arn: <WAFv2 ACL ARN HERE>
alb.ingress.kubernetes.io/shield-advanced-protection: 'true'
```

## Conclusion

The AWS Load Balancer Controller provides a Kubernetes native way to configure and manage Elastic Load Balancers that route traffic to applications running in Kubernetes clusters. I hope, I've explained to you the concept of routing, ingress groups, configure SSL, HTTP to HTTPs redirection, and other advanced concepts. You can check more on the documentation [here](https://github.com/kubernetes-sigs/aws-load-balancer-controller). I highly encourage you to give it a try and share your feedback and questions with me

I'm Siva - working as Sr. Software Architect at [Computer Enterprises Inc](https://www.ceiamerica.com) from Orlando. I'm an AWS Community builder, Auth0 Ambassador and I am going to write a lot about Cloud, Containers, IoT, and Devops. If you are interested in any of that, make sure to follow me if you haven’t already. Please follow me [@ksivamuthu](https://www.twitter.com/ksivamuthu) Twitter or check out my blogs at https://blogs.sivamuthukumar.com