<div align="center">
  <h1>Danny’s Diner SQL Case Study</h1>
  <p>Exploring Customer Behavior and Menu Insights Using SQL</p>
  <img src="CS1.png" alt="Danny's Diner">
</div>

# Case Study 1: Danny’s Diner

## Introduction

## Queries and Explanations

1. How many pizzas were ordered?
```sql
SELECT Count(Pizza_ID)
FROM customer_orders_clean;
```

This query counts the total number of pizzas ordered.

2. How many unique customer orders were made?
```sql
SELECT Count(Distinct(Order_ID))
FROM customer_orders_clean;
```

This query counts the number of unique customer orders.

3. How many successful orders were delivered by each runner?
```sql
SELECT COUNT(order_id), runner_id
FROM Runner_Orders_Clean
WHERE cancellation != 'Restaurant Cancellation' AND cancellation != 'Customer Cancellation'
GROUP BY runner_id;
```

This query counts the number of successful orders delivered by each runner, excluding canceled orders.

4. How many of each type of pizza was delivered?
```sql
SELECT Count(customer_orders_clean.Pizza_ID), pizza_names
FROM customer_orders_clean
JOIN pizza_runner.pizza_names
ON customer_orders_clean.pizza_id = pizza_names.pizza_id
GROUP BY pizza_names;
```

This query counts the number of each type of pizza delivered, joining the order data with the pizza names.

5.How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT customer_orders_clean.Customer_ID, Pizza_Name, Count(pizza_names.pizza_Name)
FROM customer_orders_clean
JOIN pizza_runner.pizza_names
ON customer_orders_clean.pizza_ID = pizza_names.pizza_ID
GROUP BY customer_ID, pizza_name
ORDER BY customer_id, pizza_name;
```

This query counts the number of Vegetarian and Meatlovers pizzas ordered by each customer.

6. What was the maximum number of pizzas delivered in a single order?
```sql
SELECT Order_ID, Count(pizza_ID) as TotalPizzas
FROM customer_orders_clean
GROUP BY Order_ID
ORDER BY TotalPizzas Desc
LIMIT 1;
```

This query finds the order with the maximum number of pizzas delivered.

7.For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT Customer_ID,
COUNT(CASE WHEN "Exclusions" = '' AND "Extras" = '' THEN 1 END) AS No_Change_Pizzas,
COUNT(CASE WHEN "Exclusions" != '' AND "Extras" != '' THEN 1 END) AS Change_Pizzas
FROM customer_orders_clean
GROUP BY Customer_ID;
```

This query calculates the number of pizzas delivered to each customer with and without changes (exclusions or extras).

8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT COUNT(CASE WHEN "Exclusions" != '' AND "Extras" != '' THEN 1 END) AS Change_Pizzas
FROM customer_orders_clean;
```

This query counts the number of pizzas with both exclusions and extras.

9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT DATE_PART('HOUR', order_time::timestamp) AS hour_of_day, count(Pizza_ID)
FROM customer_orders_clean
GROUP BY hour_of_day
ORDER BY hour_of_day ASC;
```

This query calculates the total number of pizzas ordered for each hour of the day.

10. What was the volume of orders for each day of the week?
```sql
SELECT TO_CHAR(order_time::timestamp, 'Day') AS day_of_week, COUNT(Pizza_ID)
FROM customer_orders_clean
GROUP BY day_of_week
ORDER BY day_of_week;
```

This query calculates the volume of orders for each day of the week.

11. How many runners signed up for each 1 week period?
```sql
SELECT EXTRACT(WEEK FROM pickup_time::timestamp) as Week, COUNT(runner_ID)
FROM runner_orders_clean
WHERE pickup_time &lt;&gt; '' AND pickup_time IS NOT NULL
GROUP BY Week
ORDER BY Week;
```

This query counts the number of runners who signed up for each 1-week period, filtering out null pickup times.
