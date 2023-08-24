<div align="center">
  <h1>Danny’s Diner SQL Case Study</h1>
  <p>Exploring Customer Behavior and Menu Insights Using SQL</p>
  <img src="CS1.png" alt="Danny's Diner">
</div>

# Case Study 1: Danny’s Diner

## Introduction

## A. Customer Nodes Exploration

### 1. Count of Unique Nodes
```sql
SELECT COUNT(DISTINCT node_id) as Nodes
FROM data_bank.customer_nodes;
```

This query calculates the total number of unique nodes in the Data Bank system.
| Nodes |
|-------|
|   5   |
### 2. Number of Nodes per Region
```sql
SELECT Region_Name, COUNT(DISTINCT node_id) as Nodes
FROM data_bank.customer_nodes
JOIN data_bank.regions
ON customer_nodes.region_id = regions.region_id
GROUP BY Region_Name;
```

This query retrieves the count of unique nodes in each region.
| region_name | nodes |
|-------------|-------|
|   Africa    |   5   |
|   America   |   5   |
|    Asia     |   5   |
|  Australia  |   5   |
|   Europe    |   5   |
### 3. Customers Allocated to Each Region
```sql
SELECT Region_Name, COUNT(DISTINCT Customer_Id) as UniqueCustomers
FROM data_bank.customer_nodes
JOIN data_bank.regions
ON customer_nodes.region_id = regions.region_id
GROUP BY Region_Name
ORDER BY Region_Name;
```

This query calculates the number of customers allocated to each region.
| region_name | unique_customers |
|-------------|------------------|
|   Africa    |       102        |
|   America   |       105        |
|    Asia     |       95         |
|  Australia  |       110        |
|   Europe    |       88         |

### 4. Average Days for Customer Reallocation
```sql
SELECT AVG(end_date::date - start_date::date)
FROM data_bank.customer_nodes
WHERE end_date < DATE '9999-01-01';
```

This query calculates the average number of days customers are reallocated to a different node.
|   avg    |
|----------|
| 14.6340  |

### 5. Reallocation Days Percentiles by Region
```sql
SELECT regions.region_name as Region,
  percentile_cont(0.50) within group (ORDER BY end_date::date - start_date::date) as Median,
  percentile_cont(0.80) within group (ORDER BY end_date::date - start_date::date) as "80th Percentile",
  percentile_cont(0.95) within group (ORDER BY end_date::date - start_date::date) as "95th Percentile"
FROM data_bank.customer_nodes
JOIN data_bank.regions
ON regions.region_id = customer_nodes.region_id
WHERE end_date < DATE '9999-01-01'
GROUP BY regions.region_name;
```
|  region  | median | 80th Percentile | 95th Percentile |
|----------|--------|-----------------|-----------------|
|  Africa  |   15   |       24        |       28        |
| America  |   15   |       23        |       28        |
|   Asia   |   15   |       23        |       28        |
| Australia|   15   |       23        |       28        |
|  Europe  |   15   |       24        |       28        |

## B. Customer Transactions

### 1. Unique Count and Total Amount for Each Transaction Type
```sql
SELECT txn_type as Transaction_Type, 
  COUNT(txn_type) as Count_Of_Transactions,  
  SUM(txn_amount) as Total_Transaction_Amount
FROM data_bank.customer_transactions
GROUP BY txn_type;
```

This query provides the unique count and total amount for each transaction type.
| transaction_type | count_of_transactions | total_transaction_amount |
|------------------|-----------------------|-------------------------|
|     purchase     |         1617          |          806537         |
|     deposit      |         2671          |         1359168         |
|    withdrawal    |         1580          |          793003         |

### 2. Average Historical Deposit Counts and Amounts for All Customers
```sql
WITH CTE AS (
  SELECT customer_ID,
    COUNT(customer_id) as Count_Of_Transactions,  
    AVG(txn_amount) as AVG_Transaction_Amount
  FROM data_bank.customer_transactions
  WHERE Txn_type = 'deposit'
  GROUP BY customer_ID
)
SELECT ROUND(AVG(Count_Of_Transactions), 2) as AVG_Deposits, 
  ROUND(AVG(AVG_Transaction_Amount), 2) as AVG_Deposit_Amounts
FROM CTE;
```

This query calculates the average deposit counts and amounts for all customers.
| avg_deposits | avg_deposit_amounts |
|--------------|---------------------|
|     5.34     |       508.61        |

