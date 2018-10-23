---
title: AWS Methodology
tags:
  - aws
categories:
  - - AWS
  - - Serverless
  - - Containers
  - - CI/CD
date: 2018-10-11 09:28:19
---

The Pasadena City College migration to AWS project used Infiniti’s structured approach which closely aligns with the AWS Well-Architected framework. Using the five pillars of the Well-Architected framework, including Operational Excellence, Security, Reliability, Performance Efficiency, and Cost Optimization, allowed for an efficient and seamless transition to AWS for Pasadena City College. 
At the forefront of Infiniti’s methodology, along with security, is automation. Through automation we established operational excellence by delivering business value and continually improving processes and procedures. This included creating AWS CloudFormation templates to automate the creation of resources and the underlying infrastructure. Furthermore, this methodology allows Infiniti to redeploy as opposed to repair. The automated deployment templates are reusable and version-controlled. Instead of replacing a broken or impaired part of the infrastructure, Infiniti is able to avoid configuration drift with automation. 
While we are able to apply security with automation, protecting information and systems is always a top priority. During this project, we ensured our portion of the shared responsibility model met the highest standards. This includes client and server-side data encryption and data integrity authentication as well as networking traffic protection and identity and access management. We also used the multi AWS account strategy to further secure and isolate workloads and minimize blast radius. Separate accounts define boundaries and allow for control access and visibility to auditing data. 
