# Case Study #3: Foodie-Fi - C. Challenge Payment Questions and Solutions

###  The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments

**Query**
```sql
DROP TABLE IF EXISTS payments;
CREATE TABLE payments(
    payment_id SERIAL,
    customer_id INTEGER,
    plan_id INTEGER,
    plan_name VARCHAR(20),
    payment_date DATE,
    amount DECIMAL(5,2),
    payment_order INTEGER
);

INSERT INTO payments (customer_id, plan_id, plan_name, payment_date, amount, payment_order)

WITH RECURSIVE all_subscriptions AS (
    SELECT
        s.customer_id,
        s.plan_id,
        p.plan_name,
        s.start_date,
        p.price,
        LEAD(s.start_date) OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS next_change_date,
        LEAD(s.plan_id) OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS next_plan
    FROM subscriptions s
    JOIN plans p ON s.plan_id = p.plan_id
    WHERE s.start_date <= '2020-12-31'
),
payment_generator AS (
    SELECT
        customer_id,
        plan_id,
        plan_name,
        start_date AS payment_date,
        price,
        next_change_date,
        next_plan
    FROM all_subscriptions
    WHERE plan_id NOT IN (0, 4)
    AND EXTRACT(YEAR FROM start_date) <= 2020
    
    UNION ALL
    
    SELECT
        pg.customer_id,
        pg.plan_id,
        pg.plan_name,
        (pg.payment_date + INTERVAL '1 month')::DATE,
        pg.price,
        pg.next_change_date,
        pg.next_plan
    FROM payment_generator pg
    WHERE (pg.payment_date + INTERVAL '1 month')::DATE < COALESCE(pg.next_change_date, '2021-01-01'::DATE)
    AND (pg.payment_date + INTERVAL '1 month')::DATE <= '2020-12-31'
    AND pg.plan_id != 3
),
payments_with_cumulative AS (
    SELECT
        customer_id,
        plan_id,
        plan_name,
        payment_date,
        price,
        COALESCE(
            SUM(price) OVER (
            PARTITION BY customer_id, DATE_TRUNC('month', payment_date) 
        ORDER BY payment_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING
        ), 
        0
    ) AS paid_this_month_before
    FROM payment_generator
)
SELECT
    customer_id,
    plan_id,
    plan_name,
    payment_date,
    CASE 
        WHEN paid_this_month_before > 0 THEN price - paid_this_month_before
        ELSE price
    END AS payment_amount,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY payment_date) AS payment_order
FROM payments_with_cumulative
WHERE payment_date <= '2020-12-31'
ORDER BY customer_id, payment_date;

SELECT * FROM payments
WHERE customer_id IN (1,2,13,15,16,18,19);
```

**Results**
| customer_id | plan_id | plan_name     | payment_date | payment_amount | payment_order |
| ----------- | ------- | ------------- | ------------ | -------------- | ------------- |
| 1           | 1       | basic monthly | 2020-08-08   | 9.90           | 1             |
| 1           | 1       | basic monthly | 2020-09-08   | 9.90           | 2             |
| 1           | 1       | basic monthly | 2020-10-08   | 9.90           | 3             |
| 1           | 1       | basic monthly | 2020-11-08   | 9.90           | 4             |
| 1           | 1       | basic monthly | 2020-12-08   | 9.90           | 5             |
| 2           | 3       | pro annual    | 2020-09-27   | 199.00         | 1             |
| 13          | 1       | basic monthly | 2020-12-22   | 9.90           | 1             |
| 15          | 2       | pro monthly   | 2020-03-24   | 19.90          | 1             |
| 15          | 2       | pro monthly   | 2020-04-24   | 19.90          | 2             |
| 16          | 1       | basic monthly | 2020-06-07   | 9.90           | 1             |
| 16          | 1       | basic monthly | 2020-07-07   | 9.90           | 2             |
| 16          | 1       | basic monthly | 2020-08-07   | 9.90           | 3             |
| 16          | 1       | basic monthly | 2020-09-07   | 9.90           | 4             |
| 16          | 1       | basic monthly | 2020-10-07   | 9.90           | 5             |
| 16          | 3       | pro annual    | 2020-10-21   | 189.10         | 6             |
| 18          | 2       | pro monthly   | 2020-07-13   | 19.90          | 1             |
| 18          | 2       | pro monthly   | 2020-08-13   | 19.90          | 2             |
| 18          | 2       | pro monthly   | 2020-09-13   | 19.90          | 3             |
| 18          | 2       | pro monthly   | 2020-10-13   | 19.90          | 4             |
| 18          | 2       | pro monthly   | 2020-11-13   | 19.90          | 5             |
| 18          | 2       | pro monthly   | 2020-12-13   | 19.90          | 6             |
| 19          | 2       | pro monthly   | 2020-06-29   | 19.90          | 1             |
| 19          | 2       | pro monthly   | 2020-07-29   | 19.90          | 2             |
| 19          | 3       | pro annual    | 2020-08-29   | 199.00         | 3             |

**Insights**
- The recursive CTE successfully generates monthly payment schedules while handling  upgrade and churn scenarios across all subscription types
- The query handles all edge cases: same-month upgrades with proration, new-month upgrades without proration, churn stop points, and mixed subscription paths
- Window functions successfully track cumulative monthly payments to enable accurate proration calculations when customers upgrade mid-billing cycle

---