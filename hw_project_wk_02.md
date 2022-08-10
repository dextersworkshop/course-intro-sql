## HW | Project Week 02


## Part 1:

Recall from last week's project that you had a new head of content creation who was asking you some basic questions about the data in the co:rise database. A month has passed since they joined, and they have had substantive qualitative conversations with 20 instructors and 40 learners – They have been quite busy indeed!

They now want to get back to some more **advanced quantitative analysis**. We will continue to work with the courses table we introduced in last week's project. They have learned that **nps (net promoter score)** is a **key metric** that we need to track and that it is a pretty **good indicator** statistically of the **quality of the courses** we are delivering. 

<br>

## Gross Statistics:

The head of content creation would like to understand the distribution of nps overall to get their bearings.

1. Find the minimum, maximum, average and standard deviation of nps across all courses

```sql
-- summary stats across all course runs
select
    min(nps)     as min_nps
  , max(nps)     as max_nps
  , avg(nps)     as avg_nps
  , stddev(nps)  as sd_nps
from courses;
```

At first blush, the news is good – The average seems to be reasonably high. That's a relief! However, the minimum is quite low. Perhaps there is a pocket of courses that can be improved quite a bit. They would like you to dig deeper into pockets where the nps may be underperforming. 

<br>

## Digging into some groups:

You have identified that it may be useful to dig into how the nps distribution varies across years and categories, as well as across the number of learners_registered for each course.

2. Let's group the number of learners into three buckets: low (<30 learners); medium (30-100) and high (100+ learners) – Let's get the minimum, maximum, average, and standard deviation of the nps for each of these three buckets.

```sql
-- analyze ALL course runs for classes that recorded the number of learners registered
select 
    case 
      when num_learners_registered <  30                                   then '1_low'
      when num_learners_registered >= 30 and num_learners_registered < 100 then '2_medium'
      when num_learners_registered >= 100                                  then '3_high'
      else null
    end as class_registration
  , min(nps)     as min_nps
  , avg(nps)     as avg_nps
  , max(nps)     as max_nps
  , stddev(nps)  as sd_nps
from courses
where num_learners_registered is not null
group by
    case 
      when num_learners_registered <  30                                   then '1_low'
      when num_learners_registered >= 30 and num_learners_registered < 100 then '2_medium'
      when num_learners_registered >= 100                                  then '3_high'
      else null
    end
order by class_registration asc;
```

<br>

## Analyzing 'Scaled-Courses' (Classes with Med/High Registration)

Aha! You observe something very interesting right away: All courses with <30 learners have extremely high nps. That is great, but the **premise of co:rise is to scale the quality of experience and learning for as many learners as possible.** So we would like to see the medium and high buckets also improve. 

Going forward, let us focus only on those buckets or equivalently, let us ignore the low group. We will call the **courses of interest 'scaled courses'** going forward.

3. Let us get the minimum, maximum, average and standard deviation of nps for each year for scaled courses.

