## 1. What is the total amount each customer spent at the restaurant?
```sql
select customer_id, sum(price) as total from sales s  join  
menu m on m.product_id = s.product_id group by customer_id
```

## 2. How many days has each customer visited the restaurant?
```sql
select customer_id, count(DISTINCT order_date) as visit from sales group by customer_id 
```


## 3. What was the first item from the menu purchased by each customer?
```sql
with cte as (select customer_id, product_name, order_date, row_number() over(partition by customer_id order by order_date ) as rnk  
from sales join menu on sales.product_id = menu.product_id) 
select customer_id, product_name from cte where rnk=1 
```

## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT *  
FROM ( 
    SELECT product_name, COUNT(order_date) AS total_purchased  
    FROM menu  
    JOIN sales ON menu.product_id = sales.product_id  
    GROUP BY product_name  
    ORDER BY total_purchased DESC 
)  
WHERE ROWNUM = 1
```

## 5. Which item was the most popular for each customer?
```sql
with cte as
(select customer_id, product_name, count(sales.product_id) as cnt, rank() over(partition by CUSTOMER_ID order by count(sales.product_id) desc) 
as rank from sales join menu 
on sales.product_id = menu.product_id group by customer_id, product_name)
select customer_id, product_name,cnt from cte where rank=1
```

## 6. Which item was purchased first by the customer after they became a member?
```sql
with cte as (select m.customer_id, product_id,order_date, m.join_date, row_number() over(partition by m.CUSTOMER_ID order by order_date) as rnk from sales s join members m
on s.customer_id = m.customer_id where s.order_date>= m.join_date)
select CUSTOMER_ID, product_name, order_date, join_date from cte join menu 
on cte.product_id = menu.product_id where rnk=1;
```

## 7. Which item was purchased just before the customer became a member?
```sql
with cte as (select m.customer_id, product_id,order_date, m.join_date, row_number() over(partition by m.CUSTOMER_ID 
order by order_date desc) as rnk from sales s join members m
on s.customer_id = m.customer_id where s.order_date< m.join_date)
select CUSTOMER_ID, product_name, order_date, join_date from cte join menu 
on cte.product_id = menu.product_id where rnk=1;
```

## 8. What is the total items and amount spent for each member before they became a member?
```sql
select m.customer_id, count(s.product_id) as items, sum(price) as spent from sales s join members m
on s.customer_id = m.customer_id join menu 
on s.product_id = menu.product_id where s.order_date< m.join_date
group by m.customer_id
```

## 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
select customer_id, sum(
case when PRODUCT_NAME = 'sushi' then price*20
else price*10 end
) as spent from sales s join menu m
on s.product_id = m.product_id group by customer_id
```

## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
WITH dates_cte AS(
	SELECT *, 
		DATEADD(DAY, 6, join_date) AS valid_date, 
		EOMONTH('2021-01-1') AS last_date
	FROM members)
SELECT
	s.customer_id,
	sum(CASE
		WHEN s.product_id = 1 THEN price*20
		WHEN s.order_date between d.join_date and d.valid_date THEN price*20
		ELSE price*10 
	END) as total_points
FROM dates_cte d join sales s on d.customer_id = s.customer_id
	join menu m on m.product_id = s.product_id
WHERE s.order_date <= d.last_date
GROUP BY s.customer_id;
```

## 11. Join all the tables
```sql
select s.customer_id, order_date, product_name, price,
    case
     when s.order_date>=join_date then 'Y'
     else 'N' end
     as mem
FROM members m1 right join sales s on m1.customer_id = s.customer_id
	join menu m on m.product_id = s.product_id order by m1.customer_id, order_date
```

## 12. Rank All The Things
```sql
with cte as(select s.customer_id, order_date, product_name, price,
    case
     when s.order_date>=join_date then 'Y'
     else 'N' end
     as mem
FROM members m1 right join sales s on m1.customer_id = s.customer_id
	join menu m on m.product_id = s.product_id order by m1.customer_id, order_date)
SELECT
	*,
	CASE
		WHEN mem = 'N' THEN null
		ELSE
			dense_rank() over (partition by customer_id order by order_date)
	END as rank
FROM cte;
```
