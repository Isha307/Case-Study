<h3>Data Analysis Questions</h3>

  # How many customers has Foodie-Fi ever had?
  ```sql
  select count(DISTINCT CUSTOMER_ID) as tot_customer from subscriptions
  ```
  # What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
  ```sql
 select EXTRACT(month from START_DATE) as mon, count(EXTRACT(month from START_DATE)) as Mon 
 from subscriptions  where PLAN_ID = 0 group by EXTRACT(month from START_DATE) order by EXTRACT(month from START_DATE)
  ```
  # What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
  ```sql
select PLAN_ID, EXTRACT(year from START_DATE) as plan_year , count(*) as plan_year_cnt 
from subscriptions where EXTRACT(year from START_DATE) > '2020' group by PLAN_ID, EXTRACT(year from START_DATE)
  ```
  # What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
  ```sql
SELECT COUNT(*) AS customer_count,
ROUND((CAST(COUNT(*) AS FLOAT) / (SELECT COUNT(DISTINCT customer_id) FROM  subscriptions)) * 100, 1)
AS churn_percentage FROM subscriptions WHERE plan_id = 4 group by plan_id;
  ```
  # How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
  ```sql
  ```
  # What is the number and percentage of customer plans after their initial free trial?
  ```sql
  ```
  # What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
  ```sql
  ```
  # How many customers have upgraded to an annual plan in 2020?
  ```sql
  ```
  # How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
  ```sql
  ```
  # Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
  ```sql
  ```
  # How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
  ```sql
  ```
 
