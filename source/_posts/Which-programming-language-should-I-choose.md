---
title: Which programming language should I choose?
tags:
  - python
  - serverless
categories:
  - - Programming
date: 2018-10-23 12:28:09
---

The *serverless movement* is in full force. It's more important than ever to pick up a programming language. With cloud providers abstracting pretty much everything in managed services, it boils down to being able to code. Creating the business logic via code and uploading it to a cloud provider is the new standard. The days of getting paid to run and maintain a server are coming to a screeching halt. 

Say you embrace the change and want to begin this new career path. Where do you start? What programming language should you learn? 

In my opinion, there is currently three languages that will help you adapt to this new wave of IT. 
- **Python**
- **Javascript**
- **Rust**

# Python
{% asset_img python.png %}
Python was the first programming language that I learned. People recommend it as a first language to learn because it's easy. They are right! It's easy to read and comprehend. It doesn't take much to create a little project and immediately see results. Python has been around for a long time and there is an enormous community support. This is helpful because if you get stuck, chances are a quick Google search will lead you right to the answer. Also, there are a ton of 3rd party modules that you can import into your project to do some pretty fun stuff. 

For example, you can find a list of local pizza places by using the BeautifulSoup module, running the following code, and searching for pizza:
{% codeblock lang:python %}
from bs4 import BeautifulSoup
import requests

search = input("Search for:")
params = {"q": search}
r = requests.get("http://www.bing.com/search", params=params)

soup = BeautifulSoup(r.text, "html.parser")
results = soup.find("ol", {"id": "b_results"})
links = results.findAll("li", {"class": "b_algo"})

for item in links:
    item_text = item.find("a").text
    item_href = item.find("a").attrs["href"]

    if item_text and item_href:
        print(item_text)
        print(item_href)
        print("Summary:", item.find("a").parent.parent.find("p").text)
{% endcodeblock %}

To start learning the basics and build a foundation for the language, I recommend [Code Academy](https://www.codecademy.com/catalog/language/python). There's also a ton of courses on Udemy and the [Learn Python Reddit](https://www.reddit.com/r/learnpython/) is fantastic.

## Python Above and Beyond
Since this is primarily an AWS blog, I would be remiss if I didn't mention [Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html). Boto3 is the AWS SDK for Python and provides an easy to use, object-oriented API as well as low-level direct service access to AWS services like EC2 and S3. It also works extremely well with Lambda when an event invokes a function. 

Python is also heavily used in Machine Learning and Data Science, which are extremely in-demand fields. Additionally, there are a couple popular web frameworks called Django and Flask if you'd rather get into frontend development.

As you can see, Python has a ton of valuable use cases and allows you to pursue a lot of different areas in IT. 


