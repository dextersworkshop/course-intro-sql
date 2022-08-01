
## HW | Project Week 01


#### First, they want the basics of the business:

1. How many courses have we run so far? (This is the number of rows in the table as all courses in the table have been run as you are in 2024).
    - **999 course runs have happened so far**

```sql
-- count ALL rows in table
select count(*) from courses
```

#### Now, they would like to know the next level of detail:

2. What are the distinct categories of courses that we run? This is to understand the spectrum of courses we cover.

    1. Data Engineering
    2. Data Science
    3. Deep Learning
    4. Language Processing
    5. Machine Learning
    6. Python
    7. Search

```sql
-- retrieve distinct course categories
select distinct course_category 
from courses 
order by course_category
```


#### Digging into Machine Learning courses:

They have heard from their boss that Machine Learning is a very important category for the co:rise business. Their logical next question: 

3. For the course category 'Machine Learning' how many courses have we run so far?
    - **310 runs have occured for courses in falling in the Machine Learning category**

```sql
-- count course runs for the Machine Learning category
select count(*)
from courses
where lower(course_category) = 'machine learning'
```


They want to dig in more. 

4. For the course category 'Machine Learning', what are the distinct names and descriptions of the courses?
    - **8 distinct courses for ML. too many to list here so only including SQL query.**

```sql
-- retrieve distinct names and descriptions of the courses (for ML)
select distinct course_name, course_desc
from courses
where lower(course_category) = 'machine learning'
order by course_name
```


5. What distinct course names contain the string 'machine learning' or 'ml' or 'deep learning' (We want the match to be case-insensitive).
    > **if you filter by course category machine learning (ML), then their are ONLY 5 courses meeting the criteria**
    > **however, if you include don't filter by ML then there are 7 courses meeting the criteria.**

    - Applied Machine Learning
    - Data Centric Deep Learning
    - Deep Learning Essentials
    - How to Implement ML Papers
    - MLOps: From Models to Production
    - Real Time Machine Learning
    - Search with Machine Learning

```sql
-- without ML filter
select distinct course_name
from courses
where 
     lower(course_name) like ('%machine learning%')
  or lower(course_name) like ('%ml%')
  or lower(course_name) like ('%deep learning%')
order by course_name
```

```sql
-- with ML filter
select distinct course_name, course_category
from courses
where 
    (lower(course_category) = 'machine learning')
  and 
    (lower(course_name) like ('%machine learning%')
  or lower(course_name) like ('%ml%')
  or lower(course_name) like ('%deep learning%'))
order by course_name
```


They were informed by others at co:rise that in 2020 there were some challenges running machine learning courses on time. 

6. They want to dig deeper with the following query: For all courses with course category 'Machine Learning' that started in 2020, print the course_name, the start_date, and the completion_date.

```sql
-- query
select 
    course_name
  , start_date
  , (start_date + num_weeks*7) as completion_date
from courses
where 
      lower(course_category) = 'machine learning'
  and extract(year from start_date) = 2020
order by course_name, start_date
```

They were informed by others at co:rise that machine learning courses that started in 2020 had a huge TA shortage and as a result courses were impacted. 

7. For each course from machine learning category that started in 2020 print the name of the course and the student:TA ratio (Please note that the number of TAs in the data is buggy - as with any real data - it has a bunch of zeros and negative 1s - please add a filtering condition to focus only on rows which have a positive (>0) number of TAs). 

```sql
-- query
select 
    course_name
  , cast(num_learners_registered as float) / 
         num_tas as student_ta_ratio
from courses
where 
      lower(course_category) = 'machine learning'
  and extract(year from start_date) = 2020
  and num_tas > 0
order by student_ta_ratio
```

We are trying to get a handle on the courses whose registration may have been impacted by COVID. 

8. To help with that, print the course_name and start_date of courses with the category Machine Learning that were either run in 2020 with less than 30 learners or where the number of learners was NULL (This indicates that the number of learners may have been less than 30). Do not print the row if the number of TAs was more than 4, as this is a good indicator that the course had reasonable registration.
  - **looks like 9 rows (course runs) meet these criteria**

```sql
-- query
select 
    course_name
  , start_date
from courses
where 
      lower(course_category) = 'machine learning'
  and (num_learners_registered < 30 or num_learners_registered is null)    
  and num_tas <= 4
  and extract(year from start_date) = 2020
order by start_date
```

We want to get a list of course names and start dates for courses that were run in May 2020, but only if something like the word 'machine learning' is in the description of the course. 

9. The description is a text field that has been populated from various tools, and there is no guarantee that the number of spaces is fixed or that the words are consistently capitalized. We would like to be relaxed about matching the number of spaces or the capitalization. 

```sql
-- query
select 
    course_name
  , start_date
from courses
where 
      lower(course_category) = 'machine learning'
  and start_date between '2020-05-01' and '2020-05-31'
  and lower(course_desc) like '%machine%learning%'
order by start_date
```


#### Bonus:

You have been asked to come up with three different lines of questions you could ask about this data that may be useful for understanding the business deeply. Can you formulate three English queries that can be solved with the material we have covered so far in the course? Please also write the corresponding SQL queries.