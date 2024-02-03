# Case Study 1
## Danny's Diner
![danny's diner image](https://github.com/Echooed/8-weeks-sql-challenge/assets/91009365/28516369-cfcf-48bc-967a-3616045e7f98)


## Business Problem Brief
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalized experience for his loyal customers.
Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

_**Danny has shared with us 3 key datasets for this case study:**_

- `sales`
- `menu`
- `members`

You can inspect the entity relationship diagram and example data below.

## Entity Relationship Diagram

![image](https://github.com/Echooed/8-weeks-sql-challenge/assets/91009365/e307009d-3372-44fb-8e03-5c3e3093790a)

## Requests

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


```mySQL
-- 1. What is the total amount each customer spent at the restaurant?
SELECT
  customer_id,
  SUM(price) AS Total_spent
FROM
  menu
  JOIN sales ON sales.product_id = menu.product_id
GROUP BY
  customer_id
ORDER BY
  Total_spent DESC;
```


```mySQL
-- 2. How many days has each customer visited the store?

SELECT
  customer_id,
  COUNT(order_date) AS number_of_days
FROM
  sales
GROUP BY
  customer_id;

```


```mySQL
-- 3. What was the first item from the menu purchased by each customer?

WITH ranked_sales AS (
  SELECT
    s.customer_id,
    s.order_date,
    m.product_name,
    ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS row_num
  FROM sales s
  JOIN menu m ON s.product_id = m.product_id
)

SELECT customer_id, order_date AS first_order_date, product_name
FROM ranked_sales
WHERE row_num = 1;
```

```mySQL
-- 3b. What is the most purchased item on the menu and how many times was it purchased by all customers?

SELECT
  sales.product_id,
  product_name,
  COUNT(sales.product_id) AS no_of_product
FROM
  sales
  JOIN menu ON sales.product_id = menu.product_id
GROUP BY
  sales.product_id,
  menu.product_name
ORDER BY no_of_product DESC LIMIT 1
;
```


```mysql
-- 4. How many times was the most purchased item purchased by each customer?

SELECT customer_id, product_name, COUNT(*) AS num_times_purchased FROM sales
JOIN menu
ON sales.product_id = menu.product_id
WHERE sales.product_id = (SELECT sales.product_id FROM sales
JOIN menu
ON sales.product_id = menu.product_id
GROUP BY sales.product_id
ORDER BY count(*) DESC
limit 1)
GROUP BY customer_id, product_name;
```

```mysql
-- 5. Which item was the most popular for each customer?

WITH count_product AS (
    SELECT s.customer_id, m.product_name, COUNT(s.product_id) AS no_of_product,
           ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) AS row_num
    FROM sales s
    JOIN menu m ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name
)
SELECT c.customer_id, c.product_name, c.no_of_product
FROM count_product c
WHERE c.row_num = 1;
```

```mysql
-- 6. Which item was the most popular for each customer?

WITH count_product AS (
    SELECT s.customer_id, m.product_name, COUNT(s.product_id) AS no_of_product,
           ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) AS row_num
    FROM sales s
    JOIN menu m ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name
)
SELECT c.customer_id, c.product_name, c.no_of_product
FROM count_product c
WHERE c.row_num = 1;
```

```mysql
-- 7. Which item was purchased just before the customer became a member?

WITH last_order_before_membership AS (
  SELECT
    s.customer_id,
    m.product_name,
    s.order_date,
    ROW_NUMBER() OVER (
      PARTITION BY s.customer_id
      ORDER BY
        s.order_date DESC
    ) AS purchase_rank
  FROM
    sales s
    JOIN menu m ON s.product_id = m.product_id
    JOIN members mem ON s.customer_id = mem.customer_id
  WHERE
    s.order_date < mem.join_date
)
SELECT
  customer_id,
  product_name,
  order_date
FROM
  last_order_before_membership
WHERE
  purchase_rank = 1;
```


```mysql

-- 8. What is the total items and amount spent for each member before they became a member?

SELECT
  s.customer_id,
  COUNT(s.product_id) AS purchase_count,
  CONCAT("$", SUM(m.price)) AS total_purchase
FROM
  sales s
  JOIN menu m ON s.product_id = m.product_id
  JOIN members mem ON s.customer_id = mem.customer_id
WHERE
  s.order_date < mem.join_date
GROUP BY
  s.customer_id;
```


```mysql

-- 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

WITH purchase_point AS (
  SELECT
    s.customer_id,
    SUM(m.price) AS total_spent,
    CASE
      WHEN product_name = "sushi" THEN SUM(m.price) * 20
      ELSE SUM(m.price) * 10
    END AS cust_point
  FROM
    sales s
    JOIN menu m ON s.product_id = m.product_id
  GROUP BY
    s.customer_id,
    m.product_name
  ORDER BY
    cust_point DESC
)
SELECT
  customer_id,
  CONCAT(SUM(cust_point), " ", "points") AS customer_points
FROM
  purchase_point
GROUP BY
  customer_id
```

```mysql

-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items,  not just sushi - how many points do customer A and B have at the end of January?

WITH joined_members AS (
    SELECT
        mem.customer_id,
        mem.join_date
    FROM members mem
    WHERE EXTRACT(YEAR_MONTH FROM mem.join_date) = 202101
),
customer_points AS (
    SELECT
        s.customer_id,
        m.price,
        m.product_name,
        s.order_date,
        j.join_date
    FROM sales s
    JOIN menu m ON s.product_id = m.product_id
    JOIN members j ON s.customer_id = j.customer_id
    WHERE s.order_date >= j.join_date
      AND s.order_date < DATE_ADD(j.join_date, INTERVAL 1 WEEK)
      
),
all_points AS (
    SELECT
        customer_id,
        SUM(CASE 
            WHEN DATEDIFF(order_date, join_date) <= 7 THEN price * 20
            ELSE price * 10
        END) AS total_points
    FROM customer_points
    GROUP BY customer_id
)
SELECT customer_id, CONCAT(total_points, " ", "points") members_points FROM all_points;
```

## Bonus question

```mysql
-- 1. Create a basic data table for Danny's employees

SELECT
  sales.customer_id,
  sales.order_date,
  menu.product_name,
  menu.price,
  CASE
    WHEN sales.order_date < members.join_date THEN "N"
    WHEN sales.order_date >= members.join_date THEN "Y"
    ELSE "N"
  END AS member
FROM
  sales
  JOIN menu ON sales.product_id = menu.product_id
  LEFT JOIN members ON sales.customer_id = members.customer_id
ORDER BY
  customer_id;
```



Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

```mysql
-- 2. Rank All The Things

WITH joined_table AS (
    SELECT sales.customer_id, 
           sales.order_date, 
           menu.product_name, 
           menu.price,
           CASE 
               WHEN sales.order_date < members.join_date THEN "N"
               WHEN sales.order_date >= members.join_date THEN "Y"			
               ELSE "N"
           END AS member
    FROM sales 
    JOIN menu ON sales.product_id = menu.product_id
    LEFT JOIN members ON sales.customer_id = members.customer_id 
    ORDER BY customer_id
)
SELECT customer_id, 
       order_date, 
       product_name, 
       price, 
       member,
       CASE WHEN member = "N" THEN "null"
          ELSE RANK() OVER (PARTITION BY member, customer_id ORDER BY order_date, price DESC)
       END AS ranking
FROM joined_table;
```
