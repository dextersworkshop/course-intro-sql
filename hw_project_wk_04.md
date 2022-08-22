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
    - **used LEFT JOIN to ensure that instructors that haven't taught yet are included in the output. And, used count(col-name) to avoid counts of 1 where rows were null in the right table; my approach returns counts of 0 when NULL (another way that would be even clearer would be to use a Case When statement)**

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

4. Affiliations for which neither an instructor has taught a course in 2020 nor a student has registered for a course in 2020 (please do not use outer joins for this query)
    - **I am getting just ONE affiliation returned that meets these conditions:**
        * *Mail-Well Inc.*

### Solution 01

```sql
-- affiliations without instructors running courses in 2020
select affiliation from instructors 
inner join course_run on instructors.instructor_id = course_run.instructor_id
group by affiliation
having max(case when extract(year from start_date) = 2020 then 1 else 0 end) = 0

-- identify affiliations meeting both conditions by finding where they intersect eachother
intersect

-- affiliations without students taking courses in 2020
select learners.affiliation from learners
inner join course_registration_info as reg_info 
    on learners.learner_id = reg_info.learner_id
inner join course_run 
    on reg_info.course_run_id = course_run.course_run_id
group by learners.affiliation
having max(case when extract(year from start_date) = 2020 then 1 else 0 end) = 0;
```

### Solution 02

```sql
-- list of ALL affiliations (full universe of possibilities)
(
select distinct affiliation from instructors

    -- combine ALL instructor affiliations w/ALL student affiliations
    union distinct

select distinct affiliation from learners
)

    -- subtract 2020 affiliation participants from FULL list of affiliations
    except

-- list of affiliations having courses taught in 2020 or having students enrolled in 2020

-- affiliations teaching courses in 2020
(
select distinct instructors.affiliation from instructors
inner join course_run on instructors.instructor_id = course_run.instructor_id
where extract(year from start_date) = 2020

    -- combine teacher affiliations w/student affiliations
    union distinct

-- affiliations with students enrolled in courses in 2020
select distinct affiliation from learners
inner join course_registration_info on learners.learner_id = course_registration_info.learner_id
inner join course_run on course_registration_info.course_run_id = course_run.course_run_id
where extract(year from start_date) = 2020
)
```


5. Average teaching experience of instructors who have never taught a class of more than 4 weeks

### Solution 01
    
- **Average Teaching Experience:** 4.3 Years

```sql
-- avg teaching experience for instructors who have never taught a class of more than 4 weeks long
select 
    round(avg(teaching_experience), 1) as avg_teaching_experience
from instructors
inner join (

    -- get distinct list of ALL instructors (A)
    select distinct instructor_id from instructors

        -- remove from A what is in B
        except

    -- get list of instructors having taught course more than 4 weeks long
    select distinct instructor_id from course_run
    where num_weeks > 4
    order by instructor_id
) as short_class_instructors

on instructors.instructor_id = short_class_instructors.instructor_id;
```