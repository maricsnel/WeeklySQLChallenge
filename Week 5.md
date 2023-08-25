<div align="center">
  <h1>Case Study 5: Data Mart</h1>
</div>

![image](https://github.com/maricsnel/WeeklySQLChallenge/assets/142982185/ac536ac6-b8fe-4f41-8fc1-2fa5d781b408)

## Introduction

In the realm of retail analytics, we embark on an insightful journey through a comprehensive dataset capturing weekly sales dynamics. The dataset has undergone meticulous data cleansing, laying the foundation for meaningful exploration. Harnessing the power of SQL queries, we unravel pivotal patterns within this retail landscape, encompassing factors such as weekly day trends, crucial missing data points, yearly transaction trends, platform-driven sales, and the influential role of demographics. Notably, we delve into a specific juncture in time to gauge the impact on sales, shedding light on both quantitative growth or decline and the corresponding percentage shifts. This investigation offers a strategic lens into the retail sector's nuances and evolution.

## Entity Relationship Diagram

![image](https://github.com/maricsnel/WeeklySQLChallenge/assets/142982185/65fc2440-6565-4d1a-8bf5-acca71edb81b)

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

This query creates a temporary table ```Weekly_Sales_Clean``` by selecting and transforming columns from the ```data_mart.weekly_sales``` table. The query converts date formats, categorizes age bands and demographics, calculates average transaction, and maintains other relevant columns.

## 2. Data Exploration

### 1. What day of the week is used for each ```week_date``` value?
```sql
SELECT DISTINCT(To_Char("week_date", 'DAY')) as Day_Of_Week
FROM Weekly_Sales_Clean;
```

This query retrieves the distinct days of the week corresponding to each ```week_date``` value in the ```Weekly_Sales_Clean``` table.
| day_of_week |
|---------|
| MONDAY  |

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
| week_number |
|-------------|
| 1           |
| 2           |
| 3           |
| 4           |
| 5           |
| 6           |
| 7           |
| 8           |
| 9           |
| 10          |
| 11          |
| 12          |
| 37          |
| 38          |
| 39          |
| 40          |
| 41          |
| 42          |
| 43          |
| 44          |
| 45          |
| 46          |
| 47          |
| 48          |
| 49          |
| 50          |
| 51          |
| 52          |

### 3. How many total transactions were there for each year in the dataset?
```sql
SELECT calendar_year, COUNT(Transactions)
FROM Weekly_Sales_Clean
GROUP BY calendar_year
ORDER BY calendar_year;
```

This query calculates the total number of transactions for each calendar year in the dataset by grouping the data based on the ```calendar_year``` column.

| calendar_year | count |
|---------------|-------|
| 2018          | 5698  |
| 2019          | 5708  |
| 2020          | 5711  |

### 4.What is the total sales for each region for each month?
```sql
SELECT Region, Month_number, SUM(Sales) AS Total_sales
FROM Weekly_Sales_Clean
GROUP BY Region, Month_number
ORDER BY Region, Month_number;
```

This query computes the total sales for each region and month by grouping the data based on ```Region``` and ```Month_number``` columns.
| region       | month_number | total_sales  |
|--------------|--------------|--------------|
| AFRICA       | 3            | 567767480    |
| AFRICA       | 4            | 1911783504   |
| AFRICA       | 5            | 1647244738   |
| AFRICA       | 6            | 1767559760   |
| AFRICA       | 7            | 1960219710   |
| AFRICA       | 8            | 1809596890   |
| AFRICA       | 9            | 276320987    |
| ASIA         | 3            | 529770793    |
| ASIA         | 4            | 1804628707   |
| ASIA         | 5            | 1526285399   |
| ASIA         | 6            | 1619482889   |
| ASIA         | 7            | 1768844756   |
| ASIA         | 8            | 1663320609   |
| ASIA         | 9            | 252836807    |
| CANADA       | 3            | 144634329    |
| CANADA       | 4            | 484552594    |
| CANADA       | 5            | 412378365    |
| CANADA       | 6            | 443846698    |
| CANADA       | 7            | 477134947    |
| CANADA       | 8            | 447073019    |
| CANADA       | 9            | 69067959     |
| EUROPE       | 3            | 35337093     |
| EUROPE       | 4            | 127334255    |
| EUROPE       | 5            | 109338389    |
| EUROPE       | 6            | 122813826    |
| EUROPE       | 7            | 136757466    |
| EUROPE       | 8            | 122102995    |
| EUROPE       | 9            | 18877433     |
| OCEANIA      | 3            | 783282888    |
| OCEANIA      | 4            | 2599767620   |
| OCEANIA      | 5            | 2215657304   |
| OCEANIA      | 6            | 2371884744   |
| OCEANIA      | 7            | 2563459400   |
| OCEANIA      | 8            | 2432313652   |
| OCEANIA      | 9            | 372465518    |
| SOUTH AMERICA| 3            | 71023109     |
| SOUTH AMERICA| 4            | 238451531    |
| SOUTH AMERICA| 5            | 201391809    |
| SOUTH AMERICA| 6            | 218247455    |
| SOUTH AMERICA| 7            | 235582776    |
| SOUTH AMERICA| 8            | 221166052    |
| SOUTH AMERICA| 9            | 34175583     |
| USA          | 3            | 225353043    |
| USA          | 4            | 759786323    |
| USA          | 5            | 655967121    |
| USA          | 6            | 703878990    |
| USA          | 7            | 760331754    |
| USA          | 8            | 712002790    |
| USA          | 9            | 110532368    |
### 5. What is the total count of transactions for each platform?
```sql
SELECT platform, COUNT(Transactions) AS Total_transactions
FROM Weekly_Sales_Clean
GROUP BY platform;
```

Explanation: This query calculates the total count of transactions for each platform by grouping the data based on the ```platform``` column.
| platform | total_transactions |
|----------|--------------------|
| Shopify  | 8549               |
| Retail   | 8568               |

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
| calendar_year | month_number | platform | percentage |
|---------------|--------------|----------|------------|
| 2018          | 3            | Retail   | 97.92      |
| 2018          | 3            | Shopify  | 2.08       |
| 2018          | 4            | Retail   | 97.93      |
| 2018          | 4            | Shopify  | 2.07       |
| 2018          | 5            | Retail   | 97.73      |
| 2018          | 5            | Shopify  | 2.27       |
| 2018          | 6            | Retail   | 97.76      |
| 2018          | 6            | Shopify  | 2.24       |
| 2018          | 7            | Retail   | 97.75      |
| 2018          | 7            | Shopify  | 2.25       |
| 2018          | 8            | Retail   | 97.71      |
| 2018          | 8            | Shopify  | 2.29       |
| 2018          | 9            | Retail   | 97.68      |
| 2018          | 9            | Shopify  | 2.32       |
| 2019          | 3            | Retail   | 97.71      |
| 2019          | 3            | Shopify  | 2.29       |
| 2019          | 4            | Retail   | 97.80      |
| 2019          | 4            | Shopify  | 2.20       |
| 2019          | 5            | Retail   | 97.52      |
| 2019          | 5            | Shopify  | 2.48       |
| 2019          | 6            | Retail   | 97.42


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

This query calculates the percentage of sales for each demographic group for each year. It uses CTEs to first calculate total sales for each demographic, then calculate combined sales for each year, and finally compute the percentage of sales for each demographic based on the combined sales.
| calendar_year | demographics | percentage |
|---------------|--------------|------------|
| 2018          | Couples      | 26.38      |
| 2018          | Family       | 31.99      |
| 2018          | Unknown      | 41.63      |
| 2019          | Couples      | 27.28      |
| 2019          | Family       | 32.47      |
| 2019          | Unknown      | 40.25      |
| 2020          | Couples      | 28.72      |
| 2020          | Family       | 32.73      |
| 2020          | Unknown      | 38.55      |

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
| age_band      | demographics | retail_sales |
|---------------|--------------|--------------|
| Unknown       | Unknown      | 16067285533 |
| Retirees      | Family       | 6634686916  |
| Retirees      | Couples      | 6370580014  |
| Middle Aged   | Family       | 4354091554  |
| Young Adults  | Couples      | 2602922797  |
| Middle Aged   | Couples      | 1854160330  |
| Young Adults  | Family       | 1770889293  |

This query identifies the age bands and demographic values that contribute the most to Retail sales. It filters the data for Retail platform, groups it by ```Age_band``` and ```demographics```, and calculates the sum of sales for each group, then sorts the results in descending order of Retail sales.

### 9. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?sql
```sql
SELECT week_number
FROM weekly_sales_clean
WHERE week_date = '2020-06-15' 
  AND calendar_year = '2020'
Limit 1;

WITH Total_Sales AS (
  SELECT 
    week_date, 
    week_number, 
    SUM(sales) AS total_sales
  FROM weekly_sales_clean
  WHERE (week_number BETWEEN 21 AND 28) 
    AND (calendar_year = 2020)
  GROUP BY week_date, week_number
)
, before_after_sales AS (
  SELECT 
    SUM(CASE 
      WHEN week_number BETWEEN 21 AND 24 THEN total_sales END) AS before_sales,
    SUM(CASE 
      WHEN week_number BETWEEN 25 AND 28 THEN total_sales END) AS after_sales
  FROM total_sales
)

SELECT 
  after_sales - before_sales AS sales_growth, 
  ROUND(100 * 
    (after_sales - before_sales) 
    / before_sales,2) AS percentage_growth
FROM before_after_sales;
```
| sales_growth | percentage_growth |
|--------------|-------------------|
| -26884188    | -1.15             |

## Suggestions and Conclusion
1. **Data Enrichment**: Address the missing data gaps in weeks 1 to 12 and 37 to 52 by implementing more robust data collection processes. A comprehensive dataset will provide a clearer view of sales trends throughout the entire year.
   
2. **Day-Specific Strategies**: Given the consistent Monday sales trend, consider tailoring marketing and promotional efforts to capitalize on this day's higher customer engagement.
   
3. **Platform Analysis**: Investigate the reasons behind the decline in Shopify sales after the identified date. This could involve assessing changes in user experience, marketing initiatives, or competitor activities on that platform.
   
4. **Demographic Insights**: With the "Unknown" demographic group being a significant driver of sales, delve deeper into customer profiles within this category. Conduct surveys or data analytics to understand the characteristics and preferences of this group.
   
5. **Segmented Marketing**: Leverage demographic insights to create targeted marketing campaigns. Tailor messaging and offers to appeal to specific age bands and demographics, potentially boosting sales among previously untapped segments.
   

In this exploration of retail data, we've uncovered valuable insights that can drive strategic decisions. By addressing data gaps, tailoring strategies to day-of-week trends, and delving into demographics and platforms, opportunities for growth and optimization emerge. As the retail landscape continues to evolve, leveraging these findings can pave the way for enhanced customer engagement, strategic planning, and sustainable success.
