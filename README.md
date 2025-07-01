
# Zoomato SQL Analytics Project

## 🧾 Project Overview

This project involves a detailed SQL-based analysis of Zoomato, a fictional food delivery platform. The aim is to understand customer behavior, restaurant performance, rider efficiency, and other key operational metrics through structured SQL queries.

## ⚙️ Key System Components

- **Customers Table**: Contains customer profiles.
- **Orders Table**: Includes order-level details like date, status, amount, and customer.
- **Restaurants Table**: Restaurant metadata including cuisine and location.
- **Deliveries Table**: Captures delivery timelines and status.
- **Riders Table**: Details on each rider assigned for deliveries.

## 🎯 Project Objectives

- Track customer orders and cancellations
- Identify high-value customers and restaurant rankings
- Analyze delivery patterns and rider performance
- Segment customers and measure growth KPIs
- Understand cuisine popularity and seasonal effects

## 💻 Project SQL Implementation

### 1. Top 5 most frequently restuarants ordered by customer

```sql
Select 
    c.customer_id,
    c.name,
    r.name,
    Count(*) as total_orders,
    DENSE_RANK() OVER (Order by Count(*) DESC) as Rank
from orders as o
JOIN customers as c
on c.customer_id = o.customer_id
JOIN restaurants as r
on r.restaurant_id = o.restaurant_id
where c.name = 'Dominica Kock' AND YEAR(o.order_date)=2023
Group by c.customer_id, c.name, r.name
```

### 2. Popular Time Slots

```sql
-- First Approach
SELECT
    CASE
        WHEN DATEPART(HOUR, order_time) BETWEEN 0 AND 1 THEN '00:00 - 02:00' 
        WHEN DATEPART(HOUR, order_time) BETWEEN 2 AND 3 THEN '02:00 - 04:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 4 AND 5 THEN '04:00 - 06:00' 
        WHEN DATEPART(HOUR, order_time) BETWEEN 6 AND 7 THEN '06:00 - 08:00' 
        WHEN DATEPART(HOUR, order_time) BETWEEN 8 AND 9 THEN '08:00 - 10:00' 
        WHEN DATEPART(HOUR, order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00' 
        WHEN DATEPART(HOUR, order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 22 AND 23 THEN '22:00 - 00:00'
    END AS time_slot,
    COUNT(order_id) AS order_count
FROM Orders
GROUP BY 
    CASE
        WHEN DATEPART(HOUR, order_time) BETWEEN 0 AND 1 THEN '00:00 - 02:00' 
        WHEN DATEPART(HOUR, order_time) BETWEEN 2 AND 3 THEN '02:00 - 04:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 4 AND 5 THEN '04:00 - 06:00' 
        WHEN DATEPART(HOUR, order_time) BETWEEN 6 AND 7 THEN '06:00 - 08:00' 
        WHEN DATEPART(HOUR, order_time) BETWEEN 8 AND 9 THEN '08:00 - 10:00' 
        WHEN DATEPART(HOUR, order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00' 
        WHEN DATEPART(HOUR, order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'
        WHEN DATEPART(HOUR, order_time) BETWEEN 22 AND 23 THEN '22:00 - 00:00'
    END
ORDER BY order_count DESC;

-- Second Approach
Select 
    Floor(DATEPART(HOUR, order_time)/2)*2 as Start_Time,
    Floor(DATEPART(HOUR, order_time)/2)*2 + 2 as End_Time,
    Count(*) as Total_Orders
from orders
Group By 
    Floor(DATEPART(HOUR, order_time)/2)*2,
    Floor(DATEPART(HOUR, order_time)/2)*2 + 2
Order By 3 DESC;
```

### 3. Order Value Analysis

```sql
-- Join on orders and customer table
-- where condition like more than 750 orders
-- avg function on total_amount
-- group by , order by , having clause

select 
    o.customer_id,
    c.name,
    Count(*) as Total_Orders,
    Sum(o.total_amount) as TOV, --(Total Orders Value)
    Avg(o.total_amount) as AOV  --(Average Order Value)
from orders o
Join customers c
on c.customer_id = o.customer_id
group by o.customer_id,c.name
having COUNT(o.customer_id) >= 10
order by TOV DESC;
```

### 4. High-Value Customers

```sql
select 
    o.customer_id,
    c.name,
    Count(*) as Total_Orders,
    Sum(o.total_amount) as Total_Spent 
from orders o
Join customers c
on c.customer_id = o.customer_id
group by o.customer_id,c.name
having Sum(o.total_amount) > 5000
order by Total_Spent DESC;
```

### 5. Orders Without Delivery

