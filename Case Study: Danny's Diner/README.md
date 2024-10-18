# Restaurant Data Analysis

***

## Problem Statement

A restaurant owner wants to analyze his customer data to understand their visiting patterns, spending habits, and favorite menu items. These insights will help him personalize the customer experience and decide whether to expand the customer loyalty program. He also needs to generate basic datasets for his team to inspect without using SQL.

He has provided three key datasets: sales, menu, and members, along with a sample of his customer data to write SQL queries and answer his questions.

***

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***

## Objective

The objective of this project is to enhance my PostgreSQL skills by analyzing customer data. Through this project, I aim to answer specific questions about customer visiting patterns, spending behavior, and menu preferences using SQL queries. This analysis will help Danny deliver a better and more personalized experience for his loyal customers and make informed decisions about expanding the customer loyalty program.

***

## Data

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

#### Explanation:

- Join the `sales` table with the menu table to get the price of each product.
- Group by `customer_id` and calculate the total amount spent by each customer.
- Order the results by `customer_id` in ascending order.
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

#### Explanation:

- Select the `customer_id from` the sales table.
- Count distinct `order_date` for each `customer_id`.
- Group by `customer_id` to get the number of days each customer visited.

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
#### Explanation:

- Create a CTE named `ordered_sales` that:
- Selects the `customer_id`, `order_date`, and `product_name`.
- Joins the `dannys_diner.sales` and `dannys_diner.menu` tables on the `product_id` column.
- Uses the **DENSE_RANK()** window function to rank each `customer_id` partition based on the `order_date` in ascending order.
- In the outer query: Selects the `customer_id` and `product_name`.
- Filters the results to include only the rows where the rank equals 1, representing the first item purchased by each customer.
- Groups the results by `customer_id` and `product_name`.

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

#### Explanation:

- Join the sales and menu tables using the `product_id` column.
- Group the results by `menu.product_name`.
- Count the occurrences of each product using **COUNT**(`sales.product_id`) and alias it as `amount_purchased`.
- Order the results by `amount_purchased` in descending order.
- Limit the results to the top one row using **LIMIT 1**.

#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |

- The most purchased menu item is Ramen, with 8 orders.

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

#### Explanation:

- Create a CTE named `most_popular_item` that:
- Selects the `customer_id`, `product_name`, and calculates the count of product_id occurrences for each group.
- Joins the `dannys_diner.menu` and `dannys_diner.sales` tables on the `product_id` column.
- Groups the results by `customer_id` and `product_name`.
- Uses the **DENSE_RANK()** window function to rank each `customer_id` partition based on the count of orders **COUNT**(`sales.customer_id`) in descending order.
- In the outer query: Selects the `customer_id`, `product_name`, and `order_count`.
- Filters the results to include only the rows where the rank equals 1, representing the most frequently ordered item for each customer.

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

#### Explanation:

- Create a CTE named `joined_as_member` that:
- Selects the appropriate columns and calculates the row number using **ROW_NUMBER()**. The **PARTITION BY** clause divides the data by `members.customer_id`, and the **ORDER BY** clause orders the rows within each `members.customer_id` partition by `sales.order_date`.
- Joins the `dannys_diner.members` and `dannys_diner.sales` tables on the `customer_id` column, including only sales that occurred after the member's join date (`sales.order_date` > `members.join_date`).
- In the outer query: Joins the `joined_as_member` CTE with the `dannys_diner.menu` table on the `product_id` column.
- Filters to retrieve only the rows where row_num equals 1, representing the first row within each `customer_id` partition.
- Orders the result by `customer_id` in ascending order.

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

#### Explanation:

- Create a CTE named `ranked_sales` that:
- Selects the appropriate columns and calculates the row number using **ROW_NUMBER()**. The **PARTITION** BY clause divides the data by `sales.customer_id`, and the ORDER BY clause orders the rows within each `sales.customer_id` partition by `sales.order_date` in descending order.
- Joins the `dannys_diner.sales` and `dannys_diner.members` tables on the `customer_id` column, including only sales that occurred before the member's join date (`sales.order_date` < `members.join_date`).
- In the outer query: Joins the ranked_sales CTE with the `dannys_diner.menu` table on the `product_id` column.
- Filters to retrieve only the rows where row_num equals 1, representing the most recent sale before the join date within each `customer_id` partition.
- Orders the result by `customer_id` in ascending order.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | sushi        |
| B           | sushi        |

