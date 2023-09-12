<div align="center">
  <h1>Case Study 6 - Clique Bait</h1>
</div>

![image](https://github.com/maricsnel/WeeklySQLChallenge/assets/142982185/cf73b143-5f37-4c50-9218-b5a05c8e22e0)

## Introduction
Welcome to the "Clique Bait" Case Study, where we delve into the dynamic realm of digital analytics within the broader context of the online commerce industry. As e-commerce continues to reshape traditional business models, data analytics emerges as a transformative force, driving strategic decision-making and customer-centric approaches. Within this case study, we explore how data analytics can unravel critical insights about user behavior, event interactions, and content preferences on platforms like Clique Bait. By harnessing the power of SQL queries and data interpretation, we uncover patterns that not only optimize user experiences but also hold the potential to revolutionize the way businesses engage with their online audiences. Join us in deciphering the digital footprint and untapped opportunities of the ever-evolving online commerce landscape.
## Digital Analysis


### 1.How many users are there?
```sql
SELECT Count(Distinct User_ID) as Unique_Users 
FROM clique_bait.users;
```
- The `Count(Distinct User_ID)` function calculates the number of unique user IDs in the "users" table.

| Unique Users |
|--------------|
| 500          |

### 2. How many cookies does each user have on average?
```sql
SELECT Count(Cookie_ID)/Count(Distinct User_ID) as AVG_Cookies_Per_User
FROM clique_bait.users;
```

- The `Count(Cookie_ID)` function calculates the total number of cookie IDs in the "users" table.

- The `Count(Distinct User_ID)` function calculates the count of distinct user IDs in the "users" table.

- The division `/` operator divides the count of cookie IDs by the count of distinct user IDs.

| AVG Cookies Per User |
|----------------------|
| 3                    |

### 3, What is the unique number of visits by all users per month?
```sql
SELECT Extract(month from Event_time) as Months, Count(Distinct Visit_ID) as Unique_Visits
FROM clique_bait.events
GROUP BY Months;
```
- The `Extract(month from Event_time)` function extracts the month component from the "Event_time" column.

- The `Count(Distinct Visit_ID)` function calculates the count of unique "Visit_ID" values for each month.

- The `GROUP BY Months` clause groups the results by the "Months" column, which represents the extracted month component.

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
- The `Count(Visit_ID)` function calculates the count of "Visit_ID" values for each event.

- The query performs an inner join between the "events" table and the "event_identifier" table based on matching "event_type" values.

- The `GROUP BY Event_name` clause groups the results by the "Event_name" column, which represents the name of the event.

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
- CTE :
  - This CTE calculates the total number of distinct "Visit_ID" values, representing total visits, from the "events" table.

- CTE2:
  - This CTE calculates the count of distinct "Visit_ID" values for each event, based on the "Event_name".
  - It performs an inner join between the "events" table and the "event_identifier" table based on matching "event_type" values.
  - The results are grouped by "Event_name".

- The main query:
  - This query calculates the percentage of visits for a specific event type ("Purchase").
  - It performs a cross join between the two CTEs to combine their results.
  - The percentage is calculated by dividing the count of distinct visits for the "Purchase" event by the total number of distinct visits.

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
Where checkout_page > 0),

No_Purchase as(
  Select Count (Visit_Id) as No_Purchases
from CTE
Where checkout_page > 0 and purchase = 0)

