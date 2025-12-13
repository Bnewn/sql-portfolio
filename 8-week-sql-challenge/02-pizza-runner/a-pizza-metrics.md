# Case Study #2: Pizza Runner - A. Pizza Metrics Questions and Solutions

###  1. How many pizzas were ordered?

**Query**
```sql
SELECT COUNT(*) AS total_pizzas_ordered
FROM customer_orders_clean;
```

**Results**
| total_pizzas_ordered |
| -------------------- |
| 14                   |

**Insights**
- The count shows that there are 14 total pizza ordered

---


###  2. How many unique customer orders were made?

**Query**
```sql
SELECT COUNT(DISTINCT order_id) AS unique_orders
FROM customer_orders_clean;
```

**Results**
| unique_orders |
| ------------- |
| 10            |

**Insights**
- The count shows that 10 unique orders were placed

---


###  3. How many successful orders were delivered by each runner?

**Query**
```sql
SELECT 
    runner_id,
    COUNT(order_id) AS successful_deliveries
FROM runner_orders_clean
WHERE cancellation IS NULL
GROUP BY runner_id
ORDER BY runner_id;
```

**Results**
| runner_id | successful_deliveries |
| --------- | --------------------- |
| 1         | 4                     |
| 2         | 3                     |
| 3         | 1                     |

**Insights**
- Runner 1 is the most active with 4 successful deliveries, followed by Runner 2 and 3
- Runner 3 has only completed 1 delivery, suggesting they may be new or have limited availability
- Total of 8 successful deliveries across all runners

---


###  4. How many of each type of pizza was delivered?

**Query**
```sql
SELECT 
    pn.pizza_name,
    COUNT(ro.order_id) AS successful_deliveries
FROM runner_orders_clean ro
JOIN customer_orders_clean co ON ro.order_id = co.order_id
JOIN pizza_names pn ON co.pizza_id = pn.pizza_id
WHERE ro.cancellation IS NULL
GROUP BY pn.pizza_name
ORDER BY pn.pizza_name;
```

**Results**
| pizza_name | successful_deliveries |
| ---------- | --------------------- |
| Meatlovers | 9                     |
| Vegetarian | 3                     |

**Insights**
- Meatlovers is significantly more popular with 9 deliveries (75%) compared to 3 Vegetarian (25%)
- 12 pizzas were successfully delivered out of 14 total orders
- Strong preference for meat-based options suggests menu expansion should focus on similar varieties

---


###  5. How many Vegetarian and Meatlovers were ordered by each customer?

**Query**
```sql
SELECT 
    co.customer_id,
    pn.pizza_name,
    COUNT(*) AS pizzas_ordered
FROM customer_orders_clean co
JOIN pizza_names pn ON co.pizza_id = pn.pizza_id
GROUP BY co.customer_id, pn.pizza_name
ORDER BY co.customer_id, pn.pizza_name;
```

**Results**
| customer_id | pizza_name | pizzas_ordered |
| ----------- | ---------- | -------------- |
| 101         | Meatlovers | 2              |
| 101         | Vegetarian | 1              |
| 102         | Meatlovers | 2              |
| 102         | Vegetarian | 1              |
| 103         | Meatlovers | 3              |
| 103         | Vegetarian | 1              |
| 104         | Meatlovers | 3              |
| 105         | Vegetarian | 1              |

**Insights**
- Customer 105 is the only purely vegetarian customer, while Customer 104 exclusively orders Meatlovers
- Customers 101, 102, and 103 order both types but show a clear preference for Meatlovers
- Overall, 10 Meatlovers were ordered compared to 4 Vegetarian, reinforcing the strong meat preference across the customer base

---


###  6. What was the maximum number of pizzas delivered in a single order?

**Query**
```sql
SELECT 
    MAX(pizza_count) AS max_pizza_in_order
FROM (
    SELECT
        co.order_id,
        count(*) AS pizza_count
    FROM customer_orders_clean co
    JOIN runner_orders_clean ro ON co.order_id = ro.order_id
    WHERE ro.cancellation IS NULL
    GROUP BY co.order_id
    )AS order_counts;
```

