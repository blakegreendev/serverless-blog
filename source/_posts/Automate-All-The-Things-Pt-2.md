---
title: Automate All The Things - Part 2
tags:
  - aws
categories:
  - - AWS
  - - Serverless
  - - Containers
  - - CI/CD
date: 2018-08-06 13:57:38
---


# Intro
The Terraform scripts are intended to automate the deployment of the underlying infrastructure for the project. This saves an enormous ammount of time and gives a reusable baseline to expand on post deployment. After resources are running, we'll commit the Wordpress files to AWS Codecommit and let the automated AWS CodePipeline deploy to AWS Elastic Beanstalk. With some magic from ebextensions, we'll be able to abstract media files to AWS EFS and the database to AWS RDS. This creates an immutable infrastructure because the EC2 instances running the application are completely disposable. In other words, there is nothing stateful that exists on the instance itself and when a new instance is created, it picks up the configuration automatically with autoscaling.

Here is what the architecture will look like:
{% asset_img wp.png %}

# Prerequisites
Alright, enough talk. Let's deploy it. 

But first, make sure you have [Terraform](https://www.terraform.io/) and [AWS CLI](https://aws.amazon.com/cli/) installed.

Make sure your IAM user has correct permissions to deploy the resources and "aws configure" after installing the AWS CLI because Terraform will use your AWS profile to deploy the resources. 

Next, clone this repo from my [Github](https://github.com/bgreengo/wordpress-aws-terraform-automation) account. 

Change directory into the folder and update the following:
- Type: ssh-keygen 
  - Name the key: wordpress
- Update codepipeline.tf 
  - Rename S3 bucket to something globally unique

# Deploy
Ready for lauch, in the terminal type:
```
terraform init
terraform plan  # Always best practice when using Terraform, it allows you double check what you're deploying
terraform apply
```
{% asset_img terra.gif %}

This may take a few minutes but seriosuly, what just happend?
{% asset_img jm.gif %}

In what would have taken at least a couple hours to do manually, we were able to run three commands to fully automate the deployment of an immutable infrastructure in AWS. Pretty cool, right?

Let's keep going!

# Wordpress
Now the underlying infrastructure is deployed, we need to connect to the AWS CodeCommit repository and push the Wordpress files. 

Log into the AWS console and generate HTTPS Git credentials for AWS CodeCommit with the newly created wordpress IAM user. Here's a quick reference for connecting to [AWS CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-connect.html). 

Add the Wordpress files with .ebextensions to the AWS CodeCommit repository. Modify the [mount-efs.config] in .ebextensions and update with the file system ID that was deployed. You can use the AWS CLI to perform a quick lookup by typing: [aws efs describe-file-systems]. Finally, git push the files to AWS CodeCommit and watch the magic happen.

Take a look in AWS CodePipeline and you should see that it picked up the changes to the AWS CodeCommit repository and it will eventually deploy to Elastic Beanstalk.
{% asset_img cp.gif %}

After it has successfully deployed, head over to the Elastic Beanstalk console and click on the URL.
{% asset_img eb.png %}

Now the Wordpress setup should appear. Use the RDS credentials for the DB connection. You can grab the DB information from the AWS console or [aws rds describe-db-instances] on the AWS CLI. 
{% asset_img wp-install.png %}

Finish up the install with the site information and BOOM! You now have a highly available and scalable Wordpress website. Dilly, dilly!
{% asset_img wp-page.png %}

If you don't want to keep this running, don't forget to tear it down with: [terraform destroy]

# Conclusion
As with all projects, there's always room for improvement. Like I said in Part 1, this was my first automation project using Terraform. I've already found some ways to make it better but I'm always open to suggestions. I think this is a pretty good starting point.

Some improvements include:
- Use Terraform modules 
- Store RDS secrets
- Add EFS mount targets