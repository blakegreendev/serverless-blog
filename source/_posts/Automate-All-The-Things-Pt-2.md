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
The Terraform scripts are intended to automate the deployment of the underlying infrastructure for the project. This saves an enormous amount of time and gives a reusable baseline to expand on post deployment. After resources are running, we'll commit the Wordpress files to AWS Codecommit and let the automated AWS CodePipeline deploy to AWS Elastic Beanstalk. With some magic from ebextensions, we'll be able to abstract media files to AWS EFS and the database to AWS RDS. This creates an immutable infrastructure because the EC2 instances running the application are completely disposable. In other words, there is nothing stateful that exists on the instance itself and when a new instance is created, it picks up the configuration automatically with autoscaling.

Here is what the architecture will look like:
{% asset_img wp.png %}

# Prerequisites
Alright, enough talk. Let's deploy it. 

But first, make sure you have [Terraform](https://www.terraform.io/) and [AWS CLI](https://aws.amazon.com/cli/) installed.

Next, configure your AWS profile and make sure your IAM user has the appropriate permissions to deploy the resources. Additionally, you can log into the AWS console and generate HTTPS Git credentials for AWS CodeCommit with existing IAM user account or generate with IAM user created from Terraform. Hang on to these credentials for when we push the Wordpress files to Codecommit.

Clone the [Github](https://github.com/bgreengo/wordpress-aws-terraform-automation) repo.

# Deploy
Ready for lauch, in the terminal:
- cd wordpress-aws-terrform-automation
- Generate a keypair called 'wordpress' using ssh-keygen

```
terraform init
terraform plan  
terraform apply
```
- Fill in the variables for the S3 bucket and RDS password. Make sure the S3 bucket name is GLOBALLY UNIQUE!

{% asset_img terra.gif %}

This may take a few minutes but seriosuly, what just happend?
{% asset_img jm.gif %}

In what would have taken at least a couple hours to do manually, we were able to run three commands and a couple variables to fully automate the deployment of an immutable infrastructure in AWS. It should also spit out some outputs we'll need later.

Pretty cool, right? Let's keep going!

# Wordpress
Now the underlying infrastructure is deployed, we need to connect to the AWS CodeCommit repository and push the Wordpress files. Here's a quick reference for connecting to [AWS CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-connect.html). Use the credentials generated earlier or generate from the wordpress IAM user. The wordpress files are included in the Github repo with the .ebextensions.

1. Add the Wordpress files with .ebextensions to the AWS CodeCommit repository. 
2. Update the mount-efs.config in .ebextensions with file system ID which should be in the outputs after terraform apply completes.
3. Git push the Wordpress and .ebextensions to CodeCommit.
4. Watch the magic happen!

Head over to the AWS console and take a look in AWS CodePipeline. You should see that CodePipeline picked up the changes to the AWS CodeCommit repository and it will eventually deploy to Elastic Beanstalk.

{% asset_img cp.gif %}

After it has successfully deployed, head over to the Elastic Beanstalk console and click on the URL or find the link in the Terraform outputs.

{% asset_img eb.png %}

Now the Wordpress setup should appear. Use the RDS credentials for the DB connection. You can grab the DB information from the AWS console or in the AWS CLI, type:
```
aws rds describe-db-instances
```

{% asset_img wp-install.png %}

Finish up the install with the site information and BOOM! You now have a highly available and scalable Wordpress website. Dilly, dilly!

{% asset_img wp-page.png %}

If you want to take it one step further and have a hosted zone in Route 53, create an alias record and point it to the Elastic Beanstalk environment. This will allow you to use a custom domain name for your Wordpress website.

If you don't want to keep this running, don't forget to tear it down with: 
```
terraform destroy
```

# Conclusion
As with all projects, there's always room for improvement. Like I said in Part 1, this was my first automation project using Terraform. I've already found some ways to make it better but I'm always open to suggestions. I think this is a pretty good starting point.

Some improvements include:
- Use Terraform modules 
- Store RDS secrets
