---
title: How to Make a Cheap, Fast, and Low Maintenance Serverless Blog on AWS
tags:
  - aws
categories:
  - - AWS
  - - Serverless
  - - CI/CD
date: 2018-08-28 12:09:18
---


# Intro
I started a *tech blog* because I read somewhere that it would help me become a better writer and someday I might end up Googling something to find out that I've already written a blog about it. The jury is still out.

I began with the following criteria; the blog must be **cheap, fast, and low maintenance.** Security and availability were also on my mind but ultimately, my goal was to be able to spill ideas onto a post and push content efficiently in an automated fashion. With my experience as a SysAdmin, I knew that self-hosting my blog on a server with a CMS like Wordpress or Drupal didn't meet the criteria. I was concerned about the time and effort to run and maintain a server and other hosting options were a bit too expensive.

I turned my attention to **serverless**, which has become a craze in the IT world. While serverless means different things to different people, I found that it met my criteria. This particular serverless architecture pattern is dirt cheap (less than $2/month), uses a global content delivery network with Amazon CloudFront, and there's no servers to maintain. **Bingo!**

# Overview
Now that I've explained **why** I selected a serverless architecture, I'll explain the **details** of this design pattern.

## Design
I used a static site generator called **Hexo**, which allows me to do local development to the blog among other features. After the site looks acceptable, I push the site to a **Github** repository that has a webhook to **AWS CodeBuild**. When Codebuild detects changes to the repository, it compiles and generates the site and sends a **Slack** notification with the status of the build. If the build succeeds, the site files are synced to a webstie enable **AWS S3 bucket**. The S3 bucket is an origin for an **AWS CloudFront** distribution that redirects HTTP to HTTPS and has a free SSL/TLS certificate from **AWS Certificate Manager**. Thus making a secure site to clients connecting to the blog.

## Diagram
{% asset_img serverless-blog.png %} 

# Hexo
[Hexo](https://hexo.io/) is a fast, simple and powerful blog framework. It uses **Markdown** for the blog posts which I was familiar with from writing Github readme's. To install the **Hexo CLI**, it requires **npm** and **git** as a prerequisite. Once that's complete, type:
```
npm install -g hexo-cli
hexo init blog
cd blog
```
Open **_config.yml** and update the site information:
```
title: Serverless Blog
subtitle:
description: This is a serverless blog!
keywords:
author: GreenGoCloud
language: en
timezone: America/Los_Angeles
```
Next, in the terminal type:
```
hexo server
```
There should be a link to http://localhost:4000 where you should see something like:
{% asset_img serv.png %}

The default install uses the Landscape theme (which is pretty clean). Other themes can be found [HERE](https://hexo.io/themes/index.html).

I recommend the **Hipaper** theme and here's how to change the default theme:
```
git clone https://github.com/iTimeTraveler/hexo-theme-hipaper themes/hipaper
```
Now open **_config.yml** and update the **theme** which is around line 76.
```
theme: hipaper
```

# Github
I'm a big advocate for putting everything into a version control. There are a number of benefits for using a tool like Github to store projects. I'm going to assume you've heard/know how to use Github so I won't go into detail in this section.

If you don't already have a Github account, create one and then create a public repository.  

# AWS
- **NOTE:** This assumes you have a domain name in Route 53. I'll be using **greengotech.net** for this demo.

## S3
Sign into the **AWS Management Console**, and head over to S3 console. 

Create an S3 bucket (use domain name for bucket name - i.e. blog.greengotech.net) and use the following bucket policy:
{% codeblock %}
{
  "Id": "Policy1522074684919",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1522074683215",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::blog.greengotech.net/*",
      "Principal": "*"
    }
  ]
}
{% endcodeblock %}

On the Properties tab of the bucket, enable static website hosting and set the index document to index.html. Click Save.
{% asset_img s3.png %}

## Certificate Manager
Next, make sure you're in the US-East-1 (N. Virginia) region and request an SSL/TLS certificate in the AWS Certificate Manager console. CloudFront doesn't see the certs in other regions and you need it when you create the distribution. 

