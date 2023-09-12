<div align="center">
  <h1>Case Study 4: Data Bank</h1>
</div>

![image](https://github.com/maricsnel/WeeklySQLChallenge/assets/142982185/397e37cb-8fd5-415b-b947-cfce2c353d1b)


## Introduction

In the realm of modern banking, data-driven insights play a pivotal role in understanding customer behaviors and optimizing operational strategies. In this case study, we delve into the intricacies of the "Data Bank" system, exploring the dynamics of customer nodes and transaction activities. By dissecting the unique nodes across regions, analyzing transaction patterns, and uncovering trends in customer balance growth, we aim to unearth valuable findings and recommend strategic improvements for enhanced operational efficiency and customer engagement.

## Entity Relationship Diagram
![image](https://github.com/maricsnel/WeeklySQLChallenge/assets/142982185/fcc540f3-1a0f-4bbc-9680-15e7b5dc265b)


## A. Customer Nodes Exploration

### 1. How many unique nodes are there on the Data Bank system?
```sql
SELECT COUNT(DISTINCT node_id) as Nodes
FROM data_bank.customer_nodes;
```
- The `COUNT(DISTINCT node_id)` function is used to calculate the count of unique values in the "node_id" column of the "customer_nodes" table. This means that duplicate values are not counted; each unique value is counted only once.

| Nodes |
|-------|
|   5   |
### 2. What is the number of nodes per region?

```sql
SELECT Region_Name, COUNT(DISTINCT node_id) as Nodes
FROM data_bank.customer_nodes
JOIN data_bank.regions
ON customer_nodes.region_id = regions.region_id
GROUP BY Region_Name;
```
- `COUNT(DISTINCT node_id) as Nodes`: This column calculates the count of unique values in the "node_id" column of the "customer_nodes" table. Each unique value is counted only once, and the result column is labeled as "Nodes".

- The `JOIN data_bank.regions ON customer_nodes.region_id = regions.region_id` clause performs an inner join between the "customer_nodes" table and the "regions" table.
- The `GROUP BY Region_Name` clause groups the data by the values in the "Region_Name" column. 


| region_name | nodes |
|-------------|-------|
|   Africa    |   5   |
|   America   |   5   |
|    Asia     |   5   |
|  Australia  |   5   |
|   Europe    |   5   |
### 3. How many customers are allocated to each region?

```sql
SELECT Region_Name, COUNT(DISTINCT Customer_Id) as UniqueCustomers
FROM data_bank.customer_nodes
JOIN data_bank.regions
ON customer_nodes.region_id = regions.region_id
GROUP BY Region_Name
ORDER BY Region_Name;
```

- `COUNT(DISTINCT Customer_Id) as UniqueCustomers`: This column calculates the count of unique values in the "Customer_Id" column of the "customer_nodes" table. Each unique customer ID is counted only once, and the result column is labeled as "UniqueCustomers".

- The `JOIN data_bank.regions ON customer_nodes.region_id = regions.region_id` clause performs an inner join between the "customer_nodes" table and the "regions" table.

- The `GROUP BY Region_Name` clause groups the data by the values in the "Region_Name" column. 


Overall, this query retrieves the count of unique customers for each region by joining the "customer_nodes" and "regions" tables on the common "region_id" column, grouping the results by region, and ordering the results by region name.

This query calculates the number of customers allocated to each region.
| region_name | unique_customers |
|-------------|------------------|
|   Africa    |       102        |
|   America   |       105        |
|    Asia     |       95         |
|  Australia  |       110        |
|   Europe    |       88         |

### 4. How many days on average are customers reallocated to a different node?

```sql
SELECT AVG(end_date::date - start_date::date)
FROM data_bank.customer_nodes
WHERE end_date < DATE '9999-01-01';
```

- The `SELECT AVG(end_date::date - start_date::date)` statement calculates the average duration between the "end_date" and "start_date" columns for all rows in the "customer_nodes" table. The `::date` notation is used to explicitly cast the columns to the date data type.

- The `WHERE end_date < DATE '9999-01-01'` clause filters the rows of the "customer_nodes" table to only include those where the "end_date" is earlier than January 1, 9999.

|   avg    |
|----------|
| 14.6340  |

### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
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
- The `percentile_cont(0.xx) within group (ORDER BY end_date::date - start_date::date) as Median` calculates the median duration between "end_date" and "start_date" for rows in each region. The `percentile_cont` function computes a specified percentile value within a group of values ordered by the calculated duration.

- The `JOIN data_bank.regions ON regions.region_id = customer_nodes.region_id` clause performs an inner join between the "customer_nodes" and "regions" tables based on matching values in the "region_id" columns.

- The `WHERE end_date < DATE '9999-01-01'` clause filters the rows to only include those where the "end_date" is earlier than January 1, 9999.

- The `GROUP BY regions.region_name` clause groups the results by unique region names.


|  region  | median | 80th Percentile | 95th Percentile |
|----------|--------|-----------------|-----------------|
|  Africa  |   15   |       24        |       28        |
| America  |   15   |       23        |       28        |
|   Asia   |   15   |       23        |       28        |
| Australia|   15   |       23        |       28        |
|  Europe  |   15   |       24        |       28        |

## B. Customer Transactions

### 1. What is the unique count and total amount for each transaction type?
```sql
SELECT txn_type as Transaction_Type, 
  COUNT(txn_type) as Count_Of_Transactions,  
  SUM(txn_amount) as Total_Transaction_Amount
FROM data_bank.customer_transactions
GROUP BY txn_type;
```
- The `COUNT(txn_type) as Count_Of_Transactions` calculates the count of occurrences of each unique value in the "txn_type" column. The result column is labeled as "Count_Of_Transactions".

- The `SUM(txn_amount) as Total_Transaction_Amount` calculates the sum of values in the "txn_amount" column for each unique "txn_type". The result column is labeled as "Total_Transaction_Amount".

- The `GROUP BY txn_type` clause groups the results by unique values in the "txn_type" column. This means that the subsequent calculations will be performed separately for each distinct transaction type.


| transaction_type | count_of_transactions | total_transaction_amount |
|------------------|-----------------------|-------------------------|
|     purchase     |         1617          |          806537         |
|     deposit      |         2671          |         1359168         |
|    withdrawal    |         1580          |          793003         |

### 2. What is the average total historical deposit counts and amounts for all customers?
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
- The query uses a Common Table Expression (CTE) named "CTE" to define a temporary result set:
  - The `COUNT(customer_id) as Count_Of_Transactions` calculates the count of occurrences of each unique "customer_id" where the "Txn_type" is 'deposit'. The result column is labeled as "Count_Of_Transactions".
  - The `AVG(txn_amount) as AVG_Transaction_Amount` calculates the average value of "txn_amount" for each unique "customer_id" where the "Txn_type" is 'deposit'. The result column is labeled as "AVG_Transaction_Amount".
  - The `WHERE Txn_type = 'deposit'` clause filters the rows to only include those where the "Txn_type" is 'deposit'.
  - The `GROUP BY customer_ID` clause groups the results by unique "customer_id". This means that the subsequent calculations will be performed separately for each distinct customer ID.

- The main query uses the CTE "CTE" to aggregate the results from the CTE:
  - The `SELECT ROUND(AVG(Count_Of_Transactions), 2) as AVG_Deposits` statement calculates the average count of transactions for all customers and rounds the result to two decimal places. The result column is labeled as "AVG_Deposits".
  - The `ROUND(AVG(AVG_Transaction_Amount), 2) as AVG_Deposit_Amounts` calculates the average of the average transaction amounts for all customers and rounds the result to two decimal places. The result column is labeled as "AVG_Deposit_Amounts".
  
This query calculates the average deposit counts and amounts for all customers.
| avg_deposits | avg_deposit_amounts |
|--------------|---------------------|
|     5.34     |       508.61        |

### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
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
SELECT month, COUNT(Month)
FROM CTE2
WHERE dep > 1 AND (pur = 1 OR withd = 1)
GROUP BY Month;
```
  - CTE1 ("CTE"):
    - The `EXTRACT('month' FROM txn_date) as Month` extracts the month component from the "txn_date" column and labels it as "Month".
    - The subsequent `CASE` statements create binary indicators based on the "txn_type" column. For each transaction type ('deposit', 'purchase', 'withdrawal'), a value of 1 is assigned if the condition is met, otherwise, 0 is assigned.

  - CTE2 ("CTE2"):
    - The `SELECT month, customer_id` statement selects the "month" and "customer_id" columns.
    - The `SUM(deposit) as dep`, `SUM(purchase) as pur`, and `SUM(withdrawal) as withd` calculate the sums of the binary indicators for each transaction type ('deposit', 'purchase', 'withdrawal') for each unique combination of month and customer ID.
    - The `GROUP BY Month, Customer_ID` clause groups the results by month and customer ID.

- The main query:
  - The `SELECT COUNT(Month)` statement calculates the count of unique months where the conditions are met.
  - The `WHERE dep > 1 AND (pur = 1 OR withd = 1)` clause filters the results to only include rows where the count of 'deposit' transactions is greater than 1 and either 'purchase' or 'withdrawal' transactions are present.
  - The `GROUP BY Month` clause groups the results by month.

| month | count |
| ----- | ----- |
| 1     | 115   |
| 2     | 108   |
| 3     | 113   |
| 4     | 50    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2GtQz4wZtuNNu7zXH5HtV4/3)

### 4. What is the closing balance for each customer at the end of the month?
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
- The query uses a Common Table Expression (CTE) named "CTE" to pre-process the data:
  - The `EXTRACT('month' FROM txn_date) AS Month` extracts the month component from the "txn_date" column and labels it as "Month".
  - The `CASE` statement adjusts the transaction amount based on the "txn_type". If the transaction type is 'purchase' or 'withdrawal', the transaction amount is negated; otherwise, it's kept as is.

- The main query:
  - The `COALESCE` function is used to handle cases where there is no transaction data available for a customer in a specific month. It sums the adjusted transaction amounts and includes the previous month's ending balance (using the `LAG` window function) to calculate the ending balance for the current month.
  - The `GROUP BY` clause groups the results by "customer_id" and "Month".
  - The `ORDER BY` clause orders the results by "customer_id" and "Month".

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

### 5. What is the percentage of customers who increase their closing balance by more than 5%?
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
```

- CTE ("CTE"):
  - This CTE processes the raw transaction data from the "customer_transactions" table.
  - It extracts the "customer_id", extracts the month from "txn_date" as "Month", and calculates the "adjusted_amount" based on the transaction type.
  
- CTE2:
  - This CTE calculates the ending balance for each customer for each month.
  - The adjusted transaction amounts are summed along with the previous month's ending balance using the `LAG` window function.
  
- Balances:
  - This CTE extracts the opening balance (at month 1) and final ending balance (at month 4) for each customer.
  
- CTE3:
  - This CTE filters the customers who had an opening balance and a final ending balance, and where the final ending balance is at least 5% greater than the opening balance.
  
- The final query:
  - This query calculates the percentage of customers from CTE3 relative to the total number of distinct customers in the "customer_transactions" table.
  - It performs a full outer join between "customer_transactions" and CTE3 based on the "customer_id".
  - The percentage is calculated by dividing the count of distinct customer IDs in CTE3 by the count of distinct customer IDs in "customer_transactions".
  - The result is rounded to two decimal places and labeled as "Amount5%increase".

| Amount5%increase |
|------------------|
|       29.60      |




## Suggestions and conclusion

1. **Reallocation Impact**: Investigating the effects of customer reallocation on their transaction patterns could reveal interesting trends. For instance, understanding whether reallocated customers tend to change their transaction behavior post-relocation could lead to insights on adapting customer engagement strategies.
   
3. **Segmentation for Improvement**: Segmenting customers based on their transaction behaviors and balance growth could provide a clearer picture of the diverse customer base. This segmentation could guide targeted marketing efforts, financial education initiatives, and personalized services based on specific customer needs.
   
4. **Upgrade Timing**: Collecting customer feedback on the reallocation process and overall banking experience could highlight areas for improvement. Insights from customer opinions can guide refinements to the reallocation strategy, leading to enhanced customer satisfaction and loyalty.

In conclusion, the Data Bank case study has revealed several key findings that reflect the operational health of the system and positive customer behaviors. By delving deeper into these findings and implementing the suggested improvements, the bank can enhance its operational efficiency, customer engagement, and financial well-being offerings.

## Other Case Studies

[Case Study 1: Danny's Diner](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%201:%20Danny's%20Diner.md)

[Case Study 2: Pizza Runner](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%202:%20Pizza%20Runner.md)

[Case Study 3: Foodie FI](https://github.com/maricsnel/WeeklySQLChallenge/commit/529d6a8dd0998ebdfb0eb6eaf463361799aa4f76)

[Case Study 5: Data Mart](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%205:%20Data%20Mart.md)

[Case Study 6: Clique Bait](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%206%20-%20Clique%20Bait.md)

[Case Study 7: Balanced Tree Clothing Co](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%207:%20Balanced%20Tree%20Clothing%20Co..md)
