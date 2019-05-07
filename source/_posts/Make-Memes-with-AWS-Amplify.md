---
title: Make Memes with AWS Amplify
tags:
  - aws
  - serverless
categories:
  - - AWS
date: 2019-04-30 15:12:00
---

AWS Amplify has certainly been one of the fastest growing services from AWS in 2019. It seems like there is a new release every week with an awesome feature. Static site generators in general have been a hot topic. Tools like Gatsby, Hugo, and even Hexo (which powers this blog) are going mainstream especially with the recent #QuitMedium movement. Folks are looking to run and host their own websites and Amplify is jumping onboard in a serious way. 

It's super simple and perfect to get a proof of concept up and running really quickly. Not to mention, it's completely **serverless**! 

In this tutorial, we'll create a React app to create memes powered by AWS Amplify. Besides, who doesn't love some memes.

# What is AWS Amplify?
AWS Amplify makes it easy to create, configure, and implement scalable mobile and web apps powered by AWS. Amplify seamlessly provisions and manages your mobile backend and provides a simple framework to easily integrate your backend with your iOS, Android, Web, and React Native frontends. Amplify also automates the application release process of both your frontend and backend allowing you to deliver features faster.

# Requirements
- Install Node
- AWS account

# Local Development
First, we'll start by cloning the repository with the React app:
```
git clone https://github.com/bgreengo/aws-amplify-memes.git
```
Next, change directory to the repo and start it up:
```
cd aws-amplify-memes
```
```
npm install
```
```
npm start
```
It should bring up a browser on localhost:
{% asset_img screenshot.png %}
Now it works locally, let's send it up to a new Github repository!

# Github
Create a new repository in your Github account which we'll use to connect to AWS Amplify:
{% asset_img github.png %}
Connect the directory where we ran the app locally to the Github repo and push to the new Github repo:
{% asset_img github2.png %}
Should look something like this:
{% asset_img github3.png %}
Now the app is in Github, let's deploy to AWS Amplify!

# AWS Amplify
*NOTE:* Amplify is only available in select regions at the time of this post. I have selected Oregon (us-west-2).

In the AWS console, head over to the Amplify Console and click on Get Started under the Deploy section.
{% asset_img amplify.png %}

Next, select Github and click Next:
{% asset_img amplify2.png %}

You'll need to authorize AWS Amplify with read-only access to your Github account. After successful authorization, select the repo previously created, the master branch and then click Next.
{% asset_img amplify3.png %}

It will auto-detect the React framework and preconfigure the build settings, so just click Next.
{% asset_img amplify4.png %}

Finally, click Save and Deploy!
{% asset_img amplify5.png %}

After a few minutes, the app will successfully deploy and will be available at a URL provided by Amplify. However, I want to use a custom domain and make it easy for people to find **GreenGo-Memes**! Click on Domain management on the side panel:
{% asset_img amplify6.png %}

I wanted to exclude the root domain (since that's where I serve my blog) so I created a subdomain called memes.greengocloud.com and click Save. 
{% asset_img amplify7.png %}
*NOTE:* This will take quite a bit of time while it is pending verification/deployment.

In the meantime, you can go to the URL provided by Amplify and see that the app has successfully deployed!
{% asset_img amplify8.png %}

# Update the Memes
Let's say we want to update a picture on the app. This is one of my favorites:
{% asset_img thecloud.png %}

Add the picture to the public -> images folder in the app directory and update index.js in the src -> Components -> MainPage -> index.js 
{% asset_img vscode.png %}

Now you can run the app locally and see the new picture:
```
npm start
```
{% asset_img vscode2.png %}

Commit and push the changes to the Gihub repo and Amplify will automatically deploy the updates to the webapp. 

If you refresh the page, you'll see the new picture deployed on AWS Amplify!
{% asset_img amplify9.png %}

Check back after a while and see if your domain name has been added to your Amplify app but that's pretty much it! Super simple and easy to use, AWS Amplify is perfect for getting up and running in no time. It's especially fun for these little project where you can make a silly app that everyone can use.



