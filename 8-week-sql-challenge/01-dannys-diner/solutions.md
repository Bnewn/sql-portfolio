# Case Study #1: Danny's Diner Questions and Solutions

### 1. What is the total amount each customer spent at the restaurant?

**Query**
```sql
SELECT
    s.customer_id,
    sum(m.price) as total_spent
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id
```
**Results**
| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

**Insights**
- Customer A is the highest spender at $76, making them a prime candidate for loyalty rewards
- Customer B is the second highest spender at $74
- Customer C is the least spender at $36, indicating potential for engagement opportunities

---

### 2. How many days has each customer visited the restaurant?

**Query**
```sql
SELECT
    customer_id,
    COUNT(DISTINCT order_date) AS days_visited
FROM sales
GROUP BY customer_id
ORDER BY customer_id;
```

**Results**
| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

**Insights**
- Customer B is the most frequent visitor with 6 days, showing strong engagement 
- Customer C has only visited twice, presenting an opportunity for retention campaigns
- This 3x difference in visit frequency suggests customers respond differently to the dining experience between two different people

----

### 3. What was the first item from the menu purchased by each customer?

**Query**
```sql
WITH first_purchase AS(
    SELECT 
        s.customer_id,
        s.order_date,
        m.product_name,
        RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS order_rank
    FROM sales s
    JOIN menu m ON s.product_id = m.product_id
    )
SELECT 
    customer_id,
    product_name AS first_item
FROM first_purchase
WHERE order_rank = 1;
```

**Results**
| customer_id | first_item |
| ----------- | ---------- |
| A           | curry      |
| A           | sushi      |
| B           | curry      |
| C           | ramen      |
| C           | ramen      |

**Insights**
- Curry emerges as the most popular first purchase, ordered by both Customer A and B on their initial visit. 
- Customer A ordered both curry and sushi, while Customer C ordered ramen twice. This suggests customers are comfortable making multiple purchases immediately, which could inform menu pricing strategies and combo meal offerings for new customers.

----

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

**Query**
```sql
SELECT 
    m.product_name AS most_purchased_item,
    count(s.product_id) AS times_purchased
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY m.product_name
LIMIT 1;
```

**Results**
| most_purchased_item | times_purchased |
| ------------------- | --------------- |
| ramen               | 8               |

**Insights**
- Ramen is the most purchased items, selling 8 times amongst all customers

---


### 5. Which item was the most popular for each customer?

**Query**
```sql
WITH item_counts AS (
    SELECT
        s.customer_id,
        m.product_name,
        count(s.product_id) AS times_purchased,
        RANK()OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) AS rank
    FROM sales s
    JOIN menu m ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name)
SELECT
    customer_id,
    product_name AS most_popular_item,
    times_purchased
FROM item_counts
WHERE rank = 1
ORDER BY customer_id;
```

**Results**
| customer_id | most_popular_item | times_purchased |
| ----------- | ----------------- | --------------- |
| A           | ramen             | 3               |
| B           | ramen             | 2               |
| B           | curry             | 2               |
| B           | sushi             | 2               |
| C           | ramen             | 3               |

**Insights**
- Ramen is the clear favorite for customers A and C with 3 purchases each
- Customer B shows no preference, purchasing all items equally
- This suggests ramen has broad appeal while some customers value menu diversity

---


### 6. Which item was purchased first by the customer after they became a member?

**Query**
```sql
WITH member_first_purchase AS(
    SELECT
        s.customer_id,
        s.order_date,
        m.product_name,
        mb.join_date,
        RANK()OVER (PARTITION BY s.customer_id ORDER BY order_date) AS rank
    FROM sales s
    JOIN members mb ON s.customer_id = mb.customer_id
    JOIN menu m ON s.product_id = m.product_id
    WHERE s.order_date >= mb.join_date)
SELECT
    customer_id,
    product_name as first_item_as_member,
    order_date,
    join_date
FROM member_first_purchase
WHERE rank = 1;
```

**Results**
| customer_id | first_item_as_member | order_date | join_date  |
| ----------- | -------------------- | ---------- | ---------- |
| A           | curry                | 2021-01-07 | 2021-01-07 |
| B           | sushi                | 2021-01-11 | 2021-01-09 |

---
**Insights**
- Customer A purchased curry on the same day as becoming a member, while customer B purchased sushi 2 days after becoming a member. 
- Customer C is yet to become a member

---


### 7. Which item was purchased just before the customer became a member?

**Query**
```sql
WITH purchase_before_membership AS(
    SELECT
        s.customer_id,
        s.order_date,
        m.product_name,
        mb.join_date,
        RANK()OVER (PARTITION BY s.customer_id ORDER BY order_date DESC) AS rank
    FROM sales s
    JOIN members mb ON s.customer_id = mb.customer_id
    JOIN menu m ON s.product_id = m.product_id
    WHERE s.order_date < mb.join_date)
SELECT
    customer_id,
    product_name as last_item_before_membership,
    order_date,
    join_date
FROM purchase_before_membership
WHERE rank = 1;
```

