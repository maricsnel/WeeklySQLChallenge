<div align="center">
  <h1>Pizza Runner SQL Case Study</h1>
  <p>Exploring Customer Behavior and Menu Insights Using SQL</p>
  <img src="CS1.png" alt="Danny's Diner">
</div>

# Case Study 2: Pizza Runner

## Introduction

## Data Cleanup Process
```sql
CREATE TEMP TABLE Customer_Orders_Clean AS
SELECT
    Order_ID,
    Customer_ID,
    Pizza_ID,
    CASE
        WHEN exclusions = 'null' THEN '' else exclusions
        END AS "Exclusions",
    Case
        When extras = 'null' THEN '' else extras
    END AS "Extras",
    Order_Time
FROM pizza_runner.customer_orders;
UPDATE Customer_Orders_Clean
SET "Extras" = ' '
WHERE "Extras" IS NULL;

CREATE TEMP TABLE Runner_Orders_Clean AS
SELECT
    Order_ID,
    Runner_ID,
    CASE WHEN pickup_time = 'null' THEN '' ELSE pickup_time END AS pickup_time,
	CASE WHEN distance = 'null' THEN '' ELSE 	REPLACE(distance, 'km', '') END AS 		Distance,
	CASE WHEN distance = 'null' THEN '' ELSE 	Left(Duration, 2) end as Duration,
   Case When cancellation = 'null' THEN ''
   else cancellation
   END AS cancellation  
FROM pizza_runner.runner_orders;

UPDATE runner_orders_clean
SET "cancellation" = ' '
WHERE "cancellation" is NULL;
```
### Customer Orders Cleanup

We begin by creating a temporary table called ```Customer_Orders_Clean``` to store cleaned customer order data. This table includes the following columns:

```Order_ID```: The unique identifier for each order.

```Customer_ID```: The identifier for the customer who placed the order.

```Pizza_ID```: The identifier for the pizza ordered.

```Exclusions```: A cleaned version of the "exclusions" data. If the value was originally ```'null'```, it's replaced with an empty string.

```Extras```: A cleaned version of the "extras" data. If the value was originally ```'null'```, it's replaced with an empty string.

```Order_Time```: The timestamp when the order was placed.

### Runner Orders Cleanup

We create another temporary table called ```Runner_Orders_Clean``` to store cleaned runner order data. This table includes the following columns:

```Order_ID```: The unique identifier for each order.

```Runner_ID```: The identifier for the runner who handled the order.

```pickup_time```: A cleaned version of the "pickup_time" data. If the value was originally ```'null'```, it's replaced with an empty string.

```Distance```: A cleaned version of the "distance" data. If the value was originally ```'null'```, it's replaced with an empty string. Additionally, any 'km' unit is removed.

```Duration```: A cleaned version of the "duration" data. If the value was originally ```'null'```, it's replaced with an empty string. Only the first two characters are retained.

```cancellation```: A cleaned version of the "cancellation" data. If the value was originally ```'null'```, it's replaced with an empty string.

### Update Actions 
For the ```Customer_Orders_Clean``` table:The ```Extras``` column is updated to contain an empty space (' ') where the value was originally ```NULL```.
For the ```Runner_Orders_Clean``` table:The ```cancellation``` column is updated to contain an empty string (' ') where the value was originally ```NULL```.

The cleaning steps involve converting specific 'null' values to more appropriate representations or to empty strings, making the data more consistent and suitable for analysis.

By performing these data cleanup steps, the subsequent queries can provide accurate insights without being affected by incomplete or inconsistent data values.


## Queries and Explanations

### 1. How many pizzas were ordered?
```sql
SELECT Count(Pizza_ID)
FROM customer_orders_clean;
```

This query counts the total number of pizzas ordered.

| pizzas_ordered |
| -------------- |
| 14             |

### 2. How many unique customer orders were made?
```sql
SELECT Count(Distinct(Order_ID))
FROM customer_orders_clean;
```

This query counts the number of unique customer orders.

| unique_orders |
| ------------- |
| 10            |

### 3. How many successful orders were delivered by each runner?
```sql
SELECT COUNT(order_id), runner_id
FROM Runner_Orders_Clean
WHERE cancellation != 'Restaurant Cancellation' AND cancellation != 'Customer Cancellation'
GROUP BY runner_id;
```

This query counts the number of successful orders delivered by each runner, excluding canceled orders.

| successfull_orders | runner_id |
| ------------------ | --------- |
| 1                  | 3         |
| 3                  | 2         |
| 4                  | 1         |

