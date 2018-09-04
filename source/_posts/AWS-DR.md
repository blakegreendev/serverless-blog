---
title: AWS Disaster Recovery
tags:
  - aws
  - disaster recovery
categories:
  - - AWS
date: 2018-08-02 13:30:50
---

# Intro
The term **pilot light** is often used to describe a **disaster recovery** scenario in which a minimal version of an environment is always running in the cloud. The idea of the pilot light is an analogy that comes from the gas heater. In a gas heater, a small flame that’s always on can quickly ignite the entire furnace to heat up a house.

# Overview
In this scenario, an EC2 instance is running in US-West-2, while a preconfigured AMI is stopped in US-East-1. In the event of a failure, *which in this case is initated by stopping the running US-West-2 instance,* the instance in US-East-1 is automatically started via a Lambda function and Route 53 redirects traffic to the failover instance. 

This scenario is particularly useful when there is an outage at 3am in the morning and you don't need to respond because failover is already automated. 

## DR Diagram
![AWS DR](https://user-images.githubusercontent.com/23042063/42403841-0408b49a-8139-11e8-8434-c13dac0b633f.png)

# Deploying CloudFormation Stacks

Head over to Github and clone the [AWS-DR](https://github.com/bgreengo/aws-dr) repository locally.

```
git clone https://github.com/bgreengo/aws-dr
```

- Note: This assumes that you have the AWS CLI set up and US-WEST-2 is set as the default region.

## US-EAST-1
### Step 1: Security Group
```
aws cloudformation deploy --template-file DR-East/1-sg-east.yaml --stack-name pl-sg-east --region us-east-1
```

### Step 2: EIP
**For parameter values in Step 2:**
- **Subnet ID:** *aws ec2 describe-subnets --region us-east-1*
  - Example Output: **subnet-df1937e2**
- **Security Group ID:** *aws cloudformation describe-stacks --stack-name pl-sg-east --region us-east-1*
  - Example Output: **sg-0fbcc6a0a4d1f4a72**

**Update parameter values and run:**
```
aws cloudformation deploy --template-file DR-East/2-eip-east.yaml --stack-name pl-eip-east --parameter-overrides WebSubnet=<ENTER SUBNET ID> WebServerSecurityGroup=<ENTER SECURITYGROUP ID> --region us-east-1
```

### Step 3: EC2
**For parameter values in Step 3:**
- **ENI ID:** *aws cloudformation describe-stacks --stack-name pl-eip-east --region us-east-1*
  - Example Output:

**Update parameter values and run:**
```
aws cloudformation deploy --template-file DR-East/3-ec2-east.yaml --stack-name pl-ec2-east --parameter-overrides NetworkInterface=<ENTER ENI ID>
```

### Step 4: S3
```
aws cloudformation deploy --template-file DR-East/4-s3-east.yaml --stack-name pl-s3-east --region us-east-1
```

### Step 5: Lambda
- Update **start-instance.py** with instance ID from Step 3.
  - **Instance ID:** *aws cloudformation describe-stacks --stack-name pl-ec2-east --region us-east-1*
- Zip python script and upload to S3 bucket created in Step 4.
  - **Zip:** *zip start-instance.py start-instance.py.zip*
  - **Upload to S3:** *aws s3 cp start-instance.py.zip s3://BUCKETNAME-HERE*

**For parameter values in Step 5:**
- **Email:** *Enter a valid email address*
- **S3 Bucket:** *Enter name of S3 bucket from Step 4.*
- **Zip File:** *Enter name of zip file in the S3 bucket. Example, start-instance.py.zip*

**Update parameter values and run:**
```
aws cloudformation deploy --template-file DR-East/4-lambda-east.yaml --stack-name pl-lambda-east --parameter-overrides Email=<ENTER EMAIL> S3Bucket=<ENTER S3 Bucket Name> ZipFile=<ENTER Name of Zip File>
```

- **Note:** After stack creation is complete, confirm SNS subscription via email at the address provided in the parameter.

## US-WEST-2 
### Step 1: Security Group
```
aws cloudformation deploy --template-file DR-West/1-sg-west.yaml --stack-name pl-sg-west
```

### Step 2: EIP
**For parameter values in Step 2:**
- **Subnet ID:** *aws ec2 describe-subnets*
  - Example Output: **subnet-df1937e2**
- **Security Group ID:** *aws cloudformation describe-stacks --stack-name pl-sg-west*
  - Example Output: **sg-0fbcc6a0a4d1f4a72**

**Update parameter values and run:**
```
aws cloudformation deploy --template-file DR-West/2-eip-west.yaml --stack-name pl-eip-west --parameter-overrides WebSubnet=<ENTER SUBNET ID> WebServerSecurityGroup=<ENTER SECURITYGROUP ID>
```

### Step 3: EC2
**For parameter values in Step 3:**
- **ENI ID:** *aws cloudformation describe-stacks --stack-name pl-eip-west*
  - Example Output:

**Update parameter values and run:**
```
aws cloudformation deploy --template-file DR-West/3-ec2-west.yaml --stack-name pl-ec2-west --parameter-overrides NetworkInterface=<ENTER ENI ID>
```

### Step 4: Route 53
**For parameter values in Step 4:**
- **Primary Record:** *aws cloudformation describe-stacks --stack-name pl-ec2-west*
  - Example Output: 34.244.21.56
- **Secondary Record:** *aws cloudformation describe-stacks --stack-name pl-ec2-east --region us-east-1*
  - Example Output: 34.245.22.57
- **Domain Name:** Example - aws.greengotech.net
- **Hosted Zone ID:** *aws cloudformation describe-stacks --stack-name pl-eip-west*
  - Example Output:
- **SNS ARN:** *aws cloudformation describe-stacks --stack-name pl-eip-west*
  - Example Output:

**Update parameter values and run:**
```
aws cloudformation deploy --template-file DR-West/4-route53.yaml --stack-name pl-r53-west --parameter-overrides PrimaryRecord=<US-WEST-2 EIP> SecondaryRecord=<US-EAST-1 EIP> DomainName=<Domain Name> HostedZoneId=<ID in R53> AlarmSNSTopicArn=<SNS ARN> 
```

# Failover
Get the US-West-2 instance ID and initiate the failover by stopping the instance:
```
aws ec2 describe-instances
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
```

After a few minutes, Route 53 will detect an unhealthy host in US-West-2. An SNS message will go to subscribers of the topic as well as trigger a Lambda function that will start the instance in US-East-1. Route 53 will then redirect traffic to the US-East-1 instance.


# Delete CloudFormation Stacks

- Note: Remove zip file from S3 bucket before deleting stack.
    - Example: *aws s3 rm s3://pl-s3-s3bucket-1oazy52gscksv/start-instance.py.zip*

## US-EAST-1 - Delete Stacks
```
aws cloudformation delete-stack --stack-name pl-lambda-east --region us-east-1
```
```
aws cloudformation delete-stack --stack-name pl-s3-east --region us-east-1
```
```
aws cloudformation delete-stack --stack-name pl-ec2-east --region us-east-1
```
```
aws cloudformation delete-stack --stack-name pl-eip-east --region us-east-1
```
```
aws cloudformation delete-stack --stack-name pl-sg-east --region us-east-1
```

## US-WEST-2 - Delete Stacks
```
aws cloudformation delete-stack --stack-name pl-r53-west
```
```
aws cloudformation delete-stack --stack-name pl-ec2-west
```
```
aws cloudformation delete-stack --stack-name pl-eip-west
```
```
aws cloudformation delete-stack --stack-name pl-sg-west
```
<!--
## DR Video Demo
{% youtube tXNWMlKe1Do %}
-->