```sql
select 
    o.restaurant_id,
    r.name,
    r.location,
    Count(o.restaurant_id) as TNDO   -- Total Not Delivered Orders
from orders o
LEFT Join restaurants as r
    on r.restaurant_id = o.restaurant_id
Left Join deliveries as d
    on d.order_id = o.order_id
where d.status = 'In Transit'
group by o.restaurant_id,r.name,r.location
order by TNDO DESC;
```

### 6. Restaurant Revenue Ranking

```sql
With Restaurant_revenue_rank as
(
SELECT 
    o.restaurant_id,
    r.name,
    r.location,
    SUM(o.total_amount) AS Total_Revenue,
    RANK() OVER (PARTITION BY r.location ORDER BY SUM(o.total_amount) DESC) AS Revenue_Rank
FROM 
    orders o
JOIN 
    restaurants r ON r.restaurant_id = o.restaurant_id
GROUP BY 
    o.restaurant_id, r.name, r.location
)
select * from Restaurant_revenue_rank
where Revenue_Rank = 1;
```

### 7. Most Popular Cuisine Type

```sql
-- CTE Approach
With cuisine_rank as
(
SELECT 
    r.cuisine_type,
    r.location,
    Count(*) AS Total_Orders,
    RANK() OVER (PARTITION BY r.location ORDER BY Count(o.order_id) DESC) AS Rank
FROM 
    orders o
JOIN 
    restaurants r ON r.restaurant_id = o.restaurant_id
GROUP BY 
    r.cuisine_type, r.location
)
select * from cuisine_rank
where Rank = 1;


-- Selection Approach
Select * from
(
SELECT 
    r.cuisine_type,
    r.location,
    Count(*) AS Total_Orders,
    RANK() OVER (PARTITION BY r.location ORDER BY Count(o.order_id) DESC) AS Rank
FROM 
    orders o
JOIN 
    restaurants r ON r.restaurant_id = o.restaurant_id
GROUP BY 
    r.cuisine_type, r.location
) as t1
where Rank = 1;
```

### 8. Customer Churn

```sql
SELECT 
    c.customer_id,
    c.name
FROM 
    customers AS c
LEFT JOIN 
    orders AS o ON c.customer_id = o.customer_id 
    AND YEAR(o.order_date) = 2024
WHERE 
    c.customer_id IN (
        SELECT customer_id
        FROM orders
        WHERE YEAR(order_date) = 2023
    )
GROUP BY 
    c.customer_id, c.name
HAVING 
    COUNT(o.order_id) = 0;
```

### 9. Cancellation Rate Comparison

```sql
-- First Approach
WITH CancellationRates AS (
    SELECT 
        r.restaurant_id,
        r.name,
        YEAR(o.order_date) AS order_year,
        COUNT(CASE WHEN o.status = 'Cancelled' THEN 1 END) AS cancelled_orders,
        COUNT(o.order_id) AS total_orders
    FROM 
        orders AS o
    JOIN 
        restaurants AS r ON r.restaurant_id = o.restaurant_id
    WHERE 
        YEAR(o.order_date) IN (2023, 2024)
    GROUP BY 
        r.restaurant_id, r.name, YEAR(o.order_date)
)

SELECT 
    restaurant_id,
    name,
    MAX(CASE WHEN order_year = 2023 THEN cancelled_orders * 1.0 / NULLIF(total_orders, 0) END) AS cancellation_ratio_2023,
    MAX(CASE WHEN order_year = 2024 THEN cancelled_orders * 1.0 / NULLIF(total_orders, 0) END) AS cancellation_ratio_2024
FROM 
    CancellationRates
GROUP BY 
    restaurant_id, name
ORDER BY 
    restaurant_id;
```

### 10. Rider Average Delivery Time

```sql
SELECT 
    r.name AS rider_name, 
    AVG(DATEDIFF(MINUTE, o.order_time, d.delivery_time)) AS average_delivery_time
FROM 
    riders AS r
JOIN 
    deliveries AS d ON r.rider_id = d.rider_id
JOIN 
    orders AS o ON d.order_id = o.order_id
GROUP BY 
    r.name
ORDER BY 
    average_delivery_time;
```

### 11. Monthly Restaurant Growth Ratio

