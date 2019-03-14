---
title: Deploy VSCode on AWS Fargate with Pulumi
tags:
  - aws
categories:
  - - AWS
  - - Serverless
  - - Containers
  - - CI/CD
date: 2019-03-13 15:24:22
---


In this tutorial, we'll setup Visual Studio Code to run locally in a Docker container and then deploy to AWS Fargate with Pulumi. Essentially, you'll be able to run VSCode in your browser from anywhere! Let's get started!

# Prerequisites
1. Make sure [Docker](https://docs.docker.com/install/) is installed and running
2. Install [Pulumi](https://pulumi.io/quickstart/install.html)
3. Configure [AWS](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) credentials
4. Install [Node](https://nodejs.org/en/download/)

# Run VSCode in Docker
First, let's run it locally and test that it works in Docker. I recommend creating a folder for the project:
```
mkdir vscode
```

Go ahead and clone the following repo in the new project folder which contains the Dockerfile to get up and running:

```
cd vscode
git clone https://github.com/bgreengo/code-server.git
```

Next, change direcotry into **code-server** folder and run:
```
cd code-server
docker build -t vscode-docker .
```

It will build the Docker image and then you're ready to run the container:
```
docker run -v /tmp:/projects/vscode -p 9000:9000 -t vscode-docker
```

You should now see a similar output which includes the localhost URL and password to access the application:
{% asset_img screen2.png %}

Go to https://localhost:9000 and click through the exception until you reach the login page:
{% asset_img screen1.png %}
{% asset_img screen3.png %}

Login with the password provided in the output and you will access VSCode running in a Docker container.
{% asset_img screen4.png %}

Pretty cool, right? But we want to access VSCode from anywhere! If only there was a global platform and tool to make it easy to deploy and host our VSCode application. Lucky for us, we have AWS and Pulumi to streamline this!

# Run VSCode in AWS Fargate
First, let's get a quick overview of what these tools and services are before we go global.

## What is Pulumi?
In my opinion, Pulumi is the next evolution of Infrastructure as Code (IaC). Rather than writing IaC using traditional declaritive programming languages like YAML or JSON, Pulumi provides infrastructure as *real* code. It uses imperative programming models to abstract a lot of the pain that comes with defining each and every resource with services like AWS CloudFormation. Often times with Pulumi, you will end up writing a fraction in lines of code compared to CloudFormation or Terraform. Pulumi is also "cloud agnostic" which means you can use the tool on most cloud providers out there. You'll find that Pulumi can be a powerful tool, especially if you're already familiar with one of the supported programming languages. 

## What is AWS Fargate?
Straight from the horses mouth, AWS describes Fargate as:
{% blockquote %}
AWS Fargate is a compute engine for Amazon ECS that allows you to run containers without having to manage servers or clusters. With AWS Fargate, you no longer have to provision, configure, and scale clusters of virtual machines to run containers. This removes the need to choose server types, decide when to scale your clusters, or optimize cluster packing. AWS Fargate removes the need for you to interact with or think about servers or clusters. Fargate lets you focus on designing and building your applications instead of managing the infrastructure that runs them.
{% endblockquote %}

This is perfect because who likes maintaining infrastructure nowadays? *cough* **#SERVERLESS** *cough*

## Setup 
Go ahead and change directory back out to the *vscode* folder that was created earlier. Run the following command:
```
pulumi new typescript --dir vscode-fargate
```

It's going to ask for a few details (just hit Enter):

**project name:** (vscode-fargate) 
**project description:** (A minimal TypeScript Pulumi program) 
**stack name:** (dev)
**Do you want to perform this update?** yes

Finally, it should create the stack and display a permalink (i.e. Permalink: https://app.pulumi.com/bgreengo/vscode-fargate/dev/updates/1) that will take you to the Pulumi app dashboard. 
{% asset_img screen5.png %}

You'll see some awesome details here including changes, timeline, and resources created. It's very useful.

Back to your code editor, open up the *index.ts* file in the vscode-fargate folder and paste in the following code:
{% codeblock lang:typescript %}
import * as pulumi from "@pulumi/pulumi";
import * as awsx from "@pulumi/awsx";

let listener = new awsx.elasticloadbalancingv2.NetworkListener("vscode", { port: 9000 });

let service = new awsx.ecs.FargateService("vscode", {
    desiredCount: 1,
    taskDefinitionArgs: {
        containers: {
            vscode: {
                image: awsx.ecs.Image.fromPath("vscode", "./app"),
                memory: 4096,
                cpu: 2048,
                portMappings: [listener],
            },
        },
    },
});

export const hostname = pulumi.interpolate `https://${listener.endpoint.hostname}:9000`;
{% endcodeblock %}

Next, in the vscode-fargate folder, create a folder called *app*

Copy the contents of the *code-server* folder (where we ran VSCode in Docker earlier) into the *app* folder within the *vscode-fargate* folder. The file directory for the project should look something like this (minus node_modules):
{% asset_img screen6.png %}

Finally, we need to import the awsx module:
```
npm install @pulumi/awsx
```
And, set the AWS region (I'm using Oregon):
```
pulumi config set aws:region us-west-2
```

## Deploy
That's pretty much it! You are ready to go global with VSCode running on a serverless container in AWS Fargate.

Run:
```
pulumi up
```

You will see all the awesomeness that Pulumi will deploy for you. All from the 20 lines of Typescript above.

Go ahead and accept the changes and deploy to AWS!
{% asset_img screen7.png %}

After the stack has deployed, you should be able to take note of the URL and run:
```
pulumi logs
```

This should output the password to access VSCode running on Fargate. 
{% asset_img screen8.png %}

Head to the URL and access VSCode!
{% asset_img screen9.png %}

## Next Steps
You might want to take it a step further and add an SSL certificate to the Load Balancer. Then you can use Route 53 and route traffic with your domain name. 

# Final Thought
I hope you were able to see how easy and powerful it is when you mix AWS and Pulumi. It's becoming more efficient to create IaC as the applications start to blend with the infrastructure. I encourage you to read through the Pulumi docs as they have other wonderful tutorials and hopefully they will continue to add more. I'm looking forward to the future of this product and how it continues to establish itself in this space.



