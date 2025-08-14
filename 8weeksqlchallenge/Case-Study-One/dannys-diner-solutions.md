# Danny's Diner Case Study

Danny's Diner is a small restaurant that needs help analyzing its customer data to improve business. The owner, Danny, wants to understand customer visiting patterns, spending habits, and favorite menu items. By leveraging SQL to analyze data from the sales, menu, and members tables, the goal is to gain insights that will help enhance the customer loyalty program and provide a more personalized experience. The project aims to answer key business questions and generate easy-to-read data sets for Danny's team.

---

## Case Study Questions

**1. What is the total amount each customer spent at the restaurant?**
```sql
SELECT
 	customer_id,
  SUM(price)
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
USING(product_id)
GROUP BY (customer_id);
```

**2. How many days has each customer visited the restaurant?**
```sql
SELECT DISTINCT
  	customer_id,
    COUNT(DISTINCT(order_date))
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id ASC
```

**3. What was the first item from the menu purchased by each customer?**

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

**5. Which item was the most popular for each customer?**

**6. Which item was purchased first by the customer after they became a member?**

**7. Which item was purchased just before the customer became a member?**

**8. What is the total items and amount spent for each member before they became a member?**

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**