**Results**
| max_pizza_in_order |
| ------------------ |
| 3                  |

**Insights**
- The maximum order size was 3 pizzas, indicating most orders are for individuals or small groups

---


###  7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

**Query**
```sql
SELECT 
    co.customer_id,
    SUM(
        CASE WHEN
        co.exclusions IS NOT NULL OR co.extras IS NOT NULL THEN 1
            ELSE 0
        END) AS pizza_with_changes,
    SUM(
        CASE WHEN
        co.exclusions IS NULL AND co.extras IS NULL THEN 1
            ELSE 0
        END) AS pizza_with_no_changes
FROM customer_orders_clean co
JOIN runner_orders_clean ro ON co.order_id = ro.order_id
WHERE ro.cancellation IS NULL
GROUP BY co.customer_id
ORDER BY co.customer_id;
```

**Results**
| customer_id | pizza_with_changes | pizza_with_no_changes |
| ----------- | ------------------ | --------------------- |
| 101         | 0                  | 2                     |
| 102         | 0                  | 3                     |
| 103         | 3                  | 0                     |
| 104         | 2                  | 1                     |
| 105         | 1                  | 0                     |

**Insights**
- Customer preferences are clearly divided: Customers 101 and 102 never customize their orders, while Customers 103, 104, and 105 frequently request changes
- 50% of delivered pizzas (6 out of 12) included customizations
- Customer 103 is the most particular, customizing all 3 of their delivered pizzas

---


###  8. How many pizzas were delivered that had both exclusions and extras?

**Query**
```sql
SELECT 
    SUM(
        CASE WHEN
        co.exclusions IS NOT NULL AND co.extras IS NOT NULL THEN 1
            ELSE 0
        END) AS pizza_with_exclusions_and_extras
FROM customer_orders_clean co
JOIN runner_orders_clean ro ON co.order_id = ro.order_id
WHERE ro.cancellation IS NULL;
```
**Results**
| pizza_with_exclusions_and_extras |
| -------------------------------- |
| 1                                |

**Insights**
- Only 1 pizza was delivered with both exclusions and extras out of all delivered pizzas
- Most customers who customize choose either exclusions OR extras, not both

---


###  9. What was the total volume of pizzas ordered for each hour of the day?

**Query**
```sql
SELECT 
    EXTRACT(HOUR FROM order_time) AS hour_of_day,
    COUNT(*) AS pizzas_ordered
FROM customer_orders_clean
GROUP BY hour_of_day
ORDER BY hour_of_day;
```

**Results**
| hour_of_day | pizzas_ordered |
| ----------- | -------------- |
| 11          | 1              |
| 13          | 3              |
| 18          | 3              |
| 19          | 1              |
| 21          | 3              |
| 23          | 3              |

**Insights**
- Peak ordering hours are 1 PM, 6 PM, 9 PM, and 11 PM with 3 pizzas each
- Lunch (1 PM) and dinner/evening times (6-9 PM) drive most orders
- Notably, late-night orders at 11 PM match peak dinner volumes, suggesting strong demand for late-night service

---


###  10. What was the volume of orders for each day of the week?

**Query**
```sql
SELECT 
    TO_CHAR(order_time, 'Day') AS day_of_week,
    COUNT(*) AS pizzas_ordered
FROM customer_orders_clean
GROUP BY day_of_week, EXTRACT(DOW FROM order_time)
ORDER BY EXTRACT(DOW FROM order_time);
```

**Results**
| day_of_week | pizzas_ordered |
| ----------- | -------------- |
| Wednesday   | 5              |
| Thursday    | 3              |
| Friday      | 1              |
| Saturday    | 5              |

**Insights**
- Wednesday and Saturday tie for highest volume with 5 pizzas each, indicating strong mid-week and weekend demand
- Friday has the least volume of pizzas ordered

---