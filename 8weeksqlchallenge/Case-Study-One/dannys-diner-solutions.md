# Danny's Diner Case Study

Danny's Diner is a small restaurant that needs help analyzing its customer data to improve business. The owner, Danny, wants to understand customer visiting patterns, spending habits, and favorite menu items. By leveraging SQL to analyze data from the sales, menu, and members tables, the goal is to gain insights that will help enhance the customer loyalty program and provide a more personalized experience. The project aims to answer key business questions and generate easy-to-read data sets for Danny's team.

---
## Table Schema
```sql
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```

## Case Study Questions

**1. What is the total amount each customer spent at the restaurant?**
```sql
SELECT
 	sales.customer_id,
  SUM(menu.price)
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
USING(product_id)
GROUP BY (sales.customer_id);
```

**2. How many days has each customer visited the restaurant?**
```sql
SELECT DISTINCT
  	customer_id,
    COUNT(DISTINCT(order_date)) AS count_visit
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id ASC
```

**3. What was the first item from the menu purchased by each customer?**
```sql
WITH ordered_sales AS (SELECT 
    sales.customer_id, 
    sales.order_date, 
    menu.product_name,
DENSE_RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY sales.order_date) AS rank
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id)
    
SELECT
	customer_id,
    product_name
FROM ordered_sales
WHERE rank = 1
GROUP BY customer_id, product_name;
```
Note: This will give two orders for Customer A, however order date contains no timestamps so we can not determine what this customer ordered first

Also:
```sql
SELECT DISTINCT
    s.customer_id,
    m.product_name
FROM sales s
JOIN menu m
    ON s.product_id = m.product_id
WHERE s.order_date = (
    SELECT MIN(order_date)
    FROM sales
    WHERE customer_id = s.customer_id
)
ORDER BY s.customer_id, m.product_name;
```

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
```sql
SELECT 
	COUNT(s.product_id) AS count_of_purchases,
    m.product_name
FROM sales AS s
INNER JOIN menu AS m 
USING(product_id)
GROUP BY s.product_id, m.product_name
ORDER BY s.product_id DESC
LIMIT 1
```

**5. Which item was the most popular for each customer?**
```sql
WITH count_of_purchases AS (SELECT
  	customer_id,
    product_name,
	product_id,
	COUNT(product_id) AS product_count
FROM sales
INNER JOIN menu USING(product_id)
GROUP BY customer_id, product_id, product_name
ORDER BY customer_id, product_id)

,  ranked AS (SELECT
	customer_id,
    product_name,
    product_count,
    RANK() OVER (
      PARTITION BY customer_id
      ORDER BY product_count DESC
      ) AS ranked_products
FROM count_of_purchases
)

SELECT
	customer_id,
    product_name,
    product_count
FROM ranked
WHERE ranked_products = 1
```

**6. Which item was purchased first by the customer after they became a member?**
```sql
with purchase_orders AS ( SELECT
	s.customer_id,
    m.join_date,
    s.order_date,
    mu.product_name,
    DENSE_RANK() OVER(
    PARTITION BY s.customer_id
    ORDER BY s.order_date) AS rank  
FROM
	sales AS s
INNER JOIN
	members AS m ON s.customer_id = m.customer_id
INNER JOIN
	menu AS mu ON s.product_id = mu.product_id
WHERE
	s.order_date >= m.join_date
ORDER BY s.customer_id)

SELECT
	customer_id,
    product_name
FROM
	purchase_orders
WHERE 
	rank = 1
ORDER BY
	customer_id
```

The above would assume that on the date they signed up, that they signed up first and then they ordered, hence the 's.order_date >= m.join_date'. If that isn't the case that should be replaced with 's.order_date > m.join_date'

**7. Which item was purchased just before the customer became a member?**

**8. What is the total items and amount spent for each member before they became a member?**

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**