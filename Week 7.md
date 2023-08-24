<div align="center">
  <h1>Danny’s Diner SQL Case Study</h1>
  <p>Exploring Customer Behavior and Menu Insights Using SQL</p>
  <img src="CS1.png" alt="Danny's Diner">
</div>

# Case Study 1: Danny’s Diner

## Introduction

## High-Level Sales Analysis

### 1.What was the total quantity sold for all products?

```sql
SELECT product_name, SUM(QTY) AS Total_Quantity_Sold
FROM balanced_tree.sales
JOIN balanced_tree.product_details ON product_details.product_id = sales.prod_id
GROUP BY product_name;
```
This query calculates the total quantity sold for each product.

| product_name                  | total_quantity_sold |
|------------------------------|---------------------|
| White Tee Shirt - Mens        | 3800                |
| Navy Solid Socks - Mens       | 3792                |
| Grey Fashion Jacket - Womens  | 3876                |
| Navy Oversized Jeans - Womens | 3856                |
| Pink Fluro Polkadot Socks - Mens | 3770              |
| Khaki Suit Jacket - Womens    | 3752                |
| Black Straight Jeans - Womens | 3786                |
| White Striped Socks - Mens    | 3655                |
| Blue Polo Shirt - Mens        | 3819                |
| Indigo Rain Jacket - Womens   | 3757                |
| Cream Relaxed Jeans - Womens  | 3707                |
| Teal Button Up Shirt - Mens   | 3646                |

### 2.What is the total generated revenue for all products before discounts?

```sql
WITH ProductQuantities AS (
    SELECT product_name, SUM(QTY) AS Total_Quantity_Sold
    FROM balanced_tree.sales
    JOIN balanced_tree.product_details ON product_details.product_id = sales.prod_id
    GROUP BY product_name
)
SELECT
    pq.product_name,
    pq.Total_Quantity_Sold * pd.price AS Total_Revenue_Before_Discounts
FROM ProductQuantities AS pq
JOIN balanced_tree.product_details AS pd ON pd.product_name = pq.product_name
ORDER BY Total_Revenue_Before_Discounts DESC;
```
This query calculates the total revenue generated for each product before applying discounts.

| product_name                  | total_revenue_before_discounts |
|------------------------------|--------------------------------|
| Blue Polo Shirt - Mens        | 217683                         |
| Grey Fashion Jacket - Womens  | 209304                         |
| White Tee Shirt - Mens        | 152000                         |
| Navy Solid Socks - Mens       | 136512                         |
| Black Straight Jeans - Womens | 121152                         |
| Pink Fluro Polkadot Socks - Mens | 109330                       |
| Khaki Suit Jacket - Womens    | 86296                          |
| Indigo Rain Jacket - Womens   | 71383                          |
| White Striped Socks - Mens    | 62135                          |
| Navy Oversized Jeans - Womens | 50128                          |
| Cream Relaxed Jeans - Womens  | 37070                          |
| Teal Button Up Shirt - Mens   | 36460                          |

### 3.What was the total discount amount for all products?

```sql
SELECT
    product.product_name,
    SUM(sales.qty * sales.price * sales.discount / 100) AS Total_Discount_Amount
FROM balanced_tree.sales
JOIN balanced_tree.product_details AS product ON sales.prod_id = product.product_id
GROUP BY product.product_name;
```
This query calculates the total discount amount for each product based on the sales data.

| product_name                  | total_discount_amount |
|------------------------------|-----------------------|
| White Tee Shirt - Mens        | 17968                 |
| Navy Solid Socks - Mens       | 16059                 |
| Grey Fashion Jacket - Womens  | 24781                 |
| Navy Oversized Jeans - Womens | 5538                  |
| Pink Fluro Polkadot Socks - Mens | 12344                |
| Khaki Suit Jacket - Womens    | 9660                  |
| Black Straight Jeans - Womens | 14156                 |
| White Striped Socks - Mens    | 6877                  |
| Blue Polo Shirt - Mens        | 26189                 |
| Indigo Rain Jacket - Womens   | 8010                  |
| Cream Relaxed Jeans - Womens  | 3979                  |
| Teal Button Up Shirt - Mens   | 3925                  |

## Transaction Analysis

### 1.How many unique transactions were there?