```sql
WITH MonthlyOrders AS (
    SELECT  
        o.restaurant_id,
        r.name,
        FORMAT(o.order_date, 'MM-yyyy') AS Order_Date,
        COUNT(o.order_id) AS Total_Orders
    FROM 
        orders AS o
    JOIN 
        restaurants AS r ON r.restaurant_id = o.restaurant_id
    WHERE 
        o.status = 'Completed'
    GROUP BY 
        o.restaurant_id, r.name, FORMAT(o.order_date, 'MM-yyyy')
)

SELECT 
    mo1.restaurant_id,
    mo1.name,
    mo1.Order_Date,
    mo1.Total_Orders,
    COALESCE((mo1.Total_Orders * 1.0 / NULLIF(mo2.Total_Orders, 0)) - 1, 0) AS Growth_Ratio
FROM 
    MonthlyOrders AS mo1
LEFT JOIN 
    MonthlyOrders AS mo2 ON mo1.restaurant_id = mo2.restaurant_id 
    AND DATEADD(MONTH, -1, CONVERT(DATE, '01-' + mo1.Order_Date)) = CONVERT(DATE, '01-' + mo2.Order_Date)
ORDER BY 
    mo1.Order_Date, mo1.restaurant_id;

-- Second Approach

WITH growth_ratio AS (
    SELECT
        o.restaurant_id,
        FORMAT(o.order_date, 'MM-yyyy') AS month,
        COUNT(o.order_id) AS cr_month_orders,
        LAG(COUNT(o.order_id)) OVER (PARTITION BY o.restaurant_id ORDER BY FORMAT(o.order_date, 'MM-yyyy')) AS prev_month_orders 
    FROM 
        orders AS o
    WHERE 
        o.status = 'Completed'
    GROUP BY 
        o.restaurant_id, FORMAT(o.order_date, 'MM-yyyy')
)

SELECT
    restaurant_id,
    month,
    prev_month_orders,
    cr_month_orders,
    ROUND(
        (CAST(cr_month_orders AS FLOAT) - CAST(prev_month_orders AS FLOAT)) / NULLIF(CAST(prev_month_orders AS FLOAT), 0) * 100, 
        2
    ) AS growth_ratio 
FROM 
    growth_ratio;
```

### 12. Customer Segmentation

```sql
--First Approach
WITH AOV AS (
    SELECT 
        AVG(total_amount) AS average_order_value
    FROM 
        orders
    WHERE 
        status = 'Completed'
),
CustomerSpendings AS (
    SELECT 
        o.customer_id,
        COUNT(o.order_id) AS total_orders,
        SUM(o.total_amount) AS total_revenue
    FROM 
        orders AS o
    WHERE 
        o.status = 'Completed'
    GROUP BY 
        o.customer_id
)

SELECT 
    CASE 
        WHEN cs.total_revenue > aov.average_order_value THEN 'Gold'
        ELSE 'Silver'
    END AS customer_segment,
    COUNT(cs.customer_id) AS total_customers,
    SUM(cs.total_orders) AS total_orders,
    SUM(cs.total_revenue) AS total_revenue
FROM 
    CustomerSpendings AS cs
CROSS JOIN 
    AOV AS aov
GROUP BY 
    CASE 
        WHEN cs.total_revenue > aov.average_order_value THEN 'Gold'
        ELSE 'Silver'
    END
ORDER BY 
    customer_segment;

--Second Approach

SELECT
cx_category,
Count(total_orders) as total_customers,
SUM(total_orders) as total_orders, 
SUM(total_spent) as total_revenue
from
(SELECT
customer_id,
SUM(total_amount) as total_spent,
COUNT (order_id) as total_orders,
CASE
WHEN SUM(total_amount) > (SELECT AVG(total_amount) FROM orders) THEN 'Gold' ELSE 'Silver'
END as cx_category
FROM orders
where status = 'Completed'
group by customer_id
) as ti
GROUP BY cx_category
```

### 13. Rider Monthly Earnings

```sql
select 
    d.rider_id,
    FORMAT(o.order_date, 'MM-yyyy') AS month,
    SUM(o.total_amount) as Total_Revenue,
    SUM(o.total_amount) * 0.08 as Rider_Revenue
from orders as o
Join deliveries as d
on  d.order_id = o.order_id
Group By d.rider_id, FORMAT(o.order_date, 'MM-yyyy')
Order BY 1,2
```

### 14. Order Frequency by Day

```sql
Select * from
    (
    select 
        r.name,
        --o.order_date,
        DATENAME(WEEKDAY, o.order_date) as Day,
        COUNT(o.order_id) as Total_Orders,
        DENSE_RANK() OVER (Partition by r.name order by COUNT(o.order_id) DESC) as Rank
    from orders as o
    Join restaurants as r
    on r.restaurant_id = o.restaurant_id
    Group By r.name,DATENAME(WEEKDAY, o.order_date)
    ) as t1
where Rank=1;
```

### 15. Customer Lifetime Value(CLV)

```sql
select 
    c.customer_id,
    c.name,
    SUM(o.total_amount) as CLV
from orders as o
Join customers as c
on c.customer_id = o.customer_id
Group By c.customer_id,c.name
Order By 3 DESC;
```

## 16. Monthly Sales Trend

