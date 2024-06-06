<h1>A. Customer Nodes Exploration</h1>

## How many unique nodes are there on the Data Bank system?
```sql
select count(DISTINCT node_id) AS unique_nodes from customer_nodes;
```
## What is the number of nodes per region?
```sql
select region_id, count(node_id) AS tot_nodes
from customer_nodes group by region_id order by REGION_ID;
```
## How many customers are allocated to each region?
```sql
select region_id, count(distinct CUSTOMER_ID) AS tot_customer_each_region
from customer_nodes group by region_id order by REGION_ID;
```
## How many days on average are customers reallocated to a different node?
```sql
select avg(END_DATE-START_DATE) AS avg_day from customer_nodes;
```
## What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
```sql
with cte as (select region_id, (END_DATE-START_DATE) AS diff_in_day from customer_nodes)
select region_id, PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY diff_in_day) OVER(PARTITION BY region_id) AS median,
PERCENTILE_CONT(0.8) WITHIN GROUP(ORDER BY diff_in_day) OVER(PARTITION BY region_id) AS "80th_percentile",
PERCENTILE_CONT(0.95) WITHIN GROUP(ORDER BY diff_in_day) OVER(PARTITION BY region_id) AS "95th_percentile"
from cte;
```
<h1>B. Customer Transactions</h1>

## What is the unique count and total amount for each transaction type?
```sql
select TXN_TYPE, count(CUSTOMER_ID) as unique_count , sum(TXN_AMOUNT) as TOT_TXN_AMOUNT
from customer_transactions group by TXN_TYPE;
```
## What is the average total historical deposit counts and amounts for all customers?
```sql
with cte as(select  CUSTOMER_ID, count(CUSTOMER_ID) as unique_count_for_deposit,
sum(TXN_AMOUNT) as AVG_TXN_AMOUNT_FOR_DEPOSIT
from customer_transactions where TXN_TYPE = 'deposit' group by CUSTOMER_ID)
select avg(unique_count_for_deposit) AS avg_deposit_count,
avg(AVG_TXN_AMOUNT_FOR_DEPOSIT) AS avg_total_deposit_amount
from cte
```
## For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
```
## What is the closing balance for each customer at the end of the month?
```sql
```
## What is the percentage of customers who increase their closing balance by more than 5%?
```sql
```
