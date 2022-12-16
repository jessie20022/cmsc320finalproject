# The Impact of Course Department GPA on Average GPA

CMSC320 Final Project (Fall 2022)

by Tommy Chan, Alex Chen, Jessica Wu

## Introduction
[PlanetTerp](https://planetterp.com/) is an open-source student-run platform that allows students at the University of Maryland to publish reviews for their professors in the courses that they have previously took. The website aggregates all of this review data for any user to publicly access and search through. Some of this data includes professos, classes, reviews, and course grades/average GPAs correlated to those classes. The statistics from this website are obtained through the University of Maryland Office of Institutional Reasearch, Planning and Assessment.

Reviews have become a very critical part of any student's education, especially when trying to gauge whether a professor's teaching style is effective for the student. However, reviews might sometimes be more reflective of personal resentment rather than an objective view of whether a professor's class was fair and effective in communicating the course material. Therefore, through this tutorial we aim to see whether the average GPA in a course has an impact on the reviews that that professor recieves. We specifically want to observe 

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

Now let's import these libraries with the following code:

```
import requests
import pandas as pd
import time
import numpy as np
from matplotlib import pyplot as plt
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

```
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

all_courses_df

all_courses_df.to_csv('all_courses.csv')
```
and we did a very similar procedure for the professors endpoint. However, to get grades, we had to reference our previous professors dataframe to be able to get our grades dataframe since professor is a required parameter for this endpoint. Thus,

```
grades_df = pd.DataFrame()

for i in prof_df['name']:
  api_url = 'https://api.planetterp.com/v1'
  response = requests.get(api_url + "/grades?professor=" + i)
  data = response.json()
  temp = pd.DataFrame.from_dict(data)
  grades_df = grades_df.append(temp)
  time.sleep(0.1)

grades_df

grades_df.to_csv('grades_df.csv')
```
Now, let's import our csv files as Pandas dataframes starting with courses, dropping any rows with no average_gpa or reviews. Similarly, let's do the same for professors, dropping rows of professors with no courses, average_rating or reviews. Finally, we can also import our grades.
```
all_courses_df = pd.read_csv('all_courses.csv', na_filter=True, na_values='[]', index_col=0)
all_courses_df = all_courses_df.loc[all_courses_df["average_gpa"] == all_courses_df["average_gpa"]]
all_courses_df = all_courses_df.loc[all_courses_df["reviews"] == all_courses_df["reviews"]]

prof_df = pd.read_csv('professors_data.csv', na_filter=True, na_values='[]', index_col=0)
prof_df = prof_df.loc[prof_df["courses"] == prof_df["courses"]]
prof_df = prof_df.loc[prof_df["average_rating"] == prof_df["average_rating"]]
prof_df = prof_df.loc[prof_df["reviews"] == prof_df["reviews"]]

grades_df = pd.read_csv('grades_df.csv', na_filter=True, na_values='[]', index_col=0)
```

## Exploration

Now let's observe a few entries of the 3 dataframes we created, all_courses_df, prof_df and grades_df.

```
all_courses_df.head(5)
```

|   average_gpa  |   professors                                         |   reviews                                            |   department  |   course_number  |   name      |   title                                    |   credits  |   description                                        |   is_recent  |
|----------------|------------------------------------------------------|------------------------------------------------------|---------------|------------------|-------------|--------------------------------------------|------------|------------------------------------------------------|--------------|
|   3.407091     |   ['Qin Wang', 'Abani Pradhan', 'Solmaz Alborzi'...  |   [{'professor': 'Shraddha Karanth', 'course': '...  |   NFSC        |   112            |   NFSC112   |   Food: Science and Technology             |   3.0      |   Introduction to the realm of food science, foo...  |   TRUE       |
|   2.300000     |   ['Emily Perez', 'Laura Williams', 'Douglas Ker...  |   [{'professor': 'Zita Nunes', 'course': 'AASP29...  |   AASP        |   298L           |   AASP298L  |   African-American Literature and Culture  |   3.0      |   <b>Cross-listed with:</b> ENGL234.\n<b>Credit ...  |   TRUE       |
|   3.530045     |   ['Justin Lohr', 'Sarah Pleydell', 'Catherine B...  |   [{'professor': 'Justin Lohr', 'course': 'ENGL1...  |   ENGL        |   101H           |   ENGL101H  |   Academic Writing                         |   3.0      |   <b>Additional information:</b> Any student who...  |   TRUE       |
|   2.839834     |   ['Douglas Kern', 'Zita Nunes', 'Mary Washingto...  |   [{'professor': 'Zita Nunes', 'course': 'ENGL23...  |   ENGL        |   234            |   ENGL234   |   African-American Literature and Culture  |   3.0      |   <b>Cross-listed with:</b> AASP298L.\n<b>Credit...  |   TRUE       |
|   3.625573     |   ['Guangming Zhang', 'Peter Chung', 'Abhijit Da...  |   [{'professor': 'Guangming Zhang', 'course': 'E...  |   ENME        |   470            |   ENME470   |   Finite Element Analysis                  |   3.0      |   <b>Restriction:</b> Senior standing; and permi...  |   TRUE       |
  
```
prof_df.head(5)
```
  
|   courses                                            |   average_rating  |   type       |   reviews                                            |   name                 |   slug              |
|------------------------------------------------------|-------------------|--------------|------------------------------------------------------|------------------------|---------------------|
|   ['NFSC431', 'NFSC679R', 'NFSC112', 'HLTH672', ...  |   5.0000          |   professor  |   [{'professor': 'Abani Pradhan', 'course': None...  |   Abani Pradhan        |   pradhan           |
|   ['ENME674', 'ENMA300', 'ENME684', 'ENME489Z', ...  |   4.3333          |   professor  |   [{'professor': 'Abhijit Dasgupta', 'course': '...  |   Abhijit Dasgupta     |   dasgupta_abhijit  |
|   ['ARTH389L', 'ARTH255', 'ARTH768', 'ARTH668A',...  |   2.8333          |   professor  |   [{'professor': 'Abigail McEwen', 'course': Non...  |   Abigail McEwen       |   mcewen            |
|   ['PHYS405', 'PHYS275', 'PHYS758E', 'PHYS273', ...  |   4.3333          |   professor  |   [{'professor': 'Abolhassan Jawahery', 'course'...  |   Abolhassan Jawahery  |   jawahery          |
|   ['STAT701', 'STAT700', 'STAT750', 'STAT650', '...  |   2.7000          |   professor  |   [{'professor': 'Abram Kagan', 'course': 'STAT4...  |   Abram Kagan          |   kagan             |

  
```
grades_df.head(5)
```
 
|   course    |   professor      |   semester  |   section  |   A+  |   A   |   A-  |   B+  |   B  |   B-  |   C+  |   C  |   C-  |   D+  |   D  |   D-  |   F  |   W  |   Other  |
|-------------|------------------|-------------|------------|-------|-------|-------|-------|------|-------|-------|------|-------|-------|------|-------|------|------|----------|
|   NFSC431   |   Abani Pradhan  |   201201    |   101      |   0   |   10  |   0   |   0   |   3  |   0   |   0   |   0  |   0   |   0   |   0  |   0   |   0  |   1  |   0      |
|   NFSC679R  |   Abani Pradhan  |   201208    |   101      |   2   |   5   |   0   |   0   |   0  |   0   |   0   |   0  |   0   |   0   |   0  |   0   |   0  |   0  |   0      |
|   NFSC431   |   Abani Pradhan  |   201301    |   101      |   4   |   8   |   2   |   5   |   2  |   2   |   0   |   0  |   0   |   0   |   0  |   0   |   0  |   0  |   0      |
|   NFSC679R  |   Abani Pradhan  |   201308    |   101      |   1   |   3   |   2   |   0   |   0  |   0   |   0   |   0  |   0   |   0   |   0  |   0   |   0  |   0  |   0      |
|   NFSC431   |   Abani Pradhan  |   201401    |   101      |   4   |   5   |   2   |   3   |   3  |   2   |   1   |   0  |   0   |   0   |   1  |   0   |   0  |   1  |   0      |

Now that we know what some entries in our dataset looks like, we want to get a better idea of what our dataset consists of and to do this we can create some visualizations to better understand the shape and tendencies of our data. Here, we have decided to plot the distribution of average rating, average GPA and letter grades.
  
```
from matplotlib import pyplot as plt
import numpy as np

index = np.asarray([i for i in range(1, 6)])

plt.hist(prof_df["average_rating"], bins=5)
plt.xticks(index)
plt.title("Average Rating vs Count Distribution of all courses")
plt.xlabel("Average Rating")
plt.ylabel("Count")

plt.show()
```

```
grade_counts = {grade: grades_df[grade].sum() for grade in ['A+', 'A', 'A-', 'B+', 'B', 'B-', 'C+', 'C', 'C-', 'D+', 'D', 'D-', 'F', 'W', 'Other']}

x = list(grade_counts.keys())
y = list(grade_counts.values())

plt.bar(x, y)
plt.title("Grades vs Count Distribution for all courses")
plt.xlabel("Grades")
plt.ylabel("Count")

plt.show()
```

```
index = np.asarray([i for i in range(1, 5)])

plt.hist(all_courses_df["average_gpa"], bins=12)
plt.xticks(index)
plt.title("Average GPA vs Count Distribution for all courses")
plt.xlabel("Average GPA")
plt.ylabel("Count")

plt.show()
```

From these visualizations, we can take note of a few general trends that are significant (for general data across all departments):
* Average Rating Distribution
  * It appears that much more reviews that are left on PlanetTerp are overwhelmingly positive. The most reviews left were a 5 rating, and the count incrementally decreases for each consecutive lower rating
  * This appears as a completely left skewed distribution, which agrees with our sentiment that most of the observations occur in the medium/high range of the distribution
* Grades Distribution
  * It appears that As are the mostly commonly recieved grade, which could be related to our previous observation that most reviews were a full 5 stars
  * Only considering letter grades (F-A+), this appears to be a right-skewed distribution, which means a lot of our observations occur in the left part of the distribution (the A range)
* Average GPA Distribution
  * The median average GPA across all courses appears to hover around the 3.3 area, with a slightly left skewed distribution

Now that we have an idea about our entire dataset, let's try to identify some potential new trends solely in the CS department. We will once again plot the distributions of average rating, average GPA and letter grades but this time only for entries in the CS department.

```
cmsc_courses_df = all_courses_df
cmsc_courses_df = cmsc_courses_df.loc[cmsc_courses_df["department"] == "CMSC"]
cmsc_courses_df
```

```
cmsc_prof = []
cmsc_prof_df = pd.DataFrame()
for index, row in prof_df.iterrows():
    if "CMSC" in row['courses']:
      cmsc_prof.append(row)

cmsc_prof_df = pd.DataFrame(cmsc_prof, columns=prof_df.columns)
cmsc_prof_df
```
  
```
cmsc_grades = []
cmsc_grades_df = pd.DataFrame()
for g, row in grades_df.iterrows():
    if row['course'].startswith('CMSC'):
      cmsc_grades.append(row)

cmsc_grades_df = pd.DataFrame(cmsc_grades, columns=grades_df.columns)
cmsc_grades_df
```

```
knes_courses_df = all_courses_df
knes_courses_df = knes_courses_df.loc[knes_courses_df["department"] == "KNES"]
knes_courses_df
```

```
knes_prof = []
knes_prof_df = pd.DataFrame()
for index, row in prof_df.iterrows():
    if "KNES" in row['courses']:
      knes_prof.append(row)

knes_prof_df = pd.DataFrame(knes_prof, columns=prof_df.columns)
knes_prof_df
```
  
```
knes_grades = []
knes_grades_df = pd.DataFrame()
for g, row in grades_df.iterrows():
    if row['course'].startswith('KNES'):
      knes_grades.append(row)

knes_grades_df = pd.DataFrame(knes_grades, columns=grades_df.columns)
knes_grades_df
```

## Hypothesis Test
In this section of the tutorial, we aim to see whether there is a statistically significant different in the mean average GPA of all CMSC (Computer Science) and KNES (Kinesiology) courses. We want to accomplish this using a two sample t test, where we are essentially testing whether two population means are equal. In this case, we are using the unpaired variation of the test because we can not assume that the samples are correlated. In this situation, our null hypothesis $H_0$ is that the populations 

```
#make numpy arrays out of the avg gpa column for both
cmsc_avg_gpa = cmsc_courses_df['average_gpa'].values
knes_avg_gpa = knes_courses_df['average_gpa'].values

print("CMSC Courses count: " + str(len(cmsc_avg_gpa)) + " or " + "KNES Courses count: " + str(len(knes_avg_gpa)))

#cut off 18 entries to be the same sample size
index = [35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52]

cmsc_avg_gpa2 = np.delete(cmsc_avg_gpa, index)

print("CMSC Courses count: " + str(len(cmsc_avg_gpa2)) + " or " + "KNES Courses count: " + str(len(knes_avg_gpa)))
cmsc_avg_gpa2
```
  
Expanding upon this, as we feel the need to incorporate other departments into our calculation, we would likely need an ANOVA test, which would test for whether any statistically significant differences exist between the means of three or more groups (which would be the different departments in our case). This can be followed up with Tukey's procedure to find the means that are significantly different from each other.