```sql
select
    YEAR(o.order_date) as Order_YEAR,
    MONTH(o.order_date) as Order_Month,
    SUM(o.total_amount) as Total_Sales,
    LAG(SUM(o.total_amount),1) OVER (Order by YEAR(o.order_date), MONTH(o.order_date)) as prev_month_sales
from orders as o
--where YEAR(o.order_date) = '2023'
Group By YEAR(o.order_date),MONTH(o.order_date)
```

### 17. Rider Efficiency

```sql
WITH RiderEfficiency AS (
    SELECT 
        rider_id,
        AVG(DATEDIFF(MINUTE, o.order_time, d.delivery_time)) AS Avg_Delivery_Time
    FROM 
        deliveries as d
    Join orders as o
    on o.order_id = d.order_id
    where d.status = 'Delivered'
    GROUP BY 
        rider_id
)
SELECT 
    rider_id,   
    Avg_Delivery_Time
FROM 
    RiderEfficiency
WHERE 
    Avg_Delivery_Time = (SELECT MIN(Avg_Delivery_Time) FROM RiderEfficiency)
    OR Avg_Delivery_Time = (SELECT MAX(Avg_Delivery_Time) FROM RiderEfficiency);
```

### 18. Order Item Popularity

```sql
WITH ItemPopularity AS (
    SELECT 
        r.cuisine_type as cuisine,
        YEAR(order_date) AS Order_Year,
        MONTH(order_date) AS Order_Month,
        COUNT(*) AS Order_Count
    FROM 
        orders as o
        Join restaurants as r
        on r.restaurant_id = o.restaurant_id
    GROUP BY 
        r.cuisine_type, YEAR(order_date), MONTH(order_date)
)
SELECT 
    cuisine,
    Order_Year,
    Order_Month,
    Order_Count
FROM 
    ItemPopularity
ORDER BY 
    cuisine, Order_Year, Order_Month;


-- Second Approach
Select 
    cuisine,
    Year,
    seasons,
    COUNT(order_id) as Total_orders
From
(
SELECT 
        o.order_id as order_id,
        r.cuisine_type as cuisine,
        YEAR(o.order_date) as Year,
        MONTH(o.order_date) AS Order_Month,
                CASE
                    WHEN  MONTH(o.order_date) Between 5 AND 8 THEN 'Summer'
                    WHEN  MONTH(o.order_date) Between 9 AND 11 THEN 'Autumn'
                    WHEN  MONTH(order_date) Between 3 AND 4 THEN 'Spring'
                    Else 'Winter'
                END as seasons
    FROM 
        orders as o
    Join restaurants r
    on  r.restaurant_id = o.restaurant_id
) as t1
Group By cuisine,Year,seasons
```

### 19. Monthly Restaurant Growth Ratio

```sql
SELECT 
        r.location,
        SUM(o.total_amount) as Total_Revenue,
        RANK() OVER(Order By SUM(o.total_amount) DESC) as City_Rank
    FROM 
        orders as o
    Join restaurants r
    on  r.restaurant_id = o.restaurant_id
    Group by r.location
```

### 20. High-Risk Customer Identification

```sql
WITH CustomerCancellation AS (
    SELECT 
        customer_id,
        COUNT(CASE WHEN status = 'Cancelled' THEN 1 END) AS Cancelled_Count,
        COUNT(order_id) AS Total_Orders
    FROM 
        orders
    GROUP BY 
        customer_id
)
SELECT 
    c.customer_id,
    c.name,
    (Cancelled_Count * 1.0 / NULLIF(Total_Orders, 0)) * 100 AS Cancellation_Rate
FROM 
    CustomerCancellation cc
JOIN 
    customers c ON cc.customer_id = c.customer_id
WHERE 
    (Cancelled_Count * 1.0 / NULLIF(Total_Orders, 0)) > 0.2;
```

### 21. Revenue Contribution by Cuisine Type

```sql
SELECT 
    r.cuisine_type,
    SUM(o.total_amount) AS Total_Revenue
FROM 
    orders o
JOIN 
    restaurants r ON o.restaurant_id = r.restaurant_id
GROUP BY 
    r.cuisine_type
ORDER BY 
    Total_Revenue DESC;
```

## 🏆 Achievements

- Built 20+ parameterized and analytical SQL queries
- Identified high-spending and high-risk customers
- Delivered insights on cuisine trends and restaurant growth
- Assessed rider efficiency and cancellation trends

## 📌 Conclusion

This SQL-based analytics suite provides a robust framework for analyzing critical KPIs for Zoomato. From restaurant growth to customer segmentation, the queries offer a rich view into the business, enabling strategic decision-making for marketing, operations, and logistics.