```sql
SELECT COUNT(DISTINCT txn_id) AS Unique_Transactions_Count
FROM balanced_tree.sales;
```
This query counts the number of unique transactions.

| unique_transactions_count |
|--------------------------|
|            2500           |

### 2.What is the average unique products purchased in each transaction?

```sql
SELECT COUNT(prod_id) / COUNT(DISTINCT txn_id) AS Average_Unique_Products_Per_Transaction
FROM balanced_tree.sales;
```
This query calculates the average number of unique products purchased in each transaction.

| average_unique_products_per_transaction |
|----------------------------------------|
|                  6                     |

### 3.What are the 25th, 50th and 75th percentile values for the revenue per transaction?
```sql
WITH TransactionRevenue AS (
    SELECT
        TXN_ID,
        SUM(QTY * Price) AS Revenue
    FROM balanced_tree.sales
    GROUP BY TXN_ID
)
SELECT
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY Revenue) AS "25th Percentile",
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY Revenue) AS "50th Percentile",
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY Revenue) AS "75th Percentile"
FROM TransactionRevenue;
```
This query calculates the 25th, 50th, and 75th percentile values for the revenue per transaction.

| 25th Percentile | 50th Percentile | 75th Percentile |
|-----------------|-----------------|-----------------|
|      375.75     |      509.5      |       647       |

### 4.What is the average discount value per transaction?
```sql
SELECT
    ROUND(SUM((Price::numeric * QTY::numeric) * (discount::numeric / 100)) /
    COUNT(DISTINCT TXN_ID), 2) AS Average_Discount_Value_Per_Transaction
FROM balanced_tree.sales;
```
This query calculates the average discount value applied per transaction.

| average_discount_value_per_transaction |
|----------------------------------------|
|                62.49                   |

### 5.What is the percentage split of all transactions for members vs non-members?
```sql
WITH TransactionCounts AS (
    SELECT COUNT(DISTINCT TXN_ID) AS Total_Transactions
    FROM balanced_tree.sales
),
MemberTransactions AS (
    SELECT COUNT(DISTINCT TXN_ID) AS Member_Transactions
    FROM balanced_tree.sales
    WHERE Member = true
),
NonMemberTransactions AS (
    SELECT COUNT(DISTINCT TXN_ID) AS NonMember_Transactions
    FROM balanced_tree.sales
    WHERE Member = false
)
SELECT
    ROUND(MemberTransactions.Member_Transactions::numeric / TransactionCounts.Total_Transactions::numeric * 100, 2)
    AS Member_Transaction_Percentage,
    ROUND(NonMemberTransactions.NonMember_Transactions::numeric / TransactionCounts.Total_Transactions::numeric * 100, 2)
    AS NonMember_Transaction_Percentage
FROM TransactionCounts
CROSS JOIN MemberTransactions
CROSS JOIN NonMemberTransactions;
```
This query calculates the percentage split of transactions between members and non-members.
| member_transaction_percentage | nonmember_transaction_percentage |
|------------------------------|----------------------------------|
|             60.20            |              39.80               |

### 6.What is the average revenue for member transactions and non-member transactions?
```sql
WITH MemberRevenue AS (
    SELECT SUM(QTY * Price) / COUNT(DISTINCT TXN_ID) AS Average_Revenue_Members
    FROM balanced_tree.sales
    WHERE Member = true
),
NonMemberRevenue AS (
    SELECT SUM(QTY * Price) / COUNT(DISTINCT TXN_ID) AS Average_Revenue_NonMembers
    FROM balanced_tree.sales
    WHERE Member = false
)
SELECT
    MemberRevenue.Average_Revenue_Members,
    NonMemberRevenue.Average_Revenue_NonMembers
FROM MemberRevenue, NonMemberRevenue;
```
This query calculates the average revenue for member and non-member transactions.

| average_revenue_members | average_revenue_nonmembers |
|------------------------|--------------------------|
|          516           |           515            |

### Product Analysis
### 1.What are the top 3 products by total revenue before discount?
```sql
SELECT
    product_name,
    SUM(QTY * sales.Price) AS Revenue
FROM balanced_tree.sales
JOIN balanced_tree.product_details ON sales.prod_id = product_details.product_id
GROUP BY product_name
ORDER BY Revenue DESC
LIMIT 3;
```
This query retrieves the top 3 products based on their total revenue before discounts.

