# Competitive Landscaping Project

Repo for the Ironhack project "Bootcamps"

Collaborative Project from Ironhack students:

Felipe: https://www.linkedin.com/in/felipedeavilagranja/

Guilherme: https://www.linkedin.com/in/guilherme-torres-pereira/

Lucie: https://github.com/luciefley

## Brief Wiki Description

Competitive landscape is a business analysis method that identifies direct or indirect competitors to help comprehend their mission, vision, core values, niche market, strengths, and weaknesses. Based on the volatile nature of the business world, where companies represent a competition to others, this analysis helps to establish a new mind-set which facilitates the creation of strategic competitiveness.


## Overview

The process envolved all of the above areas:

1. Scrapping - From the https://www.switchup.org/
2. Data Cleaning - We didn't want to use all the data that came from the scappring, and also it had some problems that we needed to fix.
3. Data Engineering - For this project we stored the data in a AWS cloud server.
4. Data Analysis - The final step was to analyse the data that we scrapped to come up with insights.

## How to run the code

The code can be runned, but unfortunately the results would not be the same, because the switchup.org webiste gets constantly updated.
But nevertheless, you can do it like this:

1. Either fork or download this repo.
2. Install all requirements
3. Open the " final_version_1-2.ipynb "
4. Run the code.

## Scrapping

The data was scrapped from the https://www.switchup.org/.

It consist of data from Bootcamps reviews.

```python

schools = {   
'ironhack' : 10828,
'la-capsule' : 10853,
'app-academy' : 10525,
'springboard' : 11035,
'metis' : 10886,
'practicum-by-yandex' : 11225,
'le-wagon' : 10868,
'academia-de-codigo' :10494 ,
'react-graphql-academy' : 10972
}

import re
import pandas as pd
from pandas.io.json import json_normalize
import requests



def get_comments_school(school):
    TAG_RE = re.compile(r'<[^>]+>')
    # defines url to make api call to data -> dynamic with school if you want to scrape competition
    url = "https://www.switchup.org/chimera/v1/school-review-list?mainTemplate=school-review-list&path=%2Fbootcamps%2F" + school + "&isDataTarget=false&page=3&perPage=10000&simpleHtml=true&truncationLength=250"
    #makes get request and converts answer to json
    # url defines the page of all the information, request is made, and information is returned to data variable
    data = requests.get(url).json()
    #converts json to dataframe
    reviews =  pd.DataFrame(data['content']['reviews'])
  
    #aux function to apply regex and remove tags
    def remove_tags(x):
        return TAG_RE.sub('',x)
    reviews['review_body'] = reviews['body'].apply(remove_tags)
    reviews['school'] = school
    return reviews
    
    
from pandas.io.json import json_normalize

def get_school_info(school, school_id):
    url = 'https://www.switchup.org/chimera/v1/bootcamp-data?mainTemplate=bootcamp-data%2Fdescription&path=%2Fbootcamps%2F'+ str(school) + '&isDataTarget=false&bootcampId='+ str(school_id) + '&logoTag=logo&truncationLength=250&readMoreOmission=...&readMoreText=Read%20More&readLessText=Read%20Less'

    data = requests.get(url).json()

    data.keys()

    courses = data['content']['courses']
    courses_df = pd.DataFrame(courses, columns= ['courses'])

    locations = data['content']['locations']
    locations_df = json_normalize(locations)

    badges_df = pd.DataFrame(data['content']['meritBadges'])
    
    website = data['content']['webaddr']
    description = data['content']['description']
    logoUrl = data['content']['logoUrl']
    school_df = pd.DataFrame([website,description,logoUrl]).T
    school_df.columns =  ['website','description','LogoUrl']

    locations_df['school'] = school
    courses_df['school'] = school
    badges_df['school'] = school
    school_df['school'] = school
    

    locations_df['school_id'] = school_id
    courses_df['school_id'] = school_id
    badges_df['school_id'] = school_id
    school_df['school_id'] = school_id

    return locations_df, courses_df, badges_df, school_df

locations_list = []
courses_list = []
badges_list = []
schools_list = []

for school, id in schools.items():
    print(school)
    a,b,c,d = get_school_info(school,id)
    
    locations_list.append(a)
    courses_list.append(b)
    badges_list.append(c)
    schools_list.append(d)

```

## Data Cleaning

We didn't want to use all the data that came from the scappring, and also it had some problems that we needed to fix.

![image](https://user-images.githubusercontent.com/83870535/129186608-d821472b-6691-4d4d-91f2-f35bae7a4832.png)

```python

sub_df2 = clean_schools.rename(columns={'name':'school'})
clean_comments = comments.merge(sub_df2, how='inner', on='school')
display(clean_comments.columns)
to_drop = ['anonymous', 'hostProgramName', 'graduatingYear','jobTitle', 'tagline', 'body', 'rawBody', 'createdAt',
       'queryDate', 'user', 'comments', 'review_body','school']
clean_comments.drop(to_drop, inplace=True,axis=1)
clean_comments = clean_comments.fillna(0)
clean_comments['overall'] = clean_comments['overall'].apply(lambda x : float(x))
clean_comments['overallScore'] = clean_comments['overallScore'].apply(lambda x : float(x))
clean_comments['curriculum'] = clean_comments['curriculum'].apply(lambda x : float(x))
clean_comments['jobSupport'] = clean_comments['jobSupport'].apply(lambda x : float(x))


locations_clean = locations.copy()
to_drop = ['description','country.id','country.abbrev','city.id','city.keyword','state.id','state.name','state.abbrev','state.keyword']
locations_clean.drop(to_drop,inplace=True,axis=1,)
locations_clean.rename(columns = {'id':'location_id','country.name':'country','city.name':'city'}, inplace = True)

```

## Data Engineering

We also did some feature engineering to get the data that we were interested:

```python
def badges_m(row):
    if row == 'Available Online':
        return 1
    elif row == 'Verified Outcomes':
        return 2
    elif row == 'Flexible Classes':
        return 3
    elif row == 'Job Guarantee':
        return 4
```

For this project we stored the data in a AWS cloud server:

![image](https://user-images.githubusercontent.com/83870535/129187025-06ef9bd6-68bd-4987-809a-151161a075b2.png)

## Data Analysis

And this were the final tables that we builded:

### The schools table:
![image](https://user-images.githubusercontent.com/83870535/129187084-5f491992-df0a-4e67-9309-179ca7bf8347.png)

### The badges table:
![image](https://user-images.githubusercontent.com/83870535/129187107-1fb62e7b-4c02-4770-a777-be734861db77.png)

### The schools badges table:
![image](https://user-images.githubusercontent.com/83870535/129187120-ca24a879-3b57-438f-9019-d5917d776f60.png)

### The comments table:
![image](https://user-images.githubusercontent.com/83870535/129187159-1e2975b1-b534-4888-9df2-a54a6fd4af54.png)


### The location table:
![image](https://user-images.githubusercontent.com/83870535/129187176-5c7e4bc6-509a-4417-991b-aabb2cd86246.png)


The final step was to analyse the data that we scrapped to come up with insights. The presentation is in the " Project2- Competitive landscaping.pptx " file.
