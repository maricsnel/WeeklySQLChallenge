<div align="center">
  <h1>Case Study 1: Danny’s Diner</h1>
  <img src="CS1.png" alt="Danny's Diner">
</div>

## Introduction
Welcome to the Danny’s Diner SQL case study! In this project, we'll dive into a series of SQL queries to analyze customer transactions and menu items at Danny’s Diner. Let's explore how SQL can help us answer various questions about customer behavior, popular menu items, and more.

## Entity Relationship Diagram
![image](https://github.com/maricsnel/WeeklySQLChallenge/assets/142982185/1931adad-c9c0-40c4-be08-a3208edf9468)


## Queries and Explanations

### 1. Total Amount Spent by Each Customer
```sql
SELECT
    SUM(menu.Price) AS Total_Sales_Amount,
    Sales.Customer_ID
FROM
    dannys_diner.Sales
JOIN
    dannys_diner.Menu ON Sales.Product_ID = Menu.Product_ID
GROUP BY
    Sales.Customer_ID;
```
- **Joining Data (`JOIN`):** Combine "Sales" and "Menu" tables using a shared "Product_ID".

- **Summing Prices (`SUM`):** Calculate the total prices of menu items for each customer.

- **Grouping Results (`GROUP BY`):** Organize the data by customer, allowing per-customer calculations.


| total_sales_amount | customer_id |
|-------------------|-------------|
| 74                | B           |
| 36                | C           |
| 76                | A           |


### 2. Number of Days Visited by Each Customer
```sql
SELECT
    Sales.Customer_ID AS Customer,
    COUNT(sales.Order_Date) AS Days_Visited
FROM
    dannys_diner.Sales
GROUP BY
    Sales.Customer_ID;
```

- **Grouping Data (`GROUP BY`):** Group results by "Customer_ID".

- **Aggregating Data (`COUNT`):** Count the number of times each customer's "Order_Date" appears, indicating the days they visited.

| customer | days_visited |
|----------|--------------|
| B        | 6            |
| C        | 3            |
| A        | 6            |

### 3. First Item Purchased by Each Customer
```sql
SELECT Customer, Product
FROM (
    SELECT
        Sales.Customer_ID AS Customer,
        Menu.Product_Name AS Product,
        RANK() OVER (PARTITION BY Sales.Customer_ID ORDER BY Sales.Order_Date) AS Rank
    FROM
        dannys_diner.Sales
    JOIN
        dannys_diner.Menu ON Sales.Product_ID = Menu.Product_ID
) ranked_sales
WHERE Rank = 1;
```
- **Subquery for Ranking (`SELECT`):** Within a subquery, select "Customer_ID" as "Customer", "Product_Name" as "Product", and assign a ranking using the RANK() function.

  - **Ranking by Customer (`RANK()`):** Assign ranks within each customer's partition, based on the order of "Order_Date".

- **Joining Data (`JOIN`):** Join "Sales" and "Menu" tables on matching "Product_ID".

- **Main Query for Filtering (`SELECT`):** In the main query, select "Customer" and "Product" columns from the subquery results.

- **Filtering by Rank (`WHERE`):** Keep only the rows where the rank is 1, indicating the first purchase for each customer.

| customer_id | product_name |
|-------------|--------------|
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

### 4. Most Purchased Menu Item and Total Count
```sql
SELECT 
  menu.product_name,
  COUNT(sales.product_id) AS most_purchased
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY most_purchased DESC
LIMIT 1;
```

- **Selecting and Counting (`SELECT`):** Choose "Product_Name" from "Menu" and count occurrences of "Product_ID" from "Sales" as "most_purchased".

- **Joining Data (`JOIN`):** Combine "Sales" and "Menu" tables using matching "Product_ID".

- **Grouping Data (`GROUP BY`):** Group results by "Product_Name".

- **Aggregating Data (`COUNT`):** Count the occurrences of each product in sales.

- **Ordering Results (`ORDER BY`):** Sort the groups by the count of most purchased products in descending order.

- **Limiting Results (`LIMIT`):** Keep only the first result, which will be the most purchased product.

| product_name | most_purchased |
|--------------|----------------|
| ramen        | 8              |

### 5. Most Popular Item for Each Customer
```sql
WITH product_counts AS (
    SELECT
        Customer_ID, Product_ID, COUNT(Product_ID) AS product_count
    FROM
        dannys_diner.Sales
    GROUP BY
        Customer_ID, Product_ID
)
SELECT Ranks.Customer_ID, Menu.Product_Name, Ranks.product_count, Ranks.rank
FROM (
    SELECT *,
        RANK() OVER (PARTITION BY Customer_ID ORDER BY product_count DESC) AS "rank"
    FROM product_counts
) Ranks
JOIN dannys_diner.Menu ON ranks.Product_ID = Menu.Product_ID
WHERE ranks.rank = 1
ORDER BY ranks.Customer_ID;
```

- **Common Table Expression (CTE) (`WITH`):** Create a temporary table "product_counts" to store customer-wise product purchase counts.

  - **Counting Products (`SELECT`):** Count occurrences of "Product_ID" for each "Customer_ID".

  - **Grouping Data (`GROUP BY`):** Group results by both "Customer_ID" and "Product_ID".

- **Subquery for Ranking (`SELECT`):** Within a subquery, calculate ranks for each customer based on product counts.

  - **Ranking by Product Count (`RANK()`):** Assign ranks within each customer's partition, ordered by product count in descending order.

- **Joining Data (`JOIN`):** Join "Ranks" subquery with "Menu" table using matching "Product_ID".

- **Main Query for Selecting Columns (`SELECT`):** In the main query, select "Customer_ID", "Product_Name", "product_count", and "rank".

- **Filtering by Rank (`WHERE`):** Keep only the rows where the rank is 1, indicating the most purchased product for each customer.

- **Ordering Results (`ORDER BY`):** Sort results by "Customer_ID".

| customer_id | product_name | product_count | rank |
|-------------|--------------|---------------|------|
| A           | ramen        | 3             | 1    |
| B           | sushi        | 2             | 1    |
| B           | curry        | 2             | 1    |
| B           | ramen        | 2             | 1    |
| C           | ramen        | 3             | 1    |

### 6. First Item Purchased by Customer After Joining
```sql
WITH First_Boughts AS (
  SELECT sales.*, menu.product_name,
         RANK() OVER (PARTITION BY sales.customer_id ORDER BY order_date ASC) AS "Rank"
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
  Join dannys_diner.members
  on sales.customer_id = members.customer_id
  where sales.order_date > members.join_date 
)
SELECT customer_ID, Product_Name
FROM First_Boughts
where "Rank" = 1;

```

- **Common Table Expression (CTE) (`WITH`):** Create a temporary table "First_Boughts" to store the earliest purchases for each customer.

  - **Selecting Columns and Ranking (`SELECT`):** Choose all columns from "Sales", along with "Product_Name".
  
    - **Joining Data (`JOIN`):** Combine "Sales" and "Menu" tables using matching "Product_ID".
  
    - **Joining with Membership Data (`JOIN`):** Also join with the "Members" table using matching "Customer_ID" and check if the order date is after the member's join date.
    
    - **Ranking by Order Date (`RANK()`):** Assign ranks within each customer's partition, ordered by "Order_Date" in ascending order.

- **Main Query for Selecting Columns (`SELECT`):** Select "Customer_ID" and "Product_Name" from the "First_Boughts" CTE.

- **Filtering by Rank (`WHERE`):** Keep only the rows where the rank is 1, indicating the first purchase for each customer.

| customer_id | product_name |
|-------------|--------------|
| A           | ramen        |
| B           | sushi        |

### 7. Last Item Purchased by Customer Before Joining
```sql
WITH Last_Boughts AS (
  SELECT sales.*, menu.product_name,
         RANK() OVER (PARTITION BY sales.customer_id ORDER BY order_date DESC) AS "Rank"
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
  Join dannys_diner.members
  on sales.customer_id = members.customer_id
  where sales.order_date < members.join_date 
)
SELECT customer_ID, Product_Name, Order_Date
FROM Last_Boughts
where "Rank" = 1;
```

- **Common Table Expression (CTE) (`WITH`):** Create a temporary table "Last_Boughts" to store the latest purchases before a customer's membership start.

  - **Selecting Columns and Ranking (`SELECT`):** Choose all columns from "Sales", along with "Product_Name".
  
    - **Joining Data (`JOIN`):** Combine "Sales" and "Menu" tables using matching "Product_ID".
  
    - **Joining with Membership Data (`JOIN`):** Also join with the "Members" table using matching "Customer_ID" and check if the order date is before the member's join date.
    
    - **Ranking by Order Date (`RANK()`):** Assign ranks within each customer's partition, ordered by "Order_Date" in descending order.

- **Main Query for Selecting Columns (`SELECT`):** Select "Customer_ID", "Product_Name", and "Order_Date" from the "Last_Boughts" CTE.

- **Filtering by Rank (`WHERE`):** Keep only the rows where the rank is 1, indicating the last purchase before a customer's membership.

| customer_id | product_name | order_date                |
|-------------|--------------|---------------------------|
| A           | sushi        | 2021-01-01T00:00:00.000Z  |
| A           | curry        | 2021-01-01T00:00:00.000Z  |
| B           | sushi        | 2021-01-04T00:00:00.000Z  |


### 8. Total Items and Amount Spent for Each Member Before Joining
```sql
SELECT 
  sales.customer_id, 
  COUNT(sales.product_id) AS total_items, 
  SUM(menu.price) AS total_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
  AND sales.order_date < members.join_date
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

- **Selecting and Counting (`SELECT`):** Choose "customer_id" and count occurrences of "product_id" as "total_items".
   - Also, calculate the sum of "price" from the "Menu" table as "total_sales".

- **Joining Data (`INNER JOIN`):** Combine "Sales" and "Members" tables using matching "customer_id" where the order date is before the member's join date.
   - This ensures we consider only purchases made before a customer became a member.

- **Joining Data (`INNER JOIN`):** Join the result of the previous join with the "Menu" table using matching "product_id".

- **Grouping Data (`GROUP BY`):** Group results by "customer_id".
   - This enables summarizing sales metrics on a per-customer basis.

- **Aggregating Data (`COUNT`, `SUM`):** Count the occurrences of product purchases and calculate the total sales amount for each customer.

- **Ordering Results (`ORDER BY`):** Sort the results by "customer_id".

| customer_id | total_items | total_sales |
|-------------|-------------|-------------|
| A           | 2           | 25          |
| B           | 3           | 40          |


### 9. Points Calculation for Customers
```sql
WITH Multiplier AS (
    SELECT
        Menu.Product_ID,
        CASE
            WHEN Menu.Product_ID = 1 THEN Menu.Price * 20
            WHEN Menu.Product_ID = 2 THEN Menu.Price * 10
            WHEN Menu.Product_ID = 3 THEN Menu.Price * 10
        END AS "Points"
    FROM
        dannys_diner.Menu
)
SELECT
    Sales.Customer_ID,
    SUM("Points") AS Total_Points
FROM
    dannys_diner.Sales
JOIN
    Multiplier ON Multiplier.Product_ID = Sales.Product_ID
GROUP BY
    Sales.Customer_ID;
```

- **Common Table Expression (CTE) (`WITH`):** Create a temporary table "Multiplier" to store product points multipliers.

  - **Selecting Columns and Calculating Points (`SELECT`):** Choose "Product_ID" from "Menu" and calculate "Points" based on different multipliers for each product.

    - **Using Conditional Logic (`CASE`):** Apply different multipliers based on the "Product_ID".

- **Selecting and Summing Points (`SELECT`):** Choose "Customer_ID" and sum the calculated "Points" as "Total_Points" for each customer.

- **Joining Data (`JOIN`):** Combine "Sales" and "Multiplier" tables using matching "Product_ID".

- **Grouping Data (`GROUP BY`):** Group results by "Customer_ID".

- **Aggregating Data (`SUM`):** Sum up the calculated "Points" for each customer.

- **Result:** The query produces a result set with columns "Customer_ID" and "Total_Points", representing the total points accumulated by each customer based on their purchases and the respective multipliers.

| customer_id | total_points |
|-------------|--------------|
| B           | 940          |
| C           | 360          |
| A           | 860          |


## Conclusions

In the realm of Danny’s Diner, our journey through SQL-powered data analysis has uncovered illuminating insights and valuable lessons. We've learned that understanding customer behaviors and menu preferences is pivotal in shaping exceptional dining experiences. These insights offer a roadmap to personalized offerings, increased customer loyalty, and optimized menus.

Our findings illuminate the path to success:

- Customer Understanding: Delving into spending patterns, visit frequency, and menu favorites has provided a nuanced understanding of individual customer preferences.

- Menu Optimization: Identifying popular menu items empowers Danny’s Diner to tailor offerings, ensuring a delightful dining experience for every patron.

- Membership Impact: Analyzing member behavior sheds light on the effectiveness of loyalty programs and their influence on spending patterns.

- Data-Driven Decisions: These queries exemplify the power of data analysis in steering strategic decisions that foster customer satisfaction and loyalty.

As we conclude this case study, remember that behind the numbers lies the ability to shape remarkable dining experiences. Through data-driven insights, Danny’s Diner stands poised to refine its offerings, create loyal patrons, and thrive in the competitive culinary landscape. The journey of Danny’s Diner exemplifies how harnessing the potential of data can revolutionize the world of hospitality.

## Other Case Studies

2. [Case Study 2: Pizza Runner](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%202:%20Pizza%20Runner.md)
3. [Case Study 3: Foodie FI](https://github.com/maricsnel/WeeklySQLChallenge/commit/529d6a8dd0998ebdfb0eb6eaf463361799aa4f76)
4. [Case Study 4: Data Bank](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%204:%20Data%20Bank.md)
5. [Case Study 5: Data Mart](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%205:%20Data%20Mart.md)
6. [Case Study 6: Clique Bait](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%206%20-%20Clique%20Bait.md)
7. [Case Study 7: Balanced Tree Clothing Co](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%207:%20Balanced%20Tree%20Clothing%20Co..md)
