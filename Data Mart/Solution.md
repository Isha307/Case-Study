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
