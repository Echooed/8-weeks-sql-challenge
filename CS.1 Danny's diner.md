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

## Questions

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
-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

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
