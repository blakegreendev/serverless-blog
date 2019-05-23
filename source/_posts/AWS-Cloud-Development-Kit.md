---
title: AWS Cloud Development Kit
tags:
  - aws
  - IaC
categories:
  - - AWS
date: 2019-01-18 15:52:39
---


In my previous posts about automation, it usually included CloudFormation or Terraform. These **Infrastructure as Code (IaC)** tools have provided a declartive model to provision and deploy cloud infrastructure. I've talked about how this approach to provisioning cloud infrastructure is much more efficient and reusable compared to manually creating things in the AWS console. Well, the **AWS CDK** is taking automation to the next level!

# AWS Cloud Development Kit
With the CDK, Infrastructure **IS** Code as opposed to YAML/JSON with CloudFormation or even HCL with Terraform. If you've ever created security groups in CloudFormation, you're probably aware of the numerous times you're required copy and paste amounting to several hundred lines of "code". Now, you can leverage the power of an actual programming language to loop and iterate on resources which shrinks the lines of code to maybe 10 or 15. The CDK has this concept of **"constructs"** which are essentially representations of AWS resources. You can use the **AWS Construct Library** for prebaked defaults written by AWS. 

# Let's give it a go!
First, let's cover the prerequisites:
- Install the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
- Create an IAM user and configure your AWS CLI
- Install [Node.js](https://nodejs.org/en/)
- IDE for TypeScript (I recommend [Visual Studio Code](https://code.visualstudio.com/))
- Install the AWS CDK Toolkit
{% codeblock %}
npm install -g aws-cdk
{% endcodeblock %}

Next, make an empty directory and change into it:
{% codeblock %}
mkdir cdk && cd cdk
{% endcodeblock %}


Initialize the CDK stack:
{% codeblock %}
cdk init app --language typescript
{% endcodeblock %}


Since Typescript needs to be compiled, open a new tab in your terminal and run in the cdk directory:
{% codeblock %}
npm run watch
{% endcodeblock %}


In this example, we'll create a VPC, so let's install the EC2 module:
{% codeblock %}
npm i @aws-cdk/aws-ec2
{% endcodeblock %}


Now, in the cdk directory, there is a folder called lib with a file called cdk-stack.ts. Open that file in your IDE or Visual Studio Code and copy/paste the following code:
- **Note:** After importing a module, VS Code has IntelliSense which is super useful for code completion, assistance, and linting.
{% codeblock lang:typescript %}
import cdk = require('@aws-cdk/cdk');
import ec2 = require('@aws-cdk/aws-ec2')

export class CdkStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
  
    const vpc = new ec2.VpcNetwork(this, 'VPC')
  }
}
{% endcodeblock %}

Now you can run:
{% codeblock %}
cdk synth
{% endcodeblock %}


And you'll see all of the resources that will be created. This is the raw CloudFormation that gets generated when you the CDK compiles the code.

My expression when I saw what roughly 10 lines of code can create with the CDK:
{% asset_img giphy.gif %}

Think about how quickly you would be able to reproduce this infrastructure rather than using the AWS console. Even better, think about how many lines of YAML or JSON that you didn't have to write. 

Finally, you can deploy to AWS:
{% codeblock %}
cdk deploy
{% endcodeblock %}


You can go to the AWS console and take a look at CloudFormation to see all the resources that were created. 

Clean up with:
{% codeblock %}
cdk destroy
{% endcodeblock %}


# More Info

Here the link to the [AWS CDK Documentation](https://awslabs.github.io/aws-cdk/index.html) and if you want to give it another spin, I recommend going through the [AWS CDK Workshop](https://cdkworkshop.com/). 

Here's a video from 2018 re:Invent and the presenters go through a demo using Lambda. This is awesome because if you've had to write custom resources in CloudFormation, you're probably aware of how painful that can be.
{% youtube Lh-kVC2r2AU %}

# Final Thought
While AWS CDK still new and evolving, I'm excited to see where this project goes. I have high hopes that someday this will be the preferred method to provision and deploy AWS cloud infrastructure. Anything to get away from JSON... *YUCK!* As of right now, CDK mainly supports Typescript which has been gaining a lot of popularity lately. I'm sure they'll add support for other languages eventually but this is an excuse to learn a new language. 