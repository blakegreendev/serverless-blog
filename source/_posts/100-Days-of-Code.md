---
title: 100 Days of Code
tags:
  - python
categories:
  - Programming
date: 2019-05-10 08:52:00
---

I am so excited to start this journey! I'm more movitated than ever to further my knowledge of programming. I have mentioned in previous posts that having a programming language in your back pocket is becoming more and more necessary. 

I will be focused on Python during these next 100 days. Python is such a great multi-purpose programming language and it's easy to learn. I'm looking forward to diving into machine learning, data science, web development (Django and Flask), and general purpose scripting for things like Boto3 in AWS. Python is extremely popular which means there's a large community that I will be able to lean on if I get stuck. I really think this is going to further my career and excited to see the outcome!

I decided to go with the [Talk Python](https://training.talkpython.fm/courses/details/100-days-of-code-in-python) course because it's specifically geared toward the 100 days of code challenge. I was jumping around between books, blog posts, and various Udemy courses and I didn't feel like I was actually learning. This course has structure and it lays it out for you to learn more efficiently. 

I'm in this for the long game. Learning takes time and I want to track my progress thoughout this experience. The course consists of learning a new concept every few days. I will keep updating each conecpt as I go through until the 100 days is complete. Let's go!

# Days 1-3: Datetimes
Working with dates and times can be super critical in python programs. The datetime module can be useful when you need to make it easy to work with dates and times.

The unfortunate thing about this lesson is that I found out that Christmas is still 229 days away :(
{% codeblock lang:python %}
from datetime import datetime
from datetime import date

christmas = date(2019, 12, 25)
today = date.today()

if christmas is not today:
    print("Sorry there are still " + str((christmas - today).days) + " days until Christmas!")
else:
    print("Yay it's Christmas!")
{% endcodeblock %}
**Output**
```
Sorry there are still 229 days until Christmas!
```
# Days 4-6: Collections

# Days 7-9: Data Structures

# Days 10-12: Pytest

# Days 13-15: Text Games

# Days 16-18: List Comprehensions and Generators

# Days 19-21: Iteration with itertools

# Days 22-24: Decorators

# Days 25-27: Error handling

# Days 28-30: Regular Expressions

# Days 31-33: Logging

# Days 34-36: Refactoring / Pythonic code

# Days 37-39: Using CSV data

# Days 40-42: JSON in Python

# Days 43-45: Consuming HTTP services

# Days 46-48: Web Scraping with BeautifulSoup4

# Days 49-51: Measuring performance

# Days 52-54: Parsing RSS feeds with Feedparser

# Days 55-57: Structured API clients with uplink

# Days 58-60: Twitter data analysis with Python

# Days 61-63: Using the Github API with Python

# Days 64-66: Sending emails with smtplib

# Days 67-69: Copy and Paste with Pyperclip

# Days 70-72: Excel automation with openpyxl

# Days 73-75: Automate tasks with Selenium

# Days 76-78: Getting Started with Python Flask

# Days 79-81: Basic Database Access with SQLite3

# Days 82-84: Data visualization with Plotly

# Days 85-87: Fullstack web apps made easy

# Days 88-90: Home Inventory App

# Days 91-93: Database access with SQLAlchemy

# Days 94-96: Rich GUI apps in Python

# Days 97-99: Building JSON APIs

# Day 100
:smiley: