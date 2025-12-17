# Case Study #2: Pizza Runner - D. Pricing and Ratings Questions and Solutions

###  1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

**Query**
```sql
SELECT 
    SUM(
        CASE
        WHEN co.pizza_id = 1 THEN 12
        WHEN co.pizza_id = 2 THEN 10
        END
    )AS total_revenue
FROM customer_orders_clean co
JOIN runner_orders_clean ro ON co.order_id = ro.order_id
WHERE ro.cancellation IS NULL;
```

**Results**
| total_revenue |
| ------------- |
| 138           |

**Insights**
- Pizza Runner has made $138 so far with no delivery fees
---


###  2. What if there was an additional $1 charge for any pizza extras?
- Add cheese is $1 extra

**Query**
```sql
SELECT 
    SUM(
        CASE
            WHEN co.pizza_id = 1 THEN 12
            WHEN co.pizza_id = 2 THEN 10
        END 
        +
        CASE
            WHEN co.extras IS NOT NULL
            THEN ARRAY_LENGTH(STRING_TO_ARRAY(co.extras, ','), 1)
            ELSE 0
        END
    )AS total_revenue
FROM customer_orders_clean co
JOIN runner_orders_clean ro ON co.order_id = ro.order_id
WHERE ro.cancellation IS NULL;
```

**Results**
| total_revenue |
| ------------- |
| 142           |

**Insights**
- Pizza Runner has made $142 so far with no delivery fees while charging $1 for each extra topping request

---


###  3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

**Query**
```sql
DROP TABLE IF EXISTS runner_ratings;
CREATE TABLE runner_ratings (
  rating_id SERIAL PRIMARY KEY,
  order_id INTEGER NOT NULL,
  rating INTEGER CHECK (rating >= 1 AND rating <= 5),
  rating_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  comments TEXT
);

INSERT INTO runner_ratings (order_id, rating, rating_time, comments)
VALUES
  (1, 5, '2021-01-01 19:00:00', 'Excellent service, very fast!'),
  (2, 5, '2021-01-01 20:30:00', 'Great delivery, pizza arrived hot'),
  (3, 4, '2021-01-03 00:45:00', 'Good service, took a bit longer than expected'),
  (4, 4, '2021-01-04 14:30:00', 'Professional and courteous'),
  (5, 5, '2021-01-08 22:15:00', 'Super fast delivery!'),
  (7, 5, '2021-01-08 22:00:00', 'Amazing service'),
  (8, 3, '2021-01-10 00:30:00', 'Pizza was cold, took too long'),
  (10, 4, '2021-01-11 19:15:00', 'Reliable as always');
```

**Results**
| rating_id | order_id | rating | rating_time         | comments                                      |
| --------- | -------- | ------ | ------------------- | --------------------------------------------- |
| 1         | 1        | 5      | 2021-01-01 19:00:00 | Excellent service, very fast!                 |
| 2         | 2        | 5      | 2021-01-01 20:30:00 | Great delivery, pizza arrived hot             |
| 3         | 3        | 4      | 2021-01-03 00:45:00 | Good service, took a bit longer than expected |
| 4         | 4        | 4      | 2021-01-04 14:30:00 | Professional and courteous                    |
| 5         | 5        | 5      | 2021-01-08 22:15:00 | Super fast delivery!                          |
| 6         | 7        | 5      | 2021-01-08 22:00:00 | Amazing service                               |
| 7         | 8        | 3      | 2021-01-10 00:30:00 | Pizza was cold, took too long                 |
| 8         | 10       | 4      | 2021-01-11 19:15:00 | Reliable as always                            |

**Insights**
- The `runner_ratings` table uses *order_id* to link ratings with existing delivery data. It includes a rating ID, the customer's 1-5 rating score, submission timestamp, and optional comments field for additional feedback.

---


###  4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- `customer_id`
- `order_id`
- `runner_id`
- `rating`
- `order_time`
- `pickup_time`
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas

