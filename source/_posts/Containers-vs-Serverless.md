---
title: Containers vs Serverless
tags:
  - aws
  - serverless
  - containers
categories:
  - AWS
date: 2018-10-06 18:45:24
---


{% asset_img cvs.jpg %}

Depending on who you talk to nowadays, they are probably in one tribe or the other. But which tribe will last? This has been a heated debate among thought leaders while the future of technology continues to evolve. 

As someone who is just beginning with both, I asked myself, *"Which one should I be learning?"* and even, *"Which tribe should I choose?"*

I decided to learn each technology along with special use cases and find out the enigma that surrounds this conversation.

# Containers

With my prior experience as a SysAdmin, containers made more sense to me for obvious reasons. There is still infrastructure to manage, however, containers run at a higher level of abstraction. For instance, the same way a VM is an abstraction of the physical hardware turning one server into many servers, a container is an abstraction of the operating system. This makes containers more lightweight, portable, and efficient.

{% asset_img contvm.png %}
*Retrieved from: https://www.docker.com*

## Docker

Docker is pretty much synonymous with containers. Usually when you are talking about one, you are talking about the other. Docker is by far the most popular platform or ecosystem around creating and running containers. Docker is intended to remove the old saying, "It works on my computer but doesn't work on yours." It is supposed to create better parity when moving applications from one environment to another.

I downloaded [Docker](https://www.docker.com/get-started) on my computer and gave it a spin. I quickly found out that I needed an "image" to create a container. An image is a single file with all the dependencies and configurations required to run a program. The container is just an instance of the image and runs the program. I turned to the [Docker Hub](https://hub.docker.com/) and signed up. I explored the official repositories for Docker images and settled on nginx. The details guided me on pulling the image from Docker Hub, adding content to the container with a Dockerfile and before I knew it, I had nginx running locally. 

I was pleasently surprised with the simplicity and I could see the benefits right away. I soon realized this simplicity only applied to small workloads. As the complexity increased, I needed something called container orchestration. Enter Kubernetes.

## Kubernetes 

Kubernetes is an open-source project originally created by Google. Kubernetes manages containerized workloads and services. I enabled Kubernetes on my local Docker install which includes a standalone Kubernetes server and client. 

## Kubernetes with Helm

Helm is a package manger for Kubernetes and is the best way to find, share and use software built for Kubernetes. I installed Helm with Homebrew on my Mac and did a 'helm init'. After that I was able to search through the charts at all the applications I could deploy on my local Kubernetes cluster. It was so simple and amazing. Really a game changer if you're just trying to get an application up and running quickly in Kubernetes.

## AWS Options

I remember my first AWS re:Invent in 2016 and all the talk was about containers. There were so many sessions and everything seemed so focused on running containers. Since then, AWS has continued to push hard on this area including ECS, Fargate, and most recently EKS.

### ECS

ECS is basically AWS version of Docker-as-a-Service. You can integrate with Amazon's Elastic Container Registry (ECR) which is much like Docker Hub to store Docker images. ECS helps you run microservice applications as well.

### Fargate

AWS Fargate is a compute engine for Amazon ECS that allows you to run containers without having to manage servers or clusters. With AWS Fargate, you no longer have to provision, configure, and scale clusters of virtual machines to run containers. This removes the need to choose server types, decide when to scale your clusters, or optimize cluster packing. AWS Fargate removes the need for you to interact with or think about servers or clusters. Fargate lets you focus on designing and building your applications instead of managing the infrastructure that runs them.

### EKS

Amazon EKS runs the Kubernetes management infrastructure for you across multiple AWS availability zones to eliminate a single point of failure. Amazon EKS is certified Kubernetes conformant so you can use existing tooling and plugins from partners and the Kubernetes community. Applications running on any standard Kubernetes environment are fully compatible and can be easily migrated to Amazon EKS.

# Serverless

There doesn't seem to be a clear-cut definition what exactly serverless means right now but I think the most obvious benefit of serverless allows you to run code without provisioning or managing servers. Even though there's still servers running the code, that responsibility now lies on the cloud provider. 

{% blockquote %}
Yes, there are servers in serverless!
{% endblockquote %}

The pay-as-you-go consumption model from cloud providers make serverless very appealing because you're no longer paying for idle time. In fact, when a Lambda function is invoked, you are only charged for the duration of the processed function which could be milliseconds. There are some performance considerations such as cold starts in serverless. A cold start is how long it takes for the function to process which could add latency to the application.

A common use case and what modern web applications will eventually look like in a serverless world is the combination of API Gateway, Lambda and DynamoDB. *NOTE:* There are no servers to manage in this use case.
{% asset_img serverless.png %}
*Retrieved from: https://aws.amazon.com/lambda/*

## AWS Lambda

Some would argue that Lambda is required for an architecture to be serverless. I made this [blog](https://greengocloud.com/2018/08/28/How-to-Make-a-Fast-and-Cheap-Serverless-Blog/) without using Lambda and I would consider this serverless. My point is Lambda doesn't encompass all of serverless. In fact, Lambda is considered Functions-as-a-Service or FaaS. A function is programming logic that is written to perform a specific task. When triggered, an event will perform the logic when it is required.

{% blockquote Kelsey Hightower %}
My journey to FaaS has been less about learning something new and more about unlearning something old.
{% endblockquote %}

## Serverless Framework

The [Serverless Framework](https://serverless.com/) is the easiest way to get started with serverless architectures. By installing with npm, I was able to boilerplate an entire architecture within minutes. It's extremely simple and there's a lot of great tutorials out there.

# Final Thought

There's no doubt why there's a debate. Both are very different and can be useful in their own way in certain use cases. I think *containers* will win the **battle**, but *serverless* will win the **war**. It appears that serverless can provide a competitive edge and isn't that what we are all working toward? I've longed for the days of working smarter, not harder. If that means trading in my experience with infrastructure and learning how to code some Lambda functions, 

{% blockquote Adrian Cockcroft, @ServerlessConf %}
Build serverless first. If needed move to containers.
{% endblockquote %}