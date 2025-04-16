

#  Case Study #1: Danny's Diner

## Solution

View the complete syntax [here] (https://github.com/sandeep177/SQL-Projects/blob/master/SQL%20Syntax).

***

### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT 
  s.customer_id, 
  SUM(m.price) AS total_sales
FROM sales AS s
JOIN menu AS m
  ON s.product_id = m.product_id
GROUP BY s.customer_id; 
````

#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 304         |
| B           | 296         |
| C           | 144         |

***

### 2. How many days has each customer visited the restaurant?

````sql
SELECT 
   customer_id,
   COUNT(DISTINCT order_date) AS visit_days
FROM sales
GROUP BY customer_id;
....

#### Answer:
| customer_id | visit_days |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |
***

### 3. What was the first item from the menu purchased by each customer?

````sql
WITH
 first_purchase AS (
 SELECT
   s.customer_id,
   s.order_date,
   m.product_name,
   RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rnk
 FROM sales as s
 JOIN menu as m
 ON s.product_id = m.product_id
)
SELECT 
  customer_id,
  product_name
FROM first_purchase
WHERE rnk = 1
GROUP BY customer_id, product_name;
````

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | sushi        | 
| A           | curry        | 
| B           | curry        | 
| C           | ramen        |

***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT
  m.product_name,
  COUNT(*) AS times_purchased
FROM sales as s
JOIN menu as m
  ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY times_purchased DESC 
LIMIT 1;
````

#### Answer:
| product_name | times_purchased | 
| ----------- | ----------- |
| ramen       | 32          |

***

### 5. Which item was the most popular for each customer?

````sql
WITH item_counts AS (
 SELECT
   s.customer_id,
   m.product_name,
   COUNT(*) AS item_count,
   RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(*) DESC) AS rnk
 FROM sales as s
 JOIN menu as m
 ON s.product_id = m.product_id
 GROUP BY s.customer_id, m.product_name
)
SELECT 
  customer_id,
  product_name,
  item_count
FROM item_counts
WHERE rnk = 1 ;

````

#### Answer:
| customer_id | product_name | item_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  12   |
| B           | sushi        |  8    |
| B           | curry        |  8    |
| B           | ramen        |  8    |
| C           | ramen        |  12   |

***

### 6. Which item was purchased first by the customer after they became a member?

````sql
WITH joined_sales AS (
  SELECT 
    s.customer_id, s.order_date, s.product_id, m.join_date,
   RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rnk
  FROM sales AS s
  JOIN members AS m
    ON s.customer_id = m.customer_id
  WHERE s.order_date >= m.join_date
)
SELECT
  js.customer_id,
  m2.product_name,
  js.order_date
FROM joined_sales as js
JOIN menu as m2
  ON js.product_id = m2.product_id
WHERE js.rnk = 1
GROUP BY js.customer_id, m2.product_name, js.order_date;
````

#### Answer:
| customer_id | product_name | order_date  | 
| ----------- | ---------- |----------  |
| B           | sushi      | 2021-01-11 | 
| A           | curry      |2021-01-07  |

***

### 7. Which item was purchased just before the customer became a member?

````sql
WITH before_becoming_member AS (
   SELECT s.customer_id, s.order_date, s.product_id, m.join_date,
      RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rnk
	FROM sales as s
    JOIN members as m
      ON s.customer_id = m.customer_id
	WHERE s.order_date < m.join_date
)
SELECT 
  bm.customer_id,
  m2.product_name,
  bm.order_date
FROM before_becoming_member as bm
jOIN menu as m2
  ON bm.product_id = m2.product_id
WHERE bm.rnk = 1
GROUP BY bm.customer_id, m2.product_name, bm.order_date;
....

#### Answer:
| customer_id | product_name | order_date  | 
| ----------- | ---------- |----------  |
| A           | sushi      | 2021-01-01 |
| B           | sushi      | 2021-01-04 |
| A           | curry      | 2021-01-01 |  

***

### 8. What is the total items and amount spent for each member before they became a member?

````sql
SELECT  s.customer_id,
  COUNT(*) AS total_items,
  SUM(m.price) AS total_amount
FROM sales as s
JOIN menu as m
  ON s.product_id = m.product_id
JOIN members as mb
  ON s.customer_id = mb.customer_id
WHERE s.order_date < mb.join_date
GROUP BY s.customer_id;
````

#### Answer:
| customer_id | total_items | total_amount |
| ----------- | ----------- |----------- |
| A           | 32          |  400       |
| B           | 28          |  640       |

***

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

````sql
WITH points AS
(
   SELECT *, 
      CASE
         WHEN product_id = 1 THEN price *20
         ELSE price *10
      END AS points
   FROM menu
)
SELECT s.customer_id, SUM(p.points) AS total_points
FROM points AS p
JOIN sales AS s
   ON p.product_id = s.product_id
GROUP BY s.customer_id;
````

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 3440       |
| B           | 3760       |
| C           | 1440       |

***

### 10. 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

````sql
SELECT
s.customer_id,
SUM(
CASE
WHEN s.order_date BETWEEN m.join_date AND DATE_ADD(m.join_date, INTERVAL 6 DAY)
THEN menu.price * 10 * 2
ELSE menu.price * 10
END
) AS total_points
FROM sales s
JOIN menu ON s.product_id = menu.product_id
JOIN members m ON s.customer_id = m.customer_id
WHERE s.customer_id IN ('A', 'B')
GROUP BY s.customer_id;
````

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 20320      |
| B           | 13440      |

***

