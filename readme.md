
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
This query calculates the total amount each customer spent at the restaurant by joining the 'Sales' and 'Menu' tables on the 'Product_ID'. The 'SUM' function aggregates the total sales amount for each customer.
