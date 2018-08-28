---
title: Automate All The Things - Part 1
tags:
  - aws
  - terraform
  - cloudformation
categories:
  - - AWS
  - - IaC
  - - CI/CD
date: 2018-08-03 08:28:03
---

# Intro
When I began learning AWS, I started creating resources in the console. Clicking and deploying pretty much everything. I would terminate everything at the end of the day so the resources wouldn't keep accumulating costs. But the next day, I would have issues trying to recreate what I was doing because I would miss a step. This was very frustrating but it led me to automating with Infrastructure as Code (IaC).

IaC is a process of managing and provisioning resources defined in template or script. Rather than manually clicking through the console, I could automate the deployment of resources from a pre-defined template. Best of all, with a single command or click, I could terminate all the resources deployed through the template as well as reuse it and start from where I left off. This was a game changer in my eyes and I started to learn various tools that offered IaC. 

{% asset_img automate.jpeg %}

The two primary IaC tools I became familiar with were AWS CloudFormation and Terraform from HashiCorp. I was somewhat familiar with CloudFormation from studying for various AWS certifications but I actually started with Terraform. Terraform is an open source IaC software defined in a HashiCorp Configuration Language (HCL) Terraform syntax. For some reason, Terraform was easier for me to understand compared to CloudFormation. I also liked the fact that Terraform wasn't proprietary with a specific cloud provider and I could deploy on AWS, Azure, Google, etc. 

I would go through all the manual steps I would take to deploy resources and then I would "codify" the steps into a template. It was a bit more work upfront but I started to assemble these templates to automate a lot of manual tasks which allowed me to be more efficient with my time. This has become extremely useful because I'm able to take templates that I've created, use them as a starting point to modify, and reuse them on other projects. 

In Part 2 of this post, I will demo one of my first templates I wrote using Terraform. I used Terraform to deploy the underlying infrastructure for an immutable Wordpress website. More details to follow!

