<div align="center">
  <h1>Case Study 2: Pizza Runner</h1>
</div>

![image](https://github.com/maricsnel/WeeklySQLChallenge/assets/142982185/26c26776-e700-484a-9446-3551efa73ed3)

# 

## Introduction

The "Pizza Runner" case study involves analyzing data from a pizza delivery service to gain insights into customer orders and runner activities. The dataset contains information on customer orders and runner deliveries. To facilitate the analysis, a thorough data cleanup process is conducted, addressing missing or incomplete values in the dataset. The case study explores various queries to extract valuable information, including the total number of pizzas ordered, unique customer orders, successful deliveries by each runner, pizza type distribution, and more. Through this analysis, the case study aims to provide a comprehensive understanding of the pizza delivery operations and customer preferences.

## Entity Relationship Diagram
![image](https://github.com/maricsnel/WeeklySQLChallenge/assets/142982185/0728625b-7f4b-4dd3-b891-23beb7d84ace)

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

- **Counting Records (`SELECT`):** Count the number of occurrences of "Pizza_ID" in the "customer_orders_clean" table.

- **Result:** The query produces a single number representing the count of records where "Pizza_ID" is present in the "customer_orders_clean" table.

| pizzas_ordered |
| -------------- |
| 14             |

### 2. How many unique customer orders were made?
```sql
SELECT Count(Distinct(Order_ID))
FROM customer_orders_clean;
```
- **Counting Distinct Orders (`SELECT`):** Count the number of distinct occurrences of "Order_ID" in the "customer_orders_clean" table.

- **Result:** The query produces a single number representing the count of unique orders based on distinct "Order_ID" values in the "customer_orders_clean" table.

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
- **Counting Orders per Runner (`SELECT`):** Count the occurrences of "order_id" for each "runner_id" in the "Runner_Orders_Clean" table.

- **Filtering Data (`WHERE`):** Exclude rows where the "cancellation" is either 'Restaurant Cancellation' or 'Customer Cancellation'.

- **Grouping Data (`GROUP BY`):** Group results by "runner_id".

- **Result:** The query produces a result set with columns: the count of orders ("order_id") and the respective "runner_id". The query calculates the number of orders each runner has that are not canceled by the restaurant or the customer.

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
- **Counting Pizza Orders (`SELECT`):** Count the occurrences of "Pizza_ID" for each type of pizza, represented by "pizza_names".

- **Joining Data (`JOIN`):** Combine "customer_orders_clean" and "pizza_names" tables using matching "Pizza_ID".

- **Grouping Data (`GROUP BY`):** Group results by "pizza_names".

- **Result:** The query produces a result set with columns: the count of orders ("Pizza_ID") for each type of pizza ("pizza_names"). It calculates how many times each pizza has been ordered based on the joined tables.

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

- **Selecting and Counting (`SELECT`):** Choose "Customer_ID", "Pizza_Name", and count occurrences of "pizza_names.pizza_Name".

- **Joining Data (`JOIN`):** Combine "customer_orders_clean" and "pizza_names" tables using matching "Pizza_ID".

- **Grouping Data (`GROUP BY`):** Group results by both "Customer_ID" and "pizza_name".
   - This groups orders by customer and the type of pizza they ordered.

- **Ordering Results (`ORDER BY`):** Sort the results first by "customer_id", then by "pizza_name".
  
- **Result:** The query produces a result set with columns: "Customer_ID", "Pizza_Name", and the count of orders for each specific pizza type for each customer. 

### 6. What was the maximum number of pizzas delivered in a single order?
```sql
SELECT Order_ID, Count(pizza_ID) as TotalPizzas
FROM customer_orders_clean
GROUP BY Order_ID
ORDER BY TotalPizzas Desc
LIMIT 1;
```

- **Selecting and Counting (`SELECT`):** Choose "Order_ID" and count occurrences of "pizza_ID" as "TotalPizzas".

- **Grouping Data (`GROUP BY`):** Group results by "Order_ID".

- **Ordering Results (`ORDER BY`):** Sort the results by "TotalPizzas" in descending order.
   - This arranges orders based on the total number of pizzas in each order, with the highest count first.

- **Limiting Results (`LIMIT`):** Keep only the first result, which will be the order with the most total pizzas.

- **Result:** The query produces a single result with the "Order_ID" and the total count of pizzas ("TotalPizzas") for the order with the highest number of pizzas.

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

- **Selecting and Counting (`SELECT`):** Choose "Customer_ID" and count occurrences based on conditions using the COUNT function.

- **Counting No Change Pizzas (`CASE`):** Count occurrences when both "Exclusions" and "Extras" columns are empty.
    
- **Counting Change Pizzas (`CASE`):** Count occurrences when both "Exclusions" and "Extras" columns have values.

- **Grouping Data (`GROUP BY`):** Group results by "Customer_ID".

- **Result:** The query produces a result set with columns: "Customer_ID", "No_Change_Pizzas" (count of orders with no changes to pizza), and "Change_Pizzas" (count of orders with changes to pizza). 

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

- **Selecting and Counting (`SELECT`):** Count occurrences based on a condition using the COUNT function.

- **Counting Change Pizzas (`CASE`):** Count occurrences when both "Exclusions" and "Extras" columns have values.

- **Result:** The query produces a single result with the count of orders ("Change_Pizzas") where both "Exclusions" and "Extras" columns have values. 

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

- **Extracting Hour and Counting (`SELECT`):** Extract the hour from "order_time" and count occurrences of "Pizza_ID".

- **Grouping Data (`GROUP BY`):** Group results by the extracted hour of the day.

- **Ordering Results (`ORDER BY`):** Sort the results by the hour of the day in ascending order.

- **Result:** The query produces a result set with columns: "hour_of_day" (the extracted hour from "order_time") and the count of pizza orders ("Pizza_ID") placed during each hour of the day.
  
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

- **Formatting Day of the Week and Counting (`SELECT`):** Format "order_time" as the day of the week and count occurrences of "Pizza_ID".

- **Grouping Data (`GROUP BY`):** Group results by the formatted day of the week.

- **Ordering Results (`ORDER BY`):** Sort the results by the formatted day of the week in alphabetical order.

- **Result:** The query produces a result set with columns: "day_of_week" (the day of the week extracted from "order_time") and the count of pizza orders ("Pizza_ID") placed on each day of the week. 
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
- **Extracting Week and Counting (`SELECT`):** Extract the week number from "pickup_time" and count occurrences of "runner_ID".

- **Filtering Data (`WHERE`):** Exclude rows where "pickup_time" is empty or NULL.

- **Grouping Data (`GROUP BY`):** Group results by the extracted week number.

- **Ordering Results (`ORDER BY`):** Sort the results by the week number in ascending order.

| week | count |
| ---- | ----- |
| 1    | 4     |
| 2    | 4     |


## Key Findings and Conclusions

Upon delving into the analysis of Pizza Runner's customer orders and runner activities, several significant insights have surfaced:

1. **Pizza Preferences:** The analysis of pizza types revealed that "Meatlovers" is the most favored choice among customers, closely followed by "Vegetarian." This information can guide Pizza Runner in optimizing their ingredient inventory and introducing new specials based on customer preferences.

2. **Runner Performance:** By examining successful deliveries by each runner, it became evident that some runners have consistently higher successful delivery rates than others. This insight enables Pizza Runner to recognize and reward top-performing runners, ultimately enhancing overall service quality.

3. **Peak Order Hours:** Through the analysis of order timestamps, it was found that peak order hours are during the evening, particularly around 6:00 PM to 8:00 PM. Pizza Runner can utilize this information to allocate additional resources during peak hours and minimize customer wait times.

4. **Weekly Trends:** The study of weekly runner sign-up trends highlighted that sign-ups remain steady, with four runners joining each week. This consistent trend aids Pizza Runner in forecasting staffing needs accurately and maintaining a balanced runner workforce.

In conclusion, these insights empower Pizza Runner to make data-driven decisions that positively impact their business operations. By tailoring their offerings based on popular pizza choices, recognizing and incentivizing efficient runners, and optimizing service during peak hours, Pizza Runner can elevate customer satisfaction, increase efficiency, and ensure a seamless delivery experience.

## Other Case Studies
1. [Case Study 1: Danny's Diner](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%201:%20Danny's%20Diner.md)
3. [Case Study 3: Foodie FI](https://github.com/maricsnel/WeeklySQLChallenge/commit/529d6a8dd0998ebdfb0eb6eaf463361799aa4f76)
4. [Case Study 4: Data Bank](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%204:%20Data%20Bank.md)
5. [Case Study 5: Data Mart](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%205:%20Data%20Mart.md)
6. [Case Study 6: Clique Bait](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%206%20-%20Clique%20Bait.md)
7. [Case Study 7: Balanced Tree Clothing Co](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%207:%20Balanced%20Tree%20Clothing%20Co..md)
