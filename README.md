# The Impact of Course Average GPA on Professor Reviews

CMSC320 Final Project (Fall 2022)

by Tommy Chan, Alex Chen, Jessica Wu

## Introduction
[PlanetTerp](https://planetterp.com/) is an open-source student-run platform that allows students at the University of Maryland to publish reviews for their professors in the courses that they have previously took. The website aggregates all of this review data for any user to publicly access and search through. Some of this data includes professos, classes, reviews, and course grades/average GPAs correlated to those classes. The statistics from this website are obtained through the University of Maryland Office of Institutional Reasearch, Planning and Assessment.

Reviews have become a very critical part of any student's education, especially when trying to gauge whether a professor's teaching style is effective for the student. However, reviews might sometimes be more reflective of personal resentment rather than an objective view of whether a professor's class was fair and effective in communicating the course material. Therefore, through this tutorial we aim to see whether the average GPA in a course has an impact on the reviews that that professor recieves. We specifically want to observe 

## Tools and Libraries:
Requests: https://requests.readthedocs.io/en/latest/
Requests is a library in Python created in the early 2010s that creates a user-friendly way to extract information from HTTP websites. The requests.get function found in this tutorial is used commonly to retrieve the data from the provided websites. The receiving end (commonly seen as the "response" variable) has mulitple functions and contains lots of information extracted from the HTTP. More functions and information can be found in the website linked above.

Pandas: https://pandas.pydata.org/docs/user_guide/index.html#user-guide
Python Pandas is a extremely useful data analysis/manipulation and documentation tool released in the late 2000s. The storage of data and analysis done in this tutorial is mainly done with Pandas. The user guide linked above provides documentation of functions and uses of pandas.

Time: https://docs.python.org/3/library/time.html *The time toolbox is a useful function that is used in this tutorial to configure for loops. More functions can be found in the link above. *

Numpy: https://numpy.org/doc/stable/user/quickstart.html Numpy is essentially an array. Used for storing data, mathmatical functions, and matrix operations. More information can be found in the link above.

Matplotlib: https://matplotlib.org/stable/api/index Matplotlib is ython's plotting library consisting of a variety of plotting methods. The most common one is pyplot and it creates a very basic line/dot plot on a coordinate plane. More advanced plotting methods are discussed in the link above.

Scikit-learn: https://scikit-learn.org/stable/user_guide.html Scikit-learn is Python's machine learning library consisting of tools for data modeling and predictions. Some algorithms scikit-learn is capable of are classification, regression, clustering, preprocessing and many more. More information can be found in the link above.

```
  import requests
  import pandas as pd
  import time
```
