# Case Study: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

***

**Schema:**

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

***

**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT 
  sales.customer_id, 
  SUM(menu.price) AS total_amount_spent
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
````

#### Steps:

#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

***

**2. How many days has each customer visited the restaurant?**

````sql
SELECT
  sales.customer_id,
  COUNT(distinct order_date) AS days_visited_count
FROM dannys_diner.sales
GROUP by customer_id;
````

#### Steps:

#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

***

**3. What was the first item from the menu purchased by each customer?**

````sql
WITH ordered_sales AS (
  SELECT 
    sales.customer_id, 
    sales.order_date, 
    menu.product_name,
    DENSE_RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT 
  customer_id, 
  product_name
FROM ordered_sales
WHERE rank = 1
GROUP BY customer_id, product_name;
````
#### Steps:

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

- Customer A placed an order for both curry and sushi simultaneously, making them the first items in the order.
- Customer B's first order is curry.
- Customer C's first order is ramen.

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

````sql
SELECT
  menu.product_name,
  COUNT(sales.product_id) AS amount_purchased
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY amount_purchased DESC
LIMIT 1;
````

#### Steps:

#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |

***

**5. Which item was the most popular for each customer?**

````sql
WITH most_popular_item AS (
  SELECT
  sales.customer_id,
  menu.product_name,
  COUNT(menu.product_id) AS order_count,
  DENSE_RANK() OVER (
      PARTITION BY sales.customer_id
      ORDER BY COUNT(sales.customer_id) DESC) AS rank
  FROM dannys_diner.menu
  JOIN dannys_diner.sales
    ON menu.product_id = sales.product_id
  GROUP BY sales.customer_id, menu.product_name
)

SELECT 
  customer_id, 
  product_name, 
  order_count
FROM most_popular_item 
WHERE rank = 1;
````

#### Steps:

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A and C's favourite item is ramen.
- Customer B enjoys curry and ramen equally.

***

**6. Which item was purchased first by the customer after they became a member?**

````sql
WITH joined_as_member AS (
  SELECT
    members.customer_id, 
    sales.product_id,
    DENSE_RANK() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.members
  JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND sales.order_date > members.join_date
)

SELECT 
  customer_id, 
  product_name 
FROM joined_as_member
INNER JOIN dannys_diner.menu
  ON joined_as_member.product_id = menu.product_id
WHERE rank = 1
ORDER BY customer_id ASC;
````

#### Steps:

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | ramen        |
| B           | sushi        |

- Customer A's first order as a member is ramen.
- Customer B's first order as a member is sushi.

***

**7. Which item was purchased just before the customer became a member?**

````sql
WITH ranked_sales AS (
  SELECT 
    sales.customer_id,
    sales.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY sales.customer_id
      ORDER BY sales.order_date DESC
    ) AS row_num
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
    AND sales.order_date < members.join_date
)

SELECT 
  rs.customer_id, 
  menu.product_name 
FROM ranked_sales rs
INNER JOIN dannys_diner.menu
  ON rs.product_id = menu.product_id
WHERE rs.row_num = 1
ORDER BY rs.customer_id ASC;
````

#### Steps:

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | sushi        |
| B           | sushi        |

- Both customers' last order before becoming members is sushi.

***


