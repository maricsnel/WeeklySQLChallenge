<div align="center">
  <h1>Danny’s Diner SQL Case Study</h1>
  <p>Exploring Customer Behavior and Menu Insights Using SQL</p>
  <img src="CS1.png" alt="Danny's Diner">
</div>

# Case Study 1: Danny’s Diner

## Introduction
Welcome to the Danny’s Diner SQL case study! In this project, we'll dive into a series of SQL queries to analyze customer transactions and menu items at Danny’s Diner. Let's explore how SQL can help us answer various questions about customer behavior, popular menu items, and more.

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

This query calculates the total amount each customer spent at the restaurant by joining the Sales and Menu tables on the Product_ID. The ```SUM``` function aggregates the total sales amount for each customer.

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

This query calculates the number of days each customer visited the restaurant by counting the unique ```Order_Date``` entries for each customer.

### 3. First Item Purchased by Each Customer
```sql
SELECT
    Sales.Customer_ID AS Customer,
    Menu.Product_Name AS Product
FROM
    dannys_diner.Sales
JOIN
    dannys_diner.Menu ON Sales.Product_ID = Menu.Product_ID
GROUP BY
    Customer, Product, Sales.Order_Date
ORDER BY
    Sales.Order_Date ASC;
```

This query identifies the first item purchased by each customer by joining the Sales and Menu tables and ordering the results by ```Order_Date```.

### 4. Most Purchased Menu Item and Total Count
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

This query ranks menu items based on the first purchase by each customer and selects the most purchased item by filtering the results with a rank of 1.

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
SELECT x.Customer_ID, Menu.Product_Name, x.product_count, x.rank
FROM (
    SELECT *,
        RANK() OVER (PARTITION BY Customer_ID ORDER BY product_count DESC) AS "rank"
    FROM product_counts
) x
JOIN dannys_diner.Menu ON x.Product_ID = Menu.Product_ID
WHERE x.rank = 1
ORDER BY x.Customer_ID;
```

This query uses a common table expression (CTE) to count the number of times each menu item is purchased by each customer. It then ranks items based on their purchase count and selects the most popular item for each customer.

### 6. First Item Purchased by Customer After Joining
```sql
WITH First_Boughts AS (
    SELECT
        Sales.*, Menu.Product_Name,
        RANK() OVER (PARTITION BY Sales.Customer_ID ORDER BY Sales.Order_Date ASC) AS "Rank"
    FROM
        dannys_diner.Sales
    JOIN
        dannys_diner.Menu ON Sales.Product_ID = Menu.Product_ID
    JOIN
        dannys_diner.Members ON Sales.Customer_ID = Members.Customer_ID
    WHERE
        Sales.Order_Date &gt; Members.Join_Date
)
SELECT Customer_ID, Product_Name
FROM First_Boughts
WHERE "Rank" = 1;
```

This query identifies the first item purchased by each customer after joining the membership program. It uses a CTE to filter purchases made after the customer's join date.

### 7. Last Item Purchased by Customer Before Joining
```sql
WITH Last_Boughts AS (
    SELECT
        Sales.*, Menu.Product_Name,
        RANK() OVER (PARTITION BY Sales.Customer_ID ORDER BY Sales.Order_Date DESC) AS "Rank"
    FROM
        dannys_diner.Sales
    JOIN
        dannys_diner.Menu ON Sales.Product_ID = Menu.Product_ID
    JOIN
        dannys_diner.Members ON Sales.Customer_ID = Members.Customer_ID
    WHERE
        Sales.Order_Date &lt; Members.Join_Date
)
SELECT Customer_ID, Product_Name, Order_Date
FROM Last_Boughts
WHERE "Rank" = 1;
```

This query identifies the last item purchased by each customer before joining the membership program. It uses a CTE to filter purchases made before the customer's join date.

### 8. Total Items and Amount Spent for Each Member Before Joining
```sql
SELECT
    Sales.Customer_ID,
    COUNT(Sales.Product_ID) AS Total_Items,
    SUM(Menu.Price) AS Total_Sales
FROM
    dannys_diner.Sales
INNER JOIN
    dannys_diner.Members ON Sales.Customer_ID = Members.Customer_ID
    AND Sales.Order_Date &lt; Members.Join_Date
INNER JOIN
    dannys_diner.Menu ON Sales.Product_ID = Menu.Product_ID
GROUP BY
    Sales.Customer_ID
ORDER BY
    Sales.Customer_ID;
```

This query calculates the total items and amount spent by each member before joining the membership program. It filters sales records before the member's join date and aggregates the data.

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

This query calculates the total points for each customer based on their purchases. It uses a CTE to apply point multipliers to different menu items and then sums the points for each customer.Conclusion

These SQL queries showcase the power of data analysis in understanding customer behavior and menu preferences at Danny’s Diner. By leveraging SQL, we can unravel insights and make informed decisions for optimizing the restaurant's offerings and customer experience.
