## CASE STUDY 2

### Pizza Runner
![pizza runner img](https://github.com/Echooed/8-weeks-sql-challenge/assets/91009365/a27f1d6a-c5a6-44b6-ab1d-4ae128286ca7)


Objective: Danny needs the data cleaned

---

**Problem Statement: Pizza Runner Data Optimization**

**Background:**
Danny, the owner of Pizza Runner, is keen on leveraging data science to enhance the efficiency and performance of his pizza delivery business. He has shared a dataset containing information about runners, customer orders, and pizza details within the pizza_runner database schema. As a data scientist, my task is to analyze the data, clean it, and perform various calculations to optimize Pizza Runner's operations.

**Objective:**
The primary goal is to assist Danny in improving Pizza Runner's operations by addressing key areas such as pizza metrics, runner and customer experience, ingredient optimization, and pricing and ratings. Your solutions should provide actionable insights to enhance the overall performance and customer satisfaction of Pizza Runner.

**Data Overview:**

- The dataset includes information on runners' registration dates, customer orders with pizza details, runner assignments, and pizza recipes with toppings.

**Tables available in the database**
- `runners`
- `customer_orders`
- `runner_orders`
- `pizza_names`
- `pizza_recipes`
- `pizza_toppings`

**Areas of Focus:**

1. **Pizza Metrics:**
    - Analyze the total number of orders for each pizza type.
    - Identify the top 3 pizzas by order count.
2. **Runner and Customer Experience:**
    - Calculate the average delivery time for successfully completed orders.
    - Identify the top 3 runners with the highest average distance traveled.
3. **Ingredient Optimization:**
    - Identify the most commonly excluded and included toppings in orders.
4. **Pricing and Ratings:**
    - Calculate the total revenue for each pizza type.
    - Determine the average rating for each pizza type.
5. **Bonus DML Challenges (Data Manipulation Language):**
    - Mark all orders with a null pickup_time as pending.
    - Delete all canceled orders.

**Deliverables:**
I am required to provide SQL queries for each of the tasks outlined in the problem statement. The queries should be well-structured, efficient, and capable of producing actionable insights to optimize Pizza Runner's operations.

---

This problem statement outlines the context, objectives, data overview, areas of focus, and expected deliverables for the data optimization task.

<div align="center">

### **Entity Relationship Diagram**

</div>

<p align="center">
<img src="https://github.com/Echooed/8-weeks-sql-challenge/assets/91009365/b4deeed1-4987-49d4-95ad-e52c9e92ae7f"/>
</p>

    
## Case study Requests
_The request is categorised into 4 sections depending on the area of focus_;

a). *Pizza Metrics*

b). *Runner and Customer Experience*

c). *Ingredient Optimisation*

d). *Pricing and Ratings*

e). *Bonus DML Challenges (DML = Data Manipulation Language)*

## Case study Requests


### A. Pizza Metrics

1. How many pizzas were ordered?
2. How many unique customer orders were made?
3. How many successful orders were delivered by each runner?
4. How many of each type of pizza was delivered?
5. How many Vegetarian and Meatlovers were ordered by each customer?
6. What was the maximum number of pizzas delivered in a single order?
7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
8. How many pizzas were delivered that had both exclusions and extras?
9. What was the total volume of pizzas ordered for each hour of the day?
10. What was the volume of orders for each day of the week?

### Queries
#### Data investigation and cleaning
The first course of action is to visually inspect the data tables before querying.

### Customer_order Table
![image](https://github.com/Echooed/8-weeks-sql-challenge/assets/91009365/142fac44-21ec-4804-b5d8-7bb740548f9c)

### Runner_order table
![image](https://github.com/Echooed/8-weeks-sql-challenge/assets/91009365/f5b1fffe-3c63-4aff-a6e1-ec8d221d0f4f)

### Observations:
* There is no uniformity in the exclusion column for the empty rows and null rows.
* There is no uniformity in the extra column for the empty and null rows.
* The pickup_time column has null as a string.
* The cancellation column lacks uniformity for empty rows.
* The duration column has ununiform values with some values having minute, min or just numbers.
* The null string and the different value style in the distance column are not appropriate.


The approach to cleaning this table is to create a temp table where the null, empty rows and varying value style for each columns will be unified.

### Cleaning customer_orders table
``` mysql
-- Cleaned customer_orders Table CALLED customer_orders_temp(a temp table)
-- change the "null" and " " string in the exclusion and extra table into NULL
DROP TABLE IF EXISTS pizza_runner.customer_orders_temp; 
CREATE TEMPORARY TABLE customer_orders_temp AS (
SELECT
    order_id,
    customer_id,
    pizza_id,
    order_time,
    CASE
	WHEN exclusions = ''
	OR exclusions = 'null' THEN NULL
	ELSE exclusions
    END AS exclusions,
    CASE
	WHEN extras = 'null'
        OR extras = '' THEN NULL ELSE extras
    END AS extras
FROM pizza_runner.customer_orders 
);
SELECT * FROM customer_orders_temp;
```

![image](https://github.com/Echooed/8-weeks-sql-challenge/assets/91009365/df5772b2-0beb-44e5-9d0a-02600a9ce7e0)

### Cleaning runner_orders table

``` mysql
DROP TABLE IF EXISTS pizza_runner.runner_orders_temps;
CREATE TEMP TABLE runner_orders_temp AS
SELECT 
    order_id, 
    runner_id,
    PARSE_DATETIME('%Y-%m-%d %H:%M:%S', (CASE WHEN pickup_time = 'null' THEN NULL ELSE pickup_time END)) AS pickup_time,
    CAST((CASE WHEN distance = 'null' THEN NULL 
         WHEN distance LIKE '%km' THEN REGEXP_REPLACE(distance, r'km$', '')
         ELSE distance END) AS FLOAT64) AS distance,
    CAST((CASE WHEN duration = 'null' THEN NULL 
         WHEN duration LIKE '%mins' THEN REGEXP_REPLACE(REGEXP_REPLACE(duration, r'minutes$', ''), r'mins$', '')
         WHEN duration LIKE '%minute' THEN REGEXP_REPLACE(duration, r'minute$', '')
         WHEN duration LIKE '%minutes' THEN REGEXP_REPLACE(duration, r'minutes$', '')
         ELSE duration END) AS INT64) AS duration,
    NULLIF(NULLIF(cancellation, 'null'), '') AS cancellation
FROM pizza_runner.runner_orders;
```
![image](https://github.com/Echooed/8-weeks-sql-challenge/assets/91009365/1b4a3863-649b-437d-8ada-511544630f4e)




``` mysql
-- 1. How many pizzas were ordered?

SELECT
  COUNT(order_id) AS number_of_orders
FROM
  pizza_runner.customer_orders;
```
![image](https://github.com/Echooed/8-weeks-sql-challenge/assets/91009365/f124f6c0-a507-471b-b9ad-9958e21b2981)

#### ==> The total number  of orders is 14


### 2). How many unique customer orders were made?
``` mysql
SELECT
  COUNT(DISTINCT order_id) AS no_of_orders
FROM
  pizza_runner.customer_orders;
```
![image](https://github.com/Echooed/8-weeks-sql-challenge/assets/91009365/b5016afc-94bf-4b83-a6f5-9ca75b512230)

#### ==> 10 unique order were made.


