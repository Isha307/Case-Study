<h1>A. Pizza Metrics</h1>

## How many pizzas were ordered?
```sql
select count(*) from customer_orders;
```
## How many unique customer orders were made?
```sql
select count(distinct CUSTOMER_ID) as unique_pizza_count
from customer_orders;
```
## How many successful orders were delivered by each runner?
```sql
select RUNNER_ID, count(RUNNER_ID) as Successful_orders  
from runner_orders where CANCELLATION is NULL or CANCELLATION = 'null'
group by RUNNER_ID order by RUNNER_ID

```
## How many of each type of pizza was delivered?
```sql
select PIZZA_ID, count(PIZZA_ID) 
from customer_orders c left join runner_orders r on c.order_id = r.order_id
where CANCELLATION is NULL or CANCELLATION = 'null'
group by PIZZA_ID
```
## How many Vegetarian and Meatlovers were ordered by each customer?
```sql
select CUSTOMER_ID, sum(case
    when PIZZA_ID = 1 then 1
    else 0 end)
 as Meatlovers, sum(case
    when PIZZA_ID = 2 then 1
    else 0 end)
 as Vegetarian
from customer_orders
group by customer_id
order by customer_id
```
## What was the maximum number of pizzas delivered in a single order?
```sql
select ORDER_ID, count(PIZZA_ID) as tot_order
from customer_orders 
group by ORDER_ID
order by tot_order desc;
```
## For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
with cte as (select CUSTOMER_ID, EXCLUSIONS, EXTRAS, cancellation 
    from customer_orders c left join runner_orders r on c.order_id=r.order_id)
select CUSTOMER_ID,
    sum(case
    when EXCLUSIONS is not null or EXTRAS is not null  then 1 
    else 0 end)as total_pizzas_with_changes,
    sum(case
    when EXCLUSIONS is null and EXTRAS is null then 1 
    else 0 end) as total_pizzas_without_changes
from cte where cancellation is null or CANCELLATION = 'null' group by CUSTOMER_ID order by CUSTOMER_ID ;
```
## How many pizzas were delivered that had both exclusions and extras?
```sql
with cte as (select CUSTOMER_ID, EXCLUSIONS, EXTRAS, cancellation 
    from customer_orders c left join runner_orders r on c.order_id=r.order_id)
select CUSTOMER_ID,
    sum(case
    when EXCLUSIONS is not null AND EXTRAS is not null  then 1 
    else 0 end)as total_pizzas_with_changes
from cte where cancellation is null or CANCELLATION = 'null' group by CUSTOMER_ID order by CUSTOMER_ID ;

```
## What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT
	COUNT(order_id) AS order_count,
	HOUR(order_time) AS hour
FROM cust_orders
GROUP BY hour;
```
## What was the volume of orders for each day of the week?
```sql
WITH orders_by_day AS (
SELECT
	COUNT(order_id) AS order_count,
	WEEKDAY(order_time) AS day
FROM cust_orders
GROUP BY day
ORDER BY day
)

SELECT	
	order_count,
    CASE 
	WHEN day = 0 THEN 'Monday'
    WHEN day = 1 THEN 'Tuesday'
	WHEN day = 2 THEN 'Wednesday'
	WHEN day = 3 THEN 'Thursday'
	WHEN day = 4 THEN 'Friday'
	WHEN day = 5 THEN 'Saturday'
	WHEN day = 6 THEN 'Sunday'
   END AS day
FROM orders_by_day;
```

<h1>B. Runner and Customer Experience</h1>

## How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
SELECT COUNT(runner_id) AS runner_count, WEEK(registration_date) AS week
FROM runners GROUP BY week;
```
## What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
SELECT
    r.runner_id,
    AVG(MINUTE(TIMEDIFF(r.pick_up_time, c.order_time))) AS time_mins
FROM customer_orders c
LEFT JOIN runner_orders r
	ON c.order_id = r.order_id
GROUP BY r.runner_id;
```
## What was the average distance travelled for each customer?
```sql
SELECT
    c.CUSTOMER_ID,
    AVG(NVL(TO_NUMBER(REGEXP_REPLACE(r.DISTANCE, '[^0-9\.]', '')), 0)) AS avg_distance
FROM customer_orders c
LEFT JOIN runner_orders r
    ON c.order_id = r.order_id
GROUP BY c.CUSTOMER_ID order by CUSTOMER_ID;
```
## What was the difference between the longest and shortest delivery times for all orders?
```sql
with cte as (select NVL(TO_NUMBER(REGEXP_REPLACE(DURATION, '[^0-9\.]', '')), '')
AS minute from runner_orders)
select max(minute) - min(minute) as difference from cte
```
## What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
select runner_id,
AVG(NVL(TO_NUMBER(REGEXP_REPLACE(DISTANCE, '[^0-9\.]', '')), 0)) as dis,
AVG(NVL(TO_NUMBER(REGEXP_REPLACE(DURATION, '[^0-9\.]', '')), 0) ) as time
from runner_orders GROUP BY runner_id;
```
## What is the successful delivery percentage for each runner?
```sql

```
