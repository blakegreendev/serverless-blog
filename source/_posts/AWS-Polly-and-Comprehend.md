---
title: AWS Polly and AWS Comprehend
tags:
  - aws
  - ai
  - aws polly
  - aws comprehend
  - boto3
  - python
categories:
  - - AWS
date: 2019-05-24 19:49:49
---

I have been having a lot of fun playing with the Artificial Intelligence services from AWS. Even more so, the Boto3 SDK makes it extremely simple. I have already looked at [AWS Rekognition and Boto3](https://greengocloud.com/2019/02/19/AWS-Rekognition-and-Boto3/) in a previous post but this time I'll dive into AWS Polly and AWS Comprehend. 

# Prerequisites
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) and AWS profile configured.
- Pipenv (On a Mac, I recommend just running 'brew install pipenv' in the terminal)
- Portaudio (Again, just run 'brew install portaudio', this is needed for the AWS Polly demo)

# Getting Started
Create a new directory for the project and in the terminal type:
```
pipenv --three
```
This will create a Python 3 virtual environment to install packages:
```
pipenv install boto3
```
```
pipenv install jupyterlab
```
```
pipenv install pyaudio
```
Finally, access the virtual environment and start jupyter lab:
```
pipenv shell
```
```
jupyter lab
```
**This should open a new tab in your browser and allow you to create a new Jupyter notebook**

# AWS Polly
Amazon Polly is a service that turns text into lifelike speech, allowing you to create applications that talk, and build entirely new categories of speech-enabled products. Amazon Polly is a Text-to-Speech service that uses advanced deep learning technologies to synthesize speech that sounds like a human voice.

In the Jupyter Lab, copy the following code and enjoy:
{% codeblock lang:python %}
import os
import boto3
import pyaudio

polly = boto3.client("polly")
audio = pyaudio.PyAudio()

def speak(text, voice):
    resp = polly.synthesize_speech(
        Text=text,
        TextType="text",
        OutputFormat="pcm",
        VoiceId=voice
    )
    stream = audio.open(
        format=audio.get_format_from_width(width=2),
        channels=1, rate=16000, output=True
    )
    stream.write(resp['AudioStream'].read())
    stream.stop_stream()
    stream.close()
{% endcodeblock %}
In the next cell, type:
```
speak("Hello, my name is Inigo Montoya. You killed my father. Prepare to die.", "Enrique")
```
Here's a screen shot:
{%asset_img polly.png%}
**Note: In Jupyter notebooks, you press SHIFT+ENTER to go to the next cell**

Does it sound familiar? It was the closest voice I could find to Inigo Montoya from Princess Bride but you can play with the other voices in the [Boto3 Docs for AWS Polly](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/polly.html#Polly.Client.start_speech_synthesis_task)
{% youtube 6JGp7Meg42U %}

# AWS Comprehend
Amazon Comprehend is a natural language processing (NLP) service that uses machine learning to find insights and relationships in text.

Amazon Comprehend uses machine learning to help you uncover the insights and relationships in your unstructured data. The service identifies the language of the text; extracts key phrases, places, people, brands, or events; understands how positive or negative the text is; analyzes text using tokenization and parts of speech; and automatically organizes a collection of text files by topic.

Create a new Jupyter notebook and in three lines you can get sentiment analysis on certain phrases you pass to the service.

Here's the code:
{% codeblock lang:python %}
import boto3

comprehend = boto3.client("comprehend")

comprehend.detect_sentiment(Text="I love learning AWS", LanguageCode="en")

{% endcodeblock %}
Here's some screenshots from the Jupyter notebook:
{%asset_img comp.png%}
**The sentiment was 97% positive.**

Let's see what a negative analysis looks like:
{%asset_img comp2.png%}
**I hate using hate but you can see it gives a negative sentiment score.**

If we put them both together, let's see the results:
{%asset_img comp3.png%}
**We get a mixed result because it's able to pick up on both parts of the phrase**

# Conclusion
AWS make it so simple and easy to use these machine learning and artificial intelligence services. With the batteries included, you can simply add these features or analysis to your applications. They are also fun to play around with (obviously). I'm looking forward to continuing down the list of ML and AI services that AWS provide and keep learning!