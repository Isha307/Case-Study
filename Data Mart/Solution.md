# 1. Data Cleansing Steps
## Convert the week_date to a DATE format
```
ALTER TABLE weekly_sales ADD (new_week_date DATE);
UPDATE weekly_sales SET new_week_date = TO_DATE(week_date, 'YYYY-MM-DD');
ALTER TABLE weekly_sales DROP COLUMN week_date;
ALTER TABLE weekly_sales rename column new_week_date to week_date;
```
## Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
```
ALTER TABLE weekly_sales ADD (week_number INTEGER);
UPDATE weekly_sales SET week_number = TO_NUMBER(TO_CHAR(WEEK_DATE, 'WW')); --Using WW for week of the year
```
## Add a month_number with the calendar month for each week_date value as the 3rd column
```
ALTER TABLE weekly_sales ADD (month_number INTEGER);
UPDATE weekly_sales SET month_number = TO_NUMBER(EXTRACT(month from WEEK_DATE));
```
## Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
```
ALTER TABLE weekly_sales ADD (calendar_year INTEGER);
UPDATE weekly_sales SET calendar_year = TO_NUMBER(EXTRACT(year from WEEK_DATE));
```
## Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value
Segment | 1 | 2 | 3 or 4
--- | --- | --- | --- 
age_band  | Young Adults | Middle Aged | Retirees 
```
ALTER TABLE weekly_sales ADD (age_band VARCHAR(20))
UPDATE weekly_sales set age_band = substr(SEGMENT,2,2);
UPDATE weekly_sales
SET age_band = 
    CASE 
        WHEN age_band = '1' THEN 'Young Adults'
        WHEN age_band = '2' THEN 'Middle Aged'
        WHEN age_band IN ('3', '4') THEN 'Retirees'
    END;
```
## Add a new demographic column using the following mapping for the first letter in the segment values:
Segment | C | F 
--- | --- | --- 
demographic  | Couple | Families 
```
ALTER TABLE weekly_sales ADD (demographic VARCHAR(20))
UPDATE weekly_sales set demographic = substr(SEGMENT,1,1)
UPDATE weekly_sales
SET demographic = 
    CASE 
        WHEN demographic = 'C' THEN 'Couples'
        WHEN demographic = 'F' THEN 'Families'
    END;
```
## Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
```
UPDATE weekly_sales
SET segment = COALESCE(NULLIF(segment, ''), 'unknown'),
    age_band = COALESCE(NULLIF(age_band, ''), 'unknown'),
    demographic = COALESCE(NULLIF(demographic, ''), 'unknown');
```
## Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record
```
ALTER TABLE weekly_sales ADD (avg_transaction DECIMAL)
UPDATE weekly_sales set avg_transaction = ROUND(SALES/TRANSACTIONS,2)
```

# 2. Data Exploration
## What day of the week is used for each week_date value?
```
SELECT DISTINCT(TO_CHAR(week_date, 'day')) AS week_day 
FROM weekly_sales;
```
## What range of week numbers are missing from the dataset?
```
```
## How many total transactions were there for each year in the dataset?
```
SELECT EXTRACT(year from WEEK_DATE) as "YEAR", sum(sales) as total_transaction
FROM weekly_sales group by EXTRACT(year from WEEK_DATE);
```
## What is the total sales for each region for each month?
```
SELECT REGION, EXTRACT(month from WEEK_DATE) as "MONTH", sum(sales) as total_transaction
FROM weekly_sales group by REGION, EXTRACT(month from WEEK_DATE);
```
## What is the total count of transactions for each platform?
```
SELECT PLATFORM, count(TRANSACTIONS) as total_transaction
FROM weekly_sales group by PLATFORM;
```
## What is the percentage of sales for Retail vs Shopify for each month?
```
with cte as (SELECT PLATFORM, EXTRACT(year from WEEK_DATE) as "YEAR", EXTRACT(month from WEEK_DATE) as "MONTH", 
sum(TRANSACTIONS) as total_transaction
FROM weekly_sales group by PLATFORM, EXTRACT(year from WEEK_DATE), EXTRACT(month from WEEK_DATE))
select YEAR, MONTH,  ROUND(100 * MAX (CASE 
      WHEN platform = 'Retail' THEN total_transaction ELSE NULL END) 
    / SUM(total_transaction),2) AS retail_percentage,
    ROUND(100 * MAX (CASE 
      WHEN platform = 'Shopify' THEN total_transaction ELSE NULL END)
    / SUM(total_transaction),2) AS shopify_percentage
FROM cte
GROUP BY YEAR, MONTH
```
## What is the percentage of sales by demographic for each year in the dataset?
```
with cte as (SELECT REGION, EXTRACT(year from WEEK_DATE) as "YEAR",
sum(TRANSACTIONS) as total_transaction
FROM weekly_sales group by REGION, EXTRACT(year from WEEK_DATE))
select YEAR,  ROUND(100 * MAX (CASE 
      WHEN platform = 'AFRICA' THEN total_transaction ELSE NULL END)/ SUM(total_transaction),2) AS Africa_percentage,
    ROUND(100 * MAX (CASE 
      WHEN platform = 'ASIA' THEN total_transaction ELSE NULL END)/ SUM(total_transaction),2) AS ASIA_percentage,
    ROUND(100 * MAX (CASE 
      WHEN platform = 'CANADA' THEN total_transaction ELSE NULL END)/ SUM(total_transaction),2) AS CANADA_percentage,
    ROUND(100 * MAX (CASE 
      WHEN platform = 'SOUTH AMERICA' THEN total_transaction ELSE NULL END)/ SUM(total_transaction),2) SA_percentage,
    ROUND(100 * MAX (CASE 
      WHEN platform = 'OCEANIA' THEN total_transaction ELSE NULL END)/ SUM(total_transaction),2) AS shopify_percentage.
    ROUND(100 * MAX (CASE 
      WHEN platform = 'USA' THEN total_transaction ELSE NULL END)/ SUM(total_transaction),2) AS USA_percentage,
    ROUND(100 * MAX (CASE 
      WHEN platform = 'EUROPE' THEN total_transaction ELSE NULL END)/ SUM(total_transaction),2) AS EUROPE_percentage
FROM cte
GROUP BY YEAR
```
## Which age_band and demographic values contribute the most to Retail sales?
```
with cte as (SELECT DEMOGRAPHIC, EXTRACT(year from WEEK_DATE) as "YEAR",
sum(TRANSACTIONS) as total_transaction
FROM weekly_sales group by DEMOGRAPHIC, EXTRACT(year from WEEK_DATE))
select YEAR,  ROUND(100 * MAX (CASE 
      WHEN DEMOGRAPHIC = 'unknown' THEN total_transaction ELSE NULL END)/ SUM(total_transaction),2) AS unknown_percentage,
    ROUND(100 * MAX (CASE 
      WHEN DEMOGRAPHIC = 'Couples' THEN total_transaction ELSE NULL END)/ SUM(total_transaction),2) AS Couples_percentage,
    ROUND(100 * MAX (CASE 
      WHEN DEMOGRAPHIC = 'Families' THEN total_transaction ELSE NULL END)/ SUM(total_transaction),2) AS Families_percentage
FROM cte
GROUP BY YEAR
```
## Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
```
SELECT 
  EXTRACT(year from WEEK_DATE) as "YEAR",
  platform, 
  ROUND(AVG(avg_transaction),0) AS avg_transaction_row, 
  SUM(sales) / sum(transactions) AS avg_transaction_group
FROM weekly_sales
GROUP BY EXTRACT(year from WEEK_DATE), platform;
```
