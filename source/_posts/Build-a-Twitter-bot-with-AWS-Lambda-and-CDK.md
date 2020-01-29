---
title: Build a Twitter bot with AWS Lambda and CDK
tags:
  - twitter
  - lambda
  - cdk
  - python
  - serverless
categories:
  - - AWS
date: 2020-01-24 09:49:25
---

I never know if my blog posts are any good, but I do it anyway. My hope is that they will help someone out there and encourage folks to keep learning out loud as I have tried to do. While I feel lucky for those that have visited my blog, I wanted to find a more consistent way to get my posts out to the world. I thought... what if there was a way to randomly select one of my previous blog posts and post it on Twitter every Friday at 6pm. Hopefully that would generate more interest and allow people on Twitter to easily link and share my blog posts. 

As a CDK fanboy, I immediately decided that I would build this using the CDK. The CDK app would create the CloudWatch rule that would run every Friday at 6pm using a cron expression. The CloudWatch rule would then trigger a Lambda function that would go off and scrape a random blog post from the archives and then use the Twitter API to post a tweet.

{% asset_img twitterbot.png %}

Full code is on [Github](https://github.com/bgreengo/cdk-twitter-blog)

# Prerequisites
- AWS CLI configured
- Twitter API keys - Head over to https://developer.twitter.com/ and register the bot application with Twitter. 
- Node@v12 

# AWS CDK

## The Setup
Install the latest version of the CDK CLI:
```
npm i -g aws-cdk
```

In an empty and separate folder, let's intialize the CDK project:
```
mkdir twitter-blog && cd twitter-blog
cdk init --language python
```

Now you should have the CDK skeleton where you can use the README.md to activate the virtualenv:

**NOTE:** I usually use **pipenv** to install dependencies and handle the virtual environments but whatever floats your boat.

To manually create a virtualenv on MacOS and Linux:

```
$ python3 -m venv .env
```

After the init process completes and the virtualenv is created, you can use the following
step to activate your virtualenv.

```
$ source .env/bin/activate
```

Once the virtualenv is activated, you can install the required dependencies.

```
$ pip install -r requirements.txt
```
## The Code
In the twitter_blog folder, there should be a **twitter_blog_stack.py** where you can add the following code:

{% codeblock lang:python %}
  from aws_cdk import (
      core,
      aws_events as events,
      aws_events_targets as targets,
      aws_lambda as _lambda,
      aws_iam as iam
  )

  class TwitterBlogStack(core.Stack):
      def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
          super().__init__(scope, id, **kwargs)

          ########################## LAMBDA LAYERS #######################
          # Created layers directory and installed dependencies with
          # Example: $ pip install requests -t .
          twitterlayer = _lambda.LayerVersion(
              self, 'TwitterLayer',
              code = _lambda.AssetCode('layers/twitter'),
              compatible_runtimes = [_lambda.Runtime.PYTHON_3_7]
          )

          requestslayer = _lambda.LayerVersion(
              self, 'RequestsLayer',
              code = _lambda.AssetCode('layers/requests'),
              compatible_runtimes = [_lambda.Runtime.PYTHON_3_7]
          )

          bs4layer = _lambda.LayerVersion(
              self, 'Bs4Layer',
              code = _lambda.AssetCode('layers/bs4'),
              compatible_runtimes = [_lambda.Runtime.PYTHON_3_7]
          )

          ################### LAMBDA ##############################
          tweet_lambda = _lambda.Function(
              self, 'TweetHandler',
              runtime = _lambda.Runtime.PYTHON_3_7,
              code=_lambda.Code.asset('lambda'),
              handler='handler.main',
              timeout=core.Duration.seconds(300),
              layers=[twitterlayer, requestslayer, bs4layer]
          )

          # Add permissions to get the SSM parameters for Twitter API keys
          tweet_lambda.add_to_role_policy(iam.PolicyStatement(
              effect=iam.Effect.ALLOW,
              resources=['*'],
              actions=['ssm:GetParameters','logs:CreateLogGroup','logs:CreateLogStream', 'logs:PutLogEvents']
          ))

          # CloudWatch rule that runs every Friday at 6pm 
          rule = events.Rule(
              self, "Rule",
              schedule=events.Schedule.cron(
                  minute='0',
                  hour='18',
                  month='*',
                  week_day='FRI',
                  year='*'
              )
          )

          # Add CW rule as event source to Lambda function
          rule.add_target(targets.LambdaFunction(tweet_lambda))
{% endcodeblock %}

## The Explanation
First of all, I think it's pretty cool when you only need 66 lines of code (with comments and white space) to build this project. The alternative, if you were to create this project with CloudFormation, the following command will show that it would take roughly 285 lines of yaml.
```
cdk synth twitter-blog > template.yml
```

- **Lambda Layers (lines 18-34):** 
  - Creates the layers and tells it where to get the local dependencies (twitter, requests, and bs4).
- **Lambda (lines 37-44):** 
  - Creates the function with local asset from lambda folder.
- **Lambda Role (lines 47-51):** 
  - Gives the execution role the permission to get the SSM parameters where the Twitter API keys are stored.
- **CloudWatch Rule (lines 54-63):** 
  - Create the CloudWatch rule that triggers every Friday at 6pm.
- **Add tartget (line 66):** 
  - Adds the CloudWatch rule as an event source so the Lambda function will execute every Friday at 6pm.

# AWS LAMBDA LAYERS

We need to add the dependencies necessary to use the Lambda function otherwise the function will fail. Before this project, I wasn't aware of what Lambda Layers provided. I was honestly surprised at how easy it was to add in the CDK. At first I thought I could create all the dependencies in one layer including Twitter, requests, and bs4. However, that didn't work so well so I created a layer for each. 

In the root directory, I created a folder called **layers** and then a folder for each layer, **bs4**, **requests**, **twitter**. Inside each of those folders, I created a folder called **python**. I'm not sure if that is actually required, but I read that somewhere while researching. Inside the python folder, I ran the following command:
```
$ pip install twitter -t .
```

The folder structure should look like this:
{% asset_img layers.png %}

# AWS LAMBDA
## Add Twitter API keys to SSM Parameter Store
There's probably a better way to do this but I decided to store my Twitter API keys in the AWS SSM Parameter Store. The most important thing is that these are in a secure store that Lambda can call when the function is executed. These can easily be added to the Parameter Store using the AWS CLI and the following commands:
```
aws ssm put-parameter --name /tsgt_green/access_token --value YOUR-ACCESS-TOKEN --type SecureString
aws ssm put-parameter --name /tsgt_green/access_token_secret --value YOUR-ACCESS-SECRET --type SecureString
aws ssm put-parameter --name /tsgt_green/consumer_key --value YOUR-CONSUMER-KEY --type SecureString
aws ssm put-parameter --name /tsgt_green/consumer_secret --value YOUR-CONSUMER-SECRET --type SecureString
```
- **NOTE:** Change the Twitter handle and add your API keys. 

Since we told the CDK app to grab a local asset, we need to create a folder in the root directory called **lambda**. Inside that folder, create a file called **handler.py** and copy the following code:
{% asset_img lambda.png %}
  
{% codeblock lang:python %}
import boto3
from twitter import OAuth, Twitter
import requests
from bs4 import BeautifulSoup
import random


def main(event, context):
    ####################### WEB SCRAPING ######################################

    r = requests.get('https://greengocloud.com/archives/')

    p = []

    soup = BeautifulSoup(r.text, 'html.parser')

    posts = soup.find_all(class_='archive-article-title')

    for post in posts:
        #print(post)
        title = post.get_text()
        #print(title)
        link = post.get('href')
        tl = title + ' #aws #greengocloud ' + 'https://greengocloud.com' + link
        p.append(tl)

    blog = random.choice(p)

    print(blog)

    ##################### TWITTER ############################################
    CONSUMER_KEY_PARAM_NAME = '/{}/consumer_key'.format('tsgt_green')
    CONSUMER_SECRET_PARAM_NAME = '/{}/consumer_secret'.format('tsgt_green')
    OAUTH_TOKEN_PARAM_NAME = '/{}/access_token'.format('tsgt_green')
    OAUTH_SECRET_PARAM_NAME = '/{}/access_token_secret'.format('tsgt_green')

    SSM = boto3.client('ssm')

    param_names=[
        CONSUMER_KEY_PARAM_NAME,
        CONSUMER_SECRET_PARAM_NAME,
        OAUTH_TOKEN_PARAM_NAME,
        OAUTH_SECRET_PARAM_NAME
    ]

    response = SSM.get_parameters(
        Names=param_names,
        WithDecryption=True
    )

    param_lookup = {param['Name']: param['Value'] for param in response['Parameters']}

    oauth_token=param_lookup[OAUTH_TOKEN_PARAM_NAME]
    oauth_secret=param_lookup[OAUTH_SECRET_PARAM_NAME]

    t = Twitter(
        auth=OAuth(
            oauth_token,
            oauth_secret,
            consumer_key=param_lookup[CONSUMER_KEY_PARAM_NAME],
            consumer_secret=param_lookup[CONSUMER_SECRET_PARAM_NAME],
        )
    )

    t.statuses.update(status=blog)
{% endcodeblock %}

Obviously, you'll need to update the web scraping section accordingly but hopefully this gives you a general idea how to grab a value and then pass it to the Twitter API to create a tweet.

# Wrap
Overall, it was a great experience to work on this little side project. Also, I learned that if your tweets have links to websites that don't have [Twitter Cards](https://developer.twitter.com/en/docs/tweets/optimize-with-cards/overview/abouts-cards) then they won't show a preview to your link. So, I had to add these meta tags to my blog which was interesting but worked out and makes it more appealing to click on the link in Twitter. Anyway, I guess I need to keep writing since the bot will run every week now.