---
title: Build a Serverless Resume on AWS
tags:
  - aws
  - serverless
  - zappa
categories:
  - - AWS
  - - Career Advice
date: 2019-06-07 09:10:26
---

Ditch the PDF and Word document resumes! Impress potential employers with a clean web application that truely represents your talents! This can give you a chance to stand out among the boring and old ways of creating resumes.

In this tutorial, we are going to build a serverless resume with **Flask** and then deploy it to AWS with a framework called **Zappa**.

Here's an example of what we will build: https://resume.greengocloud.com

Here's the Github repo: https://github.com/bgreengo/greengo-resume

# Prerequisites
- AWS account
- AWS profile
- Python and Pipenv
- Route 53 Hosted Zone (this is optional if you want to use a custom domain otherwise you'll get the API Gateway URL)

# Let's Get Started

First things first, create a project directory:
```
mkdir flask-resume && cd flask-resume
```

Next, I recommend just cloning or downloading the repo (it just makes it easier to start from and then you can branch off from here):
```
git clone https://github.com/bgreengo/greengo-resume.git
```

```
cd greengo-resume
```

Awesome, now let's make sure the dependencies are installed by running:
```
pipenv install
```
**NOTE:** It should automatically pick up the packages in the Pipfile and install Flask and Zappa.

Access the virtual environment with:
```
pipenv shell
```

Now you should be able to start the app and see it in your browser!
```
python app.py
```
**NOTE:** You can click the link or open a browser and go to: localhost:5000

Great! Feel free to customize the index.html to your liking and optionally the colors for the css are located in: *static/css/resume.min.css*

# Zappa Your Resume to AWS

Once everything looks good locally, you're ready to deploy to AWS! We want to world to see all the hard work we do right?

Let's initiate the Zappa project and run through the settings:
```
zappa init
```
**NOTE:** Run this inside the virtual environment (i.e. pipenv shell).

Feel free to create whatever stage you'd like, but I'll select **production** because YOLO:
{% asset_img zappa.png %}

Zappa should automatically pick up on your AWS profile if it's default, otherwise choose the correct profile. Zappa will also create a private S3 bucket where it will store the deployments in zip files. I recommend just hitting enter and accepting the default.
{% asset_img zappa2.png %}

Zappa should also automatically pick up the location of the app.py but just in case, type *app.app* here or hit enter for the default:
{% asset_img zappa3.png %}

Next, Zappa will ask if you want to deploy globally in all regions. No need for that, just select the default which is no. You'll notice in the next prompt when you have to review the Zappa settings that will be created, Zappa picks up the default region from your AWS profile. If it all looks good, hit enter to continue.
{% asset_img zappa4.png %}

You are ready to deploy to AWS!
```
zappa deploy production
```
**NOTE:** Select the stage that you picked earlier.

Excellent, if all goes well, you'll see the API Gateway URL in the terminal:
{% asset_img zappa5.png %}
**NOTE:** Add a trailing / to the end of the URL since that's the route we provided in Flask. If not, it won't come out with the CSS.

Open it up in the browser and you'll see your resume on the internet.
{% asset_img zappa6.png %}

## Add SSL to Your Resume

We want to be able to reference a custom domain for our resume rather than that ugly API Gateway URL. Here's where you'll need a Hosted Zone in Route 53. 

First, we need to log in to the AWS console, go to the N. Virginia region (us-east-1), and go to Certificate Manager.
**NOTE:** Even though your resources are deployed in another region, it's critical that you select N. Virginia to request your certificate otherwise it won't work.

I recommend requesting a new certificate with the subdomain (i.e. resume.greengocloud.com) of your choice.
{% asset_img zappa7.png %}

Next, Request a public certificate and click Request a certificate.

In the domain name, enter the subdomain that you want and click Next.
{% asset_img zappa8.png %}

I suggest selecting DNS validation, it's faster in my opinion or choose email validation and click Review.

Finally, click Confirm and request. 

Now, you can just click the 'Create record in Route 53' button and then Create for the DNS validation. Click Continue.

You might have to give it a few minutes but eventually, the certificate status will go from Pending validation to Issued. Copy the ARN to update the Zappa settings file.
{% asset_img zappa9.png %}

Add the following two lines to your *zappa_settings.json* file in your project:
{% asset_img zappa10.png %}

Last step, let's certify the app with Zappa:
```
zappa certify
```
Easy enough, you've just added SSL to your serverless resume web application.
{% asset_img zappa11.png %}

After a few minutes, if you browse to https://resume-demo.greengocloud.com, you'll see the app is fully deployed with the new URL.
{% asset_img zappa12.png %}

## Updating Your Resume
What if you get a new job or achieve another certification? Simply just update the index.html and then run:
```
zappa update production
```

## Deleting Your Resume
Let's say you get tired of paying the pennies it takes to host this web application and you want to tear it down. Again, very easily run:
```
zappa undeploy production
```

# Final Thoughts
Overall, this was a fun app to get up and running very quickly. I learned quite a bit about Flask, HTML, and CSS. Obviously, I could of made the app more dynamic by separating out the different components but my goal was to keep it simple and deploy fast. I was really impressed with the simplicity of Zappa. Compared to other frameworks like Serverless and SAM, the learning curve was quite small however, that might be because it's not a complex app.

Anyway, I really hope this helps someone out there! Maybe even get a job from it, who knows! I would love to hear about it or if you have any troubles, please feel free to reach out on Twitter!