```sql
-- summary stats for NPS by Class Registration type (scaled-courses only), by Class Start Year.
select 
    case 
      when num_learners_registered >= 30 and num_learners_registered < 100 then '2_medium'
      when num_learners_registered >= 100                                  then '3_high'
      else null
    end                           as class_registration
  , extract(year from start_date) as start_year
  , min(nps)                      as min_nps
  , round(avg(nps),1)             as avg_nps
  , max(nps)                      as max_nps
  , round(stddev(nps),1)          as sd_nps
from courses
where 
    num_learners_registered is not null
    and num_learners_registered >= 30
group by
    case 
      when num_learners_registered >= 30 and num_learners_registered < 100 then '2_medium'
      when num_learners_registered >= 100                                  then '3_high'
      else null
    end
    , extract(year from start_date)
order by class_registration asc, start_year asc;
```
Another interesting tidbit emerges - Nps  dropped quite a bit in 2020 (COVID seems to have affected folks even though everything was online – Just dealing with so many things in everyone's life has an impact on both instructors and learners, as well as the TAs and behind the scenes organizers) before coming back even better and stronger in 2021 and 2022 as every one learned to deal with COVID. 

We will treat 2020 as an aberration going forward and remove it from our analysis. 

4. get the minimum, maximum, average and standard deviation of nps for each category for scaled courses not conducted in 2020 but only for categories that have at least 10 courses in that category ('scaled categories').

```sql
-- summary stats for NPS by Course Category, by Class Registration type (scaled-courses only)
select 
    course_category
  , case 
      when num_learners_registered >= 30 and num_learners_registered < 100 then '2_medium'
      when num_learners_registered >= 100                                  then '3_high'
      else null
    end as class_registration
  , min(nps)                      as min_nps
  , round(avg(nps),1)             as avg_nps
  , max(nps)                      as max_nps
  , round(stddev(nps),1)          as sd_nps
  , count(course_name)
from courses
where 
    num_learners_registered is not null
    and num_learners_registered >= 30
    and extract(year from start_date) != 2020
group by
   course_category
   , case 
      when num_learners_registered >= 30 and num_learners_registered < 100 then '2_medium'
      when num_learners_registered >= 100                                  then '3_high'
      else null
    end

-- keep only categories w/at least 10 course runs in that category
having count(course_id) >= 10

-- order output for presentation purposes
order by course_category asc, class_registration asc;
```

This gives us some information, but there does not seem to be much to distinguish based on the category of the course. We may have to try a different slicing to see if there is anything useful that shows up.

<br>

## Analyzing Courses by Variation in Duration to better understand Variation in NPS

Anecdotally, the head of content has compiled some information after interviewing the learners, and has heard from a few learners that some of them preferred courses that ran longer and dug deeper into an area, while others preferred courses that were quick and exposed them to a new area – For these shorter courses, learners could learn a bit more on their own and then perhaps come back for an advanced class. 

You decide to investigate this a bit more in the data.

5. Let us designate courses that are less than 4 weeks old to be short, and long otherwise and call this course_duration. Let us get the average nps for each course_level and course_duration combination.  You can look at all the rows in the courses table and not worry about excluding scaled courses, scaled categories or the aberration year 2020. 

```sql
-- summary stats by Class Duration, by Course Level (for NPS)
select 
    case 
        when num_weeks <  4 then 'short'
        when num_weeks >= 4 then 'long'
        else null
    end as course_duration
  , course_level
  , round(avg(nps),1)             as avg_nps
from courses
group by
    case 
        when num_weeks <  4 then 'short'
        when num_weeks >= 4 then 'long'
        else null
    end
    , course_level
order by course_duration desc, course_level asc;
```

**Bonus:** Let's also look at the averages for a course level for each output row to see how the averages for a course level compare with individual rows (the combination of course_level and course_duration).

6. Do you see that advanced courses have a significantly lower nps level when they are of a smaller duration? And does this pattern switch for basic courses? **Yes, i see it and the pattern does switch for basic courses**

```sql
-- summary stats by Class Duration, by Course Level (for NPS)
select 
    course_level
  , case 
        when num_weeks <  4 then 'short'
        when num_weeks >= 4 then 'long'
        else null
    end as course_duration
  , round(avg(nps),1)             as avg_nps_by_level_by_duration

  -- taking avg course level by taking the average of averages (bad!)
  , round(avg(avg(nps)) over(partition by course_level),1 ) as avg_avg_nps_by_level

  -- taking a proper avg by first summing nps and counting rows from the original dataset 
  -- then applying window functions to that output
  , round( sum(sum(nps))    over(partition by course_level) / 
           sum(count(nps))  over(partition by course_level), 1)  as avg_nps_by_level

from courses
group by
    case 
        when num_weeks <  4 then 'short'
        when num_weeks >= 4 then 'long'
        else null
    end
    , course_level
order by course_level, course_duration desc;
```


<br>

## Part 2: (Bonus)

We will now go back to our offline retail store, the xacts table. 


1. We now want to understand how different user_demographics spend on different product_categories. Write a SQL query to print the total sales for each user_demographic and product_category combination. Then, assign a rank to each output row based on the rank of the row within each product_category - the rank is with respect to the total sales.

```sql
-- total sales by User Demographic, by Product Category (ranked by sales within product category)
select
  user_demographic
  , product_category
  , sum(sale_price) as total_sales
  , rank() over(partition by product_category order by sum(sale_price) desc) as ranked_sales_with_category
from xacts
group by 1, 2
order by product_category, total_sales desc
```


2. We want to understand how long it takes for a user to come back to the store. Write a SQL query to print the user_id, purchase_date and the gap between this purchase and the previous purchase.

```sql

```