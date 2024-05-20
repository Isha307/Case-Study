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
```
## For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
```
## How many pizzas were delivered that had both exclusions and extras?
```sql
```
## What was the total volume of pizzas ordered for each hour of the day?
```sql
```
## What was the volume of orders for each day of the week?
```sql
```