| product_name             | revenue |
|--------------------------|---------|
| Blue Polo Shirt - Mens   | 217683  |
| Grey Fashion Jacket - Womens | 209304 |
| White Tee Shirt - Mens   | 152000  |


### 2.What is the total quantity, revenue and discount for each segment?

```sql
SELECT
    segment_name,
    SUM(QTY) AS Total_Quantity,
    SUM(QTY * sales.Price) AS Total_Revenue,
    ROUND(SUM((QTY * sales.Price) * (discount / 100::numeric)), 2) AS Total_Discount
FROM balanced_tree.sales
JOIN balanced_tree.product_details ON sales.prod_id = product_details.product_id
GROUP BY segment_name;
```
This query calculates the total quantity, revenue, and discount for each segment.

| segment_name | total_quantity | total_revenue | total_discount |
|--------------|----------------|---------------|----------------|
| Shirt        | 11265          | 406143        | 49594.27       |
| Jeans        | 11349          | 208350        | 25343.97       |
| Jacket       | 11385          | 366983        | 44277.46       |
| Socks        | 11217          | 307977        | 37013.44       |

### 3.What is the top selling product for each segment?

```sql
WITH RankedProducts AS (
    SELECT
        segment_name,
        product_name,
        SUM(QTY) AS Total_Quantity,
        RANK() OVER (PARTITION BY segment_name ORDER BY SUM(QTY) DESC) AS Rank
    FROM balanced_tree.sales
    JOIN balanced_tree.product_details ON sales.prod_id = product_details.product_id
    GROUP BY segment_name, product_name
)
SELECT
    segment_name,
    product_name,
    total_quantity
FROM RankedProducts
WHERE Rank = 1;
```
This query identifies the top selling product for each segment.

| segment_name | product_name                | total_quantity |
|--------------|-----------------------------|----------------|
| Jacket       | Grey Fashion Jacket - Womens | 3876           |
| Jeans        | Navy Oversized Jeans - Womens | 3856           |
| Shirt        | Blue Polo Shirt - Mens      | 3819           |
| Socks        | Navy Solid Socks - Mens     | 3792           |

### 4.	What is the total quantity, revenue and discount for each category?

```sql
SELECT
    category_name,
    SUM(QTY) AS Total_Quantity,
    SUM(QTY * sales.Price) AS Total_Revenue,
    ROUND(SUM((QTY * sales.Price) * (discount / 100::numeric)), 2) AS Total_Discount
FROM balanced_tree.sales
JOIN balanced_tree.product_details ON sales.prod_id = product_details.product_id
GROUP BY category_name;
```
This query calculates the total quantity, revenue, and discount for each category.

| category_name | total_quantity | total_revenue | total_discount |
|---------------|----------------|---------------|----------------|
| Mens          | 22482          | 714120        | 86607.71       |
| Womens        | 22734          | 575333        | 69621.43       |

### 5.What is the top selling product for each category?
```sql
WITH Ranked AS (
    SELECT 
        category_name, 
        product_name, 
        SUM(QTY) as Total_Quantity,
        RANK() OVER (PARTITION BY category_name ORDER BY SUM(QTY) DESC) as Ranks 
    FROM balanced_tree.sales
    JOIN balanced_tree.product_details ON sales.prod_id = product_details.product_id
    GROUP BY category_name, product_name)

SELECT 
    category_name, 
    product_name, 
    total_quantity
FROM Ranked
WHERE Ranks = 1;
```
This query uses a Common Table Expression (CTE) named ```Ranked``` to rank products within each category based on their total quantities sold. The ```RANK()``` function is used for ranking. Then, the main query selects the products with a rank of 1 from each category, indicating the top-selling products in each category.

| category_name | product_name                | total_quantity |
|---------------|-----------------------------|----------------|
| Mens          | Blue Polo Shirt - Mens      | 3819           |
| Womens        | Grey Fashion Jacket - Womens | 3876           |


