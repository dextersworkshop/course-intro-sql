## HW | Project Week 03

<br>

## Overview

Your career at co:rise has been on the rise. As the complexity of their database for tracking their course related information has gone up, your role is now to proactively look for patterns in the data.

You are storing information in an inter-related set of tables now, and you need to bring out your ace SQL skills to satisfy your data curiosity.

We will be going through a series of analysis to identify avenues for growing the impact of the awesome courses that are coming out of co:rise. As before, we are focused on nps as the key metric to evaluate how our courses are being received by the learners. We want to continue to understand possible correlation of various pieces of our data elements with the nps.

## Analysis

You have decided that probing into the affiliation of the instructor and its correlation to nps is worth exploring.

1. Find the maximum, average and standard deviation of the nps of a course run per each affiliation (of the instructor).

```sql
-- summary statistics for course-runs aggregated by instructor affiliation
select
    instructors.affiliation,
    count(course_run.course_run_id)        as total_runs,
    max(course_run.nps)                    as nps_max,
    round(avg(course_run.nps),1)           as nps_avg,
    round(stddev(course_run.nps),1)        as nps_sd
from instructors
inner join course_run on instructors.instructor_id = course_run.instructor_id
group by instructors.affiliation
order by nps_avg desc
```


As you can notice, the instructor affiliation does not seem to correlate strongly with the nps. We have good instructors from all affiliations, so we can continue to expand our courses and keep getting instructors who are experts from all companies! That is great news!
- **i disagree. the nps_avg varies widely by affiliation; highest avg-nps is 91.3 AND the lowest avg-nps is 56.3. that tells us that affiliation does explain a portion of the variation seen in our NPS metric**

<br>

Explore if teaching experience has a correlation with nps. 

2. Retrieve the maximum, average, and standard deviation of the nps of a course run for two levels of teaching experience – early, which is 5 or fewer years of teaching experience, and experienced, which is more than 5 years of experience. 

```sql
-- summary statistics for course-runs aggregated by teaching experience (early/experienced)
select
    case 
        when teaching_experience <= 5 then 'early'
        when teaching_experience  > 5 then 'experienced'
        else null
    end as experience_level,
    count(course_run.course_run_id)        as total_runs,
    max(course_run.nps)                    as nps_max,
    round(avg(course_run.nps),1)           as nps_avg,
    round(stddev(course_run.nps),1)        as nps_sd
from instructors
inner join course_run on instructors.instructor_id = course_run.instructor_id
group by 
    case 
        when teaching_experience <= 5 then 'early'
        when teaching_experience  > 5 then 'experienced'
        else null
    end
order by nps_avg desc
```


Surprisingly, there is no obvious correlation. But you persist

<br>

Can you see if there is any correlation if you were to further slice the data into the two tiers of teaching experience combined with the level of the course?

3. Find the maximum, average and standard deviation of the nps of a course run for early and experienced instructors combined with the level of the course.

```sql
-- summary statistics for course-runs aggregated by course-level, by teaching-experience (early/experienced)
select
    course_info.course_level,
    case 
        when teaching_experience <= 5 then 'early'
        when teaching_experience  > 5 then 'experienced'
        else null
    end as experience_level,
    count(course_run.course_run_id)        as total_runs,
    min(course_run.nps)                    as nps_min,
    max(course_run.nps)                    as nps_max,
    round(avg(course_run.nps),1)           as nps_avg,
    round(stddev(course_run.nps),1)        as nps_sd
from instructors
inner join course_run on instructors.instructor_id = course_run.instructor_id
inner join course_info on course_run.course_id = course_info.course_id
group by 
    course_level,
    case 
        when teaching_experience <= 5 then 'early'
        when teaching_experience  > 5 then 'experienced'
        else null
    end
order by course_level asc
```


Aha! You hit pay dirt! It looks like early instructors are scoring high nps on basic courses relative to experienced instructors and vice versa! Interesting insight and something that you can perhaps intuitively agree with, although you should always listen to the data!! :-) If the data backs your intuition, great, but if not, verify the data twice or even thrice but go with the data in the end!

<br>

## Further Analysis

As you have been busy digging into co:rise wide statistics, you are interrupted by the account manager for **M & T Bank Corporation** (such is the life of an analyst). They want to understand if there are any challenges with basic courses being run for M & T Bank Corporation cohorts. 

Given the previous finding, you decide to look only at courses taught by experienced instructors to make sure they are working fine.

4. Get the nps for each course run that has more than 2 learners from M & T Bank Corporation and is taught by an experienced instructor for a basic course (course level = 'B').

```sql
-- will come back to solve later
```

<br>

As you are aware by now, there are always interruptions for some query or another that someone in the company would love to learn more about. So your internal reviewer of instructors reaches out and asks for the following:

5. Please give me the instructor with the highest variance in the nps scores of courses they have facilitated. This query should print only one row - output limited to one row :-)

<br>

## Bonus Question:

For each course category and the corresponding instructor affiliation pair, retrieve the average nps of the courses as well as the rank of a given affiliation within a course_category, appended to each row

For this question we are going to compute running total in two different ways. Let us define the exact table on which we will perform the running total - to keep things tractable we are going to choose five rows from course_run table using the following query (note that the query you will write should naturally generalize in its form and structure even if there were many more rows etc.)