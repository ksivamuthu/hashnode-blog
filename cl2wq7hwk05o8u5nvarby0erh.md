## EKS Blueprints - Modular Constructs for your Kubernetes cluster

Amazon Elastic Kubernetes Service (EKS) helps you manage, scale, and deploy your Kubernetes clusters. In the enterprise cluster setup, you need more than the cluster setup. To bootstrap, the kubernetes cluster requires significant features, including automated node management, ingress controller, monitoring, logging, security, networking tools, etc. 

With my experience with other cloud provider platform Kubernetes offerings such as GKE & AKS, EKS was behind in the management features - logging, ingress controllers, etc. We need to set it up manually to get the basic features. That was hard to get into the Kubernetes platform - especially if it’s the first time onboarding to Kubernetes experience and/or EKS cluster. Later the Kubernetes components from AWS speed up the operation’s onboarding process easier. 

AWS EKS Blueprints - AWS is making the operations and developer experience greater by providing modular Infrastructure as Code modules to expedite the process of onboarding the cluster and open source tools and security controls with production readiness.

## What are AWS EKS Blueprints?

[EKS Blueprints](https://github.com/aws-quickstart/cdk-eks-blueprints/) is a collection of Infrastructure as Code (IaC) modules that will help you configure and deploy EKS clusters across accounts and regions. You can use EKS Blueprints to easily bootstrap an EKS cluster with Amazon EKS add-ons as well as a wide range of popular open-source add-ons, including Prometheus, Karpenter, Nginx, Traefik, AWS Load Balancer Controller, Fluent Bit, Keda, Argo CD, and more. EKS Blueprints also helps you implement relevant security controls needed to operate workloads from multiple teams in the same cluster.

Highlights of the features:

- Deploy Well-Architected EKS clusters across accounts and regions
- Manage cluster configuration, including add-ons from a single GIt repo.
- Teams and Access management
- Continuous Delivery pipelines for deploying infrastructure
- GitOps - based workflows for workloads

![aws cdk blueprint](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/awbvfw4ox8l8812jfo8o.png)
 
Courtesy: Image from AWS Blog.

### Imperative / Declarative

When it comes to Infrastructure as Code, there are two options - Declarative (Terraform, yamls, etc.) and Imperative (CDK, etc.). AWS EKS Blueprints are available in both declarative and imperative options.

**AWS EKS Blueprints for Terraform**

In AWS EKS Blueprints for Terraform - the tools are available as terraform modules to implement. You can bootstrap the kubernetes cluster and use the adds-on modules to install the open-source tools. 

%[https://github.com/aws-ia/terraform-aws-eks-blueprints]

**AWS EKS Blueprints for CDK**

In AWS EKS Blueprints for CDK - the tools are available as CDK constructs to implement. You can bootstrap the kubernetes cluster and use the adds-on modules to install the open-source tools. 

%[https://github.com/aws-quickstart/cdk-eks-blueprints]

## Demo Walkthrough

In today’s blog, we are going to set up the EKS Blueprint CDK and set up the necessary add-ons and the team structure.

- Install the AWS-CDK
    
    ```bash
    npm install -g aws-cdk
    ```
    
- Initialize the CDK application
    
    ```bash
    mkdir eks-blueprint
    cdk init app --language typescript
    ```
    
- Create the EKS Blueprint Construct and call it from bin/<your_main_file>.ts
    
    ```bash
    ## ./bin/eks-blueprint.ts
    #!/usr/bin/env node
    import 'source-map-support/register';
    import * as cdk from 'aws-cdk-lib';
    import { EKSBlueprintConstruct } from '../lib/eks-blueprint-stack';
    
    const app = new cdk.App();
    new EKSBlueprintConstruct().build(app, 'siva-dev');
    ```
    
- The EKS Blueprint Construct has three parts.
    - Create a generic EKS cluster with a node group configuration
    - Add the addons
    - Add the teams - dev team and platform team.
    
    ```tsx
    
    export class EKSBlueprintConstruct {
    
      build(scope: Construct, id: string, props?: StackProps) {
        const devTeam = new ApplicationTeam({ name: 'dev', users: this.getUserArns(scope, 'devUsers') });
        const platformTeam = new PlatformTeam({ name: 'platform', users: this.getUserArns(scope, 'platformUsers') });
    
        const teams = [devTeam, platformTeam];
    
        const addOns = [
          new VpcCniAddOn(),
          new CoreDnsAddOn(),
          new KubeProxyAddOn(),
          new MetricsServerAddOn(),
          new AwsLoadBalancerControllerAddOn(),
          new ContainerInsightsAddOn(),
          new KarpenterAddOn()
        ];
    
        const clusterProvider = new GenericClusterProvider({
          version: KubernetesVersion.V1_21,
          managedNodeGroups: [{
            id: `${id}-nodegroup`,
            instanceTypes: [new InstanceType('t3.small')],
            minSize: 1,
            maxSize: 2
          }],
        })
    
        EksBlueprint.builder()
          .addOns(...addOns)
          .clusterProvider(clusterProvider)
          .teams(...teams)
          .region('us-east-1')
          .build(scope, id, props);
      }
    	...
    }
    ```
    
- You can add the context properties that have `devUsers` and `platformUsers` that has a list of user/role ARN to create the team.
    
    ```bash
    "context": {
        "devUsers": [
          "arn:aws:iam::xxxxxxxxxxx:user/user_name_1",
          "arn:aws:iam::xxxxxxxxxxx:user/user_name_2"
        ],
        "platformUsers": [
    		   "arn:aws:iam::xxxxxxxxxxx:user/admin_name"
    		]
      }
    ```
    
    ```tsx
      private getUserArns(scope: Construct, key: string): ArnPrincipal[] {
        const context: string[] = scope.node.tryGetContext(key);
        if (context && context.length > 0) {
          return context.map(e => new ArnPrincipal(e));
        }
        return [];
      }
    ```
    

You will see the CDK output has kubernetes config commands, the role for the dev and platform team.

```tsx
siva-dev: creating CloudFormation changeset...

 ✅  siva-dev

✨  Deployment time: 1437.28s

Outputs:
siva-dev.devsa = dev-sa
siva-dev.devteamrole = arn:aws:iam::xxxxxxxxxx:role/dev-role-name
siva-dev.platformteamadmin = none
siva-dev.sivadevClusterName9C1D1E82 = siva-dev
siva-dev.sivadevConfigCommandF9D1C39C = aws eks update-kubeconfig --name siva-dev --region us-east-1 --role-arn <role_name>
siva-dev.sivadevGetTokenCommandA8918546 = aws eks get-token --cluster-name siva-dev --region us-east-1 --role-arn <role_name>

```

### Caveats

The latest kubernetes version is not supported yet. Having issues with k8s version 1.22. You may see issues. Please use v1.21 at this time. [https://github.com/aws-quickstart/cdk-eks-blueprints/issues/350](https://github.com/aws-quickstart/cdk-eks-blueprints/issues/350)

Also, you’ve to set the region - for the AWS Load balancer controller addon to work as the container images are derived from the region variable. 

Helm release - testing - It will be a great addition if the CDK is able to fail if the addons fail to install. For e.g., without setting the region, the AWS Load balancer controller was in a pending state.

## Conclusion

AWS EKS Blueprints is definitely a big step in making the dev and operation team's life easier. AWS is also working with Industrial partners and open source tools such as Datadog, Kasten.io, ArgoCD, Hashicorp, Snyk, etc. 

In the next post of this series,  we will see how to set up the CI/CD of the infrastructure and application workloads using EKS blueprints. And also how to extend the EKS blueprint to bring your own set of tools for your enterprise.

I'm Siva - working as Sr. Software Architect at Computer Enterprises Inc from Orlando. I'm an AWS Community builder and Auth0 Ambassador. I am going to write a lot about Cloud, Containers, IoT, and Devops. If you are interested in any of that, make sure to follow me if you haven’t already. Please follow me **[@ksivamuthu](https://hashnode.com/@ksivamuthu)** on Twitter or check out my blogs at **[blog.sivamuthukumar.com](https://blog.sivamuthukumar.com/)**!