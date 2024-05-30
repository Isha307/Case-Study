<h1>Data Analysis Questions</h3>

  ## How many customers has Foodie-Fi ever had?
  ```sql
  select count(DISTINCT CUSTOMER_ID) as tot_customer from subscriptions
  ```
  ## What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
  ```sql
 select EXTRACT(month from START_DATE) as mon, count(EXTRACT(month from START_DATE)) as Mon 
 from subscriptions  where PLAN_ID = 0 group by EXTRACT(month from START_DATE) order by EXTRACT(month from START_DATE)
  ```
  ## What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
  ```sql
select PLAN_ID, EXTRACT(year from START_DATE) as plan_year , count(*) as plan_year_cnt 
from subscriptions where EXTRACT(year from START_DATE) > '2020' group by PLAN_ID, EXTRACT(year from START_DATE)
  ```
  ## What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
  ```sql
SELECT COUNT(*) AS customer_count,
ROUND((CAST(COUNT(*) AS FLOAT) / (SELECT COUNT(DISTINCT customer_id) FROM  subscriptions)) * 100, 1)
AS churn_percentage FROM subscriptions WHERE plan_id = 4 group by plan_id;
  ```
  ## How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
  ```sql
WITH cte AS (SELECT customer_id, plan_id, 
       LAG(plan_id, 1) OVER(PARTITION BY customer_id ORDER BY plan_id) AS previous_plan
FROM subscriptions)
SELECT COUNT(previous_plan) AS cnt, 
	   ROUND(COUNT(*) * 100 / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 0) AS percentage
FROM cte
WHERE plan_id = 4 and previous_plan = 0 group by plan_id,previous_plan ;
  ```
  ## What is the number and percentage of customer plans after their initial free trial?
  ```sql
 WITH cte AS (SELECT customer_id, plan_id, 
       LAG(plan_id, 1) OVER(PARTITION BY customer_id ORDER BY plan_id) AS previous_plan
FROM subscriptions)
SELECT plan_id,COUNT(previous_plan) AS cnt, 
	   ROUND(COUNT(*) * 100 / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 2) AS percentage
FROM cte
WHERE previous_plan = 0 and plan_id <> 0 group by plan_id,previous_plan order by plan_id ;
  ```
  ## What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
  ```sql
with cte as (select PLAN_ID,customer_id,start_date, --count(CUSTOMER_ID), --round(count(CUSTOMER_ID)*100/(select count(*) from subscriptions),2) as each_percent
    rank() over(partition by customer_id order by start_date desc) as rnk
from subscriptions where START_DATE < ='31-DEC-20' 
order by customer_id, plan_id)
select PLAN_ID,count(CUSTOMER_ID),
round(count(CUSTOMER_ID)*100/(select count(*) from cte),2) as each_percent    
from cte where rnk = 1 group by PLAN_ID 
  ```
  ## How many customers have upgraded to an annual plan in 2020?
  ```sql
SELECT COUNT(DISTINCT customer_id) AS unique_customers
FROM subscriptions
WHERE plan_id = 3 AND start_date <= '31-DEC-20'
  ```
  ## How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
  ```sql
with cte as (SELECT CUSTOMER_ID, PLAN_ID, START_DATE 
FROM subscriptions
WHERE plan_id = 0),
cte2 as (SELECT CUSTOMER_ID, PLAN_ID, START_DATE 
FROM subscriptions
WHERE plan_id = 3)
select round(avg(cte2.START_DATE - cte.START_DATE),0) as date_diff
from cte join cte2 on cte.CUSTOMER_ID = cte2.CUSTOMER_ID 
  ```
  ## Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
  ```sql
  ```
  ## How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
  ```sql
WITH cte AS (SELECT customer_id,plan_id,start_date,
LEAD(plan_id) OVER(PARTITION BY customer_id ORDER BY plan_id) AS next_plan
FROM subscriptions)
SELECT COUNT(*) AS downgraded
FROM cte
WHERE start_date <= '31-DEC-20'
AND plan_id = 2 AND next_plan = 1;
  ```
