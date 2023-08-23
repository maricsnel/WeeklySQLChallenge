<div align="center">
  <h1>Foodie FI SQL Case Study</h1>
  <img src="CS1.png" alt="Danny's Diner">
</div>

# Case Study 2: Foodie FI

## Introduction
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

This query counts the number of distinct customer IDs in the subscriptions table, which gives the total number of customers Foodie-Fi has ever had.
| count    |
|----------|
| 1000                 |

### 2. What is the monthly distribution of trial plan start_date values for our dataset?
```sql
SELECT DATE_TRUNC('MONTH', subscriptions.Start_date) AS month, COUNT(subscriptions.Plan_ID)
FROM foodie_fi.subscriptions
WHERE subscriptions.plan_id = 0
GROUP BY month
ORDER BY month;
```

This query groups the trial plan start dates by month and counts the number of trial plans started in each month. It helps visualize the distribution of trial plan sign-ups over time.
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

This query identifies plans that were started after the year 2020 and provides a breakdown of how many plans of each type were started in that period.
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

This query calculates the number and percentage of customers who have churned (subscribed to plan_id 4), providing insights into the customer retention rate.
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
| PercentageChurnAfterTrial   | ChurnAfterTrial  |
|-----------------------------|------------------|
| 9                           | 92               |

This query calculates the percentage and count of customers who have churned after their initial free trial period. It uses a common table expression (CTE) to identify the next subscription plan for each customer and then calculates the churn rate for those who moved from trial to churn.

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

This query calculates the number and percentage of customers who subscribed to different plans immediately after their initial free trial. It uses a CTE to assign row numbers based on the start date for each customer, helping to identify the second subscription (after the trial) and then calculates the distribution of plans.
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

This query calculates the customer count and percentage breakdown of all plan_name values at the end of the year 2020. It uses a CTE to filter subscriptions before January 1, 2021, and then groups and calculates the distribution of plans based on the latest subscription for each customer.
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

This query calculates the number of customers who upgraded to an annual plan in the year 2020. It specifically counts subscriptions with plan_id 3 (annual plan) and falling within the specified date range.
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

This query calculates the average number of days it takes for a customer to upgrade to an annual plan from the day they join Foodie-Fi. It uses two CTEs to first find the join date and upgrade date for each customer, and then calculates the average difference in days.

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
This query analyzes customer subscription upgrades by categorizing the time between their subscription start date and upgrade date into specific periods (e.g., '0-30 days', '31-60 days'). It then counts the number of customers in each period who upgraded their plans. This provides insights into how quickly customers upgrade after joining.

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
In summary, the query counts the number of customers who had a subscription plan of 2 and followed it with a subscription plan of 1, considering only subscriptions that started before January 1, 2021. This helps analyze how many customers switched from plan 2 to plan 1 within a specified time frame.
| count |
|-------|
| 0     |
