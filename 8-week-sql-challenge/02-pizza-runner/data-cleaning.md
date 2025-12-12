# Case Study #2: Pizza Runner - Data Cleaning

## Data Cleaning The `customer_orders` Table
    
Upon inspection, the `customer_orders` table has a few columns that need to be cleaned up as suggested by the case study directions

- There are empty ' ' strings and 'null' as text instead of NULL within the *exclusions* and *extras* column that need to be cleaned up

**The table, `customer_orders`, before data cleaning:**
| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        | null       | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        | null       | null   | 2020-01-08 21:03:13 |
| 7        | 105         | 2        | null       | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        | null       | null   | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        | null       | null   | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |


In order to clean this table up:
- a temporary table will be created with all data values
- 'null' text will be converted to NULL
- ' ' blank spaces will be removed

**Query used for data cleaning:**
```sql
DROP TABLE IF EXISTS customer_orders_clean;

CREATE TEMP TABLE customer_orders_clean AS
SELECT 
    order_id,
    customer_id,
    pizza_id,
    CASE 
        WHEN exclusions = '' THEN NULL
        WHEN exclusions = 'null' THEN NULL
        ELSE exclusions
    END AS exclusions,
    CASE 
        WHEN extras = '' THEN NULL
        WHEN extras = 'null' THEN NULL
        ELSE extras
    END AS extras,
    order_time
FROM customer_orders;
```

**Temporary table, `customer_orders_clean`, created after data cleaning has been performed:**
| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
| 1        | 101         | 1        | null       | null   | 2020-01-01 18:05:02 |
| 2        | 101         | 1        | null       | null   | 2020-01-01 19:00:52 |
| 3        | 102         | 1        | null       | null   |  2020-01-02 23:51:23 |
| 3        | 102         | 2        | null       | null   |  2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          | null   | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          | null   | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          | null   | 2020-01-04 13:23:46 |
| 5        | 104         | 1        | null       | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        | null       | null   |  2020-01-08 21:03:13 |
| 7        | 105         | 2        | null       | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        | null       | null   | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        | null       | null   | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |

---
## Data Cleaning The `runner_orders` Table
The `runner_orders` table has some known data issues that need to be cleaned up as suggested by the case study directions

- There are empty ' ' strings and 'null' as text instead of NULL throughout the table.
- Units such as 'mins,' 'minutes,' 'minute,' and 'km' throughout the table that also need to be addressed.

**The table, `runner_orders`, before data cleaning:**

| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
| -------- | --------- | ------------------- | -------- | ---------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40         |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15         |                         |
| 6        | 3         | null                | null     | null       | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
| 9        | 2         | null                | null     | null       | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |

In order to clean this table up:

- a temporary table will be created with all data values
- 'null' text will be converted to NULL
- ' ' blank spaces will be removed
- units need to be removed from the columns

**Query used for data cleaning:**
```sql
DROP TABLE IF EXISTS runner_orders_clean;

CREATE TEMP TABLE runner_orders_clean AS
SELECT 
    order_id,
    runner_id,
    CASE 
        WHEN pickup_time = 'null' THEN NULL
        ELSE pickup_time::TIMESTAMP
    END AS pickup_time,
    CASE 
        WHEN distance = 'null' THEN NULL
        WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)::NUMERIC
        ELSE distance::NUMERIC
    END AS distance,
    CASE 
        WHEN duration = 'null' THEN NULL
        WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)::INTEGER
        WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)::INTEGER  
        WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)::INTEGER
        ELSE duration::INTEGER
    END AS duration,
    CASE 
        WHEN cancellation IN ('', 'null') THEN NULL
        ELSE cancellation
    END AS cancellation
FROM runner_orders;
```
**Temporary table, `runner_orders_clean`, created after data cleaning has been performed:**
| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
| -------- | --------- | ------------------- | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       | null                    |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       | null                    |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       | null                    |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       | null                    |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       | null                    |
| 6        | 3         | null                | null     | null     | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       | null                    |
| 9        | 2         | null                | null     | null     | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       | null                    |

---

- Temporary tables `customer_orders_clean` and `runner_orders_clean` have been created as a result of data cleaning with respect to the original `customer_orders` and `runner_orders` tables. 
- Subsequent queries to answer case study questions will be performed on the newly created temporary tables.