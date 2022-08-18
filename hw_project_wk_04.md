## HW | Project Week 04

<br>

## Overview

Your life as a data analyst at co:rise has been a blast! Your previous successes at helping co:rise understand the core drivers of their business has set you up very well at co:rise. You have been promoted to be a head analyst and are now expected to proactively ask the questions that will unearth value to the business folks at co:rise. Here are some questions that have been playing around in your mind - you want to write SQL for these English queries. 

<br>

## Questions

1. Names of Students who have not registered for any course using outer joins (You can assume course_run_id is not null in the table course_registration_info - this is definitely true for the data you have in the table now. As an advanced topic - although course_run_id is not a primary key for the table course_registration_info, let us assume it has been declared as not null - this means we can not insert a row into that table with a null value for course_run_id - the DBMS will ensure that.)

```sql
-- student(s) who have not registered for any courses
select 
    learners.learner_id,
    learners.name,
    case when reg_info.course_run_id is null then 'No' 
         else 'Yes'
    end as registered
from learners
left join course_registration_info as reg_info
    on learners.learner_id = reg_info.learner_id
where reg_info.course_run_id is null;
```

<br>

2. Names of Students who have not registered for any course using except
    - **including learner-id to make it explicit which student has not enrolled (necessary due to student names being repeated for registered learners who have the same name)**

```sql
-- give me students unique to learners table when compared to course-reg table
select 
    learner_id, 
    name as student_name
from learners

except

select distinct
    reg_info.learner_id,
    learners.name
from course_registration_info as reg_info
left join learners on reg_info.learner_id = learners.learner_id;
```


<br>

3. Retrieve instructor_id and names of each instructor in corise and the  number of  courses they have taught

```sql
-- count of courses run by CoRise instructors
select 
    instructors.instructor_id,
    instructors.name,
    count(course_run.course_id) as courses_taught   -- using column name to get 0 instead of 1 where NULL
from instructors
left join course_run on instructors.instructor_id = course_run.instructor_id
group by 
    instructors.instructor_id,
    instructors.name
order by instructor_id;
```