### 4. How many of each type of pizza was delivered?
```sql
SELECT Count(customer_orders_clean.Pizza_ID), pizza_names
FROM customer_orders_clean
JOIN pizza_runner.pizza_names
ON customer_orders_clean.pizza_id = pizza_names.pizza_id
GROUP BY pizza_names;
```

This query counts the number of each type of pizza delivered, joining the order data with the pizza names.

| count | pizza_names    |
| ----- | -------------- |
| 10    | (1, Meatlovers) |
| 4     | (2, Vegetarian) |

### 5.How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT customer_orders_clean.Customer_ID, Pizza_Name, Count(pizza_names.pizza_Name)
FROM customer_orders_clean
JOIN pizza_runner.pizza_names
ON customer_orders_clean.pizza_ID = pizza_names.pizza_ID
GROUP BY customer_ID, pizza_name
ORDER BY customer_id, pizza_name;
```

| customer_id | pizza_name  | count |
| ----------- | ----------- | ----- |
| 101         | Meatlovers  | 2     |
| 101         | Vegetarian  | 1     |
| 102         | Meatlovers  | 2     |
| 102         | Vegetarian  | 1     |
| 103         | Meatlovers  | 3     |
| 103         | Vegetarian  | 1     |
| 104         | Meatlovers  | 3     |
| 105         | Vegetarian  | 1     |

This query counts the number of Vegetarian and Meatlovers pizzas ordered by each customer.

### 6. What was the maximum number of pizzas delivered in a single order?
```sql
SELECT Order_ID, Count(pizza_ID) as TotalPizzas
FROM customer_orders_clean
GROUP BY Order_ID
ORDER BY TotalPizzas Desc
LIMIT 1;
```

This query finds the order with the maximum number of pizzas delivered.

| order_id | totalpizzas |
| -------- | ----------- |
| 4        | 3           |

### 7.For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT Customer_ID,
COUNT(CASE WHEN "Exclusions" = '' AND "Extras" = '' THEN 1 END) AS No_Change_Pizzas,
COUNT(CASE WHEN "Exclusions" != '' AND "Extras" != '' THEN 1 END) AS Change_Pizzas
FROM customer_orders_clean
GROUP BY Customer_ID;
```

This query calculates the number of pizzas delivered to each customer with and without changes (exclusions or extras).

| customer_id | no_change_pizzas | change_pizzas |
| ----------- | ---------------- | ------------- |
| 101         | 3                | 0             |
| 103         | 0                | 1             |
| 104         | 1                | 1             |
| 105         | 0                | 0             |
| 102         | 2                | 0             |

### 8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT COUNT(CASE WHEN "Exclusions" != '' AND "Extras" != '' THEN 1 END) AS Change_Pizzas
FROM customer_orders_clean;
```

This query counts the number of pizzas with both exclusions and extras.

| change_pizzas |
| ------------- |
| 2             |

### 9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT DATE_PART('HOUR', order_time::timestamp) AS hour_of_day, count(Pizza_ID)
FROM customer_orders_clean
GROUP BY hour_of_day
ORDER BY hour_of_day ASC;
```

This query calculates the total number of pizzas ordered for each hour of the day.

| hour_of_day | count |
| ----------- | ----- |
| 11          | 1     |
| 13          | 3     |
| 18          | 3     |
| 19          | 1     |
| 21          | 3     |
| 23          | 3     |

### 10. What was the volume of orders for each day of the week?
```sql
SELECT TO_CHAR(order_time::timestamp, 'Day') AS day_of_week, COUNT(Pizza_ID)
FROM customer_orders_clean
GROUP BY day_of_week
ORDER BY day_of_week;
```

This query calculates the volume of orders for each day of the week.

| day_of_week | count |
| ----------- | ----- |
| Friday      | 1     |
| Saturday    | 5     |
| Thursday    | 3     |
| Wednesday   | 5     |

### 11. How many runners signed up for each 1 week period?
```sql
SELECT EXTRACT(WEEK FROM pickup_time::timestamp) as Week, COUNT(runner_ID)
FROM runner_orders_clean
WHERE pickup_time &lt;&gt; '' AND pickup_time IS NOT NULL
GROUP BY Week
ORDER BY Week;
```
This query counts the number of runners who signed up for each 1-week period, filtering out null pickup times.

| week | count |
| ---- | ----- |
| 1    | 4     |
| 2    | 4     |
