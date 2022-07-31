
## HW | Project Week 01


#### First, they want the basics of the business:

1. How many courses have we run so far? (This is the number of rows in the table as all courses in the table have been run as you are in 2024).

```sql
-- count ALL rows in table
select count(*) from courses
```

    - **999 course runs have happened so far**


#### Now, they would like to know the next level of detail:

2. What are the distinct categories of courses that we run? This is to understand the spectrum of courses we cover.

```sql
-- retrieve distinct course categories
select distinct course_category 
from courses 
order by course_category
```

    1. Data Engineering
    2. Data Science
    3. Deep Learning
    4. Language Processing
    5. Machine Learning
    6. Python
    7. Search




#### Digging into Machine Learning courses:

They have heard from their boss that Machine Learning is a very important category for the co:rise business. Their logical next question: 

3. For the course category 'Machine Learning' how many courses have we run so far?

They want to dig in more. For the course category 'Machine Learning', what are the distinct names and descriptions of the courses?

What distinct course names contain the string 'machine learning' or 'ml' or 'deep learning' (We want the match to be case-insensitive)

They were informed by others at co:rise that in 2020 there were some challenges running machine learning courses on time. They want to dig deeper with the following query: For all courses with course category 'Machine Learning' that started in 2020, print the course_name, the start_date, and the completion_date.

They were informed by others at co:rise that machine learning courses that started in 2020 had a huge TA shortage and as a result courses were impacted. For each course from machine learning category that started in 2020 print the name of the course and the student:TA ratio (Please note that the number of TAs in the data is buggy - as with any real data - it has a bunch of zeros and negative 1s - please add a filtering condition to focus only on rows which have a positive (>0) number of TAs). 

We are trying to get a handle on the courses whose registration may have been impacted by COVID. To help with that, print the course_name and start_date of courses with the category Machine Learning that were either run in 2020 with less than 30 learners or where the number of learners was NULL (This indicates that the number of learners may have been less than 30). Do not print the row if the number of TAs was more than 4, as this is a good indicator that the course had reasonable registration.

We want to get a list of course names and start dates for courses that were run in May 2020, but only if something like the word 'machine learning' is in the description of the course. The description is a text field that has been populated from various tools, and there is no guarantee that the number of spaces is fixed or that the words are consistently capitalized. We would like to be relaxed about matching the number of spaces or the capitalization. 

Bonus:

You have been asked to come up with three different lines of questions you could ask about this data that may be useful for understanding the business deeply. Can you formulate three English queries that can be solved with the material we have covered so far in the course? Please also write the corresponding SQL queries.