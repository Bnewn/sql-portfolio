# Case Study #2: Pizza Runner - B. Runner and Customer Experience Questions and Solutions

###  1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

**Query**
```sql
SELECT
    FLOOR((registration_date - '2021-01-01'::DATE) / 7) + 1 AS week_number,
    COUNT(runner_id) AS runners_signed_up
FROM runners
GROUP BY week_number
ORDER BY week_number;
```

**Results**
| week_number | runners_signed_up |
| ----------- | ----------------- |
| 1           | 2                 |
| 2           | 1                 |
| 3           | 1                 |


**Insights**
- Strong initial interest with 2 runners signing up in the first week, suggesting effective launch marketing
- Signups stabilized at 1 runner per week for weeks 2 and 3
- Total of 4 runners recruited in 3 weeks

---

###  2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

**Query**
```sql
SELECT 
    ro.runner_id,
    ROUND(AVG(EXTRACT(EPOCH FROM (ro.pickup_time - co.order_time)) / 60)::NUMERIC, 2) AS avg_pickup_time_minutes
FROM runner_orders_clean ro
JOIN (
    SELECT order_id, MIN(order_time) AS order_time
    FROM customer_orders_clean
    GROUP BY order_id
) co ON ro.order_id = co.order_id
WHERE ro.cancellation IS NULL AND co.order_time IS NOT NULL
GROUP BY ro.runner_id
ORDER BY ro.runner_id;
```

**Results**
| runner_id | avg_pickup_time_minutes |
| --------- | ----------------------- |
| 1         | 14.33                   |
| 2         | 20.01                   |
| 3         | 10.47                   |

**Insights**
- Runner 3 has the shortest average pickup time, followed by runner 1 and 2


---

###  3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

**Query**
```sql
SELECT 
    pizza_count,
    ROUND(AVG(prep_time_minutes)::NUMERIC , 2) AS avg_prep_time_minutes
FROM (
    SELECT 
        co.order_id,
        count(co.pizza_id) AS pizza_count,
        EXTRACT(EPOCH FROM(ro.pickup_time - co.order_time)) / 60 AS prep_time_minutes
    FROM customer_orders_clean co
    JOIN runner_orders_clean ro ON ro.order_id = co.order_id
    WHERE ro.cancellation IS NULL
    GROUP BY co.order_id, ro.pickup_time, co.order_time
) AS order_prep
GROUP BY pizza_count
ORDER BY pizza_count;
```

**Results**
| pizza_count | avg_prep_time_minutes |
| ----------- | --------------------- |
| 1           | 12.36                 |
| 2           | 18.38                 |
| 3           | 29.28                 |

**Insights**
- Positive correlation exists between pizza count and prep time, more pizzas require longer preparation
- Each additional pizza adds approximately 6-11 minutes to prep time

---

###  4. What was the average distance traveled for each customer?

**Query**
```sql
SELECT 
    co.customer_id,
    ROUND(AVG(ro.distance)::NUMERIC, 2) AS avg_distance_km
FROM customer_orders_clean co
JOIN runner_orders_clean ro ON co.order_id = ro.order_id
GROUP BY co.customer_id
ORDER BY co.customer_id;
```

**Results**
| customer_id | avg_distance_km |
| ----------- | --------------- |
| 101         | 20.00           |
| 102         | 16.73           |
| 103         | 23.40           |
| 104         | 10.00           |
| 105         | 25.00           |

**Insights**
- Customer 105 is the farthest at 25 km, while Customer 104 is the closest at only 10 km
- Average distances of 16-25 km suggest Pizza Runner handles medium to long-distance deliveries, which impacts delivery times and costs

---

###  5. What was the difference between the longest and shortest delivery times for all orders?

**Query**
```sql
SELECT 
    MAX(duration) AS longest_delivery_minutes,
    MIN(duration) AS shortest_delivery_minutes,
    MAX(duration) - MIN(duration) AS difference_minutes
FROM runner_orders_clean
WHERE cancellation IS NULL;
```

**Results**
| longest_delivery_minutes | shortest_delivery_minutes | difference_minutes |
| ------------------------ | ------------------------- | ------------------ |
| 40                       | 10                        | 30                 |

**Insights**
- The difference between the longest and shortest delivery times for all orders is 30 minutes

---

###  6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

**Query**
```sql
SELECT 
    runner_id,
    order_id,
    distance AS distance_km,
    duration AS duration_minutes,
    ROUND((distance / duration * 60)::NUMERIC, 2) AS avg_speed_kmh
FROM runner_orders_clean
WHERE cancellation IS NULL
    AND duration > 0
    AND distance > 0
ORDER BY runner_id, order_id;
```

**Results**
| runner_id | order_id | distance_km | duration_minutes | avg_speed_kmh |
| --------- | -------- | ----------- | ---------------- | ------------- |
| 1         | 1        | 20          | 32               | 37.50         |
| 1         | 2        | 20          | 27               | 44.44         |
| 1         | 3        | 13.4        | 20               | 40.20         |
| 1         | 10       | 10          | 10               | 60.00         |
| 2         | 4        | 23.4        | 40               | 35.10         |
| 2         | 7        | 25          | 25               | 60.00         |
| 2         | 8        | 23.4        | 15               | 93.60         |
| 3         | 5        | 10          | 15               | 40.00         |

**Insights**
- Excluding the outlier, delivery speeds range from 35-60 km/h, which is reasonable for urban/suburban delivery
- Both Runner 1 and Runner 2 demonstrate increasing speeds over successive deliveries

---

###  7. What is the successful delivery percentage for each runner?

**Query**
```sql
SELECT 
    runner_id,
    COUNT(*) AS total_orders,
    SUM(
        CASE
        WHEN cancellation IS NULL THEN 1 ELSE 0
        END) AS successful_delivery,
    ROUND((SUM(
        CASE
        WHEN cancellation IS NULL THEN 1 ELSE 0
        END)::NUMERIC / count(*) * 100) , 2) AS success_percentage
FROM runner_orders_clean
GROUP BY runner_id
ORDER by runner_id;
```

**Results**
| runner_id | total_orders | successful_delivery | success_percentage |
| --------- | ------------ | ------------------- | ------------------ |
| 1         | 4            | 4                   | 100.00             |
| 2         | 4            | 3                   | 75.00              |
| 3         | 2            | 1                   | 50.00              |

**Insights**
- Runner 1 maintains a 100% success rate with all 4 deliveries completed successfully
- Runner 2 has a 75% success rate with 1 cancellation out of 4 orders
- Runner 3 shows the poorest performance at 50%, with half their orders (1 of 2) cancelled

---