### 3. Customers with Multiple Deposits and Either 1 Purchase or 1 Withdrawalsql
```sql
WITH CTE AS (
  SELECT customer_id, 
    EXTRACT('month' FROM txn_date) as Month,
    CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END as deposit,
    CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END as purchase,
    CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END as withdrawal
  FROM data_bank.customer_transactions
),
CTE2 AS (
  SELECT month, customer_id, 
    SUM(deposit) as dep, 
    SUM(purchase) as pur, 
    SUM(withdrawal) as withd
  FROM CTE
  GROUP BY Month, Customer_ID
  ORDER BY Month, Customer_ID
)
SELECT COUNT(Month)
FROM CTE2
WHERE dep > 1 AND (pur = 1 OR withd = 1)
GROUP BY Month;
```

This query counts the number of customers who make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month.
| count |
|-------|
|  115  |
|  108  |
|  113  |
|  50   |

### 4. Closing Balance for Each Customer at Month-endsql
```sql
WITH CTE AS (
  SELECT 
    customer_id, 
    EXTRACT('month' FROM txn_date) AS Month, 
    CASE
      WHEN txn_type IN ('purchase', 'withdrawal') THEN -txn_amount
      ELSE txn_amount
    END AS adjusted_amount
  FROM data_bank.customer_transactions
)
SELECT 
  cust.customer_id, 
  months.Month, 
  COALESCE(SUM(cte.adjusted_amount), 0) 
    + CASE WHEN LAG(SUM(cte.adjusted_amount), 1) OVER (PARTITION BY cust.customer_id ORDER BY months.Month) IS NULL 
           THEN 0 
           ELSE LAG(SUM(cte.adjusted_amount), 1) OVER (PARTITION BY cust.customer_id ORDER BY months.Month) 
      END AS Ending_Balance
FROM 
  (SELECT DISTINCT customer_id FROM data_bank.customer_transactions) AS cust
CROSS JOIN 
  (SELECT DISTINCT EXTRACT('month' FROM txn_date) AS Month FROM data_bank.customer_transactions) AS months
LEFT JOIN 
  CTE AS cte 
ON 
  cust.customer_id = cte.customer_id AND months.Month = cte.Month
GROUP BY 
  cust.customer_id, months.Month
ORDER BY 
  cust.customer_id, months.Month;
```

This query calculates the closing balance for each customer at the end of each month.
| customer_id | month | ending_balance |
|-------------|-------|----------------|
|      1      |   1   |      312       |
|      1      |   2   |      312       |
|      1      |   3   |     -952       |
|      1      |   4   |     -952       |
|      2      |   1   |      549       |
|      2      |   2   |      549       |
|      2      |   3   |       61       |
|      2      |   4   |       61       |
|      3      |   1   |      144       |
|      3      |   2   |     -821       |

### 5. Percentage of Customers Increasing Balance by More Than 5%
```sql
WITH CTE AS (
  SELECT 
    customer_id, 
    EXTRACT('month' FROM txn_date) AS Month, 
    CASE
      WHEN txn_type IN ('purchase', 'withdrawal') THEN -txn_amount
      ELSE txn_amount
    END AS adjusted_amount
  FROM data_bank.customer_transactions
),

CTE2 AS (
  SELECT 
    cust.customer_id, 
    months.Month, 
    COALESCE(SUM(cte.adjusted_amount), 0) 
      + CASE WHEN LAG(SUM(cte.adjusted_amount), 1) OVER (PARTITION BY cust.customer_id ORDER BY months.Month) IS NULL 
             THEN 0 
             ELSE LAG(SUM(cte.adjusted_amount), 1) OVER (PARTITION BY cust.customer_id ORDER BY months.Month) 
        END AS Ending_Balance 
  FROM 
    (SELECT DISTINCT customer_id FROM data_bank.customer_transactions) AS cust
  CROSS JOIN 
    (SELECT DISTINCT EXTRACT('month' FROM txn_date) AS Month FROM data_bank.customer_transactions) AS months
  LEFT JOIN 
    CTE AS cte 
  ON 
    cust.customer_id = cte.customer_id AND months.Month = cte.Month
  GROUP BY 
    cust.customer_id, months.Month
  ORDER BY 
    cust.customer_id, months.Month
),
Balances as(
SELECT Customer_ID,
Case when month = 1 then ending_balance else 0 end as Opening_Balance,
Case when month = 4 then ending_balance else 0 end as Final_ending_Balance
From CTE2
),

CTE3 as (
SELECT b1.customer_id, b1.opening_balance, b2.final_ending_balance
FROM Balances as b1
Join Balances as b2
on b1.customer_id = b2.customer_id
WHERE b1.Opening_Balance != 0 and b2.final_ending_balance != 0 and 
b2.final_ending_balance > b1.Opening_Balance *1.05)

Select Round(Count(Distinct cte3.customer_id):: decimal/ Count(Distinct customer_transactions.customer_id)*100, 2) As "Amount5%increase"
From data_bank.customer_transactions
Full outer Join CTE3
ON customer_transactions.customer_id = cte3.customer_id

| Amount5%increase |
|------------------|
|       29.60      |

```