### 6. What is the percentage split of revenue by product for each segment?
```sql
WITH Segments AS (
    SELECT 
        product_details.segment_name, 
        product_name, 
        SUM(QTY*sales.Price) as Revenue
    FROM balanced_tree.sales
    JOIN balanced_tree.product_details ON sales.prod_id = product_details.product_id
    GROUP BY 
        product_details.segment_name, 
        product_name),
Total AS ( 
    SELECT 
        SUM(QTY*sales.Price) as Total_Revenue, 
        product_details.segment_name
    FROM balanced_tree.sales
    JOIN balanced_tree.product_details
    ON product_details.product_id = sales.prod_ID
    GROUP BY product_details.segment_name)

SELECT 
    segments.segment_name, 
    product_name, 
    ROUND((revenue::numeric/total_revenue::numeric)*100, 2) as Revenue_Split 
FROM segments
JOIN total ON segments.segment_name = total.segment_name
ORDER BY 
    segment_name;
```
This query calculates the percentage split of revenue by product within each segment. The ```Segments``` CTE calculates the revenue for each product within a segment, while the ```Total``` CTE calculates the total revenue for each segment. The main query then computes the revenue split percentage for each product within its segment.

| segment_name | product_name                | revenue_split |
|--------------|-----------------------------|---------------|
| Jacket       | Indigo Rain Jacket - Womens | 19.45         |
| Jacket       | Khaki Suit Jacket - Womens  | 23.51         |
| Jacket       | Grey Fashion Jacket - Womens | 57.03         |
| Jeans        | Navy Oversized Jeans - Womens | 24.06         |
| Jeans        | Black Straight Jeans - Womens | 58.15         |
| Jeans        | Cream Relaxed Jeans - Womens | 17.79         |
| Shirt        | White Tee Shirt - Mens      | 37.43         |
| Shirt        | Blue Polo Shirt - Mens      | 53.60         |
| Shirt        | Teal Button Up Shirt - Mens | 8.98          |
| Socks        | Navy Solid Socks - Mens     | 44.33         |
| Socks        | White Striped Socks - Mens  | 20.18         |
| Socks        | Pink Fluro Polkadot Socks - Mens | 35.50   |


### 7.What is the percentage split of revenue by segment for each category?
```sql
WITH Segments AS (
    SELECT 
        product_details.category_name, 
        segment_name, 
        SUM(QTY*sales.Price) as Revenue
    FROM  balanced_tree.sales
    JOIN balanced_tree.product_details ON sales.prod_id = product_details.product_id
    GROUP BY 
        product_details.category_name, 
        segment_name
),
Total AS ( 
    SELECT 
        SUM(QTY*sales.Price) as Total_Revenue, 
        product_details.category_name
    FROM balanced_tree.sales
    JOIN balanced_tree.product_details
    ON product_details.product_id = sales.prod_ID
    GROUP BY product_details.category_name
)

SELECT 
    segments.category_name, 
    segment_name, 
    ROUND((revenue::numeric/total_revenue::numeric)*100, 2) as Revenue_Split 
FROM segments
JOIN total
ON segments.category_name = total.category_name
ORDER By category_name;
```
This query calculates the percentage split of revenue by segment within each category. The ```Segments``` CTE calculates the revenue for each segment within a category, while the ```Total``` CTE calculates the total revenue for each category. The main query then computes the revenue split percentage for each segment within its category.

| category_name | segment_name | revenue_split |
|---------------|--------------|---------------|
| Mens          | Socks        | 43.13         |
| Mens          | Shirt        | 56.87         |
| Womens        | Jeans        | 36.21         |
| Womens        | Jacket       | 63.79         |


### 8.What is the percentage split of total revenue by category?
```sql
WITH Segments AS (
    SELECT 
        category_name, 
        SUM(QTY*sales.Price) as Revenue
    FROM balanced_tree.sales
    JOIN balanced_tree.product_details
    ON sales.prod_id = product_details.product_id
    GROUP BY category_name
),
Total AS ( 
    SELECT 
        SUM(QTY*sales.Price) as Total_Revenue
    FROM balanced_tree.sales
)

SELECT 
    category_name, 
    ROUND((revenue::numeric/total_revenue::numeric)*100, 2) as Revenue_Split 
FROM 
    segments
CROSS JOIN 
    Total;
```

This query calculates the percentage split of total revenue by category. The ```Segments``` CTE calculates the revenue for each category, while the ```Total``` CTE calculates the overall total revenue. The main query then computes the revenue split percentage for each category. Since there's a single value for total revenue, a cross join with the ```Total``` CTE is used to calculate the percentage split for each category.
| category_name | revenue_split |
|---------------|---------------|
| Mens          | 55.38         |
| Womens        | 44.62         |

