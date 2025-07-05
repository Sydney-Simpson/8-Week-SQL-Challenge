# 8 Week SQL Challenge Case #1 Danny's Diner

![dinerlogo](https://github.com/user-attachments/assets/235d3e70-1ff4-4a84-b8f2-1ba54708261a)

This case study is from the 8 week SQL challenge, and can be found [here](https://8weeksqlchallenge.com/case-study-1/).

# Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

# Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/5fd12d34-b733-4be7-87ef-3e4f3e653d3e)


# Case Study Questions and Solutions

All of these queries were executing using PostgreSQL on [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

***

**1. What is the total amount each customer spent at the restaurant?**

```sql

SELECT
  	s.customer_id,
   	SUM(m.price)
FROM menu m
INNER JOIN sales s
ON m.product_id = s.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;

```
<br/>

| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

<br/>

***

**2. How many days has each customer visited the restaurant?**

```sql

SELECT 
	s.customer_id,
    COUNT(DISTINCT s.order_date)
FROM sales
GROUP BY s.customer_id
ORDER BY s.customer_id;

```
<br/>

| customer_id  | product_name |
|--------------|--------------|
| A            | 4            |
| B            | 6            |
| C            | 2            |

- Customer A visited the restaurant 6 days
- Customer B visited the restaurant 6 days
- Customer C visited the restaurant 3 days
  
<br/>

***


**3. What was the first item from the menu purchased by each customer?**

```sql

WITH first_order as (
  SELECT customer_id,
  min(order_date) as order_date
  FROM sales
  GROUP BY customer_id)
SELECT 
	s.customer_id,
    m.product_name
FROM sales s
INNER JOIN first_order fo
ON s.customer_id = fo.customer_id
AND s.order_date = fo.order_date
INNER JOIN menu m
ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name
ORDER BY s.customer_id, m.product_name;

```

<br/>

| customer_id  | product_name |
|--------------|--------------|
| A            | curry        |
| A            | sushi        |
| B            | curry        |
| C            | ramen        |


- Customer A's first order consisted of curry and sushi
- Customer B's first order was curry
- Customer C's first order was ramen
  
<br/>

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql

SELECT	
  	m.product_name as most_purchased_product,
    COUNT(m.product_name) as times_purchased
FROM sales as s
INNER JOIN menu m
ON s.product_id = m.product_id
GROUP BY most_purchased_product
ORDER BY times_purchased DESC
LIMIT 1;

WITH most_purchased as(
  SELECT
  product_id,
  COUNT(product_id) as times_purchased
  FROM sales
  GROUP BY product_id
  ORDER BY times_purchased DESC
  LIMIT 1)
  
SELECT 
 	s.customer_id,
    count(s.product_id) as number_of_ramen_orders
FROM sales as s
INNER JOIN most_purchased mp
ON s.product_id = mp.product_id
GROUP BY s.customer_id;


```

<br/>

| most_purchased_product | times_purchased |
|------------------------|-----------------|
| ramen                  | 8               |


- Ramen is the most popular product and was purchased a total of 8 times- specifically

<br/>

| customer_id  | number_of_ramen_orders |
|--------------|------------------------|
| A            | 3                      |
| B            | 2                      |
| C            | 3                      |


- Customer A purchased ramen 3 times
- Customer B purchased ramen 2 times
- Customer C purchased ramen 3 times

<br/>

***

**5. Which item was the most popular for each customer?**
``` sql

WITH customer_favorites as ( 
  SELECT
  customer_id,
  product_id,
  COUNT(product_id) as times_purchased,
  RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(product_id) DESC) as item_rank
  FROM sales
  GROUP BY customer_id, product_id)
SELECT
	cf.customer_id,
    m.product_name,
    cf.times_purchased
FROM menu m
INNER JOIN customer_favorites cf
ON m.product_id = cf.product_id
WHERE item_rank = 1
ORDER BY cf.customer_id;

```

<br/>

| customer_id | product_name | times_purchased |
| ----------- | ----------   |------------     |
| A           | ramen        |  3              |
| B           | sushi        |  2              |
| B           | curry        |  2              |
| B           | ramen        |  2              |
| C           | ramen        |  3              |

- Customer A and C purchased ramen more than anyother tiem
- Customer B purchased all items an equal number of times

<br/>

***

6. Which item was purchased first by the customer after they became a member?
```sql

WITH ranked_dates as (
  SELECT
	s.customer_id,
    s.product_id,
    s.order_date,
    RANK() OVER (PARTITION BY s.customer_id ORDER BY order_date) as rnk
	FROM sales s
	LEFT JOIN members mem
	ON s.customer_id = mem.customer_id
	WHERE s.order_date >= mem.join_date)
SELECT 
	s.customer_id,
    m.product_name
FROM sales s
INNER JOIN ranked_dates rd
ON s.customer_id = rd.customer_id
AND s.order_date = rd.order_date
INNER JOIN menu m
ON s.product_id = m.product_id 
WHERE rnk =1
ORDER BY s.customer_id;

```

<br/>

| customer_id | product_name |
| ----------- | ----------   |
| A           | curry        |
| B           | sushi        |

- Customer A's first purchase after becoming a member was curry. Note that they purchased curry the same day that they became a member. We're going to assume that they were so thrilled with the membership that they bought curry directly afterwards.
- Customer B's first purchase as a member was sushi.
  
<br/>

 ***
 
**7. Which item was purchased just before the customer became a member?**

```sql
WITH ranked_dates as (
  SELECT
	s.customer_id,
    s.product_id,
    s.order_date,
    RANK() OVER (PARTITION BY s.customer_id ORDER BY order_date DESC) as rnk
	FROM sales s
	LEFT JOIN members mem
	ON s.customer_id = mem.customer_id
	WHERE s.order_date < mem.join_date)
SELECT 
	s.customer_id,
    m.product_name,
    s.order_date
FROM sales s
INNER JOIN ranked_dates rd
ON s.customer_id = rd.customer_id
AND s.order_date = rd.order_date
AND s.product_id = rd.product_id
INNER JOIN menu m
ON s.product_id = m.product_id 
WHERE rnk =1
ORDER BY s.customer_id;

```

<br/>

| customer_id | product_name |
| ----------- | ----------   |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

- Customer A purchased sushi and curry before becoming a member. Since both sushi and curry were purchased on the same day we assume they were bought at the same time, as we do not have any information regarding the time of purchase.
- Customer B bought sushi before becoming a member.
  
<br/>

***

8. What is the total items and amount spent for each member before they became a member?

```sql

WITH orders_before_membership as (
  SELECT
	s.customer_id,
    s.product_id
	FROM sales s
	LEFT JOIN members mem
	ON s.customer_id = mem.customer_id
	WHERE s.order_date < mem.join_date)
SELECT
	obm.customer_id,
    COUNT(obm.product_id) as number_of_items_bought,
    SUM(m.price) as total_spent
FROM orders_before_membership obm
LEFT JOIN menu m
ON obm.product_id = m.product_id
GROUP BY obm.customer_id
ORDER BY obm.customer_id;

```

<br/>

| customer_id | number_of_items_bought | total_spent |
| ----------- | -----------------------|-------------|
| A           | 2                      | 25          |
| B           | 3                      | 40          |

- Customer A bought 2 items and spent $25 before becoming a member.
- Customer B bought 3 items and spent $40 before becoming a member.
  
<br/>

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```sql
WITH points_per_order as (
  SELECT 
	s.customer_id,
	CASE WHEN m.product_name = 'sushi' THEN m.price * 20
	ELSE m.price * 10
	END AS points
	FROM sales s
	INNER JOIN menu m
	ON s.product_id = m.product_id)
SELECT 
	ppo.customer_id,
    SUM(ppo.points) as total_points
FROM points_per_order ppo
GROUP BY ppo.customer_id
ORDER BY ppo.customer_id;

```

<br/>

| customer_id | total_points |
| ----------- | ----------   |
| A           | 860          |
| A           | 940          |
| B           | 360          |
  
<br/>

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql

WITH dates AS (
  SELECT 
    customer_id, 
      join_date, 
      join_date + 6 AS first_week_date, 
      DATE_TRUNC(
        'month', '2021-01-31'::DATE)
        + interval '1 month' 
        - interval '1 day' AS last_date
  FROM members
)

SELECT 
  s.customer_id, 
  SUM(CASE
    WHEN m.product_name = 'sushi' THEN 2 * 10 * m.price
    WHEN s.order_date BETWEEN d.join_date AND d.first_week_date THEN 2 * 10 * m.price
    ELSE 10 * m.price END) AS points
FROM sales s
INNER JOIN dates d
  ON s.customer_id = d.customer_id
  AND d.join_date <= s.order_date
  AND s.order_date <= d.last_date
INNER JOIN menu m
  ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;

```

<br/>

| customer_id | total_points |
| ----------- | ----------   |
| A           | 1020         |
| A           | 320          |

- Customer A had 1020 points at the end of January
- Customer B had 320 points at the end of January
