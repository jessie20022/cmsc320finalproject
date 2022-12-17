# How Does Your Major Affect Your Average GPA?

CMSC320 Final Project (Fall 2022)

by Tommy Chan, Alex Chen, Jessica Wu

## Introduction
[PlanetTerp](https://planetterp.com/) is an open-source student-run platform that allows students at the University of Maryland to publish reviews for their professors in the courses that they have previously taken. The website aggregates all of this review data for any user to publicly access and search through. Some of this data includes professors, classes, reviews, and course grades/average GPAs correlated to those classes. The statistics from this website are obtained through the University of Maryland Office of Institutional Research, Planning and Assessment.

Reviews have become a very critical part of any college student’s education, especially when trying to gauge whether a professor’s teaching style is effective for the student. However, reviews might not necessarily be an objective view of whether a professor’s class was fair and effective in communicating the course material. Through this tutorial, we wanted to explore this phenomenon and find factors that might affect a student’s ratings and how they can be used to interpret the effectiveness of a professor.

Undergraduate students may find this tutorial project useful in determining which classes they may want to take, or which professor one might prefer. In addition, students that have an undecided major or high school students going into college could use this tutorial to help guide them in which major they may find have good reviews with a good resulting grade that still align to their educational interests. Overall, many students could find a lot of useful information in making decisions with future class enrollment or insight on other courses and professors. 

By analyzing review data from PlanetTerp, we hope to be able to help students identify methods to make more informed decisions about their choice of professor and class in the upcoming semesters.

## Tools and Libraries:
We will be using Python 3 (you can download the latest version [here](https://www.python.org/downloads/)) for this final tutorial. This will be the programming language that all of our code will be written in and is the programming language of choice for many data science applications.

Additionally, we will be using external libraries on top of what is provided to us in the default Python environment. Appropriate documentation and additional information is given.


[Requests](https://requests.readthedocs.io/en/latest/): Requests is a library in Python created in the early 2010s that creates a user-friendly way to extract information from HTTP websites. The requests.get function found in this tutorial is used commonly to retrieve the data from the provided websites. The receiving end (commonly seen as the "response" variable) has mulitple functions and contains lots of information extracted from the HTTP. More functions and information can be found in the website linked above.

[Pandas](https://pandas.pydata.org/docs/user_guide/index.html#user-guide): 
Python Pandas is a extremely useful data analysis/manipulation and documentation tool released in the late 2000s. The storage of data and analysis done in this tutorial is mainly done with Pandas. The user guide linked above provides documentation of functions and uses of pandas.

[Time](https://docs.python.org/3/library/time.html): The time toolbox is a useful function that is used in this tutorial to configure for loops. More functions can be found in the link above.

[Numpy](https://numpy.org/doc/stable/user/quickstart.html): Numpy is essentially an array. Used for storing data, mathmatical functions, and matrix operations. More information can be found in the link above.

[Matplotlib](https://matplotlib.org/stable/api/index): Matplotlib is ython's plotting library consisting of a variety of plotting methods. The most common one is pyplot and it creates a very basic line/dot plot on a coordinate plane. More advanced plotting methods are discussed in the link above.

[Scikit-learn](https://scikit-learn.org/stable/user_guide.html): Scikit-learn is Python's machine learning library consisting of tools for data modeling and predictions. Some algorithms scikit-learn is capable of are classification, regression, clustering, preprocessing and many more. More information can be found in the link above.

[Scipy](https://docs.scipy.org/doc/scipy/tutorial/index.html#user-guide): A powerful open source statistical computation library that is later used in this tutorial to run a two sample t test. More information can be found in the user guide linked above.

Now let's import these libraries with the following code:


```python
import requests
import pandas as pd
import time
import numpy as np
from matplotlib import pyplot as plt
import scipy.stats as stats
from sklearn import linear_model
```

## Data Collection
We will be collecting our data using the publicly available PlantTerp [API](https://planetterp.com/api/), where we can retrieve data on courses, professors, reviews and grades. There are 3 different endpoints that we want to consider in this project: /courses, /professors and /grades. 

The following is the response schema for each of the 3 endpoints.

### Course

|      Name     |   Type   |                              Description                              |
|:-------------:|:--------:|:---------------------------------------------------------------------:|
| department    | string   | The department in which the course is offered (i.e. CMSC, MATH, BMGT) |
| course_number | string   | The numerical part of a course identifier (i.e. 320 in CMSC320)       |
| title         | string   | The name of the course (i.e. "Introduction to Data Science")          |
| description   | string   | The description of the course as shown on Testudo                     |
| credits       | int      | The number of credits associated with the course.                     |
| professors    | [string] | A list of professors that teach the course.                           |
| average_gpa   | double   | The average GPA of all students who have taken the course.            |

### Professor

|      Name      |   Type   |                                 Description                                 |
|:--------------:|:--------:|:---------------------------------------------------------------------------:|
| name           | string   | The name of the professor (Jon Snow)                                        |
| slug           | string   | A unique identifier for the professor, commonly their last name (e.g. Snow) |
| type           | string   | Professor or TA                                                             |
| courses        | [string] | A list of courses that this professor teaches                               |
| average_rating | double   | The average of all ratings given to this professor                          |

### Grades

|    Name   |   Type  |                      Description                      |
|:---------:|:-------:|:-----------------------------------------------------:|
| course    | string  | The course we are getting grades for                  |
| professor | string  | The professor giving the following grades             |
| semester  | string  | The semester when the grades were given               |
| section   | string  | The section of the course where the grades were given |
| A+        | integer | Count of A+                                           |
| A         | integer | Count of A                                            |
| A-        | integer | Count of A-                                           |
| B+        | integer | Count of B+                                           |
| B         | integer | Count of B                                            |
| B-        | integer | Count of B-                                           |
| C+        | integer | Count of C+                                           |
| C         | integer | Count of C                                            |
| C-        | integer | Count of C-                                           |
| D+        | integer | Count of D+                                           |
| D         | integer | Count of D                                            |
| D-        | integer | Count of D-                                           |
| F         | integer | Count of F                                            |
| W         | integer | Count of W                                            |
| Other     | integer | Count of Other                                        |

Using the requests library, we can interact with our API by sending HTTP GET requests to the endpoints and specifying parameters. On our initial observation using the PlanetTerp API, we found that the API has a limit of 100 responses with an offset parameter. Therefore intuitively, to build our dataframes, we would have to query our API, increment the offset by 100 and then continue to query until we run out of data. This is what we attempted to do in the following code. 

However, we found that this block took roughly ~10 min to generate the dataframe, which we found to be unsustainable for later, when we desire to have immediate new copies of our dataframe. Therefore, we decided to generate a csv file for our data, and then call read_csv to generate our dataframe in an efficient manner.


```python
all_courses_df = pd.DataFrame()

base_url = 'https://api.planetterp.com/v1'
response = requests.get(base_url + "/courses?&reviews=true")
data = response.json()
all_courses_df = pd.DataFrame.from_dict(data)

for i in range (1, 113):
  r = "/courses?&reviews=true&offset=" + str(i) + "00"
  response = requests.get(base_url + r)
  data = response.json()
  temp = pd.DataFrame.from_dict(data)
  all_courses_df = pd.concat([all_courses_df, temp])
  time.sleep(0.1)

all_courses_df.to_csv('all_courses.csv')

all_courses_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>average_gpa</th>
      <th>professors</th>
      <th>reviews</th>
      <th>department</th>
      <th>course_number</th>
      <th>name</th>
      <th>title</th>
      <th>credits</th>
      <th>description</th>
      <th>is_recent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3.728530</td>
      <td>[Jordan Boyd-Graber, A Seyed, Vanessa Frias-Ma...</td>
      <td>[]</td>
      <td>INST</td>
      <td>737</td>
      <td>INST737</td>
      <td>Introduction to Data Science</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; INST627 and (LBSC690, LBS...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>[Aaron Goldman, Mark Hill, Robert DiLutis, Eri...</td>
      <td>[]</td>
      <td>MUSC</td>
      <td>800W</td>
      <td>MUSC800W</td>
      <td>Advanced Seminar in Music Pedagogy</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; MUSC400; or students who ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3.407091</td>
      <td>[Qin Wang, Abani Pradhan, Solmaz Alborzi, Yang...</td>
      <td>[{'professor': 'Shraddha Karanth', 'course': '...</td>
      <td>NFSC</td>
      <td>112</td>
      <td>NFSC112</td>
      <td>Food: Science and Technology</td>
      <td>3.0</td>
      <td>Introduction to the realm of food science, foo...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3.923333</td>
      <td>[Abani Pradhan, Abani Pradhan, Abani Pradhan]</td>
      <td>[]</td>
      <td>NFSC</td>
      <td>679R</td>
      <td>NFSC679R</td>
      <td>Selected Topics in Food Science; Food Safety a...</td>
      <td>4.0</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2.300000</td>
      <td>[Emily Perez, Laura Williams, Douglas Kern, Is...</td>
      <td>[{'professor': 'Zita Nunes', 'course': 'AASP29...</td>
      <td>AASP</td>
      <td>298L</td>
      <td>AASP298L</td>
      <td>African-American Literature and Culture</td>
      <td>3.0</td>
      <td>&lt;b&gt;Cross-listed with:&lt;/b&gt; ENGL234.\n&lt;b&gt;Credit ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>95</th>
      <td>3.000000</td>
      <td>[Hester Baer]</td>
      <td>[]</td>
      <td>FILM</td>
      <td>469D</td>
      <td>FILM469D</td>
      <td>Special Topics in Film Theories II; Thinking, ...</td>
      <td>3.0</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>96</th>
      <td>3.216260</td>
      <td>[Humberto Coronado, Humberto Coronado, Humbert...</td>
      <td>[]</td>
      <td>BULM</td>
      <td>700</td>
      <td>BULM700</td>
      <td>Business Fundamentals for Supply Chain Managers I</td>
      <td>2.0</td>
      <td>&lt;b&gt;Restriction:&lt;/b&gt; Permission of BMGT-Robert ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>97</th>
      <td>2.850000</td>
      <td>[Humberto Coronado, Humberto Coronado, Humbert...</td>
      <td>[]</td>
      <td>BULM</td>
      <td>701</td>
      <td>BULM701</td>
      <td>Business Fundamentals for Supply Chain Manager...</td>
      <td>2.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; BULM700; or permission of...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>98</th>
      <td>3.729032</td>
      <td>[Humberto Coronado]</td>
      <td>[]</td>
      <td>BUSM</td>
      <td>758X</td>
      <td>BUSM758X</td>
      <td>Special Topics in Business; Process Improvement</td>
      <td>2.0</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>99</th>
      <td>NaN</td>
      <td>[Isabel Lloyd, Isabel Lloyd]</td>
      <td>[]</td>
      <td>ENMA</td>
      <td>672</td>
      <td>ENMA672</td>
      <td>Additive Manufacturing of Materials</td>
      <td>3.0</td>
      <td>&lt;b&gt;Restriction:&lt;/b&gt; Must be in one of the foll...</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
<p>11300 rows × 10 columns</p>
</div>



We did a very similar procedure for the professors endpoint.


```python
prof_df = pd.DataFrame()

api_url = 'https://api.planetterp.com/v1'
response = requests.get(api_url + "/professors?type=professor&reviews=true")
data = response.json()
prof_df = pd.DataFrame.from_dict(data)

for i in range (1, 117):
  r = "/professors?type=professor&reviews=true&offset=" + str(i) + "00"
  response = requests.get(api_url + r)
  data = response.json()
  temp = pd.DataFrame.from_dict(data)
  prof_df = pd.concat([prof_df, temp])
  time.sleep(0.1)

prof_df.to_csv('professors_data.csv')

prof_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>courses</th>
      <th>average_rating</th>
      <th>type</th>
      <th>reviews</th>
      <th>name</th>
      <th>slug</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>[INST737, ENPM808W, ENPM808W, ENPM808W]</td>
      <td>NaN</td>
      <td>professor</td>
      <td>[]</td>
      <td>A Seyed</td>
      <td>seyed</td>
    </tr>
    <tr>
      <th>1</th>
      <td>[MUSC800W, MUSC830W, MUSC830W]</td>
      <td>NaN</td>
      <td>professor</td>
      <td>[]</td>
      <td>Aaron Goldman</td>
      <td>goldman_aaron</td>
    </tr>
    <tr>
      <th>2</th>
      <td>[THET678, THET499]</td>
      <td>NaN</td>
      <td>professor</td>
      <td>[]</td>
      <td>Aaron Posner</td>
      <td>posner</td>
    </tr>
    <tr>
      <th>3</th>
      <td>[NFSC431, NFSC679R, NFSC112, HLTH672, HLTH710,...</td>
      <td>5.0</td>
      <td>professor</td>
      <td>[{'professor': 'Abani Pradhan', 'course': None...</td>
      <td>Abani Pradhan</td>
      <td>pradhan</td>
    </tr>
    <tr>
      <th>4</th>
      <td>[CMLT235, ENGL101, ENGL234, ENGL101H, AASP298L]</td>
      <td>NaN</td>
      <td>professor</td>
      <td>[]</td>
      <td>Abbey Morgan</td>
      <td>morgan_abbey</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>95</th>
      <td>[]</td>
      <td>NaN</td>
      <td>professor</td>
      <td>[]</td>
      <td>Theresa Nebel Robinson</td>
      <td>robinson</td>
    </tr>
    <tr>
      <th>96</th>
      <td>[]</td>
      <td>5.0</td>
      <td>professor</td>
      <td>[{'professor': 'Siqiao Ao', 'course': 'COMM107...</td>
      <td>Siqiao Ao</td>
      <td>ao</td>
    </tr>
    <tr>
      <th>97</th>
      <td>[]</td>
      <td>2.0</td>
      <td>professor</td>
      <td>[{'professor': 'Lance Shapiro', 'course': 'BSC...</td>
      <td>Lance Shapiro</td>
      <td>shapiro_lance</td>
    </tr>
    <tr>
      <th>98</th>
      <td>[]</td>
      <td>5.0</td>
      <td>professor</td>
      <td>[{'professor': 'Rudy Kim', 'course': 'COMM107'...</td>
      <td>Rudy Kim</td>
      <td>kim_rudy</td>
    </tr>
    <tr>
      <th>99</th>
      <td>[]</td>
      <td>5.0</td>
      <td>professor</td>
      <td>[{'professor': 'Rishabh Mikkar', 'course': 'ST...</td>
      <td>Rishabh Mikkar</td>
      <td>mikkar_rishabh</td>
    </tr>
  </tbody>
</table>
<p>11700 rows × 6 columns</p>
</div>



Now, let's import our csv files as Pandas dataframes starting with courses, dropping any rows with no average_gpa or reviews. Similarly, let's do the same for professors, dropping rows of professors with no courses, average_rating or reviews.


```python
all_courses_df = pd.read_csv('all_courses.csv', na_filter=True, na_values='[]', index_col=0)
all_courses_df = all_courses_df.loc[all_courses_df["average_gpa"] == all_courses_df["average_gpa"]]
all_courses_df = all_courses_df.loc[all_courses_df["reviews"] == all_courses_df["reviews"]]

prof_df = pd.read_csv('professors_data.csv', na_filter=True, na_values='[]', index_col=0)
prof_df = prof_df.loc[prof_df["courses"] == prof_df["courses"]]
prof_df = prof_df.loc[prof_df["average_rating"] == prof_df["average_rating"]]
prof_df = prof_df.loc[prof_df["reviews"] == prof_df["reviews"]]
```

Finally, to get grades, we have to reference our previous professors dataframe to be able to get our grades dataframe since professor is a required parameter for this endpoint. Thus, we do the following:


```python
grades_df = pd.DataFrame()

for i in prof_df['name']:
  api_url = 'https://api.planetterp.com/v1'
  response = requests.get(api_url + "/grades?professor=" + i)
  data = response.json()
  temp = pd.DataFrame.from_dict(data)
  grades_df = pd.concat([grades_df, temp])
  time.sleep(0.1)

grades_df.to_csv('grades_df.csv')

grades_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>course</th>
      <th>professor</th>
      <th>semester</th>
      <th>section</th>
      <th>A+</th>
      <th>A</th>
      <th>A-</th>
      <th>B+</th>
      <th>B</th>
      <th>B-</th>
      <th>C+</th>
      <th>C</th>
      <th>C-</th>
      <th>D+</th>
      <th>D</th>
      <th>D-</th>
      <th>F</th>
      <th>W</th>
      <th>Other</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NFSC431</td>
      <td>Abani Pradhan</td>
      <td>201201</td>
      <td>0101</td>
      <td>0</td>
      <td>10</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NFSC679R</td>
      <td>Abani Pradhan</td>
      <td>201208</td>
      <td>0101</td>
      <td>2</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NFSC431</td>
      <td>Abani Pradhan</td>
      <td>201301</td>
      <td>0101</td>
      <td>4</td>
      <td>8</td>
      <td>2</td>
      <td>5</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NFSC679R</td>
      <td>Abani Pradhan</td>
      <td>201308</td>
      <td>0101</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NFSC431</td>
      <td>Abani Pradhan</td>
      <td>201401</td>
      <td>0101</td>
      <td>4</td>
      <td>5</td>
      <td>2</td>
      <td>3</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>LARC151</td>
      <td>Deni Ruggeri</td>
      <td>202201</td>
      <td>0104</td>
      <td>8</td>
      <td>7</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>LARC720</td>
      <td>Deni Ruggeri</td>
      <td>202201</td>
      <td>0101</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>0</th>
      <td>PLCY388A</td>
      <td>Brandi Slaughter</td>
      <td>202201</td>
      <td>0101</td>
      <td>14</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>PLCY388A</td>
      <td>Brandi Slaughter</td>
      <td>202201</td>
      <td>0201</td>
      <td>7</td>
      <td>0</td>
      <td>5</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>0</th>
      <td>FMSC330</td>
      <td>Tanner Kilpatrick</td>
      <td>202201</td>
      <td>0201</td>
      <td>7</td>
      <td>12</td>
      <td>8</td>
      <td>6</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>83322 rows × 19 columns</p>
</div>



And now, we are reading it from our downloaded csv for easier manipulation later on:


```python
grades_df = pd.read_csv('grades_df.csv', na_filter=True, na_values='[]', index_col=0)

grades_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>course</th>
      <th>professor</th>
      <th>semester</th>
      <th>section</th>
      <th>A+</th>
      <th>A</th>
      <th>A-</th>
      <th>B+</th>
      <th>B</th>
      <th>B-</th>
      <th>C+</th>
      <th>C</th>
      <th>C-</th>
      <th>D+</th>
      <th>D</th>
      <th>D-</th>
      <th>F</th>
      <th>W</th>
      <th>Other</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NFSC431</td>
      <td>Abani Pradhan</td>
      <td>201201</td>
      <td>0101</td>
      <td>0</td>
      <td>10</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NFSC679R</td>
      <td>Abani Pradhan</td>
      <td>201208</td>
      <td>0101</td>
      <td>2</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NFSC431</td>
      <td>Abani Pradhan</td>
      <td>201301</td>
      <td>0101</td>
      <td>4</td>
      <td>8</td>
      <td>2</td>
      <td>5</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NFSC679R</td>
      <td>Abani Pradhan</td>
      <td>201308</td>
      <td>0101</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NFSC431</td>
      <td>Abani Pradhan</td>
      <td>201401</td>
      <td>0101</td>
      <td>4</td>
      <td>5</td>
      <td>2</td>
      <td>3</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>LARC151</td>
      <td>Deni Ruggeri</td>
      <td>202201</td>
      <td>0104</td>
      <td>8</td>
      <td>7</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>LARC720</td>
      <td>Deni Ruggeri</td>
      <td>202201</td>
      <td>0101</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>0</th>
      <td>PLCY388A</td>
      <td>Brandi Slaughter</td>
      <td>202201</td>
      <td>0101</td>
      <td>14</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>PLCY388A</td>
      <td>Brandi Slaughter</td>
      <td>202201</td>
      <td>0201</td>
      <td>7</td>
      <td>0</td>
      <td>5</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>0</th>
      <td>FMSC330</td>
      <td>Tanner Kilpatrick</td>
      <td>202201</td>
      <td>0201</td>
      <td>7</td>
      <td>12</td>
      <td>8</td>
      <td>6</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>83322 rows × 19 columns</p>
</div>



## Exploration

Now let's observe a few entries of the 3 dataframes we created, all_courses_df, prof_df and grades_df.


```python
all_courses_df.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>average_gpa</th>
      <th>professors</th>
      <th>reviews</th>
      <th>department</th>
      <th>course_number</th>
      <th>name</th>
      <th>title</th>
      <th>credits</th>
      <th>description</th>
      <th>is_recent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>3.407091</td>
      <td>['Qin Wang', 'Abani Pradhan', 'Solmaz Alborzi'...</td>
      <td>[{'professor': 'Shraddha Karanth', 'course': '...</td>
      <td>NFSC</td>
      <td>112</td>
      <td>NFSC112</td>
      <td>Food: Science and Technology</td>
      <td>3.0</td>
      <td>Introduction to the realm of food science, foo...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2.300000</td>
      <td>['Emily Perez', 'Laura Williams', 'Douglas Ker...</td>
      <td>[{'professor': 'Zita Nunes', 'course': 'AASP29...</td>
      <td>AASP</td>
      <td>298L</td>
      <td>AASP298L</td>
      <td>African-American Literature and Culture</td>
      <td>3.0</td>
      <td>&lt;b&gt;Cross-listed with:&lt;/b&gt; ENGL234.\n&lt;b&gt;Credit ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>5</th>
      <td>3.530045</td>
      <td>['Justin Lohr', 'Sarah Pleydell', 'Catherine B...</td>
      <td>[{'professor': 'Justin Lohr', 'course': 'ENGL1...</td>
      <td>ENGL</td>
      <td>101H</td>
      <td>ENGL101H</td>
      <td>Academic Writing</td>
      <td>3.0</td>
      <td>&lt;b&gt;Additional information:&lt;/b&gt; Any student who...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2.839834</td>
      <td>['Douglas Kern', 'Zita Nunes', 'Mary Washingto...</td>
      <td>[{'professor': 'Zita Nunes', 'course': 'ENGL23...</td>
      <td>ENGL</td>
      <td>234</td>
      <td>ENGL234</td>
      <td>African-American Literature and Culture</td>
      <td>3.0</td>
      <td>&lt;b&gt;Cross-listed with:&lt;/b&gt; AASP298L.\n&lt;b&gt;Credit...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>8</th>
      <td>3.625573</td>
      <td>['Guangming Zhang', 'Peter Chung', 'Abhijit Da...</td>
      <td>[{'professor': 'Guangming Zhang', 'course': 'E...</td>
      <td>ENME</td>
      <td>470</td>
      <td>ENME470</td>
      <td>Finite Element Analysis</td>
      <td>3.0</td>
      <td>&lt;b&gt;Restriction:&lt;/b&gt; Senior standing; and permi...</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
prof_df.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>courses</th>
      <th>average_rating</th>
      <th>type</th>
      <th>reviews</th>
      <th>name</th>
      <th>slug</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>['NFSC431', 'NFSC679R', 'NFSC112', 'HLTH672', ...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Abani Pradhan', 'course': None...</td>
      <td>Abani Pradhan</td>
      <td>pradhan</td>
    </tr>
    <tr>
      <th>6</th>
      <td>['ENME674', 'ENMA300', 'ENME684', 'ENME489Z', ...</td>
      <td>4.3333</td>
      <td>professor</td>
      <td>[{'professor': 'Abhijit Dasgupta', 'course': '...</td>
      <td>Abhijit Dasgupta</td>
      <td>dasgupta_abhijit</td>
    </tr>
    <tr>
      <th>8</th>
      <td>['ARTH389L', 'ARTH255', 'ARTH768', 'ARTH668A',...</td>
      <td>2.8333</td>
      <td>professor</td>
      <td>[{'professor': 'Abigail McEwen', 'course': Non...</td>
      <td>Abigail McEwen</td>
      <td>mcewen</td>
    </tr>
    <tr>
      <th>11</th>
      <td>['PHYS405', 'PHYS275', 'PHYS758E', 'PHYS273', ...</td>
      <td>4.3333</td>
      <td>professor</td>
      <td>[{'professor': 'Abolhassan Jawahery', 'course'...</td>
      <td>Abolhassan Jawahery</td>
      <td>jawahery</td>
    </tr>
    <tr>
      <th>12</th>
      <td>['STAT701', 'STAT700', 'STAT750', 'STAT650', '...</td>
      <td>2.7000</td>
      <td>professor</td>
      <td>[{'professor': 'Abram Kagan', 'course': 'STAT4...</td>
      <td>Abram Kagan</td>
      <td>kagan</td>
    </tr>
  </tbody>
</table>
</div>




```python
grades_df.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>course</th>
      <th>professor</th>
      <th>semester</th>
      <th>section</th>
      <th>A+</th>
      <th>A</th>
      <th>A-</th>
      <th>B+</th>
      <th>B</th>
      <th>B-</th>
      <th>C+</th>
      <th>C</th>
      <th>C-</th>
      <th>D+</th>
      <th>D</th>
      <th>D-</th>
      <th>F</th>
      <th>W</th>
      <th>Other</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NFSC431</td>
      <td>Abani Pradhan</td>
      <td>201201</td>
      <td>0101</td>
      <td>0</td>
      <td>10</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NFSC679R</td>
      <td>Abani Pradhan</td>
      <td>201208</td>
      <td>0101</td>
      <td>2</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NFSC431</td>
      <td>Abani Pradhan</td>
      <td>201301</td>
      <td>0101</td>
      <td>4</td>
      <td>8</td>
      <td>2</td>
      <td>5</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NFSC679R</td>
      <td>Abani Pradhan</td>
      <td>201308</td>
      <td>0101</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NFSC431</td>
      <td>Abani Pradhan</td>
      <td>201401</td>
      <td>0101</td>
      <td>4</td>
      <td>5</td>
      <td>2</td>
      <td>3</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Now that we know what some entries in our dataset looks like, we want to get a better idea of what our dataset consists of and to do this we can create some visualizations to better understand the shape and tendencies of our data. Here, we have decided to plot the distribution of average rating, average GPA and letter grades.


```python
index = np.asarray([i for i in range(1, 6)])

plt.hist(prof_df["average_rating"], bins=4)
plt.xticks(index)
plt.title("Average Rating vs Count Distribution of all courses")
plt.xlabel("Average Rating")
plt.ylabel("Count")

plt.show()
```


    
![png](output_17_0.png)
    



```python
grade_counts = {grade: grades_df[grade].sum() for grade in ['A+', 'A', 'A-', 'B+', 'B', 'B-', 'C+', 'C', 'C-', 'D+', 'D', 'D-', 'F', 'W', 'Other']}

x = list(grade_counts.keys())
y = list(grade_counts.values())

plt.bar(x, y)
plt.title("Grades vs Count Distribution for all courses")
plt.xlabel("Grades")
plt.ylabel("Count")

plt.show()
```


    
![png](output_18_0.png)
    



```python
index = np.asarray([i for i in range(1, 5)])

plt.hist(all_courses_df["average_gpa"], bins=12)
plt.xticks(index)
plt.title("Average GPA vs Count Distribution for all courses")
plt.xlabel("Average GPA")
plt.ylabel("Count")

plt.show()
```


    
![png](output_19_0.png)
    


From these visualizations, we can take note of a few general trends that are significant (for general data across all departments):
* Average Rating Distribution
  * It appears that much more reviews that are left on PlanetTerp are overwhelmingly positive. The most reviews left were a 5 rating, and the count incrementally decreases for each consecutive lower rating
  * This appears as a completely left skewed distribution, which agrees with our sentiment that most of the observations occur in the medium/high range of the distribution
* Grades Distribution
  * It appears that As are the mostly commonly recieved grade, which could be related to our previous observation that most reviews were a full 5 stars
  * Only considering letter grades (A+ to F), this appears to be a right-skewed distribution, which means a lot of our observations occur in the left part of the distribution (the A range)
* Average GPA Distribution
  * The median average GPA across all courses appears to hover around the 3.3 area, with a slightly left skewed distribution

Now that we have an idea about our entire dataset, let's try to identify some potential new trends in the CS department. We will once again plot the distributions of average rating, average GPA and letter grades but this time only for entries in the CS department.


```python
cmsc_courses_df = all_courses_df
cmsc_courses_df = cmsc_courses_df.loc[cmsc_courses_df["department"] == "CMSC"]
cmsc_courses_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>average_gpa</th>
      <th>professors</th>
      <th>reviews</th>
      <th>department</th>
      <th>course_number</th>
      <th>name</th>
      <th>title</th>
      <th>credits</th>
      <th>description</th>
      <th>is_recent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>26</th>
      <td>3.199664</td>
      <td>['Atif Memon', 'Adam Porter', 'Charles Song', ...</td>
      <td>[{'professor': 'Adam Porter', 'course': 'CMSC4...</td>
      <td>CMSC</td>
      <td>436</td>
      <td>CMSC436</td>
      <td>Programming Handheld Systems</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>79</th>
      <td>2.672616</td>
      <td>['Amol Deshpande', 'Nicholas Roussopoulos', 'P...</td>
      <td>[{'professor': 'Amol Deshpande', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>424</td>
      <td>CMSC424</td>
      <td>Database Design</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>92</th>
      <td>3.461364</td>
      <td>['Charles Kassir', 'Amy Vaillancourt', 'Alyssa...</td>
      <td>[{'professor': 'Corie Brown', 'course': 'CMSC1...</td>
      <td>CMSC</td>
      <td>100</td>
      <td>CMSC100</td>
      <td>Bits and Bytes of Computer and Information Sci...</td>
      <td>1.0</td>
      <td>&lt;b&gt;Restriction:&lt;/b&gt; For first time freshmen an...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2.449417</td>
      <td>['Alan Sussman', 'Larry Herman', 'Neil Spring'...</td>
      <td>[{'professor': 'Nelson Padua-Perez', 'course':...</td>
      <td>CMSC</td>
      <td>216</td>
      <td>CMSC216</td>
      <td>Introduction to Computer Systems</td>
      <td>4.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2.746843</td>
      <td>['Chau-Wen Tseng', 'Larry Herman', 'Dave Levin...</td>
      <td>[{'professor': 'Jeffrey Foster', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>330</td>
      <td>CMSC330</td>
      <td>Organization of Programming Languages</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>47</th>
      <td>2.380112</td>
      <td>['Ashok Agrawala', 'Samrat Bhattacharjee', 'Ne...</td>
      <td>[{'professor': 'Ashok Agrawala', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>417</td>
      <td>CMSC417</td>
      <td>Computer Networks</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>68</th>
      <td>2.584963</td>
      <td>['Michelle Hugue', 'Clyde Kruskal', 'Thomas Re...</td>
      <td>[{'professor': 'John Aloimonos', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>250</td>
      <td>CMSC250</td>
      <td>Discrete Structures</td>
      <td>4.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>69</th>
      <td>2.386852</td>
      <td>['Hector Corrada Bravo', 'Mohammad Hajiaghayi'...</td>
      <td>[{'professor': 'Michelle Hugue', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>351</td>
      <td>CMSC351</td>
      <td>Algorithms</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>29</th>
      <td>3.014400</td>
      <td>['Hal Daume', 'Dana Nau', 'Donald Perlis', 'Li...</td>
      <td>[{'professor': 'Hal Daume', 'course': 'CMSC421...</td>
      <td>CMSC</td>
      <td>421</td>
      <td>CMSC421</td>
      <td>Introduction to Artificial Intelligence</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2.746742</td>
      <td>['Clyde Kruskal', 'David Mount', 'Jessica Chan...</td>
      <td>[{'professor': 'Samir Khuller', 'course': 'CMS...</td>
      <td>CMSC</td>
      <td>451</td>
      <td>CMSC451</td>
      <td>Design and Analysis of Computer Algorithms</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>33</th>
      <td>2.491505</td>
      <td>['Fawzi Emad', 'Pedram Sadeghian', 'Jandelyn P...</td>
      <td>[{'professor': 'Fawzi Emad', 'course': 'CMSC12...</td>
      <td>CMSC</td>
      <td>122</td>
      <td>CMSC122</td>
      <td>Introduction to Computer Programming via the Web</td>
      <td>3.0</td>
      <td>&lt;b&gt;Restriction:&lt;/b&gt; Must not have completed an...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>67</th>
      <td>3.148883</td>
      <td>['James Reggia', 'Hal Daume', 'Venkatramanan S...</td>
      <td>[{'professor': 'Hal Daume', 'course': 'CMSC422...</td>
      <td>CMSC</td>
      <td>422</td>
      <td>CMSC422</td>
      <td>Introduction to Machine Learning</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>90</th>
      <td>2.972768</td>
      <td>['Hector Corrada Bravo', 'Mihai Pop', 'Todd Tr...</td>
      <td>[{'professor': 'Mihai Pop', 'course': 'CMSC423...</td>
      <td>CMSC</td>
      <td>423</td>
      <td>CMSC423</td>
      <td>Bioinformatic Algorithms, Databases, and Tools</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>15</th>
      <td>3.373276</td>
      <td>['Evan Golub', 'Jon Froehlich', 'James Gilkeso...</td>
      <td>[{'professor': 'Ben Shneiderman', 'course': 'C...</td>
      <td>CMSC</td>
      <td>434</td>
      <td>CMSC434</td>
      <td>Introduction to Human-Computer Interaction</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>45</th>
      <td>3.062698</td>
      <td>['Atif Memon', 'James Purtilo', 'Garrett Vanho...</td>
      <td>[{'professor': 'James Purtilo', 'course': 'CMS...</td>
      <td>CMSC</td>
      <td>435</td>
      <td>CMSC435</td>
      <td>Software Engineering</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; 1 course with a minimum g...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>46</th>
      <td>2.946301</td>
      <td>['James Reggia']</td>
      <td>[{'professor': 'James Reggia', 'course': 'CMSC...</td>
      <td>CMSC</td>
      <td>289I</td>
      <td>CMSC289I</td>
      <td>Rise of the Machines: Artificial Intelligence ...</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>True</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2.883774</td>
      <td>['Jeffrey Foster', 'Chau-Wen Tseng', 'David Va...</td>
      <td>[{'professor': 'Jeffrey Foster', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>430</td>
      <td>CMSC430</td>
      <td>Introduction to Compilers</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>5</th>
      <td>3.220892</td>
      <td>['David Jacobs', 'John Aloimonos', 'Larry Davi...</td>
      <td>[{'professor': 'John Aloimonos', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>426</td>
      <td>CMSC426</td>
      <td>Computer Vision</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>33</th>
      <td>2.366667</td>
      <td>['Jonathan Katz', 'James Schafer', 'Lawrence W...</td>
      <td>[{'professor': 'Jonathan Katz', 'course': 'CMS...</td>
      <td>CMSC</td>
      <td>456</td>
      <td>CMSC456</td>
      <td>Cryptography</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; (CMSC106, CMSC131, or ENE...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>35</th>
      <td>3.693187</td>
      <td>['Hal Daume', 'Jordan Boyd-Graber', 'Marine Ca...</td>
      <td>[{'professor': 'Jing Lin', 'course': 'CMSC723'...</td>
      <td>CMSC</td>
      <td>723</td>
      <td>CMSC723</td>
      <td>Computational Linguistics I</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; CMSC421 or CMSC422; or st...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>54</th>
      <td>2.889347</td>
      <td>['Chau-Wen Tseng', 'Michelle Hugue', 'Alan Sus...</td>
      <td>[{'professor': 'Michelle Hugue', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>411</td>
      <td>CMSC411</td>
      <td>Computer Systems Architecture</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>55</th>
      <td>2.940404</td>
      <td>['Michelle Hugue', 'Hanan Samet', 'Venkatraman...</td>
      <td>[{'professor': 'Michelle Hugue', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>420</td>
      <td>CMSC420</td>
      <td>Advanced Data Structures</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>56</th>
      <td>2.817027</td>
      <td>['Jonathan Katz', 'Peter Keleher', 'Elaine Shi...</td>
      <td>[{'professor': 'William Arbaugh', 'course': 'C...</td>
      <td>CMSC</td>
      <td>414</td>
      <td>CMSC414</td>
      <td>Computer and Network Security</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>45</th>
      <td>2.474748</td>
      <td>['A.U. Shankar', 'Neil Spring', 'Jeffrey Holli...</td>
      <td>[{'professor': 'William Arbaugh', 'course': 'C...</td>
      <td>CMSC</td>
      <td>412</td>
      <td>CMSC412</td>
      <td>Operating Systems</td>
      <td>4.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>47</th>
      <td>2.367742</td>
      <td>['Nelson Padua-Perez', 'Ilchul Yoon', 'Anthony...</td>
      <td>[{'professor': 'Larry Herman', 'course': 'CMSC...</td>
      <td>CMSC</td>
      <td>106</td>
      <td>CMSC106</td>
      <td>Introduction to C Programming</td>
      <td>4.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; MATH115.\n&lt;b&gt;Restriction:...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>49</th>
      <td>2.442320</td>
      <td>['Jandelyn Plane', 'Fawzi Emad', 'Evan Golub',...</td>
      <td>[{'professor': 'Fawzi Emad', 'course': 'CMSC13...</td>
      <td>CMSC</td>
      <td>131</td>
      <td>CMSC131</td>
      <td>Object-Oriented Programming I</td>
      <td>4.0</td>
      <td>&lt;b&gt;Corequisite:&lt;/b&gt; MATH140.\n&lt;b&gt;Credit only g...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>50</th>
      <td>2.535442</td>
      <td>['Nelson Padua-Perez', 'Fawzi Emad', 'Larry He...</td>
      <td>[{'professor': 'Fawzi Emad', 'course': 'CMSC13...</td>
      <td>CMSC</td>
      <td>132</td>
      <td>CMSC132</td>
      <td>Object-Oriented Programming II</td>
      <td>4.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>94</th>
      <td>3.650000</td>
      <td>['Peter Keleher', 'Peter Keleher']</td>
      <td>[{'professor': 'Peter Keleher', 'course': 'CMS...</td>
      <td>CMSC</td>
      <td>818E</td>
      <td>CMSC818E</td>
      <td>Advanced Topics in Computer Systems; Clouds, C...</td>
      <td>3.0</td>
      <td>&lt;i&gt;The guiding philosophy of this course is th...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>69</th>
      <td>3.444074</td>
      <td>['Clyde Kruskal', 'Evan Golub', 'William Gasar...</td>
      <td>[{'professor': 'William Gasarch', 'course': 'C...</td>
      <td>CMSC</td>
      <td>250H</td>
      <td>CMSC250H</td>
      <td>Discrete Structures</td>
      <td>4.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>55</th>
      <td>2.813682</td>
      <td>['Adam Porter', 'Walter Cleaveland', 'Evan Gol...</td>
      <td>[{'professor': 'William Pugh', 'course': 'CMSC...</td>
      <td>CMSC</td>
      <td>433</td>
      <td>CMSC433</td>
      <td>Programming Language Technologies and Paradigms</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>47</th>
      <td>2.979952</td>
      <td>['David Mount', 'Zia Khan', 'Roger Eastman', '...</td>
      <td>[{'professor': 'David Jacobs', 'course': 'CMSC...</td>
      <td>CMSC</td>
      <td>427</td>
      <td>CMSC427</td>
      <td>Computer Graphics</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; MATH240; and minimum grad...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>70</th>
      <td>3.481818</td>
      <td>['Amol Deshpande', 'Nicholas Roussopoulos', 'A...</td>
      <td>[{'professor': 'Amol Deshpande', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>724</td>
      <td>CMSC724</td>
      <td>Database Management Systems</td>
      <td>3.0</td>
      <td>&lt;b&gt;Restriction:&lt;/b&gt; Must be in one of the foll...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>93</th>
      <td>2.884185</td>
      <td>['William Gasarch', 'Clyde Kruskal', 'Clyde Kr...</td>
      <td>[{'professor': 'William Gasarch', 'course': 'C...</td>
      <td>CMSC</td>
      <td>452</td>
      <td>CMSC452</td>
      <td>Elementary Theory of Computation</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>21</th>
      <td>3.320096</td>
      <td>['Dana Nau', 'Dana Nau', 'Dana Nau']</td>
      <td>[{'professor': 'Dana Nau', 'course': 'CMSC722'...</td>
      <td>CMSC</td>
      <td>722</td>
      <td>CMSC722</td>
      <td>Artificial Intelligence Planning</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; CMSC421; or students who ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2.689017</td>
      <td>['Mohammad Hajiaghayi', 'Dana Nau', 'Walter Cl...</td>
      <td>[{'professor': 'Dana Nau', 'course': 'CMSC474'...</td>
      <td>CMSC</td>
      <td>474</td>
      <td>CMSC474</td>
      <td>Introduction to Computational Game Theory</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>64</th>
      <td>3.263595</td>
      <td>['David Mount', 'Roger Eastman', 'Stevens Mill...</td>
      <td>[{'professor': 'Stevens Miller', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>425</td>
      <td>CMSC425</td>
      <td>Game Programming</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>87</th>
      <td>3.664706</td>
      <td>['Hector Corrada Bravo', 'Hal Daume', 'Jordan ...</td>
      <td>[{'professor': 'Jordan Boyd-Graber Ying', 'cou...</td>
      <td>CMSC</td>
      <td>726</td>
      <td>CMSC726</td>
      <td>Machine Learning</td>
      <td>3.0</td>
      <td>An introduction to modern statistical data ana...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>0</th>
      <td>3.376209</td>
      <td>['Hector Corrada Bravo', 'John Dickerson', 'Am...</td>
      <td>[{'professor': 'Hector Corrada Bravo', 'course...</td>
      <td>CMSC</td>
      <td>320</td>
      <td>CMSC320</td>
      <td>Introduction to Data Science</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>81</th>
      <td>3.226063</td>
      <td>['Nelson Padua-Perez', 'Richard Johnson', 'Ped...</td>
      <td>[{'professor': 'Nelson Padua-Perez', 'course':...</td>
      <td>CMSC</td>
      <td>389N</td>
      <td>CMSC389N</td>
      <td>Special Topics in Computer Science; Introducti...</td>
      <td>3.0</td>
      <td>&lt;i&gt;\n&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>47</th>
      <td>3.365746</td>
      <td>['James Purtilo', 'Thomas Reinhardt', 'Nelson ...</td>
      <td>[{'professor': 'Fawzi Emad', 'course': 'CMSC13...</td>
      <td>CMSC</td>
      <td>132H</td>
      <td>CMSC132H</td>
      <td>Object-Oriented Programming II</td>
      <td>4.0</td>
      <td>Introduction to use of computers to solve prob...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>47</th>
      <td>3.577941</td>
      <td>['Neil Spring', 'Samrat Bhattacharjee', 'Samra...</td>
      <td>[{'professor': 'Neil Spring', 'course': 'CMSC2...</td>
      <td>CMSC</td>
      <td>216H</td>
      <td>CMSC216H</td>
      <td>Introduction to Computer Systems</td>
      <td>4.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>49</th>
      <td>3.314828</td>
      <td>['Nelson Padua-Perez', 'John Dickerson']</td>
      <td>[{'professor': 'John Dickerson', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>389K</td>
      <td>CMSC389K</td>
      <td>Special Topics in Computer Science; Full-stack...</td>
      <td>1.0</td>
      <td>&lt;i&gt;Prerequisites: Minimum grade of C- in CMSC2...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>29</th>
      <td>3.678721</td>
      <td>['Dave Levin', 'Thomas Goldstein', 'Franklin Y...</td>
      <td>[{'professor': 'Thomas Goldstein', 'course': '...</td>
      <td>CMSC</td>
      <td>389O</td>
      <td>CMSC389O</td>
      <td>Special Topics in Computer Science; The Coding...</td>
      <td>1.0</td>
      <td>&lt;i&gt;\n&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>33</th>
      <td>2.059444</td>
      <td>['David Van Horn']</td>
      <td>[{'professor': 'David Van Horn', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>131A</td>
      <td>CMSC131A</td>
      <td>Object-Oriented Programming I</td>
      <td>4.0</td>
      <td>Introduction to programming and computer scien...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>73</th>
      <td>3.955172</td>
      <td>['Furong Huang', 'Ramani Duraiswami', 'Ramani ...</td>
      <td>[{'professor': 'Nicholas Roussopoulos', 'cours...</td>
      <td>CMSC</td>
      <td>828R</td>
      <td>CMSC828R</td>
      <td>Advanced Topics in Information Processing; Dee...</td>
      <td>1.0</td>
      <td>&lt;i&gt;Seminar course reviewing current literature...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>22</th>
      <td>3.602151</td>
      <td>['Matthias Zwicker', 'Matthias Zwicker']</td>
      <td>[{'professor': 'Matthias Zwicker', 'course': '...</td>
      <td>CMSC</td>
      <td>740</td>
      <td>CMSC740</td>
      <td>Advanced Computer Graphics</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; MATH240 and CMSC420; or p...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>74</th>
      <td>3.518644</td>
      <td>['Niki Vazou', 'Marshini Chetty', 'Furong Huang']</td>
      <td>[{'professor': 'Niki Vazou', 'course': 'CMSC49...</td>
      <td>CMSC</td>
      <td>498V</td>
      <td>CMSC498V</td>
      <td>Selected Topics in Computer Science; Advanced ...</td>
      <td>3.0</td>
      <td>Prerequisites: Minimum grade of C- in CMSC422 ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>52</th>
      <td>2.395266</td>
      <td>['Xiaodi Wu', 'Andrew Childs', 'Xiaodi Wu', 'X...</td>
      <td>[{'professor': 'Xiaodi Wu', 'course': 'CMSC457...</td>
      <td>CMSC</td>
      <td>457</td>
      <td>CMSC457</td>
      <td>Introduction to Quantum Computing</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; 1 course with a minimum g...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>52</th>
      <td>3.690336</td>
      <td>['Jason Filippou', 'Alexander Brassel', 'Jason...</td>
      <td>[{'professor': 'Jason Filippou', 'course': 'CM...</td>
      <td>CMSC</td>
      <td>389E</td>
      <td>CMSC389E</td>
      <td>Special Topics in Computer Science; Digital Lo...</td>
      <td>1.0</td>
      <td>&lt;i&gt;\n&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>62</th>
      <td>3.370213</td>
      <td>['Marine Carpuat', 'Jordan Boyd-Graber Ying', ...</td>
      <td>[{'professor': 'Rachel Rudinger', 'course': 'C...</td>
      <td>CMSC</td>
      <td>470</td>
      <td>CMSC470</td>
      <td>Introduction to Natural Language Processing</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in CM...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>72</th>
      <td>3.483969</td>
      <td>['Ben Shneiderman', 'Zhicheng Liu', 'Zhicheng ...</td>
      <td>[{'professor': 'Zhicheng Liu', 'course': 'CMSC...</td>
      <td>CMSC</td>
      <td>734</td>
      <td>CMSC734</td>
      <td>Information  Visualization</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; CMSC434; or students who ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>74</th>
      <td>3.502857</td>
      <td>['Robert Maxwell', 'Jonathan Katz', 'Roger Eas...</td>
      <td>[{'professor': 'Roger Eastman', 'course': 'CMS...</td>
      <td>CMSC</td>
      <td>498N</td>
      <td>CMSC498N</td>
      <td>Selected Topics in Computer Science; Digital M...</td>
      <td>3.0</td>
      <td>&lt;i&gt;Shared with ARTT489M 0101\n&lt;b&gt;Prerequisite:...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>46</th>
      <td>3.739286</td>
      <td>['Zia Khan', 'Robert Patro', 'Robert Patro']</td>
      <td>[{'professor': 'Robert Patro', 'course': 'CMSC...</td>
      <td>CMSC</td>
      <td>858D</td>
      <td>CMSC858D</td>
      <td>Advanced Topics in Theory of Computing; Algori...</td>
      <td>3.0</td>
      <td>&lt;i&gt;\n&lt;b&gt;Restriction:&lt;/b&gt; Must be a graduate st...</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
cmsc_prof = []
cmsc_prof_df = pd.DataFrame()
for index, row in prof_df.iterrows():
    if "CMSC" in row['courses']:
      cmsc_prof.append(row)

cmsc_prof_df = pd.DataFrame(cmsc_prof, columns=prof_df.columns)
cmsc_prof_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>courses</th>
      <th>average_rating</th>
      <th>type</th>
      <th>reviews</th>
      <th>name</th>
      <th>slug</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>17</th>
      <td>['CMSC433', 'CMSC436', 'CMSC798', 'CMSC436', '...</td>
      <td>2.5000</td>
      <td>professor</td>
      <td>[{'professor': 'Adam Porter', 'course': 'CMSC4...</td>
      <td>Adam Porter</td>
      <td>porter_adam</td>
    </tr>
    <tr>
      <th>52</th>
      <td>['ENEE627', 'ENEE322H', 'ENEE620', 'ENEE626', ...</td>
      <td>2.8750</td>
      <td>professor</td>
      <td>[{'professor': 'Alexander Barg', 'course': 'EN...</td>
      <td>Alexander Barg</td>
      <td>barg</td>
    </tr>
    <tr>
      <th>17</th>
      <td>['CMSC424', 'CMSC724', 'CMSC498O', 'CMSC641', ...</td>
      <td>2.8889</td>
      <td>professor</td>
      <td>[{'professor': 'Amol Deshpande', 'course': 'CM...</td>
      <td>Amol Deshpande</td>
      <td>deshpande</td>
    </tr>
    <tr>
      <th>53</th>
      <td>['CMSC858K', 'CMSC451', 'CMSC858Q', 'CMSC798',...</td>
      <td>4.7500</td>
      <td>professor</td>
      <td>[{'professor': 'Andrew Childs', 'course': 'CMS...</td>
      <td>Andrew Childs</td>
      <td>childs</td>
    </tr>
    <tr>
      <th>8</th>
      <td>['ENAE380', 'CMSC106', 'ENAE380']</td>
      <td>2.7500</td>
      <td>professor</td>
      <td>[{'professor': 'Anthony Banes', 'course': 'ENA...</td>
      <td>Anthony Banes</td>
      <td>banes</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>84</th>
      <td>['CMSC100', 'INST101', 'CMSC100', 'INST101']</td>
      <td>4.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Charlotte Avery', 'course': 'C...</td>
      <td>Charlotte Avery</td>
      <td>avery_charlotte</td>
    </tr>
    <tr>
      <th>91</th>
      <td>['CMSC411', 'CMSC818J']</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Bahar Asgari', 'course': 'CMSC...</td>
      <td>Bahar Asgari</td>
      <td>asgari_bahar</td>
    </tr>
    <tr>
      <th>5</th>
      <td>['CMSC216', 'CMSC351', 'CMSC351', 'CMSC436']</td>
      <td>4.3333</td>
      <td>professor</td>
      <td>[{'professor': 'Herve Franceschi', 'course': '...</td>
      <td>Herve Franceschi</td>
      <td>franceschi_herve</td>
    </tr>
    <tr>
      <th>7</th>
      <td>['CMSC250', 'CMSC250']</td>
      <td>3.6667</td>
      <td>professor</td>
      <td>[{'professor': 'Paul Kline', 'course': 'CMSC25...</td>
      <td>Paul Kline</td>
      <td>kline_paul</td>
    </tr>
    <tr>
      <th>14</th>
      <td>['CMSC451', 'CMSC858N']</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Laxman Dhulipala', 'course': '...</td>
      <td>Laxman Dhulipala</td>
      <td>dhulipala_laxman</td>
    </tr>
  </tbody>
</table>
<p>158 rows × 6 columns</p>
</div>




```python
cmsc_grades = []
cmsc_grades_df = pd.DataFrame()
for g, row in grades_df.iterrows():
    if row['course'].startswith('CMSC'):
      cmsc_grades.append(row)

cmsc_grades_df = pd.DataFrame(cmsc_grades, columns=grades_df.columns)
cmsc_grades_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>course</th>
      <th>professor</th>
      <th>semester</th>
      <th>section</th>
      <th>A+</th>
      <th>A</th>
      <th>A-</th>
      <th>B+</th>
      <th>B</th>
      <th>B-</th>
      <th>C+</th>
      <th>C</th>
      <th>C-</th>
      <th>D+</th>
      <th>D</th>
      <th>D-</th>
      <th>F</th>
      <th>W</th>
      <th>Other</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CMSC433</td>
      <td>Adam Porter</td>
      <td>201201</td>
      <td>0201</td>
      <td>0</td>
      <td>10</td>
      <td>0</td>
      <td>0</td>
      <td>17</td>
      <td>1</td>
      <td>0</td>
      <td>11</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CMSC436</td>
      <td>Adam Porter</td>
      <td>201308</td>
      <td>0101</td>
      <td>0</td>
      <td>10</td>
      <td>7</td>
      <td>8</td>
      <td>17</td>
      <td>6</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>CMSC433</td>
      <td>Adam Porter</td>
      <td>201401</td>
      <td>0201</td>
      <td>0</td>
      <td>23</td>
      <td>0</td>
      <td>0</td>
      <td>12</td>
      <td>0</td>
      <td>0</td>
      <td>11</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CMSC436</td>
      <td>Adam Porter</td>
      <td>201408</td>
      <td>0101</td>
      <td>0</td>
      <td>20</td>
      <td>0</td>
      <td>0</td>
      <td>21</td>
      <td>0</td>
      <td>0</td>
      <td>7</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>CMSC433</td>
      <td>Adam Porter</td>
      <td>201501</td>
      <td>0101</td>
      <td>0</td>
      <td>17</td>
      <td>0</td>
      <td>0</td>
      <td>15</td>
      <td>0</td>
      <td>0</td>
      <td>8</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>4</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>0</th>
      <td>CMSC657</td>
      <td>Daniel Gottesman</td>
      <td>202108</td>
      <td>0101</td>
      <td>4</td>
      <td>15</td>
      <td>3</td>
      <td>2</td>
      <td>7</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CMSC433</td>
      <td>Liyi Li</td>
      <td>202201</td>
      <td>0101</td>
      <td>23</td>
      <td>44</td>
      <td>11</td>
      <td>15</td>
      <td>17</td>
      <td>19</td>
      <td>9</td>
      <td>19</td>
      <td>12</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>6</td>
      <td>15</td>
      <td>2</td>
    </tr>
    <tr>
      <th>0</th>
      <td>CMSC351</td>
      <td>Erika Melder</td>
      <td>202201</td>
      <td>0101</td>
      <td>4</td>
      <td>19</td>
      <td>15</td>
      <td>17</td>
      <td>72</td>
      <td>31</td>
      <td>17</td>
      <td>59</td>
      <td>19</td>
      <td>2</td>
      <td>10</td>
      <td>2</td>
      <td>8</td>
      <td>6</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CMSC351</td>
      <td>Erika Melder</td>
      <td>202201</td>
      <td>0201</td>
      <td>3</td>
      <td>14</td>
      <td>5</td>
      <td>6</td>
      <td>41</td>
      <td>14</td>
      <td>12</td>
      <td>33</td>
      <td>4</td>
      <td>2</td>
      <td>4</td>
      <td>0</td>
      <td>1</td>
      <td>7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>0</th>
      <td>CMSC100</td>
      <td>Charlotte Avery</td>
      <td>202201</td>
      <td>0201</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>2562 rows × 19 columns</p>
</div>




```python
index = np.asarray([i for i in range(1, 6)])

plt.hist(cmsc_prof_df["average_rating"], bins=4)
plt.xticks(index)
plt.title("Average Rating vs Count Distribution of CMSC courses")
plt.xlabel("Average Rating")
plt.ylabel("Count")

plt.show()
```


    
![png](output_24_0.png)
    


The CMSC average rating distribution appears to be slightly different than the overall distribution. Noticeably, there are a relatively higher number of 2-3 and 3-4 ratings.


```python
grade_counts = {grade: cmsc_grades_df[grade].sum() for grade in ['A+', 'A', 'A-', 'B+', 'B', 'B-', 'C+', 'C', 'C-', 'D+', 'D', 'D-', 'F', 'W', 'Other']}

x = list(grade_counts.keys())
y = list(grade_counts.values())

plt.bar(x, y)
plt.title("Grades vs Count Distribution for CMSC courses")
plt.xlabel("Grades")
plt.ylabel("Count")

plt.show()
```


    
![png](output_26_0.png)
    


The CMSC grade distribution also follows a relatively similar shape, but with a relatively higher amounts of the B-D grades, shown by its wider tails.


```python
index = np.asarray([i for i in range(1, 5)])

plt.hist(cmsc_courses_df["average_gpa"], bins=12)
plt.xticks(index)
plt.title("Averge GPA vs Count Distribution for CMSC courses")
plt.xlabel("Average GPA")
plt.ylabel("Count")

plt.show()
```


    
![png](output_28_0.png)
    


The CMSC average GPA distribution appears noticeably left shifted compared to the original all courses average GPA distribution, with much wider variance. Now we will plot distributions of average rating, average GPA and letter grades but this time only for entries in the KNES department.


```python
knes_courses_df = all_courses_df
knes_courses_df = knes_courses_df.loc[knes_courses_df["department"] == "KNES"]
knes_courses_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>average_gpa</th>
      <th>professors</th>
      <th>reviews</th>
      <th>department</th>
      <th>course_number</th>
      <th>name</th>
      <th>title</th>
      <th>credits</th>
      <th>description</th>
      <th>is_recent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>18</th>
      <td>3.262241</td>
      <td>['James Hagberg', 'Jane Clark', 'Eva Chin', 'J...</td>
      <td>[{'professor': 'Stephen McDaniel', 'course': '...</td>
      <td>KNES</td>
      <td>497</td>
      <td>KNES497</td>
      <td>Kinesiology Senior Seminar</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; A professional writing co...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>99</th>
      <td>3.604040</td>
      <td>['Andrea Romeo', 'Jo Zimmerman', 'Ana Palla-Ka...</td>
      <td>[{'professor': 'Andrew Ginsberg', 'course': 'K...</td>
      <td>KNES</td>
      <td>201</td>
      <td>KNES201</td>
      <td>Kinesiological Principles of Physical Activity</td>
      <td>1.0</td>
      <td>&lt;b&gt;Corequisite:&lt;/b&gt; Any physical activity cour...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>19</th>
      <td>3.808006</td>
      <td>['Laura Rush', 'Andrew Ginsberg', 'Kristi Tred...</td>
      <td>[{'professor': 'Anna Posbergh', 'course': 'KNE...</td>
      <td>KNES</td>
      <td>157N</td>
      <td>KNES157N</td>
      <td>Physical Education Activities: Coed; Weight Tr...</td>
      <td>1.0</td>
      <td>&lt;i&gt;Attendance is required on the first day of ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>52</th>
      <td>2.685705</td>
      <td>['Marc Rogers', 'Rosemary Lindle', 'Kathleen D...</td>
      <td>[{'professor': 'Marc Rogers', 'course': 'KNES3...</td>
      <td>KNES</td>
      <td>360</td>
      <td>KNES360</td>
      <td>Physiology of Exercise</td>
      <td>4.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in BS...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>77</th>
      <td>2.991701</td>
      <td>['Ben Hurley']</td>
      <td>[{'professor': 'Ben Hurley', 'course': 'KNES46...</td>
      <td>KNES</td>
      <td>461</td>
      <td>KNES461</td>
      <td>Exercise and Body Composition</td>
      <td>3.0</td>
      <td>An in-depth overview on how body composition i...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>75</th>
      <td>3.864691</td>
      <td>['Andrew Ginsberg', 'Ronald Mower', 'Michele F...</td>
      <td>[{'professor': 'Mike Hamberger', 'course': 'KN...</td>
      <td>KNES</td>
      <td>100O</td>
      <td>KNES100O</td>
      <td>Physical Education Activities: Coed; Basketbal...</td>
      <td>2.0</td>
      <td>&lt;i&gt;Attendance is required on the first day of ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>80</th>
      <td>2.711621</td>
      <td>['Michael Friedman', 'David Andrews', 'Adam Be...</td>
      <td>[{'professor': 'Michael Friedman', 'course': '...</td>
      <td>KNES</td>
      <td>287</td>
      <td>KNES287</td>
      <td>Sport and American Society</td>
      <td>3.0</td>
      <td>&lt;b&gt;Recommended:&lt;/b&gt; Minimum grade of C- in KNE...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>91</th>
      <td>3.725503</td>
      <td>['Dennis Vacante', 'David Maggiacomo']</td>
      <td>[{'professor': 'Dennis Vacante', 'course': 'KN...</td>
      <td>KNES</td>
      <td>389E</td>
      <td>KNES389E</td>
      <td>Topical Investigations; Childrens Development ...</td>
      <td>1.0</td>
      <td>&lt;i&gt;Students will be required to complete finge...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>21</th>
      <td>3.026405</td>
      <td>['Elizabeth Brown', 'J Carson Smith', 'Evan Br...</td>
      <td>[{'professor': 'Elizabeth Brown', 'course': 'K...</td>
      <td>KNES</td>
      <td>350</td>
      <td>KNES350</td>
      <td>The Psychology of Sports &amp; Exercise</td>
      <td>3.0</td>
      <td>An exploration of personality factors, includi...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>23</th>
      <td>3.701969</td>
      <td>['Elizabeth Brown', 'Jay Goldstein', 'Elizabet...</td>
      <td>[{'professor': 'Elizabeth Brown', 'course': 'K...</td>
      <td>KNES</td>
      <td>451</td>
      <td>KNES451</td>
      <td>Children and Sport: A Psychosocial Perspective</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in KN...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>85</th>
      <td>2.912519</td>
      <td>['Rodolphe Gentili', 'Jae Kun Shim', 'Ross Mil...</td>
      <td>[{'professor': 'Hossein Ehsani', 'course': 'KN...</td>
      <td>KNES</td>
      <td>300</td>
      <td>KNES300</td>
      <td>Biomechanics of Human Motion</td>
      <td>4.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in BS...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>28</th>
      <td>2.457373</td>
      <td>['Jennie Phillips', 'James Hagberg', 'Andrea R...</td>
      <td>[{'professor': 'Rosemary Lindle', 'course': 'K...</td>
      <td>KNES</td>
      <td>260</td>
      <td>KNES260</td>
      <td>Science of Physical Activity and Cardiovascula...</td>
      <td>3.0</td>
      <td>Course details (1) the public health importanc...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3.144200</td>
      <td>['Marvin Scott', 'Jay Goldstein', 'Jay Goldste...</td>
      <td>[{'professor': 'Marvin Scott', 'course': 'KNES...</td>
      <td>KNES</td>
      <td>457</td>
      <td>KNES457</td>
      <td>Managing Youth Programs: Educational, Fitness ...</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in KN...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>16</th>
      <td>3.820435</td>
      <td>['Jeff Maynor', 'Jeff Maynor', 'Jeff Maynor']</td>
      <td>[{'professor': 'Jeff Maynor', 'course': 'KNES2...</td>
      <td>KNES</td>
      <td>289L</td>
      <td>KNES289L</td>
      <td>Topical Investigations; Golf for Business and ...</td>
      <td>3.0</td>
      <td>&lt;i&gt;Students must pay a $50.00 Facility fee. Th...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>9</th>
      <td>3.598878</td>
      <td>['Kenneth Klotz', 'Kenneth Klotz', 'Kenneth Kl...</td>
      <td>[{'professor': 'Kenneth Klotz', 'course': 'KNE...</td>
      <td>KNES</td>
      <td>144Q</td>
      <td>KNES144Q</td>
      <td>Physical Education Activities: Coed; Martial A...</td>
      <td>2.0</td>
      <td>&lt;i&gt;Attendance is required on the first day of ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>10</th>
      <td>3.576775</td>
      <td>['Kenneth Klotz', 'Michael Dorothy', 'Michael ...</td>
      <td>[{'professor': 'Kenneth Klotz', 'course': 'KNE...</td>
      <td>KNES</td>
      <td>144R</td>
      <td>KNES144R</td>
      <td>Physical Education Activities: Coed; Martial A...</td>
      <td>2.0</td>
      <td>&lt;i&gt;Attendance is required on the first day of ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>11</th>
      <td>3.536119</td>
      <td>['Kenneth Klotz', 'Kenneth Klotz', 'Kenneth Kl...</td>
      <td>[{'professor': 'Kenneth Klotz', 'course': 'KNE...</td>
      <td>KNES</td>
      <td>144T</td>
      <td>KNES144T</td>
      <td>Physical Education Activities: Coed; Self-Defense</td>
      <td>2.0</td>
      <td>&lt;i&gt;Attendance is required on the first day of ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>6</th>
      <td>3.035401</td>
      <td>['Larry Plotkin']</td>
      <td>[{'professor': 'Larry Plotkin', 'course': 'KNE...</td>
      <td>KNES</td>
      <td>498T</td>
      <td>KNES498T</td>
      <td>Special Topics in Kinesiology; Principles and ...</td>
      <td>3.0</td>
      <td>&lt;i&gt;Prerequisites: KNES300/KNES300H and KNES360...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>91</th>
      <td>2.867470</td>
      <td>['Marvin Scott', 'Lindsey Winter']</td>
      <td>[{'professor': 'Marvin Scott', 'course': 'KNES...</td>
      <td>KNES</td>
      <td>386</td>
      <td>KNES386</td>
      <td>Experiential Learning</td>
      <td>3.0</td>
      <td>Explore and analyze concepts and procedures re...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>93</th>
      <td>2.756108</td>
      <td>['Michael Friedman', 'Samuel Clevenger', 'Dami...</td>
      <td>[{'professor': 'Michael Friedman', 'course': '...</td>
      <td>KNES</td>
      <td>293</td>
      <td>KNES293</td>
      <td>History of Sport in America</td>
      <td>3.0</td>
      <td>The growth and development of sport in America...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>94</th>
      <td>2.995556</td>
      <td>['Michael Friedman', 'Shaun Edmonds', 'Michael...</td>
      <td>[{'professor': 'Michael Friedman', 'course': '...</td>
      <td>KNES</td>
      <td>484</td>
      <td>KNES484</td>
      <td>Sporting Hollywood</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in KN...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>40</th>
      <td>2.575027</td>
      <td>['Marcio Oliveira', 'Rodolphe Gentili', 'Tim K...</td>
      <td>[{'professor': 'Rodolphe Gentili', 'course': '...</td>
      <td>KNES</td>
      <td>385</td>
      <td>KNES385</td>
      <td>Motor Control and Learning</td>
      <td>3.0</td>
      <td>&lt;b&gt;Restriction:&lt;/b&gt; Must be in a major within ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>55</th>
      <td>3.335480</td>
      <td>['Ross Miller', 'Tim Kiemel']</td>
      <td>[{'professor': 'Jennie Phillips', 'course': 'K...</td>
      <td>KNES</td>
      <td>289W</td>
      <td>KNES289W</td>
      <td>The Cybernetic Human</td>
      <td>3.0</td>
      <td>&lt;b&gt;Credit only granted for:&lt;/b&gt; KNES289W OR KN...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>72</th>
      <td>3.240186</td>
      <td>['Shannon Jette', 'Katelyn Esmonde', 'Anna Pos...</td>
      <td>[{'professor': 'Anna Posbergh', 'course': 'KNE...</td>
      <td>KNES</td>
      <td>400</td>
      <td>KNES400</td>
      <td>The Foundations of Public Health in Kinesiology</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in KN...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>44</th>
      <td>2.323661</td>
      <td>['Stephen McDaniel', 'Stephen McDaniel', 'Step...</td>
      <td>[{'professor': 'Stephen McDaniel', 'course': '...</td>
      <td>KNES</td>
      <td>355</td>
      <td>KNES355</td>
      <td>Sport Management</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in KN...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>32</th>
      <td>3.224306</td>
      <td>['Marcio Oliveira', 'Yue Du', 'Ana Palla-Kane'...</td>
      <td>[{'professor': 'Tim Kiemel', 'course': 'KNES37...</td>
      <td>KNES</td>
      <td>370</td>
      <td>KNES370</td>
      <td>Motor Development</td>
      <td>3.0</td>
      <td>&lt;b&gt;Restriction:&lt;/b&gt; Must be in a major within ...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>67</th>
      <td>3.742382</td>
      <td>['Anne Contee', 'Shilpa Reddy', 'Jesse Bresele...</td>
      <td>[{'professor': 'Shilpa Reddy', 'course': 'KNES...</td>
      <td>KNES</td>
      <td>161T</td>
      <td>KNES161T</td>
      <td>Physical Education Activities: Coed; Yoga</td>
      <td>1.0</td>
      <td>&lt;i&gt;Attendance on the first day of class is man...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>73</th>
      <td>3.440000</td>
      <td>['Rodolphe Gentili']</td>
      <td>[{'professor': 'Rodolphe Gentili', 'course': '...</td>
      <td>KNES</td>
      <td>462</td>
      <td>KNES462</td>
      <td>Neural Basis of Human Movement</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in BS...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2.706757</td>
      <td>['Marc Rogers']</td>
      <td>[{'professor': 'Marc Rogers', 'course': 'KNES4...</td>
      <td>KNES</td>
      <td>466</td>
      <td>KNES466</td>
      <td>Graded Exercise Testing</td>
      <td>3.0</td>
      <td>Functional and diagnostic examination of the c...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2.585388</td>
      <td>['Seppo Iso-Ahola', 'Seppo Iso-Ahola', 'Seppo ...</td>
      <td>[{'professor': 'Seppo Iso-Ahola', 'course': 'K...</td>
      <td>KNES</td>
      <td>442</td>
      <td>KNES442</td>
      <td>Psychology of Exercise and Health</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in KN...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>27</th>
      <td>3.563158</td>
      <td>['Jo Zimmerman', 'Andrew Ginsberg', 'Andrea Ro...</td>
      <td>[{'professor': 'Andrea Liberto', 'course': 'KN...</td>
      <td>KNES</td>
      <td>200</td>
      <td>KNES200</td>
      <td>Introduction to Kinesiology</td>
      <td>3.0</td>
      <td>&lt;b&gt;Restriction:&lt;/b&gt; Must be in Kinesiology pro...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>36</th>
      <td>3.204815</td>
      <td>['Samuel Clevenger', 'Damion Thomas', 'Ronald ...</td>
      <td>[{'professor': 'Samuel Clevenger', 'course': '...</td>
      <td>KNES</td>
      <td>289R</td>
      <td>KNES289R</td>
      <td>Hoop Dreams: Black Masculinity and Sport</td>
      <td>3.0</td>
      <td>&lt;b&gt;Credit only granted for:&lt;/b&gt; KNES289R OR KN...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>49</th>
      <td>3.373958</td>
      <td>['Jay Goldstein', 'Jay Goldstein', 'Jay Goldst...</td>
      <td>[{'professor': 'Jay Goldstein', 'course': 'KNE...</td>
      <td>KNES</td>
      <td>246</td>
      <td>KNES246</td>
      <td>Transformational Leader in Sport: The Art and ...</td>
      <td>3.0</td>
      <td>Highlights the expectations and ethical proble...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>56</th>
      <td>3.808247</td>
      <td>['Joanne Klossner', 'Larry Plotkin', 'Joanne K...</td>
      <td>[{'professor': 'Larry Plotkin', 'course': 'KNE...</td>
      <td>KNES</td>
      <td>405</td>
      <td>KNES405</td>
      <td>Principles &amp; Techniques of Manual Muscle Testing</td>
      <td>3.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in BS...</td>
      <td>True</td>
    </tr>
    <tr>
      <th>35</th>
      <td>3.225754</td>
      <td>['Kathleen Dondero', 'Steven Prior', 'Jeffrey ...</td>
      <td>[{'professor': 'Jeffrey Beans', 'course': 'KNE...</td>
      <td>KNES</td>
      <td>320</td>
      <td>KNES320</td>
      <td>Physiological Basis of Physical Activity and H...</td>
      <td>4.0</td>
      <td>&lt;b&gt;Prerequisite:&lt;/b&gt; Minimum grade of C- in BS...</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
knes_prof = []
knes_prof_df = pd.DataFrame()
for index, row in prof_df.iterrows():
    if "KNES" in row['courses']:
      knes_prof.append(row)

knes_prof_df = pd.DataFrame(knes_prof, columns=prof_df.columns)
knes_prof_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>courses</th>
      <th>average_rating</th>
      <th>type</th>
      <th>reviews</th>
      <th>name</th>
      <th>slug</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>57</th>
      <td>['KNES100O', 'KNES131O', 'KNES131V', 'KNES157O...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Andrew Ginsberg', 'course': 'K...</td>
      <td>Andrew Ginsberg</td>
      <td>ginsberg</td>
    </tr>
    <tr>
      <th>77</th>
      <td>['KNES461', 'KNES497', 'KNES461H']</td>
      <td>1.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Ben Hurley', 'course': 'KNES46...</td>
      <td>Ben Hurley</td>
      <td>hurley_ben</td>
    </tr>
    <tr>
      <th>87</th>
      <td>['KNES289Y', 'KNES287', 'KNES614', 'KNES485', ...</td>
      <td>4.4444</td>
      <td>professor</td>
      <td>[{'professor': 'David Andrews', 'course': 'KNE...</td>
      <td>David Andrews</td>
      <td>andrews_david</td>
    </tr>
    <tr>
      <th>68</th>
      <td>['KNES333', 'KNES389E', 'KNES689J']</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Dennis Vacante', 'course': 'KN...</td>
      <td>Dennis Vacante</td>
      <td>vacante</td>
    </tr>
    <tr>
      <th>69</th>
      <td>['KNES350', 'KNES451', 'KNES451H', 'KNES498', ...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Elizabeth Brown', 'course': 'K...</td>
      <td>Elizabeth Brown</td>
      <td>brown_elizabeth</td>
    </tr>
    <tr>
      <th>69</th>
      <td>['HLTH366', 'EPIB641', 'KNES689Y', 'SPHL611', ...</td>
      <td>1.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Evelyn King-Marshall', 'course...</td>
      <td>Evelyn King-Marshall</td>
      <td>king-marshall</td>
    </tr>
    <tr>
      <th>12</th>
      <td>['KNES497', 'KNES711', 'KNES260', 'KNES695', '...</td>
      <td>4.0000</td>
      <td>professor</td>
      <td>[{'professor': 'James Hagberg', 'course': 'KNE...</td>
      <td>James Hagberg</td>
      <td>hagberg</td>
    </tr>
    <tr>
      <th>61</th>
      <td>['KNES451', 'KNES457', 'KNES152N', 'KNES152O',...</td>
      <td>2.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Jay Goldstein', 'course': 'KNE...</td>
      <td>Jay Goldstein</td>
      <td>goldstein_jay</td>
    </tr>
    <tr>
      <th>72</th>
      <td>['KNES137N', 'KNES137O', 'KNES289L', 'KNES137N...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Jeff Maynor', 'course': 'KNES2...</td>
      <td>Jeff Maynor</td>
      <td>maynor</td>
    </tr>
    <tr>
      <th>2</th>
      <td>['KNES161V', 'KNES260', 'KNES497', 'KNES161R',...</td>
      <td>3.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Jennie Phillips', 'course': 'K...</td>
      <td>Jennie Phillips</td>
      <td>phillips_jennie</td>
    </tr>
    <tr>
      <th>59</th>
      <td>['KNES497', 'KNES157O', 'KNES332', 'KNES201', ...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Jo Zimmerman', 'course': 'KNES...</td>
      <td>Jo Zimmerman</td>
      <td>zimmerman_jo</td>
    </tr>
    <tr>
      <th>68</th>
      <td>['KNES497', 'KNES282', 'KNES689T', 'KNES405', ...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Joanne Klossner', 'course': 'K...</td>
      <td>Joanne Klossner</td>
      <td>klossner</td>
    </tr>
    <tr>
      <th>27</th>
      <td>['KNES144Q', 'KNES144R', 'KNES144T', 'KNES144Q...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Kenneth Klotz', 'course': 'KNE...</td>
      <td>Kenneth Klotz</td>
      <td>klotz</td>
    </tr>
    <tr>
      <th>8</th>
      <td>['KNES498T', 'KNES405', 'KNES305', 'KNES305H',...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Larry Plotkin', 'course': 'KNE...</td>
      <td>Larry Plotkin</td>
      <td>plotkin</td>
    </tr>
    <tr>
      <th>76</th>
      <td>['KNES360', 'KNES466', 'KNES498F', 'KNES460', ...</td>
      <td>2.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Marc Rogers', 'course': 'KNES3...</td>
      <td>Marc Rogers</td>
      <td>rogers_marc</td>
    </tr>
    <tr>
      <th>61</th>
      <td>['KNES240', 'KNES386', 'KNES457', 'KNES286']</td>
      <td>1.5000</td>
      <td>professor</td>
      <td>[{'professor': 'Marvin Scott', 'course': None,...</td>
      <td>Marvin Scott</td>
      <td>scott_marvin</td>
    </tr>
    <tr>
      <th>68</th>
      <td>['KNES287', 'KNES293', 'KNES484', 'KNES389C', ...</td>
      <td>2.2500</td>
      <td>professor</td>
      <td>[{'professor': 'Michael Friedman', 'course': '...</td>
      <td>Michael Friedman</td>
      <td>friedman_michael</td>
    </tr>
    <tr>
      <th>63</th>
      <td>['KNES300', 'KNES497', 'KNES462', 'KNES385', '...</td>
      <td>4.8000</td>
      <td>professor</td>
      <td>[{'professor': 'Rodolphe Gentili', 'course': '...</td>
      <td>Rodolphe Gentili</td>
      <td>gentili</td>
    </tr>
    <tr>
      <th>80</th>
      <td>['KNES210', 'SPHL498O', 'KNES360', 'KNES214', ...</td>
      <td>4.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Rosemary Lindle', 'course': 'K...</td>
      <td>Rosemary Lindle</td>
      <td>lindle</td>
    </tr>
    <tr>
      <th>22</th>
      <td>['KNES293', 'KNES289R', 'KNES497', 'PHSC497']</td>
      <td>2.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Samuel Clevenger', 'course': '...</td>
      <td>Samuel Clevenger</td>
      <td>clevenger</td>
    </tr>
    <tr>
      <th>96</th>
      <td>['KNES442', 'HONR258O', 'KNES440', 'KNES498Y',...</td>
      <td>4.3333</td>
      <td>professor</td>
      <td>[{'professor': 'Seppo Iso-Ahola', 'course': 'K...</td>
      <td>Seppo Iso-Ahola</td>
      <td>iso-ahola</td>
    </tr>
    <tr>
      <th>12</th>
      <td>['KNES498A', 'KNES600', 'KNES400', 'KNES497', ...</td>
      <td>4.7500</td>
      <td>professor</td>
      <td>[{'professor': 'Shannon Jette', 'course': 'KNE...</td>
      <td>Shannon Jette</td>
      <td>jette</td>
    </tr>
    <tr>
      <th>25</th>
      <td>['KNES287', 'KNES201', 'KNES157O', 'KNES157N',...</td>
      <td>4.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Shaun Edmonds', 'course': 'KNE...</td>
      <td>Shaun Edmonds</td>
      <td>edmonds_shaun</td>
    </tr>
    <tr>
      <th>98</th>
      <td>['KNES355', 'KNES483', 'KNES222', 'KNES355H', ...</td>
      <td>2.5000</td>
      <td>professor</td>
      <td>[{'professor': 'Stephen McDaniel', 'course': '...</td>
      <td>Stephen McDaniel</td>
      <td>mcdaniel</td>
    </tr>
    <tr>
      <th>51</th>
      <td>['KNES689W', 'KNES689D', 'KNES497', 'KNES385',...</td>
      <td>3.9000</td>
      <td>professor</td>
      <td>[{'professor': 'Tim Kiemel', 'course': 'KNES28...</td>
      <td>Tim Kiemel</td>
      <td>kiemel</td>
    </tr>
    <tr>
      <th>28</th>
      <td>['UNIV100', 'KNES201', 'SPHL240', 'KNES386']</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Lindsey Winter', 'course': 'UN...</td>
      <td>Lindsey Winter</td>
      <td>winter_lindsey</td>
    </tr>
    <tr>
      <th>91</th>
      <td>['KNES160N', 'KNES160O', 'KNES157N', 'KNES157O...</td>
      <td>2.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Steven Kahl', 'course': 'KNES2...</td>
      <td>Steven Kahl</td>
      <td>kahl</td>
    </tr>
    <tr>
      <th>51</th>
      <td>['KNES161O', 'KNES157O', 'KNES293', 'SOCY100',...</td>
      <td>3.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Eric Stone', 'course': 'KNES28...</td>
      <td>Eric Stone</td>
      <td>stone_eric</td>
    </tr>
    <tr>
      <th>79</th>
      <td>['KNES100O', 'KNES131O', 'KNES131V', 'KNES157N']</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Mike Hamberger', 'course': 'KN...</td>
      <td>Mike Hamberger</td>
      <td>hamberger</td>
    </tr>
    <tr>
      <th>42</th>
      <td>['KNES464', 'KNES464H', 'KNES691', 'KNES497', ...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Sarah Glancy', 'course': 'KNES...</td>
      <td>Sarah Glancy</td>
      <td>glancy</td>
    </tr>
    <tr>
      <th>83</th>
      <td>['KNES692', 'KNES465', 'KNES465H', 'KNES360', ...</td>
      <td>1.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Sushant Ranadive', 'course': N...</td>
      <td>Sushant Ranadive</td>
      <td>ranadive</td>
    </tr>
    <tr>
      <th>89</th>
      <td>['KNES161T', 'EXST030', 'SOCY100']</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Shilpa Reddy', 'course': 'KNES...</td>
      <td>Shilpa Reddy</td>
      <td>reddy</td>
    </tr>
    <tr>
      <th>65</th>
      <td>['KNES293', 'KNES225', 'KNES285', 'KNES225', '...</td>
      <td>4.8000</td>
      <td>professor</td>
      <td>[{'professor': 'Brandon Wallace', 'course': 'K...</td>
      <td>Brandon Wallace</td>
      <td>wallace_brandon</td>
    </tr>
    <tr>
      <th>36</th>
      <td>['KNES100O', 'KNES131V', 'KNES157O', 'KNES289R...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Ronald Mower', 'course': 'KNES...</td>
      <td>Ronald Mower</td>
      <td>mower</td>
    </tr>
    <tr>
      <th>87</th>
      <td>['KNES293', 'KNES689Q', 'KNES498K', 'KNES289R']</td>
      <td>4.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Damion Thomas', 'course': 'KNE...</td>
      <td>Damion Thomas</td>
      <td>thomas_damion</td>
    </tr>
    <tr>
      <th>77</th>
      <td>['KNES131V', 'KNES157N', 'KNES201', 'KNES400',...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Anna Posbergh', 'course': 'KNE...</td>
      <td>Anna Posbergh</td>
      <td>posbergh</td>
    </tr>
    <tr>
      <th>23</th>
      <td>['KNES320', 'SPHL399', 'KNES320', 'SPHL399', '...</td>
      <td>1.2000</td>
      <td>professor</td>
      <td>[{'professor': 'Jeffrey Beans', 'course': 'KNE...</td>
      <td>Jeffrey Beans</td>
      <td>beans</td>
    </tr>
    <tr>
      <th>65</th>
      <td>['KNES161T', 'KNES161T', 'KNES161T', 'KNES161T...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Kate Shin', 'course': 'KNES161...</td>
      <td>Kate Shin</td>
      <td>shin_kate</td>
    </tr>
    <tr>
      <th>63</th>
      <td>['KNES300', 'KNES300H']</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Hossein Ehsani', 'course': 'KN...</td>
      <td>Hossein Ehsani</td>
      <td>ehsani</td>
    </tr>
    <tr>
      <th>14</th>
      <td>['KNES131V', 'KNES201', 'KNES786', 'KNES157O',...</td>
      <td>5.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Andrea Liberto', 'course': 'KN...</td>
      <td>Andrea Liberto</td>
      <td>liberto_andrea</td>
    </tr>
    <tr>
      <th>1</th>
      <td>['KNES285']</td>
      <td>1.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Tori Justin', 'course': 'KNES2...</td>
      <td>Tori Justin</td>
      <td>justin_tori</td>
    </tr>
    <tr>
      <th>2</th>
      <td>['KNES306', 'KNES306']</td>
      <td>1.0000</td>
      <td>professor</td>
      <td>[{'professor': 'Ian Fothergill', 'course': 'KN...</td>
      <td>Ian Fothergill</td>
      <td>fothergill_ian</td>
    </tr>
  </tbody>
</table>
</div>




```python
knes_grades = []
knes_grades_df = pd.DataFrame()
for g, row in grades_df.iterrows():
    if row['course'].startswith('KNES'):
      knes_grades.append(row)

knes_grades_df = pd.DataFrame(knes_grades, columns=grades_df.columns)
knes_grades_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>course</th>
      <th>professor</th>
      <th>semester</th>
      <th>section</th>
      <th>A+</th>
      <th>A</th>
      <th>A-</th>
      <th>B+</th>
      <th>B</th>
      <th>B-</th>
      <th>C+</th>
      <th>C</th>
      <th>C-</th>
      <th>D+</th>
      <th>D</th>
      <th>D-</th>
      <th>F</th>
      <th>W</th>
      <th>Other</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>KNES100O</td>
      <td>Andrew Ginsberg</td>
      <td>201201</td>
      <td>0001</td>
      <td>0</td>
      <td>40</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>KNES100O</td>
      <td>Andrew Ginsberg</td>
      <td>201201</td>
      <td>0002</td>
      <td>0</td>
      <td>38</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>KNES131O</td>
      <td>Andrew Ginsberg</td>
      <td>201201</td>
      <td>0001</td>
      <td>0</td>
      <td>40</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>KNES131V</td>
      <td>Andrew Ginsberg</td>
      <td>201201</td>
      <td>0002</td>
      <td>0</td>
      <td>60</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>KNES157O</td>
      <td>Andrew Ginsberg</td>
      <td>201201</td>
      <td>0002</td>
      <td>0</td>
      <td>31</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>22</th>
      <td>KNES131V</td>
      <td>Andrea Liberto</td>
      <td>202201</td>
      <td>0101</td>
      <td>0</td>
      <td>27</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>23</th>
      <td>KNES131V</td>
      <td>Andrea Liberto</td>
      <td>202201</td>
      <td>0102</td>
      <td>0</td>
      <td>24</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>24</th>
      <td>KNES131V</td>
      <td>Andrea Liberto</td>
      <td>202201</td>
      <td>0103</td>
      <td>0</td>
      <td>23</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>25</th>
      <td>KNES200</td>
      <td>Andrea Liberto</td>
      <td>202201</td>
      <td>0101</td>
      <td>0</td>
      <td>62</td>
      <td>0</td>
      <td>0</td>
      <td>16</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>26</th>
      <td>KNES201</td>
      <td>Andrea Liberto</td>
      <td>202201</td>
      <td>0101</td>
      <td>0</td>
      <td>11</td>
      <td>0</td>
      <td>0</td>
      <td>11</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>1674 rows × 19 columns</p>
</div>




```python
index = np.asarray([i for i in range(1, 6)])

plt.hist(knes_prof_df["average_rating"], bins=4)
plt.xticks(index)
plt.title("Average Rating vs Count Distribution of KNES courses")
plt.xlabel("Average Rating")
plt.ylabel("Count")

plt.show()
```


    
![png](output_33_0.png)
    


The KNES average rating is quite interesting in that it follows a different shape than its overall and CMSC counterparts. 4-5 average rating is most common, followed by 1-2, 2-3, and then 3-4. This could potentially be the result of a lot of specifically 1 star reviews, shifting the distribution for the 1-2, 2-3 and 3-4 buckets.


```python
grade_counts = {grade: knes_grades_df[grade].sum() for grade in ['A+', 'A', 'A-', 'B+', 'B', 'B-', 'C+', 'C', 'C-', 'D+', 'D', 'D-', 'F', 'W', 'Other']}

x = list(grade_counts.keys())
y = list(grade_counts.values())

plt.bar(x, y)
plt.title("Grades vs Count Distribution for KNES courses")
plt.xlabel("Grades")
plt.ylabel("Count")

plt.show()
```


    
![png](output_35_0.png)
    


Once again, we get a similar shape for grade distribution for KNES courses (compared to overall and CMSC). If anything, we have more As and our tails are longer and flatter.


```python
index = np.asarray([i for i in range(1, 5)])

plt.hist(knes_courses_df["average_gpa"], bins=12)
plt.xticks(index)
plt.title("Averge GPA vs Count Distribution for KNES courses")
plt.xlabel("Average GPA")
plt.ylabel("Count")

plt.show()
```


    
![png](output_37_0.png)
    


We notice here that the distribution of average GPAs for KNES is much more in line with the distribution for all courses, but the variance is still noticeably larger.


```python
def calcAvgGPA(prof, df):
  sum = 0
  count = 0
  for i, row in df.iterrows():
    if row['professor'] == prof:
      sum = sum + 4*row['A+'] + 4*row['A'] + 3.7*row['A-'] + 3.3*row['B+'] + 3*row['B'] + 2.7*row['B-'] + 2.3*row['C+'] + 2*row['C'] + 1.7*row['C-'] + 1.3*row['D+'] + 1*row['D'] + .7*row['D-'] + 0*row['F']
      count = count + row['A+'] + row['A'] + row['A-'] + row['B+'] + row['B'] + row['B-'] + row['C+'] + row['C'] + row['C-'] + row['D+'] + row['D'] + row['D-'] + row['F']
  if count == 0:
    return sum
  else:
    return sum / count
```

Later, you will see that we want to graph a scatter plot of average grades compared to the average reviews for each professor in the CMSC and KNES departments. This function will help get the average GPA for each professor entered into its parameter.


```python
scat_cmsc_df = pd.DataFrame()

scat_cmsc_df['name'] = cmsc_prof_df['name']
scat_cmsc_df['average_rating'] = cmsc_prof_df['average_rating']
scat_cmsc_df['total_average_gpa'] = 0

for i, row in scat_cmsc_df.iterrows():
  scat_cmsc_df.at[i, 'total_average_gpa'] = calcAvgGPA(row['name'], cmsc_grades_df)


scat_cmsc_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>average_rating</th>
      <th>total_average_gpa</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>17</th>
      <td>Adam Porter</td>
      <td>2.5000</td>
      <td>3.034218</td>
    </tr>
    <tr>
      <th>52</th>
      <td>Alexander Barg</td>
      <td>2.8750</td>
      <td>3.566851</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Amol Deshpande</td>
      <td>2.8889</td>
      <td>3.034218</td>
    </tr>
    <tr>
      <th>53</th>
      <td>Andrew Childs</td>
      <td>4.7500</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Anthony Banes</td>
      <td>2.7500</td>
      <td>3.176316</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>84</th>
      <td>Charlotte Avery</td>
      <td>4.0000</td>
      <td>3.461538</td>
    </tr>
    <tr>
      <th>91</th>
      <td>Bahar Asgari</td>
      <td>5.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Herve Franceschi</td>
      <td>4.3333</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Paul Kline</td>
      <td>3.6667</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Laxman Dhulipala</td>
      <td>5.0000</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
<p>158 rows × 3 columns</p>
</div>



Here, we are creating a new dataframe to store all of the data we will need to graph our scatterplot by using existing data from previous dataframes and calculating the average GPAs for each professor using the prior function created.


```python
#remove 0 avg_gpa to prevent a wrong liner regression line
scat_cmsc_df = scat_cmsc_df.loc[scat_cmsc_df["total_average_gpa"] != 0]

scat_cmsc_df.plot.scatter(x='total_average_gpa', y='average_rating')

x = scat_cmsc_df['total_average_gpa']
y = scat_cmsc_df['average_rating']

# linear regression through scatterplot
m, b = np.polyfit(x, y, 1)
print("Linear model equation: " + str(m) + "x + " + str(b))

correlation = np.corrcoef(x, y)[0, 1]
print("Pearson correlation coefficient: " + str(correlation))

# graph linear model
coef = np.polyfit(x, y, 1)
poly1d_fn = np.poly1d(coef)
plt.plot(x, poly1d_fn(x), color='red')
plt.title('CMSC average GPA vs average rating per professor')
```

    Linear model equation: -0.11501074131718242x + 3.8904571872568803
    Pearson correlation coefficient: -0.04040787989242778





    Text(0.5, 1.0, 'CMSC average GPA vs average rating per professor')




    
![png](output_43_2.png)
    


Graphed into a scatterplot, the data from the scat_cmsc_df dataframe is displayed. Originally, the scatterplot would show many points in the 0 GPA area with ratings, so we decided to remove this to prevent an inaccurate linear regression model. The reason for many points in the 0 region for average GPA is because the grades_df contains rows of grade data where no grade was earned, resulting in a 0 GPA. This occurs for a few professors. However, after removal, we can see that there are many professors with high average GPA and high average ratings and low points where it's a high average GPA with a low rating. The data points seen on the scatterplot was then used to create a linear regression line as seen in red. From intuitive observation, the model seems underfit. This means this is not a good predictor for average rating or average GPA. This is further supported by the pearson correlation coefficient being very close to 0, indicating no relationship between the variables here. From the slope of the linear regression line, we can see that having a higher average GPA is not representative of having higher average ratings.


```python
scat_knes_df = pd.DataFrame()

scat_knes_df['name'] = knes_prof_df['name']
scat_knes_df['average_rating'] = knes_prof_df['average_rating']
scat_knes_df['total_average_gpa'] = 0

for i, row in scat_knes_df.iterrows():
  scat_knes_df.at[i, 'total_average_gpa'] = calcAvgGPA(row['name'], knes_grades_df)


scat_knes_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>average_rating</th>
      <th>total_average_gpa</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>57</th>
      <td>Andrew Ginsberg</td>
      <td>5.0000</td>
      <td>3.934829</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Ben Hurley</td>
      <td>1.0000</td>
      <td>3.564583</td>
    </tr>
    <tr>
      <th>87</th>
      <td>David Andrews</td>
      <td>4.4444</td>
      <td>3.322287</td>
    </tr>
    <tr>
      <th>68</th>
      <td>Dennis Vacante</td>
      <td>5.0000</td>
      <td>2.834697</td>
    </tr>
    <tr>
      <th>69</th>
      <td>Elizabeth Brown</td>
      <td>5.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>69</th>
      <td>Evelyn King-Marshall</td>
      <td>1.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>James Hagberg</td>
      <td>4.0000</td>
      <td>3.201448</td>
    </tr>
    <tr>
      <th>61</th>
      <td>Jay Goldstein</td>
      <td>2.0000</td>
      <td>3.001803</td>
    </tr>
    <tr>
      <th>72</th>
      <td>Jeff Maynor</td>
      <td>5.0000</td>
      <td>3.934766</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jennie Phillips</td>
      <td>3.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>59</th>
      <td>Jo Zimmerman</td>
      <td>5.0000</td>
      <td>3.758888</td>
    </tr>
    <tr>
      <th>68</th>
      <td>Joanne Klossner</td>
      <td>5.0000</td>
      <td>2.834697</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Kenneth Klotz</td>
      <td>5.0000</td>
      <td>3.770343</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Larry Plotkin</td>
      <td>5.0000</td>
      <td>3.208092</td>
    </tr>
    <tr>
      <th>76</th>
      <td>Marc Rogers</td>
      <td>2.0000</td>
      <td>2.677517</td>
    </tr>
    <tr>
      <th>61</th>
      <td>Marvin Scott</td>
      <td>1.5000</td>
      <td>3.001803</td>
    </tr>
    <tr>
      <th>68</th>
      <td>Michael Friedman</td>
      <td>2.2500</td>
      <td>2.834697</td>
    </tr>
    <tr>
      <th>63</th>
      <td>Rodolphe Gentili</td>
      <td>4.8000</td>
      <td>3.514286</td>
    </tr>
    <tr>
      <th>80</th>
      <td>Rosemary Lindle</td>
      <td>4.0000</td>
      <td>3.190791</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Samuel Clevenger</td>
      <td>2.0000</td>
      <td>2.884536</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Seppo Iso-Ahola</td>
      <td>4.3333</td>
      <td>2.858613</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Shannon Jette</td>
      <td>4.7500</td>
      <td>3.201448</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Shaun Edmonds</td>
      <td>4.0000</td>
      <td>3.205428</td>
    </tr>
    <tr>
      <th>98</th>
      <td>Stephen McDaniel</td>
      <td>2.5000</td>
      <td>2.808143</td>
    </tr>
    <tr>
      <th>51</th>
      <td>Tim Kiemel</td>
      <td>3.9000</td>
      <td>3.212644</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Lindsey Winter</td>
      <td>5.0000</td>
      <td>3.627273</td>
    </tr>
    <tr>
      <th>91</th>
      <td>Steven Kahl</td>
      <td>2.0000</td>
      <td>3.781377</td>
    </tr>
    <tr>
      <th>51</th>
      <td>Eric Stone</td>
      <td>3.0000</td>
      <td>3.212644</td>
    </tr>
    <tr>
      <th>79</th>
      <td>Mike Hamberger</td>
      <td>5.0000</td>
      <td>3.927143</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Sarah Glancy</td>
      <td>5.0000</td>
      <td>3.186353</td>
    </tr>
    <tr>
      <th>83</th>
      <td>Sushant Ranadive</td>
      <td>1.0000</td>
      <td>3.147117</td>
    </tr>
    <tr>
      <th>89</th>
      <td>Shilpa Reddy</td>
      <td>5.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>65</th>
      <td>Brandon Wallace</td>
      <td>4.8000</td>
      <td>3.930233</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Ronald Mower</td>
      <td>5.0000</td>
      <td>3.302688</td>
    </tr>
    <tr>
      <th>87</th>
      <td>Damion Thomas</td>
      <td>4.0000</td>
      <td>3.322287</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Anna Posbergh</td>
      <td>5.0000</td>
      <td>3.564583</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Jeffrey Beans</td>
      <td>1.2000</td>
      <td>3.386224</td>
    </tr>
    <tr>
      <th>65</th>
      <td>Kate Shin</td>
      <td>5.0000</td>
      <td>3.930233</td>
    </tr>
    <tr>
      <th>63</th>
      <td>Hossein Ehsani</td>
      <td>5.0000</td>
      <td>3.514286</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Andrea Liberto</td>
      <td>5.0000</td>
      <td>3.770510</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Tori Justin</td>
      <td>1.0000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ian Fothergill</td>
      <td>1.0000</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>



Like scat_cmsc_df, we are once again creating a new dataframe to store all of the data we will need to graph our scatterplot for KNES professors by using existing data and calculating the average GPAs for each professor in KNES using the prior function calcAvgGPA.


```python
# remove 0 avg_gpa to prevent a wrong linear regression line
scat_knes_df = scat_knes_df.loc[scat_knes_df["total_average_gpa"] != 0]

scat_knes_df.plot.scatter(x='total_average_gpa', y='average_rating')

x = scat_knes_df['total_average_gpa']
y = scat_knes_df['average_rating']

# linear regression through scatterplot
m, b = np.polyfit(x, y, 1)
print("Linear model equation: " + str(m) + "x + " + str(b))

correlation = np.corrcoef(x, y)[0, 1]
print("Pearson correlation coefficient: " + str(correlation))

# graph linear model
coef = np.polyfit(x, y, 1)
poly1d_fn = np.poly1d(coef)
plt.plot(x, poly1d_fn(x), color='red')
plt.title('KNES average GPA vs average rating per professor')
```

    Linear model equation: 1.5404094807111284x + -1.303469194241732
    Pearson correlation coefficient: 0.4129817063211394





    Text(0.5, 1.0, 'KNES average GPA vs average rating per professor')




    
![png](output_47_2.png)
    


This scatterplot also shows the comparison of total average GPA to the average rating. There are much fewer data points than compared to the CMSC scatter plot, however the points look more distributed, possibly an upward trend prior to the linear regression line. However, once we graphed the linear regression line, we were able to determine that there was a more positive relationship between the total average GPA and the average reviews. From our observation, the model is also underfit despite the positive relationship as observed. However, we still cannot conclude anything definitively, especially given the pearson correlation coefficient's closeness to 0.

## Application
An application that might prove to be potentially useful is coming up with some sort of ranking for professors based on their average rating that better reflects reality. Currently, the metric of average_rating is not necessarily the most accurate in determining how popular a professor is with his/her students because a single 5 rating will automatically put that professor at the top. Is there a way to better capture this percieved popularity? A possible solution is to use the Bayesian average of the average rating, which takes into account the amount of reviews that that professor recieved. Let's try it out on CS professors and see if it matches our expectations as a CS student.


```python
cmsc_prof_df["review_count"] = cmsc_prof_df["reviews"].apply(lambda x: x.count('{'))
avg = cmsc_prof_df["average_rating"].median()
C = np.quantile(cmsc_prof_df["review_count"], 0.25)

cmsc_prof_df["bayesian_avg"] = (cmsc_prof_df["average_rating"] * cmsc_prof_df["review_count"] + C * avg) / (cmsc_prof_df["review_count"] + C)
cmsc_prof_df.sort_values(by=["bayesian_avg"], ascending=False).head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>courses</th>
      <th>average_rating</th>
      <th>type</th>
      <th>reviews</th>
      <th>name</th>
      <th>slug</th>
      <th>review_count</th>
      <th>bayesian_avg</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12</th>
      <td>['MATH206', 'MATH241', 'MATH410', 'MATH695', '...</td>
      <td>4.8579</td>
      <td>professor</td>
      <td>[{'professor': 'Justin Wyss-Gallifent', 'cours...</td>
      <td>Justin Wyss-Gallifent</td>
      <td>wyss-gallifent</td>
      <td>183</td>
      <td>4.845022</td>
    </tr>
    <tr>
      <th>30</th>
      <td>['AMSC460', 'CMSC460', 'MATH140', 'MATH410', '...</td>
      <td>4.9032</td>
      <td>professor</td>
      <td>[{'professor': 'Stefan Doboszczak', 'course': ...</td>
      <td>Stefan Doboszczak</td>
      <td>doboszczak</td>
      <td>31</td>
      <td>4.828261</td>
    </tr>
    <tr>
      <th>39</th>
      <td>['MATH456', 'MATH620', 'MATH406', 'MATH808N', ...</td>
      <td>4.8710</td>
      <td>professor</td>
      <td>[{'professor': 'Lawrence Washington', 'course'...</td>
      <td>Lawrence Washington</td>
      <td>washington_lawrence</td>
      <td>31</td>
      <td>4.798012</td>
    </tr>
    <tr>
      <th>89</th>
      <td>['MATH141', 'MATH241H', 'MATH401', 'MATH241', ...</td>
      <td>4.8286</td>
      <td>professor</td>
      <td>[{'professor': 'Wiseley Wong', 'course': 'MATH...</td>
      <td>Wiseley Wong</td>
      <td>wong_wiseley</td>
      <td>35</td>
      <td>4.765795</td>
    </tr>
    <tr>
      <th>15</th>
      <td>['TLPL101', 'SPHL601', 'SPHL602', 'SPHL603', '...</td>
      <td>4.9167</td>
      <td>professor</td>
      <td>[{'professor': 'Elias Gonzalez', 'course': 'CM...</td>
      <td>Elias Gonzalez</td>
      <td>gonzalez_elias</td>
      <td>12</td>
      <td>4.738129</td>
    </tr>
  </tbody>
</table>
</div>



Here, we were able to find the 5 professors with the new best Bayesian average rating for average GPA, which we got as

1. Justin Wyss-Gallifent
2. Stefan Doboszczak
3. Lawrence Washington
4. Wiseley Wong
5. Elias Gonzalez

and by intuition, this list feels quite accurate based on percieved reputation here at the University of Maryland.

## Hypothesis Test
In this section of the tutorial, we aim to see whether there is a statistically significant different in the mean average GPA of all CMSC (Computer Science) and KNES (Kinesiology) courses. We want to accomplish this using a two sample t test, where we are essentially testing whether two population means are equal. In this case, we are using the unpaired variation of the test because we can not assume that the samples are correlated. In this situation, our null hypothesis $H0$ is that the population means are equal, i.e. $\mu{difference} = 0$. Our alternative hypothesis $HA$ then is that the population means are not equal, i.e. $\mu{difference} \neq 0$.

There are a few things to consider before we can actually run our two sample t test on the average GPAs in the computer science and kinesiology departments. First of all, we have to make sure the samples that we take are equal in size, and we find that there are 52 entries for computer science and 35 entries for kinesiology. Therefore, we cut off the last 18 entries in the kinesiology array to make their sizes equal.


```python
# make numpy arrays out of the avg gpa column for both
cmsc_avg_gpa = cmsc_courses_df['average_gpa'].values
knes_avg_gpa = knes_courses_df['average_gpa'].values

print("CMSC Courses count: " + str(len(cmsc_avg_gpa)) + " or " + "KNES Courses count: " + str(len(knes_avg_gpa)))

# cut off 18 entries to be the same sample size
index = [35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52]

cmsc_avg_gpa2 = np.delete(cmsc_avg_gpa, index)

print("CMSC Courses count: " + str(len(cmsc_avg_gpa2)) + " or " + "KNES Courses count: " + str(len(knes_avg_gpa)))
cmsc_avg_gpa2
```

    CMSC Courses count: 53 or KNES Courses count: 35
    CMSC Courses count: 35 or KNES Courses count: 35





    array([3.19966395, 2.67261646, 3.46136364, 2.44941702, 2.74684288,
           2.38011222, 2.58496257, 2.3868524 , 3.0144    , 2.74674179,
           2.49150456, 3.14888281, 2.97276786, 3.37327632, 3.06269841,
           2.94630137, 2.88377393, 3.22089235, 2.36666667, 3.69318681,
           2.88934678, 2.94040448, 2.81702712, 2.47474849, 2.36774194,
           2.44232042, 2.53544221, 3.65      , 3.44407407, 2.81368201,
           2.97995227, 3.48181818, 2.88418491, 3.32009569, 2.68901734])



Additionally, we check the variances of the computer sciences and kinesiology average GPAs to see if they are equal enough to run the test and this check passes for us empirically (even if this does not hold, setting equal_var=False yields a very similar p-value). Therefore, we run the two sample t test on both NumPy arays, which we can do through the SciPy library.


```python
print(np.var(cmsc_avg_gpa2), np.var(knes_avg_gpa))
stats.ttest_ind(a=cmsc_avg_gpa2, b=knes_avg_gpa, equal_var=True)
```

    0.14580867979024528 0.18786909196167234





    Ttest_indResult(statistic=-3.1433783582687727, pvalue=0.002475133785922839)



We find that we get a p-value of ~0.002475, which would reject the null hypothesis for all reasonable values of significance level $\alpha$ in favor of the alternative hypothesis. Therefore, we can conclude that the CMSC and KNES average GPA means are not equal, and that the CMSC and KNES students have significantly different average GPAs.

Expanding upon this, as we feel the need to incorporate other departments into our calculation, we would likely need an ANOVA test, which would test for whether any statistically significant differences exist between the means of three or more groups (which would be the different departments in our case). This can be followed up with Tukey's procedure to find the means that are significantly different from each other.

## Conclusion
This tutorial explores the different statistics of the courses offered at University of Maryland. By using the current data made available on PlanetTerp, our tutorial analyzes and manipulates data to provide a visual insight (graphs and plots) and more specific information in addition to what was provided. By using pandas dataframe and many other libraries we were able to create several more variables and models including seperate dataframes for computer science courses and kinesiology, courses that a professor taught and their average ratings, and average GPAs vs count distribution. We then extracted information on average GPAs from classes of these two courses and predicted an average overall GPA using the two sample t test. In the end, we concluded by calculating average ratings of professors and how that correlates to their final class GPA. By providing lots of useful information and data evaluation, this tutorial can help students choose their future classes and can benefit their performance in college.
