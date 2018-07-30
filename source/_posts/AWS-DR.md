---
title: AWS-DR
tags:
  - aws
categories:
  - - AWS
date: 2018-07-30 13:30:50
---

# AWS Disaster Recovery

In this demo, AWS Route 53 health checks will automatically failover from an instance running in US-WEST-2 over to an instance running in US-EAST-1. AWS Lambda is invoked to start the instance running in the US-EAST-1 region while the sysadmin receives an SNS notification that there has been a failover.

