<div align="center">
  <h1>Case Study 7: Balanced Tree Clothing Co.</h1>
</div>

![image](https://github.com/maricsnel/WeeklySQLChallenge/assets/142982185/1d76902e-6769-4a2c-ab8b-76fa3dbc2dcf)

## Introduction

## High-Level Sales Analysis

### 1.What was the total quantity sold for all products?

```sql
SELECT product_name, SUM(QTY) AS Total_Quantity_Sold
FROM balanced_tree.sales
JOIN balanced_tree.product_details ON product_details.product_id = sales.prod_id
GROUP BY product_name;
```
- The `SELECT` clause includes two columns: "product_name" and the sum of "QTY" (quantity).

- The `JOIN` clause joins the "sales" and "product_details" tables based on matching "product_id" values.

- The `GROUP BY product_name` clause groups the results by unique "product_name" values.

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
- CTE "ProductQuantities":
  - This CTE calculates the total quantity sold for each product by joining the "sales" and "product_details" tables in the "balanced_tree" database.
  - It retrieves the "product_name" and sums the "QTY" values.
  - The results are grouped by "product_name".

- The main query:
  - This query calculates the total revenue before discounts for each product.
  - It performs a join between the "ProductQuantities" CTE and the "product_details" table based on matching "product_name" values.
  - The calculated revenue is the product of the total quantity sold and the price of the product.

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
- The calculation for the total discount amount is performed using the `SUM` function on the product of "qty" (quantity), "price", and "discount" divided by 100 (to get the actual discount value).

- The `GROUP BY product.product_name` clause groups the results by unique "product_name" values.

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
- The `COUNT(DISTINCT txn_id)` function calculates the number of unique transaction IDs in the "sales" table.

| unique_transactions_count |
|--------------------------|
|            2500           |

### 2.What is the average unique products purchased in each transaction?

```sql
SELECT COUNT(prod_id) / COUNT(DISTINCT txn_id) AS Average_Unique_Products_Per_Transaction
FROM balanced_tree.sales;
```

- The `COUNT(prod_id)` function calculates the total count of "prod_id" values in the "sales" table.

- The `COUNT(DISTINCT txn_id)` function calculates the count of distinct transaction IDs in the "sales" table.

- The division `/` operator calculates the ratio of the total product IDs to the total distinct transaction IDs.

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
- CTE "TransactionRevenue":
  - This CTE calculates the total revenue for each transaction by summing the product of "QTY" (quantity) and "Price" columns.
  - The results are grouped by "TXN_ID".

- The main query:
  - This query calculates the specified percentiles (25th, 50th, and 75th percentiles) of transaction revenues.
  - The `PERCENTILE_CONT` function is used with the `WITHIN GROUP` clause to calculate the specified percentiles.

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
- The calculation for the average discount value per transaction is performed as follows:
  - The `SUM` function calculates the total sum of the discounted revenue for all transactions. It multiplies the product of "Price", "QTY", and the discount percentage ("discount / 100").
  - The `COUNT(DISTINCT TXN_ID)` function calculates the count of distinct transaction IDs in the "sales" table.
  - The division `/` operator calculates the ratio of the total discounted revenue to the total distinct transaction IDs.

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
- The query calculates the percentage of member and non-member transactions based on the "sales" table in the "balanced_tree" database.

- CTE "TransactionCounts":
  - This CTE calculates the total count of distinct transaction IDs as "Total_Transactions".

- CTE "MemberTransactions":
  - This CTE calculates the count of distinct transaction IDs for member transactions as "Member_Transactions". It filters rows where "Member" is true.

- CTE "NonMemberTransactions":
  - This CTE calculates the count of distinct transaction IDs for non-member transactions as "NonMember_Transactions". It filters rows where "Member" is false.

- The main query:
  - This query calculates the percentages of member and non-member transactions.
  - It performs cross joins between the three CTEs to combine the results.
  - The percentages are calculated by dividing the counts of member and non-member transactions by the total transactions and multiplying by 100.

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
- CTE "MemberRevenue":
  - This CTE calculates the average revenue per transaction for members.
  - It divides the sum of the product of "QTY" (quantity) and "Price" by the count of distinct transaction IDs where "Member" is true.

- CTE "NonMemberRevenue":
  - This CTE calculates the average revenue per transaction for non-members.
  - It divides the sum of the product of "QTY" and "Price" by the count of distinct transaction IDs where "Member" is false.

- The main query:
  - This query retrieves the calculated average revenue values from the "MemberRevenue" and "NonMemberRevenue" CTEs.
  - The results are obtained through cross joins between the two CTEs.

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

- The calculation for the total revenue is performed by multiplying the "QTY" and "Price" for each sale.

- The `JOIN` clause joins the "sales" table with the "product_details" table based on matching "prod_id" and "product_id" values.

- The `GROUP BY product_name` clause groups the results by unique "product_name" values.

- The `LIMIT 3` clause limits the output to the top three products.

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
- The calculations involve the following:
  - Total_Quantity: The `SUM(QTY)` function calculates the total quantity sold for each segment.
  - Total_Revenue: The `SUM(QTY * sales.Price)` function calculates the total revenue generated by summing the product of quantity and price for each segment.
  - Total_Discount: The `SUM((QTY * sales.Price) * (discount / 100::numeric))` function calculates the total discount amount by summing the discounted revenue for each segment. It multiplies the product of quantity, price, and discount percentage.

- The `JOIN` clause joins the "sales" table with the "product_details" table based on matching "prod_id" and "product_id" values.

- The `GROUP BY segment_name` clause groups the results by unique "segment_name" values.

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
- CTE "RankedProducts":
  - This CTE calculates the total quantity sold for each product within each segment.
  - It assigns a rank to each product within each segment based on the total quantity sold in descending order.

- The main query:
  - This query retrieves the segment name, product name, and total quantity sold for products that have a rank of 1 within their respective segments.
  - The `WHERE Rank = 1` clause filters the results to only include products with a rank of 1.

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
- The calculations involve the following:
  - Total_Quantity: The `SUM(QTY)` function calculates the total quantity sold for each product category.
  - Total_Revenue: The `SUM(QTY * sales.Price)` function calculates the total revenue generated by summing the product of quantity and price for each product category.
  - Total_Discount: The `SUM((QTY * sales.Price) * (discount / 100::numeric))` function calculates the total discount amount by summing the discounted revenue for each product category. It multiplies the product of quantity, price, and discount percentage.

- The `JOIN` clause joins the "sales" table with the "product_details" table based on matching "prod_id" and "product_id" values.

- The `GROUP BY category_name` clause groups the results by unique "category_name" values.

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
- CTE "Ranked":
  - This CTE calculates the total quantity sold for each product within each category.
  - It assigns a rank to each product within each category based on the total quantity sold in descending order.

- The main query:
  - This query retrieves the category name, product name, and total quantity sold for products that have a rank of 1 within their respective categories.
  - The `WHERE Ranks = 1` clause filters the results to only include products with a rank of 1.

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
- CTE "Segments":
  - This CTE calculates the revenue generated by each product within segments.
  - It calculates the sum of the product of "QTY" (quantity) and "Price" for each product within each segment.

- CTE "Total":
  - This CTE calculates the total revenue generated within each segment.
  - It calculates the sum of the product of "QTY" and "Price" for all products within each segment.

- The main query:
  - The calculation for the revenue split percentage is done by dividing the product's revenue by the total revenue for the segment and then multiplying by 100.
  - The `JOIN` clause combines the "Segments" CTE with the "Total" CTE based on the segment name.

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
- CTE "Segments":
  - This CTE calculates the revenue generated by each segment within each category.
  - It calculates the sum of the product of "QTY" (quantity) and "Price" for each segment within each category.

- CTE "Total":
  - This CTE calculates the total revenue generated within each category.
  - It calculates the sum of the product of "QTY" and "Price" for all segments within each category.

- The main query:
  - The calculation for the revenue split percentage is done by dividing the segment's revenue by the total revenue for the category and then multiplying by 100.
  - The `JOIN` clause combines the "Segments" CTE with the "Total" CTE based on the category name.

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

- CTE "Segments":
  - This CTE calculates the total revenue generated for each product category.
  - It calculates the sum of the product of "QTY" (quantity) and "Price" for all products within each category.

- CTE "Total":
  - This CTE calculates the overall total revenue generated across all categories.
  - It calculates the sum of the product of "QTY" and "Price" for all products.

- The main query:
  - This query retrieves the category name and revenue split percentage for each product category.
  - The calculation for the revenue split percentage is done by dividing the category's revenue by the total revenue across all categories and then multiplying by 100.
  - The `CROSS JOIN` clause combines the "Segments" CTE with the "Total" CTE, resulting in a single-row "Total" value that can be used to calculate the percentage for each category.

| category_name | revenue_split |
|---------------|---------------|
| Mens          | 55.38         |
| Womens        | 44.62         |