**Results**
| customer_id | last_item_before_membership | order_date | join_date  |
| ----------- | --------------------------- | ---------- | ---------- |
| A           | sushi                       | 2021-01-01 | 2021-01-07 |
| A           | curry                       | 2021-01-01 | 2021-01-07 |
| B           | sushi                       | 2021-01-04 | 2021-01-09 |

**Insights**
- Both customers A and B purchased sushi just before joining the loyalty program
- Customer A ordered both sushi and curry on their final pre-membership visit
- There was a 5-6 day gap between last purchase and membership signup, suggesting customers joined after experiencing the restaurant

---


### 8. What is the total items and amount spent for each member before they became a member?

**Query**
```sql
SELECT 
    s.customer_id,
    count(m.product_name) as total_items,
    sum(m.price) as amount_spent
FROM sales s
JOIN members mb USING(customer_id)
JOIN menu m USING(product_id)
WHERE s.order_date < mb.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

**Results**
| customer_id | total_items | amount_spent |
| ----------- | ----------- | ------------ |
| A           | 2           | 25           |
| B           | 3           | 40           |

**Insights**
- Customer B spent $40 on 3 items before joining, while Customer A spent $25 on 2 items
- Both customers established purchasing history before converting to membership, suggesting they joined after positive initial experiences
- Customer B's higher pre-membership spending indicates stronger early engagement with the restaurant

---


### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

**Query**
```sql
SELECT 
    s.customer_id,
    SUM(CASE 
            WHEN m.product_name = 'sushi' THEN m.price * 20
            ELSE m.price * 10
        END
        ) AS total_points
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

**Results**
| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

**Insights**
- Customer B leads in with 940 loyalty points, followed closely by Customer A with 860 points 
- Customer C would have 360 points if they joined the program. The lower points indicate they are either spending less overall, or they're buying fewer high-value items like sushi
- Customer C is a prime target for membership conversion. They are not benefiting from the rewards program 

---


### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

**Query**
```sql
SELECT 
    s.customer_id,
    SUM(
        CASE
            WHEN s.order_date BETWEEN mb.join_date and mb.join_date + 6 then m.price * 20
            WHEN m.product_name = 'sushi' then m.price * 20
            ELSE m.price * 10
        END
        ) AS total_points
FROM sales s 
JOIN menu m ON s.product_id = m.product_id
JOIN members mb ON s.customer_id = mb.customer_id
WHERE s.order_date <= '2021-01-31' AND s.order_date >= mb.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

**Results**
| customer_id | total_points |
| ----------- | ------------ |
| A           | 1020         |
| B           | 320          |

**Insights**
- Customer A earned a total of 1020 points, Customer B earned a total of 320 points. 
- Customer A has almost 3 times the amount of points that customer B
- Customer C is not a member therefore is not able to earn any points on his purchases

---


### Bonus Question 1. Join all the things

**Query**
```sql
SELECT
    s.customer_id,
    s.order_date,
    m.product_name,
    m.price,
    CASE
        WHEN s.order_date >= mb.join_date THEN 'Y'
        ELSE 'N'
    END AS member
FROM sales s
LEFT JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mb on s.customer_id = mb.customer_id
ORDER BY customer_id, order_date, product_name;
```

**Results**
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

**Insights**
- Displays all sales per customer with order date, product name, price and their membership status on the purchase date

---


### Bonus Question 2. Rank all the things

**Query**
```sql
WITH member_data AS(
SELECT
    s.customer_id,
    s.order_date,
    m.product_name,
    m.price,
    CASE
        WHEN s.order_date >= mb.join_date THEN 'Y'
        ELSE 'N'
    END AS member
FROM sales s
LEFT JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mb on s.customer_id = mb.customer_id)
SELECT
    *,
    CASE 
        WHEN member = 'Y' THEN RANK()OVER(PARTITION BY customer_id, member ORDER BY order_date)
        ELSE NULL
    END AS ranking
FROM member_data
ORDER BY customer_id, order_date, product_name;
```

**Results**
| customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | curry        | 15    | N      | null    |
| A           | 2021-01-01 | sushi        | 10    | N      | null    |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      | null    |
| B           | 2021-01-02 | curry        | 15    | N      | null    |
| B           | 2021-01-04 | sushi        | 10    | N      | null    |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      | null    |
| C           | 2021-01-01 | ramen        | 12    | N      | null    |
| C           | 2021-01-07 | ramen        | 12    | N      | null    |

**Insights**
- Displays all sales per customer with order date, product name, price, their membership status on the purchase date, and customer's purchase rankings dependent on membership status
- Customer C's rankings display as null because they never opened a membership

---