I suggest issuing a cert for the top level domain as well as adding a wildcard to the cert for future use. For example, domain name would be greengotech.net and then add another name to the cert would be *.greengotech.net. The wild card will cover all subdomains including blog.greengotech.net. 

If you choose DNS validation, it's rather quick but still might take a few minutes before you can move to CloudFront.

## CloudFront
Once a certificate has been successfully issued in Certificate Manager, head over to the CloudFront console. Click create distribution and select the Web option. Fill in the following options and keep the rest as default values:

1. **Origin Domain Name** - use the static website hosting endpoint for the S3 bucket (i.e. **blog.greengotech.net.s3-website-us-west-2.amazonaws.com**)
2. **Viewer Protocol Policy** - change to **Redirect HTTP to HTTPS**
3. **Object Caching** - change to **Customize** and set the **Default TTL** to **600**
4. **Alternate Domain Names (CNAMEs)** - add the domain name of the site (i.e. **blog.greengotech.net**)
5. **SSL Certificate** - change to **Custom SSL Certificate** and select the cert from the previous step.
6. Click **Create Distribution**

This will take some time as the distribution deploys so let's head over to CodeBuild in the meantime. 

## CodeBuild
**Create a new project** and give the project a name. Fill in the following options and keep the rest as default values:

1. **Source provider** select **Github**
2. **Repository** select **Use a repository in my account**
3. **Choose a repository** select the repo created in the previous step from the drop down selection
4. **Webhook** - check the box 
5. **Branch filter** type **master**
6. **Operating system** select **Ubuntu**
7. **Runtime** select **Node.js**
8. **Runtime version** select **aws/codebuild/nodejs:10.1.0**
9. **Role name** - give the role a name
10. Click **Continue**

## Route 53
By now, the CloudFront distribution should be complete and you can go to the Route 53 console to create an alias record. Create a record and point the domain name or subdomain to the CloudFront domain name.

Here's an example:
{% asset_img r53.png %}

# Slack
I wanted to make sure the builds were successful without logging into the AWS console, so I researched **AWS** and **Slack** integrations. I found a very helpful tutorial [HERE](https://github.com/bgreengo/aws-to-slack). 

After you create the Slack webhook, you launch the CloudFormation template and fill in the parameters with the Slack webhook information. After that, create a CloudWatch event and select CodeBuild as the source and the new Slack Lambda function as the target and every time a build is triggered, you'll be notified in your Slack channel. 

Here's what the CloudWatch event looks like:
{% asset_img cb.png %}

Here's what the Slack notifications look like:
{% asset_img slack.png %}

# Deploy
Finally, the last step! The **buildspec.yml** file is required for CodeBuild to generate the site and sync it to S3.

Create a **buildspec.yml** file in the **blog** directory and enter the following:
- Make sure you update the S3 bucket with your bucket name.
{% codeblock %}
version: 0.2

phases:
  install:
    commands: 
      - npm install -g hexo-cli
      - npm install
      - apt-get update
      - apt-get install -y awscli

  build: 
    commands:
      - hexo generate

  post_build: 
    commands:
      - cd public/ && aws s3 sync . s3://blog.greengotech.net/
{% endcodeblock %}

Last but not least, commit and push the files to **Github** and let the CI/CD roll! 

If all goes well, you should see something like this:
{% asset_img web.png %}

You can continue to customize the blog in the **_config.yml** file and edit the layout.

To create new posts just type:
```
hexo new **BLOG TITLE**
```

# Conclusion
It was a bit of a journey to get here but you're only a **git push** away from updating your blog content. Now you can focus on writing valuable posts rather than worrying about maintaining the underlying infrastructure. So far I've been very pleased with the low cost, high performance, and lack of adminstrative burden of my serverless blog on AWS. I must give a shout out to [Mohamed Labouardy](https://hackernoon.com/build-a-serverless-production-ready-blog-b1583c0a5ac2) for the inspiration.