Select Round(No_Purchases::numeric/Checkout_Page_Views::numeric*100, 2) as Percentage_No_Purchase
From Totalviews
Cross Join No_Purchase;
```
- "CTE":
  - This CTE aggregates data for each "visit_id", counting occurrences of certain event types and page IDs.
  - The `CASE` statements are used to selectively count events based on conditions.
  - "Checkout_Page" is calculated as the sum of events with event_type 1 and page_id 12.
  - "Purchase" is calculated as the sum of events with event_type 3.

- "Totalviews":
  - This CTE counts the total number of checkout page views from the "CTE" (Calculated in the previous CTE).
  - It filters rows from the CTE where the "checkout_page" count is greater than 0.

- "No_Purchase":
  - This CTE counts the number of visits where there was a checkout page view (checkout_page count > 0) but no purchase (purchase count = 0).

- The main query:
  - This query calculates the percentage of visits without a purchase relative to the total number of checkout page views.
  - It performs a cross join between the "Totalviews" and "No_Purchase" CTEs.
  - The percentage is calculated by dividing the count of visits without a purchase by the total number of checkout page views.
    
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

- The `SELECT` clause includes two columns: "page_name" and the count of "visit_id" values.

- The `FROM` clause specifies the two tables to be used: "page_hierarchy" and "events".

- The `WHERE` clause filters the results to only include rows where the "event_type" is 1 (indicating a view event).

- The `GROUP BY page_name` groups the results by unique "page_name" values.
  
- The `LIMIT 3` clause restricts the output to the top three most viewed pages.
  
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
- CTE "AllViews":
  - This CTE calculates the count of page views for each "Product_Category".
  - The `WHERE` clause filters rows where the "event_type" is 1 (indicating a view event).
  - The results are grouped by "Product_Category".

- CTE "Cart":
  - This CTE calculates the count of cart additions for each "Product_Category".
  - The `WHERE` clause filters rows where the "event_type" is 2 (indicating a cart addition event).
  - The results are grouped by "Product_Category".

- The main query:
  - This query retrieves the "Product_Category", page views ("page_view"), and cart additions ("cart_adds") from the two CTEs.
  - It performs a FULL JOIN between the "AllViews" and "Cart" CTEs based on matching "product_category" values.

| Product Category | Page Views | Cart Adds |
|------------------|------------|-----------|
| null             | 7059       | null      |
| Luxury           | 3032       | 1870      |
| Shellfish        | 6204       | 3792      |
| Fish             | 4633       | 2789      |

## Suggestions and Conclusion
1. **Checkout-to-Purchase Conversion**: A significant percentage of visits involving the checkout page do not culminate in a purchase event. This suggests an opportunity to improve conversion rates within the checkout process. Optimize the checkout experience to bridge the gap between checkout page views and actual purchases. Streamline the checkout form, offer guest checkout options, and provide transparency about security and return policies to encourage user trust and conversion.
   
2. **Cart Adds**: There are a high number of cart additions, indicating user interest, but this interest is not consistently translating into purchases. Implement Cart Abandonment Strategies: Deploy automated cart abandonment emails or notifications. Remind users about their pending cart items, possibly with additional incentives like discounts or limited-time offers, to entice them to complete the purchase.
   
3. **User Interaction Trends by Month**: The analysis reveals varying levels of user engagement across different months. While some months witness higher unique visits, others experience a decline. To maintain consistent engagement implement monthly campaigns. Design targeted marketing campaigns aligned with user trends. Offer season-specific promotions, events, or content to sustain user interest and engagement throughout the year.
   
The "Clique Bait" analysis has illuminated crucial patterns in user engagement and behavior. By acting on these insights—optimizing conversion pathways, refining checkout experiences, and harnessing ad impact—Clique Bait has the power to forge stronger connections, drive conversions, and ultimately elevate its position in the digital marketplace. The road ahead is one of strategic adaptation, where data-driven decisions pave the way for enhanced customer experiences and sustained growth.

## Other Case Studies

[Case Study 1: Danny's Diner](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%201:%20Danny's%20Diner.md)

[Case Study 2: Pizza Runner](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%202:%20Pizza%20Runner.md)

[Case Study 3: Foodie FI](https://github.com/maricsnel/WeeklySQLChallenge/commit/529d6a8dd0998ebdfb0eb6eaf463361799aa4f76)

[Case Study 4: Data Bank](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%204:%20Data%20Bank.md)

[Case Study 5: Data Mart](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%205:%20Data%20Mart.md)

[Case Study 7: Balanced Tree Clothing Co](https://github.com/maricsnel/WeeklySQLChallenge/blob/WeeklySQLChallenge/Case%20Study%207:%20Balanced%20Tree%20Clothing%20Co..md)

