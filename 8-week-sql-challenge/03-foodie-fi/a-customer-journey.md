# Case Study #3: Foodie-Fi - A. Customer Journey Questions and Solutions

###  1. Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

**Query**
```sql
SELECT
    ss.customer_id,
    ss.plan_id,
    p.plan_name,
    p.price,
    ss.start_date
FROM subscriptions ss
JOIN plans p USING(plan_id)
WHERE customer_id IN (1, 2, 11, 13, 15, 16, 18, 19) 
ORDER BY customer_id, start_date;
```

**Results**
| customer_id | plan_id | plan_name     | price  | start_date |
| ----------- | ------- | ------------- | ------ | ---------- |
| 1           | 0       | trial         | 0.00   | 2020-08-01 |
| 1           | 1       | basic monthly | 9.90   | 2020-08-08 |
| 2           | 0       | trial         | 0.00   | 2020-09-20 |
| 2           | 3       | pro annual    | 199.00 | 2020-09-27 |
| 11          | 0       | trial         | 0.00   | 2020-11-19 |
| 11          | 4       | churn         |        | 2020-11-26 |
| 13          | 0       | trial         | 0.00   | 2020-12-15 |
| 13          | 1       | basic monthly | 9.90   | 2020-12-22 |
| 13          | 2       | pro monthly   | 19.90  | 2021-03-29 |
| 15          | 0       | trial         | 0.00   | 2020-03-17 |
| 15          | 2       | pro monthly   | 19.90  | 2020-03-24 |
| 15          | 4       | churn         |        | 2020-04-29 |
| 16          | 0       | trial         | 0.00   | 2020-05-31 |
| 16          | 1       | basic monthly | 9.90   | 2020-06-07 |
| 16          | 3       | pro annual    | 199.00 | 2020-10-21 |
| 18          | 0       | trial         | 0.00   | 2020-07-06 |
| 18          | 2       | pro monthly   | 19.90  | 2020-07-13 |
| 19          | 0       | trial         | 0.00   | 2020-06-22 |
| 19          | 2       | pro monthly   | 19.90  | 2020-06-29 |
| 19          | 3       | pro annual    | 199.00 | 2020-08-29 |

**Insights**
- All customers utilized the trail plan before proceeding with a more premium, paid plan
- Customer 1 utilized the free trial for 7 days before signing up for the basic monthly plan
- Customer 2 utilized the free trail and then signed up for the pro annual plan
- Customer 11 utilized the free trail and then did not sign up for a premium plan afterwards
- Customer 13 utilized the trail and then signed up for the basic plan before switching to the pro monthly plan the next month
- Customer 15 utilized the trial, signed up for the pro monthly before cancelling services the following month
- Customer 16 utilized the free trail, signed up for the basic monthly plan and then committed to the pro annual plan 4 months later
- Customer 18 utilized the free trail and switched to the pro monthly plan
- Customer 19 utilized the trail before subscribing to the pro monthly plan and then committing to the pro annual plan a month later

---