**Query**
```sql
SELECT
    co.customer_id,
    co.order_id,
    ro.runner_id,
    rr.rating,
    co.order_time,
    ro.pickup_time,
    ROUND((EXTRACT(EPOCH FROM (ro.pickup_time - co.order_time)) / 60)::NUMERIC, 2) AS time_to_pickup_minutes,
    ro.duration AS delivery_duration_minutes,
    ROUND((ro.distance / ro.duration * 60)::NUMERIC, 2) AS average_speed_kmh,
    COUNT(co.pizza_id) AS total_pizzas
FROM runner_ratings rr
JOIN runner_orders_clean ro ON rr.order_id = ro.order_id
JOIN customer_orders_clean co ON rr.order_id = co.order_id
WHERE ro.cancellation IS NULL
GROUP BY co.customer_id,
    co.order_id,
    ro.runner_id,
    rr.rating,
    co.order_time,
    ro.pickup_time,
    ro.duration,
    ro.distance
ORDER BY co.customer_id, ro.runner_id;
```

**Results**
| customer_id | order_id | runner_id | rating | order_time          | pickup_time         | time_to_pickup_minutes | delivery_duration_minutes | average_speed_kmh | total_pizzas |
| ----------- | -------- | --------- | ------ | ------------------- | ------------------- | ---------------------- | ------------------------- | ----------------- | ------------ |
| 101         | 1        | 1         | 5      | 2020-01-01 18:05:02 | 2020-01-01 18:15:34 | 10.53                  | 32                        | 37.50             | 1            |
| 101         | 2        | 1         | 5      | 2020-01-01 19:00:52 | 2020-01-01 19:10:54 | 10.03                  | 27                        | 44.44             | 1            |
| 102         | 3        | 1         | 4      | 2020-01-02 23:51:23 | 2020-01-03 00:12:37 | 21.23                  | 20                        | 40.20             | 2            |
| 102         | 8        | 2         | 3      | 2020-01-09 23:54:33 | 2020-01-10 00:15:02 | 20.48                  | 15                        | 93.60             | 1            |
| 103         | 4        | 2         | 4      | 2020-01-04 13:23:46 | 2020-01-04 13:53:03 | 29.28                  | 40                        | 35.10             | 3            |
| 104         | 10       | 1         | 4      | 2020-01-11 18:34:49 | 2020-01-11 18:50:20 | 15.52                  | 10                        | 60.00             | 2            |
| 104         | 5        | 3         | 5      | 2020-01-08 21:00:29 | 2020-01-08 21:10:57 | 10.47                  | 15                        | 40.00             | 1            |
| 105         | 7        | 2         | 5      | 2020-01-08 21:20:29 | 2020-01-08 21:30:45 | 10.27                  | 25                        | 60.00             | 1            |

**Insights**
- This table consolidates customer orders, delivery metrics, and ratings into a single comprehensive view for analyzing runner performance and customer satisfaction

---


###  5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

**Query**
```sql
WITH 
revenue AS (
    SELECT 
    SUM(
        CASE
        WHEN co.pizza_id = 1 THEN 12
        WHEN co.pizza_id = 2 THEN 10
        END
    ) AS total_revenue
    FROM customer_orders_clean co
    JOIN runner_orders_clean ro ON co.order_id = ro.order_id
    WHERE ro.cancellation IS NULL
),
costs AS (
    SELECT 
        ROUND(SUM(distance) * 0.30, 2) AS runner_cost
    FROM runner_orders_clean
    WHERE cancellation IS NULL
)
SELECT 
    revenue.total_revenue,
    costs.runner_cost,
    ROUND((revenue.total_revenue - costs.runner_cost)::NUMERIC, 2) AS net_profit
FROM revenue, costs;
```

**Results**
| total_revenue | runner_cost | net_profit |
| ------------- | ----------- | ---------- |
| 138           | 43.56       | 94.44      |

**Insights**
- The total revenue with no costs for extras is $138, with runner costs being a total of $43.56 at $0.30 per kilometer, Pizza Runner is left over with $94.44 after deliveries

---