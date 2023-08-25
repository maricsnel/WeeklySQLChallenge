<div align="center">
  <h1> Case Study 3: Foodie FI </h1>
</div>

![image](https://github.com/maricsnel/WeeklySQLChallenge/assets/142982185/eacb01c8-b3e0-4501-aa14-a1a202d96929)



## Introduction

In the realm of subscription-based culinary platforms, Foodie-Fi stands as a prime example. This case study delves into Foodie-Fi's customer engagement dynamics, subscription plan preferences, and retention strategies. By analyzing key data points, we uncover valuable insights that can guide strategic decisions to enhance customer experiences and drive business growth.

## Entity Relationship Diagram

![image](https://github.com/maricsnel/WeeklySQLChallenge/assets/142982185/e1c4c40e-86b1-472d-bd33-c9ffa18136ca)


## Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about 3 customers onboarding journey.
```sql
SELECT Customer_ID, start_date, plan_name 
FROM foodie_fi.subscriptions
Join foodie_fi.plans
On subscriptions.plan_ID = plans.plan_ID
Where customer_ID in (1, 2, 11, 13, 15, 16, 18, 19)
Order by Customer_id, start_date

```

Customer 1:Started with a trial on 2020-08-01.Subscribed to the "basic monthly" plan on 2020-08-08.Ongoing plan with no churn record.

Customer 2:Started with a trial on 2020-09-20.Subscribed to the "pro annual" plan on 2020-09-27.Ongoing plan with no churn record.

Customer 11:Started with a trial on 2020-11-19.Churned on 2020-11-26, but the plan continued until the end of the billing period.

## Data Analysis Questions:

### 1. How many customers has Foodie-Fi ever had?
```sql
SELECT COUNT(DISTINCT Customer_ID)
FROM foodie_fi.subscriptions;
```
- The `COUNT(DISTINCT Customer_ID)` function calculates the number of unique customer IDs in the "subscriptions" table.
  
| count    |
|----------|
| 1000    |

### 2. What is the monthly distribution of trial plan start_date values for our dataset?
```sql
SELECT DATE_TRUNC('MONTH', subscriptions.Start_date) AS month, COUNT(subscriptions.Plan_ID)
FROM foodie_fi.subscriptions
WHERE subscriptions.plan_id = 0
GROUP BY month
ORDER BY month;
```

- The `DATE_TRUNC('MONTH', subscriptions.Start_date)` function is used to truncate the dates in the "Start_date" column to the beginning of their respective months. This creates a new column labeled as "month" in the output.

- The `COUNT(subscriptions.Plan_ID)` function calculates the count of rows where the "Plan_ID" column has a value.
  
- The `WHERE` clause filters the rows to only include those where the "Plan_ID" is 0.


| month                        | count |
|------------------------------|-------|
| 2020-01-01T00:00:00.000Z      | 88    |
| 2020-02-01T00:00:00.000Z      | 68    |
| 2020-03-01T00:00:00.000Z      | 94    |
| 2020-04-01T00:00:00.000Z      | 81    |
| 2020-05-01T00:00:00.000Z      | 88    |
| 2020-06-01T00:00:00.000Z      | 79    |
| 2020-07-01T00:00:00.000Z      | 89    |
| 2020-08-01T00:00:00.000Z      | 88    |
| 2020-09-01T00:00:00.000Z      | 87    |
| 2020-10-01T00:00:00.000Z      | 79    |
| 2020-11-01T00:00:00.000Z      | 75    |
| 2020-12-01T00:00:00.000Z      | 84    |
### 3. What plan start_date values occur after the year 2020?
```sql
SELECT plan_id, COUNT(Plan_id)
FROM foodie_fi.subscriptions
WHERE start_date > '2020-12-31'
GROUP BY plan_id
ORDER BY plan_id;
```

- The `COUNT(Plan_id)` function calculates the count of occurrences of each unique plan ID.

- The `WHERE start_date > '2020-12-31'` clause filters the data to include only rows where the "start_date" is after December 31, 2020.

| month                        | count |
|------------------------------|-------|
| 2020-01-01T00:00:00.000Z      | 88    |
| 2020-02-01T00:00:00.000Z      | 68    |
| 2020-03-01T00:00:00.000Z      | 94    |
| 2020-04-01T00:00:00.000Z      | 81    |
| 2020-05-01T00:00:00.000Z      | 88    |
| 2020-06-01T00:00:00.000Z      | 79    |
| 2020-07-01T00:00:00.000Z      | 89    |
| 2020-08-01T00:00:00.000Z      | 88    |
| 2020-09-01T00:00:00.000Z      | 87    |
| 2020-10-01T00:00:00.000Z      | 79    |
| 2020-11-01T00:00:00.000Z      | 75    |
| 2020-12-01T00:00:00.000Z      | 84    |
### 4. What is the customer count and percentage of customers who have churned?
```sql
SELECT 
    COUNT(DISTINCT Customer_ID) AS ChurnedCustomers,
    ROUND(COUNT(DISTINCT Customer_ID)::numeric / (SELECT COUNT(DISTINCT Customer_ID) FROM Foodie_fi.subscriptions) * 100, 1) AS percentage
FROM foodie_fi.subscriptions
WHERE plan_id = 4;
```

- The `COUNT(DISTINCT Customer_ID)` function calculates the count of unique customer IDs in the "subscriptions" table for customers who have churned (plan_id = 4).

- Inside the `ROUND` function, the division calculation `(COUNT(DISTINCT Customer_ID)::numeric / (SELECT COUNT(DISTINCT Customer_ID) FROM Foodie_fi.subscriptions) * 100)` calculates the percentage of churned customers out of the total distinct customer count.
  
- The `WHERE plan_id = 4` clause filters the data to include only rows where the "plan_id" is 4, representing churned customers.

| churnedcustomers | percentage    |
| 307             | 30.7          |

### 5. How many customers have churned after their initial free trial?
```sql
WITH CTE AS (
    SELECT *,
        LEAD(plan_id, 1) OVER (PARTITION BY  Customer_id) AS NextSubscription
    FROM foodie_FI.subscriptions
)
SELECT 
    ROUND(COUNT(plan_id)::float / (SELECT COUNT(DISTINCT Customer_ID) FROM Foodie_fi.subscriptions) * 100) AS "PercentageChurnAfterTrial",
    COUNT(plan_id) AS "ChurnAfterTrial"
FROM CTE
WHERE plan_id = 0 AND nextsubscription = 4;
```
- A Common Table Expression (CTE) named `CTE` is defined to enrich the "subscriptions" data with an additional column named "NextSubscription". The 'LEAD' is used to find out the next subscription.

- Inside the main `SELECT`:
  - The calculation `ROUND(COUNT(plan_id)::float / (SELECT COUNT(DISTINCT Customer_ID) FROM Foodie_fi.subscriptions) * 100)` calculates the percentage of customers who churned after a trial period. The percentage is rounded to the nearest whole number.
  - The calculation `COUNT(plan_id)` calculates the count of customers who churned after a trial period.

- The `WHERE plan_id = 0 AND nextsubscription = 4` clause filters the data to include only rows where the initial plan was 0 (trial period) and the subsequent subscription was 4 (indicating churn).


| PercentageChurnAfterTrial   | ChurnAfterTrial  |
|-----------------------------|------------------|
| 9                           | 92               |

### 6. What is the number and percentage of customer plans after their initial free trial?
```sql
WITH CTE AS (
    SELECT *,
        ROW_NUMBER() OVER (PARTITION BY Customer_Id ORDER BY Start_Date ASC) AS Row
    FROM foodie_FI.subscriptions
)
SELECT 
    Plan_Id, 
    COUNT(Plan_ID), 
    COUNT(Plan_ID)::float / (SELECT COUNT(DISTINCT Customer_ID) FROM Foodie_fi.subscriptions) * 100 AS "PercentagePerPlan"
FROM CTE
WHERE Row = 2
GROUP BY Plan_ID;
```

- A Common Table Expression (CTE) named `CTE` is defined to assign a row number to each subscription record for every customer. The `ROW_NUMBER()` function is used with the `PARTITION BY Customer_Id` to number rows for each customer separately, and `ORDER BY Start_Date ASC` to order them by start date in ascending order.

- Inside the main `SELECT`:
  - The `Plan_Id` column is selected, representing the plan ID of each subscription.
  - The calculation `COUNT(Plan_ID)` counts the number of subscriptions for each unique plan ID.
  - The calculation `COUNT(Plan_ID)::float / (SELECT COUNT(DISTINCT Customer_ID) FROM Foodie_fi.subscriptions) * 100` calculates the percentage of subscriptions for each plan relative to the total distinct customer count.
  - 
- The `WHERE Row = 2` clause filters the data to include only the second subscription for each customer.


| plan_id               | count | PercentagePerPlan|
|-----------------------|-------|------------------|
| 1                     | 546   | 54.60            |
| 2                     | 325   | 32.50            |
| 3                     | 37    | 3.70             |
| 4                     | 92    | 9.20             |

### 7. What is the customer count and percentage breakdown of all plan_name values at 2020-12-31?
```sql
WITH CTE AS (
    SELECT *,
        ROW_NUMBER() OVER (PARTITION BY Customer_Id ORDER BY Start_Date DESC) AS Row
    FROM foodie_FI.subscriptions
    WHERE start_date < DATE '2021-01-01'
)
SELECT 
    Plan_ID, 
    COUNT(Plan_ID),
    COUNT(Plan_ID)::float / (SELECT COUNT(DISTINCT Customer_ID) FROM CTE) * 100 AS "PercentagePerPlan"
FROM CTE
WHERE Row = 1
GROUP BY Plan_ID;
```
- A Common Table Expression (CTE) named `CTE` is defined to assign a row number to each subscription record for every customer. The `ROW_NUMBER()` function is used with the `PARTITION BY Customer_Id` to number rows for each customer separately, and `ORDER BY Start_Date DESC` to order them by start date in descending order.

- The `WHERE start_date < DATE '2021-01-01'` clause filters the data to include only subscription records with start dates before January 1, 2021.

- Inside the main `SELECT`:
  - The calculation `COUNT(Plan_ID)` counts the number of subscriptions for each unique plan ID.
  - The calculation `COUNT(Plan_ID)::float / (SELECT COUNT(DISTINCT Customer_ID) FROM CTE) * 100` calculates the percentage of subscriptions for each plan relative to the total distinct customer count from the filtered CTE. 

- The `WHERE Row = 1` clause filters the data to include only the most recent subscription for each customer.


| plan_id               | count | Percentage_Per_Plan |
|-----------------------|-------|-------------------|
| 0                     | 19    | 1.90              |
| 1                     | 224   | 22.40             |
| 2                     | 326   | 32.60             |
| 3                     | 195   | 19.50             |
| 4                     | 236   | 23.60             |

### 8. How many customers have upgraded to an annual plan in 2020?
```sql
SELECT 
    COUNT(plan_id) AS "UpgradeAnnual"
FROM foodie_FI.subscriptions
WHERE plan_id = 3 AND start_date BETWEEN DATE '2020-01-01' AND DATE '2020-12-31';
```

- The calculation `COUNT(plan_id)` counts the number of subscription records.

- The `WHERE plan_id = 3` clause filters the data to include only subscription records with a plan ID of 3.

- The `start_date BETWEEN DATE '2020-01-01' AND DATE '2020-12-31'` clause further filters the data to include only subscription records with start dates within the range of January 1, 2020, to December 31, 2020.

| UpgradeAnnual |
|------------|
| 195   |
### 9. How many days on average does it take for a customer to upgrade to an annual plan?
```sql
WITH Join_Date AS (
    SELECT Customer_ID, MIN(Start_Date) AS Joined
    FROM foodie_FI.subscriptions
    GROUP BY Customer_ID
),
Upgrade AS (
    SELECT Customer_ID, Start_Date AS Upgrade
    FROM foodie_FI.subscriptions
    WHERE plan_ID = 3
)
SELECT AVG((upgrade.upgrade::date - join_date.joined::date)::integer) AS AverageUpgradeDays
FROM Join_Date
JOIN Upgrade ON Join_Date.Customer_ID = Upgrade.Customer_ID;
```

- Two Common Table Expressions (CTEs) named `Join_Date` and `Upgrade` are defined to organize the data for analysis.

- In the `Join_Date` CTE:
  - The `MIN(Start_Date)` function calculates the earliest subscription start date for each customer.
  - This CTE compiles a list of customers along with their corresponding earliest subscription start date (the date they joined).

- In the `Upgrade` CTE:
  - The `Start_Date` column is selected as "Upgrade" for each subscription record with a plan ID of 3.
  - This CTE compiles a list of customers who performed an upgrade to plan ID 3 along with the respective upgrade date.

- The calculation `(upgrade.upgrade::date - join_date.joined::date)::integer` computes the difference in days between the upgrade date and join date for each customer.

- The `JOIN Upgrade ON Join_Date.Customer_ID = Upgrade.Customer_ID` clause joins the data with the `Upgrade` CTE based on matching customer IDs.

- The calculation `AVG(...)` calculates the average of the computed differences in days for all customers who upgraded to plan ID 3.


| average_upgrade_days |
|-------------------|
| 104.62 |

### 10. What is the average time taken to upgrade broken down into 30-day periods?
```sql
With Join_Date as (
  Select Customer_ID, Min(Start_Date) as Joined
  From foodie_FI.subscriptions
  Group by (Customer_ID)
),
  Upgrade as (
  Select Customer_ID, Start_Date as Upgrade
  From foodie_FI.subscriptions
  Where plan_ID = 3 )
Select 
CASE
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 0 AND 30 THEN '0-30 days'
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 31 AND 60 THEN '31-60 days'
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 61 AND 90 THEN '61-90 days'
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 91 AND 120 THEN '91-120 days'
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 121 AND 150 THEN '121-150 days'
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 151 AND 180 THEN '151-180 days'
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 181 AND 210 THEN '181-210 days'
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 211 AND 240 THEN '211-240 days'
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 241 AND 270 THEN '241-270 days'
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 271 AND 300 THEN '271-300 days'
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 301 AND 330 THEN '301-330 days'
WHEN (upgrade.upgrade::date - join_date.joined::date) BETWEEN 331 AND 360 THEN '331-360 days'
  END AS Period,
Count (Join_Date.Customer_ID)
from Join_Date
Join Upgrade ON Join_Date.Customer_ID = Upgrade.Customer_ID
Group by Period;
```

- Two Common Table Expressions (CTEs) named `Join_Date` and `Upgrade` are defined to organize the data for analysis.

- In the `Join_Date` CTE:
  - This CTE compiles a list of customers along with their corresponding earliest subscription start date (the date they joined).

- In the `Upgrade` CTE:
  - This CTE compiles a list of customers who performed an upgrade to plan ID 3 along with the respective upgrade date.

- For each customer's upgrade, the conditions within the `CASE` expression determine which period category the upgrade falls into (e.g., '0-30 days', '31-60 days', and so on).

- The `Count (Join_Date.Customer_ID)` calculates the number of customers who fall into each period category.

- The `JOIN Upgrade ON Join_Date.Customer_ID = Upgrade.Customer_ID` clause joins the data with the `Upgrade` CTE based on matching customer IDs.

| period            | count |
|-------------------|-------|
| 0-30 days         | 49    |
| 121-150 days      | 42    |
| 151-180 days      | 36    |
| 181-210 days      | 26    |
| 211-240 days      | 4     |
| 241-270 days      | 5     |
| 271-300 days      | 1     |
| 301-330 days      | 1     |
| 31-60 days        | 24    |
| 331-360 days      | 1     |
| 61-90 days        | 34    |
| 91-120 days       | 35    |

### 11.	How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql
With CTE as(
	SELECT *,
    LEAD(plan_id, 1) Over (partition by Customer_ID) as NextSubscription
 From foodie_FI.subscriptions
Where start_date < date '2021-01-01')
   
Select Count(Customer_Id)
From CTE
Where plan_ID = 2 and nextsubscription = 1
```

- A Common Table Expression (CTE) named `CTE` is defined to enrich the "subscriptions" data with an additional column named "NextSubscription." This column holds the value of the "plan_id" for the subsequent subscription of the same customer using the `LEAD` window function. The `PARTITION BY Customer_ID` ensures that the function operates on each customer's data.

- The `WHERE start_date < date '2021-01-01'` clause filters the data to include only subscription records with start dates before January 1, 2021.

- Inside the main `SELECT`:
  - The `Count(Customer_Id)` function calculates the number of customers who match the specified conditions.
  
- The `WHERE plan_ID = 2 and nextsubscription = 1` clause filters the data to include only rows where the initial plan was 2 and the subsequent subscription was 1.

| count |
|-------|
| 0     |


## Key Findings and conclusion

1. **Popular Plans**: The "basic monthly" plan is the preferred choice for most customers, followed by the "pro monthly" plan. This indicates a demand for flexibility in commitment. Offering more diverse plans or personalized add-ons can cater to different customer preferences.

2. **Annual Commitments**: A significant percentage (19.5%) upgraded to annual plans. Capitalize on this interest by introducing more annual subscription benefits such as exclusive content, discounts, or early access, enticing more customers to commit for longer periods.
   
3. **Upgrade Timing**: The faster upgrade pattern within the initial 30 days suggests that customers are eager to explore and invest in the platform early on. Enhance the onboarding experience, provide clear plan differentiators, and highlight success stories during this crucial period to encourage swift upgrades.

In this Foodie-Fi case exploration, we've unearthed vital insights from customer onboarding and subscription data. The findings emphasize the significance of timely conversions after trials, the popularity of flexible plans, and the allure of longer commitments. By leveraging these insights, Foodie-Fi can refine its strategies, tailor plans, and elevate user experiences to foster sustained growth and customer satisfaction.


