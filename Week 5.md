<div align="center">
  <h1>Danny’s Diner SQL Case Study</h1>
  <p>Exploring Customer Behavior and Menu Insights Using SQL</p>
  <img src="CS1.png" alt="Danny's Diner">
</div>

# Case Study 1: Danny’s Diner

## Introduction

## 1. Data Cleansing Steps
```sql
CREATE TEMPORARY TABLE Weekly_Sales_Clean AS 
SELECT
  TO_DATE(week_date, 'dd-mm-yy') AS week_date,
  EXTRACT('week' FROM TO_DATE(week_date, 'dd-mm-yy')) AS week_number,
  EXTRACT('MONTH' FROM TO_DATE(week_date, 'dd-mm-yy')) AS month_number,
  EXTRACT('year' FROM TO_DATE(week_date, 'dd-mm-yy')) AS calendar_year,
  CASE 
    WHEN segment LIKE '%1%' THEN 'Young Adults'
    WHEN segment LIKE '%2%' THEN 'Middle Aged'
    WHEN segment LIKE '%3%' OR segment LIKE '%4%' THEN 'Retirees'
    ELSE 'Unknown'
  END AS Age_Band,
  CASE 
    WHEN segment LIKE '%C%' THEN 'Couples'
    WHEN segment LIKE '%F%' THEN 'Family' 
    ELSE 'Unknown'
  END AS Demographics,
  ROUND(sales/transactions, 2) AS avg_transaction, 
  Sales,
  Transactions,
  Customer_type,
  Region,
  Platform
FROM data_mart.weekly_sales;
```

Explanation: This query creates a temporary table ```Weekly_Sales_Clean``` by selecting and transforming columns from the ```data_mart.weekly_sales``` table. The query converts date formats, categorizes age bands and demographics, calculates average transaction, and maintains other relevant columns.

## 2. Data Exploration

### 1. What day of the week is used for each ```week_date``` value?
```sql
SELECT DISTINCT(To_Char("week_date", 'DAY')) 
FROM Weekly_Sales_Clean;
```

This query retrieves the distinct days of the week corresponding to each ```week_date``` value in the ```Weekly_Sales_Clean``` table.

### 2. What range of week numbers are missing from the dataset?
```sql
WITH AllWeekNumbers AS (
  SELECT week_number
  FROM generate_series(1, 52) AS week_number
)
SELECT awn.week_number
FROM AllWeekNumbers awn
LEFT JOIN Weekly_Sales_Clean wsc ON awn.week_number = EXTRACT('week' FROM wsc.week_date)
WHERE wsc.week_number IS NULL
ORDER BY awn.week_number;
```

This query identifies the missing week numbers within the range 1 to 52 by generating all week numbers and left joining them with the actual data. It then retrieves the week numbers that are missing from the ```Weekly_Sales_Clean``` table.

### 3. How many total transactions were there for each year in the dataset?
```sql
SELECT calendar_year, COUNT(Transactions)
FROM Weekly_Sales_Clean
GROUP BY calendar_year
ORDER BY calendar_year;
```

Explanation: This query calculates the total number of transactions for each calendar year in the dataset by grouping the data based on the ```calendar_year``` column.

### 4.What is the total sales for each region for each month?
```sql
SELECT Region, Month_number, SUM(Sales) AS Total_sales
FROM Weekly_Sales_Clean
GROUP BY Region, Month_number
ORDER BY Region, Month_number;
```

This query computes the total sales for each region and month by grouping the data based on ```Region``` and ```Month_number``` columns.

### 5. What is the total count of transactions for each platform?
```sql
SELECT platform, COUNT(Transactions) AS Total_transactions
FROM Weekly_Sales_Clean
GROUP BY platform;
```

Explanation: This query calculates the total count of transactions for each platform by grouping the data based on the ```platform``` column.

### 6.What is the percentage of sales for Retail vs Shopify for each month?
```sql
WITH CTE AS (
  SELECT 
    calendar_year, 
    month_number,
    Platform, 
    SUM(sales) AS TotalSales
  FROM Weekly_Sales_Clean
  GROUP BY calendar_year, month_number, Platform
),
CTE2 AS (
  SELECT 
    SUM(totalsales) AS combinedsales, 
    month_number, 
    calendar_year
  FROM CTE
  GROUP BY calendar_year, month_number
)
SELECT 
  cte.calendar_year, 
  cte.month_number, 
  Platform, 
  ROUND(totalsales/combinedsales*100, 2) AS Percentage
FROM CTE
JOIN CTE2 ON cte.month_number = cte2.month_number AND cte.calendar_year = cte2.calendar_year
ORDER BY calendar_year, month_number, Platform;
```

This query calculates the percentage of sales for each platform (Retail and Shopify) for each month and year. It uses Common Table Expressions (CTEs) to first calculate total sales for each platform, then calculate combined sales for each month, and finally computes the percentage of sales for each platform based on the combined sales.

### 7. What is the percentage of sales by demographic for each year in the dataset?
```sql
WITH CTE AS (
  SELECT 
    calendar_year,
    demographics, 
    SUM(sales) AS TotalSales
  FROM Weekly_Sales_Clean
  GROUP BY calendar_year, demographics
),
CTE2 AS (
  SELECT 
    SUM(totalsales) AS combinedsales, 
    calendar_year
  FROM CTE
  GROUP BY calendar_year
)
SELECT 
  cte.Calendar_year, 
  demographics, 
  ROUND(totalsales/combinedsales*100, 2) AS Percentage
FROM CTE
JOIN CTE2 ON cte.calendar_year = cte2.calendar_year
ORDER BY calendar_year, demographics;
```

This query calculates the percentage of sales for each demographic group for each year. It uses CTEs to first calculate total sales for each demographic, then calculate combined sales for each year, and finally computes the percentage of sales for each demographic based on the combined sales.

### 8. Which ```age_band``` and ```demographic``` values contribute the most to Retail sales?
```sql
SELECT 
  Age_band, 
  demographics,
  SUM(sales) AS Retail_Sales
FROM Weekly_Sales_Clean
WHERE platform = 'Retail'
GROUP BY Age_band, demographics
ORDER BY retail_sales DESC;
```

This query identifies the age bands and demographic values that contribute the most to Retail sales. It filters the data for Retail platform, groups it by ```Age_band``` and ```demographics```, and calculates the sum of sales for each group, then sorts the results in descending order of Retail sales.

### 9. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?sql
```sql
SELECT week_number
FROM weekly_sales_clean
WHERE week_date = '2020-06-15' 
  AND calendar_year = '2020'
LIMIT 1;

WITH Total_Sales AS (
  SELECT 
    week_date, 
    week_number, 
    SUM(sales) AS total_sales
  FROM weekly_sales_clean
  WHERE (week_number BETWEEN 21 AND 28) 
    AND (calendar_year = 2020)
  GROUP BY week_date, week_number
),
before_after_sales AS (
  SELECT 
```
