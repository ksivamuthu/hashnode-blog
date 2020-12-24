## Tech Talk: Building Event-Driven Apps with Dapr in Kubernetes

## Building Event-Driven Apps with Dapr in Kubernetes

In this article, I will summarize the tech talk I've done in various meetups on *Building Event-Driven Apps with Dapr in Kubernetes*

### Talk Abstract 
In this talk, we will go through an overview of Dapr, how it enables distributed application development. Dapr brings proven patterns and practices to you. It unifies event-driven actors' semantics into a simple, consistent programming model, and Dapr supports all programming languages without framework lock-in. This talk goes more in-depth into these building blocks of distributed microservices such as state stores, messaging, resource bindings, etc., with demos and workflow of the deployment of services and runtime.

### Details

This talk covers Dapr in the below points.

### 1. What is Dapr?

Dapr (Distributed Application Runtime) is a portable, event-driven runtime that makes it easy for developers to build resilient, microservice stateless, and stateful applications that run on the cloud. 

### 2. What problem does Dapr solve?

Microservices have different capabilities when it comes to a distributed system. Each capability is abstracted through a Dapr Building Blocks that defines a simple HTTP / gRPC API decoupled from the technology chosen to implement it through the use of pluggable components. 

This best practice of decoupling frees developers from the burden of integrating and having a deep understanding of the underlying technologies they use. The use of building blocks also allows swapping implementations when the application is deployed in different hosting environments or when needs change as the application evolves without affecting your code or without any vendor lock-in.


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608779062829/sxaT5gXCC.png)

### 3. How does it run? Sidecar?

Dapr can run as a sidecar as a process to standalone apps and as sidecar containers to the apps in kubernetes (using sidecar architecture).

**Standalone app**
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608779345065/KHR8f2J3o.png)

**Kubernetes app**

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608779376867/OC_cwC7a_.png)

### 4. Dapr in Kubernetes

Dapr can be configured to run on any Kubernetes cluster. In Kubernetes, the `dapr-sidecar-injector` and `dapr-operator` services provide first-class integration to launch Dapr as a sidecar container in your deployments.

The dapr-sidecar-injector also injects the environment variables DAPR_HTTP_PORT and DAPR_GRPC_PORT into all the containers in the pod to enable user-defined applications .to communicate with Dapr without hardcoding Dapr port values easily.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608779543555/yoTc0vAnc.png)

You've to add the below annotations in your deployment YAML to enable the dapr sidecar injection in your pod.

```
  annotations:
    dapr.io/enabled: "true"
    dapr.io/app-id: "nodeapp"
    dapr.io/app-port: "3000"
```

### 5. Configuring Components.

Dapr components can be configured using YAML. Each building block has the type and the metadata to configure the implementation.

You can add more components and switch the component's type at the higher / other environments.

E.g., The below-configured message bus component configured with Redis type can be changed later to Kafka without changing any application code.

**Redis**

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: messagebus
spec:
  type: pubsub.redis
  metadata:
  - name: redisHost
    value: localhost:6379
  - name: redisPassword
    value: ""
```
**Kafka**

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: messagebus
spec:
  type: pubsub.kafka
  version: v1
  metadata:
    - name: brokers
      value: "kafka.default.svc.cluster.local:9092"
    - name: authRequired
      value: "false"    
```

### 6. Food Delivery App - Dapr Demo

The demo runs three microservices in the kubernetes cluster. The demo explains how the order service, kitchen service, delivery service works together to process the food delivery system powered by Dapr.

1. Dapr Runtime - Sidecar Injection Annotations
2. State Store Demo - Redis
3. Pub-Sub Demo - Redis & Kafka / NATS
4. Bindings Demo - Twillio SMS
5. Github Actions - Deploying Dapr Components and Microservices in AKS (Azure Kubernetes Service)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608780373269/IlA_vEBED.png)

### 7. Open Source Contributions

I'm one of the contributors and members of the Dapr team in Github. This talk also explains how to contribute to Dapr and the different repositories you can look for to make contributions. If you are new to Golang, that's fine. Everyone is more welcome to contribute. Look at the issues you can start contributing. Please reach me or anyone on the Dapr team for any help. 

### Source Code

The source code of the demo is here. Slides are in the repo.

%[https://github.com/ksivamuthu/cloud-dapr-demo]

### Video Recordings

**Atlanta .NET User Group**

%[https://www.youtube.com/watch?v=SBumJ9KT8Pk]

**St.Pete .NET User Group**

%[https://www.youtube.com/watch?v=5zuNhx1ye0s]

**Orlando .NET User Group**

%[https://www.youtube.com/watch?v=jSFAdn1UmPI]

The same talk is presented in various meetup groups - Columbus App Dev User Group, New Hampshire .NET User Group, and internal presentations. This talk is also scheduled for 2021 to talk in meetups and user groups.

Do you want me to talk at your meetups/conferences?  Reach me out at my Twitter DM [@ksivamuthu](https://twitter.com/ksivamuthu) or [Linkedin](https://linkedin.com/in/ksivamuthu). Happy to have a conversation with you all.

> Hello, I'm Siva. I'm speaking in meetups and tech conferences on technical topics - Kubernetes, Mobile - Flutter / Xamarin, and others. I decided to create the [Tech Talk Blog series](https://hashnode.com/series/tech-talks-and-demos-ckj29avqn03p025s1gv6o6932) in my [blog](https://blog.sivamuthukumar.com) summarizing the talk I've done with the abstract, the notes, or layout on how I structured the talk, the demo, and the video recordings if any. It would be beneficial for me to document. Also, I hope it's helpful for you to learn, review, and provide feedback on the technologies you love.



