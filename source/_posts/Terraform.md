---
title: Terraform Cloud and Terraform v0.12
tags:
  - aws
  - IaC
  - terraform
categories:
  - - AWS
date: 2019-05-23 09:43:41
---

The folks at Hashicorp have obviously been very busy! They have recently released a new **Terraform Cloud** and new version with a bunch of awesome features in **Terraform v0.12**. I have been a big fan of Terraform (actually, all of the Hashicorp products) and I'm very excited about these releases. In this post, I'm going to dive in and explore these new announcements from the Hashicorp team. More specifically, I'll deply an EC2 instance with a remote state in Terraform Cloud and then go through some of the syntax changes in the new v0.12 release.

{% twitter [tweet-url](https://twitter.com/mitchellh/status/1131303953990242305) %}

# Terraform Cloud
In the past, maintaining the state files for Terraform has always been a bit of a pain point. You might have several projects within a project and if they each have local state files, it can become a burden to keep track of those files and their purpose. Not to mention, if you are collaborating with other team members and they need to make infrastructure changes. You could somewhat avoid this by using a remote backend with AWS S3 but it was still a bit cumbersome especially when auditing changes and whatnot. 

I think Hashicorp heard the discontent from users and administrators and have launched the Terraform Cloud. Terraform Cloud is a SaaS application that brings free collaboration features to individual users and teams with additional paid feature sets that provide team management, self-service infrastructure, and governance features. **NOTE:** I haven't read anything about the security (i.e. encryption in-transit/at-rest). 

## Getting Started
Go to https://app/terraform.io and sign up for an account (don't worry, it's free AFAIK). 

Once you've signed in, you'll create a New Organization and enter an email address (for notification purposes). 

Now, you'll see the Workspaces page. In the top right corner, click the user icon and click User Settings.
{% asset_img tcloud.png %}

Next, click on Tokens on the left, type a description, click Generate token, and then copy the token into your clipboard (or leave this page open, you'll need to use the token in a minute).
{% asset_img tcloud2.png %}

In the terminal, you'll need to create a new file:
```
vim ~/.terraformrc
```
{% asset_img tcreds.png %}
**NOTE:** Substitute your token.

Great, now you can pretty much proceed as normal except, in the project directory, add a file called: **remote-state.tf**

In the remote-state.tf file add the following (also note, you can use any name for the workspace):
{% asset_img tremote.png %}

Let's create another file in the same directory called **main.tf** and add an EC2 instance:
{% asset_img tec2.png %}

Finally, you can deploy and it will pick up the remote state configuration.
```
terraform apply
```

When you head back over to the Workspaces page on Terraform Cloud, you'll see the newly created Workspace. 
{% asset_img worksp.png %}

Click on the Workspace name, and you'll be able to see all of the state changes on this Workspace (i.e. adding tags to the EC2 instance, etc.)
{% asset_img tstate.png %}

# Terraform v0.12
Yet again, Hashicorp listened when people were turned off by some of the required syntax and lack of traditional programming features such as loops. It was a long awaited update to get to v0.12 because I believe they rewrote the entire code base for Terraform. I remember reading about it a year ago but it is finally here. 

One of my favorite updates is how they dropped the string interpolation syntax and now you can use references and expressions directly. If you noticed in the example above, in **main.tf**, I'm not required to wrap the variable in the traditional ${}. Instead, I'm able to just call on it directly and HCL knows what to do.
{% asset_img tec2.png %}

Another way in which data structures were inconvenient in prior versions was the lack of any general iteration constructs that could perform transformations on lists and maps.

Terraform v0.12 introduces a new for operator that allows building one collection from another by mapping and filtering input elements to output elements:
{% asset_img iter.png %}

# Conclusion
Overall, I'm looking forward to learning more about these new features but the Terraform team is certainly on the right path. Especially, with other tools like Pulumi and AWS CDK entering the IaC game, Terraform is evolving and meeting the developers where they are while making their lives a little bit easier.