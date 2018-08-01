---
title: AWS DR
tags:
  - aws
categories:
  - - AWS
date: 2018-07-30 13:30:50
---

# AWS Disaster Recovery

The term pilot light is often used to describe a DR scenario in which a minimal version of an environment is always
running in the cloud. The idea of the pilot light is an analogy that comes from the gas heater. In a gas heater, a small
flame that’s always on can quickly ignite the entire furnace to heat up a house.

This scenario has a running EC2 instance running in US-West-2 while a preconfigured AMI is stopped in US-East-1. In the event of a failure, which in this case is initated by stopping the running instance, the instance in US-East-1 is automatically started and Route 53 redirects traffic to the failover instance. 

## DR Diagram
![AWS DR](https://user-images.githubusercontent.com/23042063/42403841-0408b49a-8139-11e8-8434-c13dac0b633f.png)

## DR Video Demo
{% youtube tXNWMlKe1Do %}

## Sequence of events in this scenario:
1. Stopped instance in US-West-2 to initiate failover
2. Route 53 health check becomes unhealthy
3. SNS notification goes to subscribers and invokes Lambda function
4. Lambda starts instance in US-East-1
5. Route 53 serves traffic to failover instance

Here is a link to the CloudFormation templates on [Github](https://github.com/bgreengo/aws-dr)