- Both customers' last order before becoming members is sushi.

***

**8. What is the total items and amount spent for each member before they became a member?**

````sql
WITH sales_before_membership AS (
  SELECT 
    sales.customer_id,
    sales.product_id,
    sales.order_date,
    menu.price
  FROM dannys_diner.sales
  JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  WHERE sales.order_date < members.join_date
)

SELECT 
  sales_before_membership.customer_id,
  COUNT(sales_before_membership.product_id) AS total_items,
  SUM(sales_before_membership.price) AS total_amount_spent
FROM sales_before_membership
GROUP BY sales_before_membership.customer_id
ORDER BY sales_before_membership.customer_id ASC;
````

#### Explanation:

- Create a CTE named `sales_before_membership` that:
- Selects the appropriate columns, including `customer_id`,`product_id`, `order_date`, and `price`.
- Joins the `dannys_diner.sales` and `dannys_diner.members` tables on the `customer_id` column.
- Joins the `dannys_diner.sales` and `dannys_diner.menu` tables on the `product_id` column.
- Applies a condition to include only sales that occurred before the member's join date (`sales.order_date` < `members.join_date`).
- In the outer query: Groups the results by `customer_id` and calculates the total number of items and the total amount spent before membership for each customer.
- Orders the result by `customer_id` in ascending order.

#### Answer:
| customer_id | total_items | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

Before becoming members,
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items.

***

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

````sql
WITH sales_as_points AS (
  SELECT
    sales.customer_id,
    sales.product_id,
    menu.product_name,
    menu.price,
    CASE
      WHEN menu.product_name = 'sushi' then (menu.price * 2 * 10)
      ELSE (menu.price * 10)
    END AS points
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT
  sales_as_points.customer_id,
  SUM(sales_as_points.points) AS total_points
FROM sales_as_points
GROUP BY sales_as_points.customer_id
ORDER BY sales_as_points.customer_id ASC;
````

#### Explanation:

- Create a CTE named `sales_as_points` that:
- Selects the appropriate columns, including `customer_id`, `product_id`, `product_name`, and `price`.
- Joins the `dannys_diner.sales` and `dannys_diner.menu` tables on the `product_id` column.
- Uses a CASE statement to calculate points:
- Multiplies the price by 20 for sushi (2x multiplier).
- Multiplies the price by 10 for all other products.
- In the outer query: Groups the results by `customer_id` and sums the total points for each customer.
- Orders the result by `customer_id` in ascending order.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Total points for Customer A is 860.
- Total points for Customer B is 940.
- Total points for Customer C is 360.

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

````sql
WITH sales_as_points AS (
  SELECT
    sales.customer_id,
    sales.product_id,
    sales.order_date,
    menu.product_name,
    menu.price,
    members.join_date,
    CASE
      WHEN sales.order_date BETWEEN members.join_date 
      AND members.join_date + INTERVAL '6 days' THEN (menu.price * 2 * 10)
      WHEN menu.product_name = 'sushi' THEN (menu.price * 2 * 10)
      ELSE (menu.price * 10)
    END AS points
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
)

SELECT
  sales_as_points.customer_id,
  SUM(sales_as_points.points) AS total_points
FROM sales_as_points
WHERE sales_as_points.order_date <= '2021-01-31'
AND sales_as_points.customer_id IN ('A', 'B')
GROUP BY sales_as_points.customer_id
ORDER BY sales_as_points.customer_id ASC;
````

#### Explanation:

- Create a CTE named `sales_as_points` that:
- Selects the appropriate columns, including `customer_id`, `product_id`, `order_date`, `product_name`, `price`, and `join_date`.
- Joins the `dannys_diner.sales` and `dannys_diner.menu` tables on the `product_id` column.
- Joins the `dannys_diner.sales` and `dannys_diner.members` tables on the `customer_id` column.
- Uses a CASE statement to calculate points:
- Applies a 2x points multiplier (price * 2 * 10) for orders within the first week after joining (from `join_date` to `join_date` + **INTERVAL** '6 days').
- Applies a 2x points multiplier for sushi (price * 2 * 10) outside the first week.
- Applies the standard points multiplier (price * 10) for all other cases.
- In the outer query: Filters the results to include only sales up to January 31, 2021, and only for customers A and B.
- Groups the results by `customer_id` and sums the total points for each customer.
- Orders the result by `customer_id` in ascending order.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 820 |

- Total points for Customer A is 1,370.
- Total points for Customer B is 820.

***

