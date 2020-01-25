---
title: Build and Deploy a Serverless app with AWS CDK and Python
tags:
  - serverless
  - aws cdk
  - python
  - fargate
  - vue
categories:
  - AWS
date: 2019-12-30 15:01:16
---

There's quite a few AWS CDK examples that use Typescript but I haven't come across a lot for Python. In fact, while at re:Invent 2019, I asked one of the AWS CDK team members why they initially chose Typescript over other languages. The response was that it was the preferred language of the person who started the project and no other particular reasons. I thought that was quite interesting. Anyway, as an aspiring Pythonista and Python fanboy, I wanted to dig into a useful example of how to utilize my preferred language to build and deploy a serverless app with AWS CDK and Python.

# Prerequisites 
- Node v12 
- AWS CLI
- Python 3.7
  
# The App
We're going to build a HackerNews clone build upon HackerNews official Firebase API, Vue 2.0 + Vue Router + Vuex, with server-side rendering. 

Here's what the app looks like:
{% asset_img hn-pic.png %} 

Here's what the application architecture looks like:
{% asset_img hn-arch.png %} 

Let's start with bringing down the Vue example:
```
git clone https://github.com/vuejs/vue-hackernews-2.0
```

We can start it locally by installing dependencies and then running the following in dev mode:
```
npm install
npm run dev
```

Now that the application looks good locally, let's containerize it by adding a Dockerfile to the root folder. 

{% codeblock lang:yaml %}
FROM node:10

WORKDIR /usr/src

COPY package*.json ./

RUN npm install

COPY . . 

EXPOSE 8080

RUN npm run build

CMD [ "npm", "start" ]
{% endcodeblock %}

I renamed the directory to *webapp* because we'll need to move it into the CDK app later.

It should looks something like this:
{% asset_img app-struct.png %} 

# The Infrastructure

Now that we've Dockerized the application locally, let's deploy to AWS! The architecture is pretty simple and consists of only a few main components. 

## Fargate
{% blockquote %}
AWS Fargate is a compute engine for Amazon ECS that allows you to run containers without having to manage servers or clusters. With AWS Fargate, you no longer have to provision, configure, and scale clusters of virtual machines to run containers. This removes the need to choose server types, decide when to scale your clusters, or optimize cluster packing. AWS Fargate removes the need for you to interact with or think about servers or clusters. Fargate lets you focus on designing and building your applications instead of managing the infrastructure that runs them.
{% endblockquote %}


Install the latest version of the CDK CLI:
```
npm i -g aws-cdk
```

In an empty and separate folder, let's intialize the CDK project:
```
mkdir cdk_hack && cd cdk_hack
cdk init --language python
```

This creates the skeleton of the project for the CDK application which should looks something like this:
{% asset_img cdk-struct.png %}

First, let's go into **setup.py** and add the dependencies we'll need to deploy our infrastructure. It should look like this:

{% codeblock lang:python %}
    install_requires=[
        "aws-cdk.core",
        "aws-cdk.aws-ecs-patterns",
        "aws-cdk.aws-ecr-assets",
        "aws-cdk.aws-certificatemanager",
        "aws-cdk.aws-route53",
        "aws-cdk.aws-ec2",
    ],
{% endcodeblock %}

Next, activate your virtualenv:
```
source .env/bin/activate
```

Like the instructions say, once the virtualenv is activated, you can install the required dependencies.
```
pip install -r requirements.txt
```

Last step before we can start coding the infrastructure is moving the webapp directory where our Dockerfile is located to the CDK directory. 
```
mv webapp cdk_hack/cdk_hack/
```

It should look something like this:
{% asset_img cdk2-struct.png %}

```
export USER_AWS_ACCOUNT='111111111'
export USER_AWS_REGION='us-west-2'
```

## Test the Infrastructure
The pytest framework makes it easy to write small yet functional testing for applications and libraries. Since we're using Python to model our infrastructure, we can use pytest to validate our resources. 

For example, let's say we 

{% codeblock lang:python %}
import json
import pytest

from aws_cdk import core
from cdk_hack.cdk_hack_stack import CdkHackStack

def get_template():
    app = core.App()
    CdkHackStack(app, "cdk-hack", env=core.Environment(account="987092829714", region="us-west-2"))
    return json.dumps(app.synth().get_stack("cdk-hack").template)

def test_ecs_service():
    assert("Memory" and "512" in get_template())
{% endcodeblock %}

# Clean Up

```
cdk destroy
```