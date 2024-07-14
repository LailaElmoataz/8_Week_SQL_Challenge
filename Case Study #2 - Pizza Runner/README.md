# Case Study #2: Pizza Runner
![alt text](header.png)

## Table of Contents:
* [Introduction](#introduction)
* [Tools](#tools)
* [Problem Statement](#problem-statement)
* [Dataset](#dataset)
* [Data Cleaning & Transformation](#data-cleaning--transformation)
* [Analysis:](#analysis)
    * [A. Pizza Metrics](#a-pizza-metrics) 

## Introduction

Welcome to this SQL project, which is part of [Danny Ma's 8WeekSQLChallenge](https://8weeksqlchallenge.com/)! Our objective is to use SQL to tackle eight fascinating case studies.

Please note that all the information needed for the case study will be obtained from [here](https://8weeksqlchallenge.com/case-study-2/).


## Tools
* **PostgreSQL**: database management system to create and manage the database schema for this project.
* **VS Code**: code editor for developing and executing SQL queries.
* **Git & Github**: for version control, project tracking and sharing my scripts and analysis.

## Problem Statement
Danny is expanding his new Pizza Empire, and wants to uberize it, and so Pizza Runner was launched!
He recruited “runners” to deliver the pizza from Pizza Runner Headquarters (his house).\
He needs our help to clean his data and apply some basic calculations so he can better direct his runners and optimize Pizza Runner’s operations.

## Dataset
Danny provided us with 6 tables: customer_orders, runner_orders, runners, pizza_names, pizza_recipes, and pizza_toppings.
* **customer_orders:** Captures the customers orders including the type of pizza, any additional or excluded ingredients, and the order date.
* **runner_orders:** Shows the runner that the order was assigned to, the timestamp at which the runner picks up the order at the headquarters, delivery distance and duration, and cancellation status.
* **runners:** Holds information on the runner and their registration date.
* **pizza_names:** Contains the names of the pizzas currently offered by Pizza Runner.
* **pizza_recipes:** Holds information on the recipe (mix of toppings) for each pizza.
* **pizza_toppings:** Contains the names of available pizza toppings.

Below is the entity relationship diagram (ERD) showing the relationships between the tables:
![alt text](erd.png)

## Data Cleaning & Transformation
**customer_orders:**
> extras: convert string "null" and "" to NULL\
> exclusions: convert string "null" and "" to NULL
```sql
UPDATE customer_orders
SET exclusions = NULL
WHERE exclusions IN ('null', '');
UPDATE customer_orders
SET extras = NULL
WHERE extras IN ('null', '');
```

**runner_orders:**
> cancellation: populate with "Not Cancelled" if not classified as cancelled
```sql
UPDATE runner_orders
SET cancellation = 'Not Cancelled' 
WHERE cancellation IS NULL 
OR cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation');
``` 

## Analysis

### A. Pizza Metrics

#### 1. How many pizzas were ordered?
 
```sql
SELECT 
    COUNT(*) AS pizza_count
FROM customer_orders;
```
There were 14 pizzas ordered in total.
| pizza_count |
| ----------- |
| 14          |


#### 2. How many unique customer orders were made?
```sql
SELECT 
    COUNT(DISTINCT order_id) AS order_count
FROM customer_orders;
```
There are 10 unique customer orders.
| order_count |
| ----------- |
| 10          |

#### 3. How many successful orders were delivered by each runner?
```sql
SELECT 
    runner_id, 
    COUNT(*) AS successful_order_count
FROM runner_orders
WHERE cancellation = 'Not Cancelled'
GROUP BY 1;
```
Table below shows that runners 1, 2, and 3 made 4, 3 and 1 successful orders respectively.
| runner_id | successful_order_count |
| --------- | ---------------------- |
| 1         | 4                      |
| 2         | 3                      |
| 3         | 1                      |

#### 4. How many of each type of pizza was delivered?
```sql
SELECT 
    pizza_name, 
    COUNT(*) AS pizza_count
FROM customer_orders
JOIN runner_orders
USING(order_id)
JOIN pizza_names
USING(pizza_id)
WHERE cancellation = 'Not Cancelled'
GROUP BY 1;
```
9 Meatlovers pizzas and 3 Vegetarian pizzas were delivered.

| pizza_name | pizza_count |
| ---------- | ----------- |
| Meatlovers | 9           |
| Vegetarian | 3           |

#### 5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT 
    customer_id, 
    COUNT(*) FILTER (WHERE pizza_name = 'Meatlovers') AS meatlovers_count, 
    COUNT(*) FILTER (WHERE pizza_name = 'Vegetarian') AS vegetarian_count
FROM customer_orders
JOIN pizza_names
USING(pizza_id)
GROUP BY 1
ORDER BY 1;
```
The table below summarizes customer orders, indicating how many "Meatlovers" and "Vegetarian" pizzas each customer purchased.

| customer_id | meatlovers_count | vegetarian_count |
| ----------- | ---------------- | ---------------- |
| 101         | 2                | 1                |
| 102         | 2                | 1                |
| 103         | 3                | 1                |
| 104         | 3                | 0                |
| 105         | 0                | 1                |

#### 6. What was the maximum number of pizzas delivered in a single order?
```sql
SELECT 
    MAX(pizza_count) AS max_pizza_per_order
FROM (    
    SELECT order_id, COUNT(*) pizza_count
    FROM customer_orders
    JOIN runner_orders
    USING(order_id)
    WHERE cancellation = 'Not Cancelled'
    GROUP BY 1
    ) sub;
```
Maximum number of pizzas delivered in a single order is 3 pizzas.

| max_pizza_per_order |
| ------------------- |
| 3                   |

#### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT 
    customer_id,
    COUNT(pizza_id) FILTER(WHERE exclusions IS NULL AND extras IS NULL) AS no_changes,
    COUNT(pizza_id) FILTER(WHERE exclusions IS NOT NULL OR extras IS NOT NULL) AS at_least_1_change, 
    ROUND(AVG(CASE WHEN exclusions IS NOT NULL OR extras IS NOT NULL THEN 1 ELSE 0 END) * 100, 0) || '%' AS pct_customized
FROM customer_orders
JOIN runner_orders
USING(order_id)
WHERE cancellation = 'Not Cancelled'
GROUP BY 1
ORDER BY 1;
```
Looking at the table below, it appears some customers only ordered their pizzas with at least one customization (customers 103, 105), while others only ordered their pizzas without any changes (customers 101, 102), while others .
Customer 104 ordered a mix of both.

| customer_id | no_changes | at_least_1_change | pct_customized |
| ----------- | ---------- | ----------------- | ----------------------- |
| 101         | 2          | 0                 | 0%                       |
| 102         | 3          | 0                 | 0%                       |
| 103         | 0          | 3                 | 100%                     |
| 104         | 1          | 2                 | 67%                      |
| 105         | 0          | 1                 | 100%                     |
#### 8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT 
    COUNT(*) AS pizza_count
FROM customer_orders
JOIN runner_orders
USING(order_id)
WHERE cancellation = 'Not Cancelled'
AND exclusions IS NOT NULL
AND extras IS NOT NULL;
```
Only one of the delivered pizza had both exclusions and extras!
| pizza_count |
| ----------- |
| 1           |

#### 9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT 
    EXTRACT(HOUR FROM order_time) AS order_hour,
    COUNT(*) AS pizza_count
FROM customer_orders
GROUP BY 1
ORDER BY 1;
```
Peak hours for Pizza Runner, based on volume of pizzas, are considered to be 1 pm, 6 pm, 9 pm, and 11 pm with 3 pizzas ordered each.\
11 am and 7 pm are considered to be off-peak hours with only 1 pizza ordered each.
| order_hour | pizza_count |
| ---------- | ----------- |
| 11         | 1           |
| 13         | 3           |
| 18         | 3           |
| 19         | 1           |
| 21         | 3           |
| 23         | 3           |

#### 10. What was the volume of orders for each day of the week?
```sql
SELECT
    TO_CHAR(order_time, 'Day') AS order_day, 
    COUNT(DISTINCT order_id) AS order_count
FROM customer_orders
GROUP BY 1
ORDER BY 1;
```
Looking at the table, we can see that Wednesday is the most popular day with the most orders placed (5), while Friday is the least popular day with only one order.

| order_day | order_count |
| --------- | ----------- |
| Friday    | 1           |
| Saturday  | 2           |
| Thursday  | 2           |
| Wednesday | 5           |


## Thank you!