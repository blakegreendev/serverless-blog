---
title: AWS Rekognition and Boto3
tags:
  - aws
categories:
  - - AWS
  - - Serverless
  - - Containers
  - - CI/CD
date: 2019-02-19 14:29:00
---

Rekognition is one of the coolest services from AWS. Rekognition makes it easy to add image and video analysis to your applications. It's super simple and easy to use, especially with Boto3 (the AWS SDK for Python). 

In this quick demo, I'm going to show how you can use AWS Rekognition and Boto3 to compare two photos and the results will show the similarity score.

If you don't already know, I'm a huge San Francisco 49ers fan. Jimmy Garoppolo, who was a backup to Tom Brady and the Patriots, is the starting QB for the 49ers. I'm hoping one day Jimmy G will bring six championships to San Francisco, much like Tom Brady did for New England. So, I thought maybe if they look alike, maybe they'll play alike. 

# Prerequisites
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
- AWS Account and User
- [Boto3](https://pypi.org/project/boto3/)

# Rekognition
I recommend creating a directory for your code and images:
{% codeblock %}
mkdir aws-rekognition
{% endcodeblock %}

Next, save these images into the new *aws-rekognition* folder.
- Saved as download.jpeg.
{% asset_img download.jpeg %}
- Saved as download1.jpeg.
{% asset_img download1.jpeg %}

Now, create rekognition.py in *aws-rekognition* and copy the following code into the file.

{% codeblock lang:python %}
import boto3
import pprint
 
if __name__ == "__main__":
 
    sourceFile='download.jpeg'
    targetFile='download1.jpeg'
 
    client=boto3.client('rekognition')
 
    imageSource=open(sourceFile,'rb')
    imageTarget=open(targetFile,'rb')
 
response=client.compare_faces(SimilarityThreshold=20,
                        SourceImage={'Bytes': imageSource.read()},
                        TargetImage={'Bytes': imageTarget.read()})
 
for faceMatch in response['FaceMatches']:
    position = faceMatch['Face']['BoundingBox']
    similarity = str(faceMatch['Similarity'])

pprint.pprint('The face at ' +
        str(position['Left']) + ' ' +
        str(position['Top']) +
        ' matches with ' + similarity + '% confidence')
 
imageSource.close()
imageTarget.close()
{% endcodeblock %}

Let's see the results by running:
{% codeblock %}
python rekognition.py
{% endcodeblock %}

You should see something like this:
{% asset_img terminal.png %}

Apparently, the similarity score is only 34%. Hopefully, Jimmy will play more like Tom since he doesn't really look like him.

For contrast, let's add another picture of Jimmy and see the results.

Save this image and update the targetFile in rekognition.py to download2.jpeg (or whatever you want to call it).
{% asset_img download2.jpeg %}

Now, we should see much better results with the similarity score at 99%.
{% asset_img terminal2.png %}