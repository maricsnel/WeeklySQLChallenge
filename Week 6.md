<div align="center">
  <h1>Danny’s Diner SQL Case Study</h1>
  <p>Exploring Customer Behavior and Menu Insights Using SQL</p>
  <img src="CS1.png" alt="Danny's Diner">
</div>

# Case Study 1: Danny’s Diner

## Introduction

## Digital Analysis


### 1.How many users are there?
```sql
SELECT Count(Distinct User_ID) as Unique_Users 
FROM clique_bait.users;
```

This query calculates the count of distinct user IDs in the "users" dataset, providing the number of unique users.
| Unique Users |
|--------------|
| 500          |

### 2. How many cookies does each user have on average?
```sql
SELECT Count(Cookie_ID)/Count(Distinct User_ID) as AVG_Cookies_Per_User
FROM clique_bait.users;
```

This query calculates the average number of cookies per user by dividing the total count of cookies by the count of distinct user IDs in the "users" dataset.
| AVG Cookies Per User |
|----------------------|
| 3                    |

### 3, What is the unique number of visits by all users per month?
```sql
SELECT Extract(month from Event_time) as Months, Count(Distinct Visit_ID) as Unique_Visits
FROM clique_bait.events
GROUP BY Months;
```

This query groups the events by month, extracting the month from the event time. It then counts the distinct visit IDs for each month, giving the unique number of visits by all users per month.
| Months | Unique Visits |
|--------|---------------|
| 1      | 876           |
| 2      | 1488          |
| 3      | 916           |
| 4      | 248           |
| 5      | 36            |

### 4. What is the number of events for each event type?
```sql
SELECT Event_name, Count(Visit_ID) as Visits
FROM clique_bait.events
JOIN clique_bait.event_identifier
ON events.event_type = event_identifier.event_type 
GROUP BY Event_name;
```

This query joins the "events" dataset with the "event_identifier" dataset based on the event type. It then counts the number of events (visits) for each event name and groups the results accordingly.
| Event Name     | Visits |
|----------------|--------|
| Purchase       | 1777   |
| Ad Impression  | 876    |
| Add to Cart    | 8451   |
| Page View      | 20928  |
| Ad Click       | 702    |

### 5.What is the percentage of visits which have a purchase event?
```sql
WITH CTE as(
  Select Count(Distinct Visit_ID) as Total_Visits
  From clique_bait.events),

CTE2 as(
SELECT Event_name, Count(Distinct Visit_ID) as Visits
FROM clique_bait.events
JOIN clique_bait.event_identifier
ON events.event_type = event_identifier.event_type 
GROUP BY Event_name)

Select Round(CTE2.Visits::numeric/CTE.Total_Visits::numeric*100, 2)
From CTE
Cross Join CTE2
Where event_name = 'Purchase';
```

This query calculates the percentage of visits that have a "Purchase" event. It uses Common Table Expressions (CTEs) to first calculate the total number of visits and the number of visits for each event type. Then, it divides the "Purchase" event visits by the total visits and presents the result as a percentage.
| Percentage |
|------------|
| 49.86      |

### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql
WITH CTE as(
  SELECT
    visit_id,
    Sum(CASE WHEN event_type = 1 and page_id = 12 THEN 1 ELSE 0 END) AS Checkout_Page,
    Sum(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS Purchase
  FROM clique_bait.events
  GROUP BY visit_id),

Totalviews as (
Select Count (Visit_Id) as Checkout_Page_Views
from CTE
Where checkout_page &gt; 0),

No_Purchase as(
  Select Count (Visit_Id) as No_Purchases
from CTE
Where checkout_page &gt; 0 and purchase = 0)

Select Round(No_Purchases::numeric/Checkout_Page_Views::numeric*100, 2) as Percentage_No_Purchase
From Totalviews
Cross Join No_Purchase;
```

This query calculates the percentage of visits that view the checkout page but do not have a purchase event. It first calculates the number of checkout page views and the number of visits without purchases using CTEs. Then, it calculates and presents the percentage of visits without purchases out of the total checkout page views.
| Percentage No Purchase |
|------------------------|
| 15.50                  |


### 7. What are the top 3 pages by number of views?
```sql
Select page_name, Count(visit_id) as Views
FROM clique_bait.page_hierarchy
JOIN clique_bait.events
ON page_hierarchy.page_id = events.page_id
WHERE event_type = 1
GROUP BY page_name
ORDER BY Views Desc
LIMIT 3;
```

This query identifies the top 3 pages with the highest number of views. It joins the "page_hierarchy" dataset with the "events" dataset based on page IDs and filters for event type 1 (page view). The results are grouped by page name, ordered by views in descending order, and limited to the top 3.
| Page Name     | Views |
|---------------|-------|
| All Products  | 3174  |
| Checkout      | 2103  |
| Home Page     | 1782  |

### 8. What is the number of views and cart adds for each product category?
```sql
WITH AllViews as(
Select Product_Category, Count(visit_id) as Page_view
FROM clique_bait.page_hierarchy
JOIN clique_bait.events
ON page_hierarchy.page_id = events.page_id
WHERE event_type = 1
GROUP BY Product_Category),

Cart as(
Select Product_Category, Count(visit_id) as Cart_adds
FROM clique_bait.page_hierarchy
JOIN clique_bait.events
ON page_hierarchy.page_id = events.page_id
WHERE event_type = 2
GROUP BY Product_Category)

Select allviews.product_category, allviews.page_view, cart.cart_adds
FROM AllViews
FULL JOIN Cart
ON cart.product_category = allviews.product_category;
```
| Product Category | Page Views | Cart Adds |
|------------------|------------|-----------|
| null             | 7059       | null      |
| Luxury           | 3032       | 1870      |
| Shellfish        | 6204       | 3792      |
| Fish             | 4633       | 2789      |

This query provides the number of views and cart additions for each product category. It calculates the page views and cart adds separately using CTEs. Then, it combines the results using a full join on the product category, ensuring that all categories are included in the output.
