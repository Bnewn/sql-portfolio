# Case Study #3: Foodie-Fi - B. Data Analysis Questions and Solutions

###  1. How many customers has Foodie-Fi ever had?

**Query**
```sql
SELECT
    COUNT(DISTINCT customer_id) AS total_customers
FROM subscriptions;
```

**Results**
| total_customers |
| --------------- |
| 1000            |

**Insights**
- Foodie-Fi has had a total of 1000 customers since launch

---


###  2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

**Query**
```sql
SELECT 
    EXTRACT(MONTH FROM start_date) AS month,
    COUNT(*) AS trial_signups
FROM subscriptions
WHERE plan_id = 0
GROUP BY EXTRACT(MONTH FROM start_date)
ORDER BY month;
```

**Results**
| month | trial_signups |
| ----- | ------------- |
| 1     | 88            |
| 2     | 68            |
| 3     | 94            |
| 4     | 81            |
| 5     | 88            |
| 6     | 79            |
| 7     | 89            |
| 8     | 88            |
| 9     | 87            |
| 10    | 79            |
| 11    | 75            |
| 12    | 84            |

**Insights**
- Trial signups show relatively consistent distribution throughout the year, ranging from 68 to 94 signups per month
- March leads with 94 signups, while February has only 68 signups

---


###  3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

**Query**
```sql
SELECT 
    p.plan_name,
    COUNT(*) AS event_count
FROM subscriptions s
JOIN plans p USING(plan_id)
WHERE EXTRACT(YEAR FROM s.start_date) > 2020
GROUP BY p.plan_name
ORDER BY event_count;
```

**Results**
| plan_name     | event_count |
| ------------- | ----------- |
| basic monthly | 8           |
| pro monthly   | 60          |
| pro annual    | 63          |
| churn         | 71          |

**Insights**
- All trial signups occurred in 2020, with no new customer acquisition after that year
- Churn leads post-2020 activity with 71 events, indicating attrition from 2020 cohort
- Pro annual nearly matches pro monthly, suggesting customers value the long-term commitment option

---


###  4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

**Query**
```sql
WITH cte AS(
    SELECT 
        COUNT(DISTINCT customer_id) AS total_customers,
        COUNT(CASE
            WHEN plan_id = 4 THEN customer_id
            END) AS churned_customers
    FROM subscriptions)
SELECT
    total_customers,
    churned_customers,
    round((churned_customers::NUMERIC / total_customers::NUMERIC * 100), 1) AS percentage
FROM cte;
```

**Results**
| total_customers | churned_customers | percentage |
| --------------- | ----------------- | ---------- |
| 1000            | 307               | 30.7       |

**Insights**
- Nearly one-third of customers (30.7%) have churned, representing some attrition
- 307 churned customers out of 1,000 total indicates a retention rate of 69.3%
- The 30.7% figure suggests either: (a) customers are churning quickly after trial periods, (b) the service struggles with long-term engagement, or (c) pricing/content doesn't meet expectations

---


###  5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

**Query**
```sql
WITH customer_plans AS (
    SELECT 
        customer_id,
        plan_id,
        LAG(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS previous_plan
    FROM subscriptions
)
SELECT 
    COUNT(DISTINCT customer_id) AS churned_after_trial,
    ROUND((COUNT(DISTINCT customer_id)::NUMERIC / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) * 100), 0) AS percentage
FROM customer_plans
WHERE previous_plan = 0
    AND plan_id = 4;
```

**Results**
| churned_after_trial | percentage |
| ------------------- | ---------- |
| 92                  | 9          |

**Insights**
- Only 9% of customers churned immediately after their free trial, indicating the trial effectively converts most users to paid plans
- This represents just 30% of total churns (92 of 307), meaning the majority of customer loss (70%) occurs after customers have already converted to paid subscriptions

---


###  6. What is the number and percentage of customer plans after their initial free trial?

**Query**
```sql
WITH customer_plans AS (
    SELECT 
        customer_id,
        plan_id,
        LAG(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS previous_plan
    FROM subscriptions
)
SELECT 
    p.plan_name,
    COUNT(DISTINCT customer_id) AS customer_count,
    ROUND((COUNT(DISTINCT customer_id)::NUMERIC / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) * 100), 1) AS percentage
FROM customer_plans cp
JOIN plans p ON cp.plan_id = p.plan_id
WHERE cp.previous_plan = 0
GROUP BY p.plan_name
ORDER BY customer_count DESC;
```

**Results**
| plan_name     | customer_count | percentage |
| ------------- | -------------- | ---------- |
| basic monthly | 546            | 54.6       |
| pro monthly   | 325            | 32.5       |
| churn         | 92             | 9.2        |
| pro annual    | 37             | 3.7        |

**Insights**
- Over half of customers choose basic monthly after trial, indicating strong price sensitivity and preference for the lowest-commitment paid option
- Monthly plans dominate compared to annual plan

---


###  7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

**Query**
```sql
WITH latest_plans AS (
    SELECT 
        customer_id,
        plan_id,
        start_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date DESC) AS rn
    FROM subscriptions
    WHERE start_date <= '2020-12-31'
)
SELECT 
    p.plan_name,
    COUNT(*) AS customer_count,
    ROUND((COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER () * 100), 1) AS percentage
FROM latest_plans lp
JOIN plans p ON lp.plan_id = p.plan_id
WHERE lp.rn = 1
GROUP BY p.plan_name
ORDER BY customer_count DESC;
```

**Results**
| plan_name     | customer_count | percentage |
| ------------- | -------------- | ---------- |
| pro monthly   | 326            | 32.6       |
| churn         | 236            | 23.6       |
| basic monthly | 224            | 22.4       |
| pro annual    | 195            | 19.5       |
| trial         | 19             | 1.9        |

**Insights**
- Pro monthly leads at 32.6%, showing it becomes the preferred plan
- Pro gained traction demonstrating customers gain confidence to commit long-term after experiencing the service
- Customer journey reveals an upgrade pattern of trial → basic monthly → pro monthly/annual

---


###  8. How many customers have upgraded to an annual plan in 2020?

**Query**
```sql
SELECT 
    COUNT(DISTINCT customer_id) AS customers_upgraded_to_annual
FROM subscriptions
WHERE plan_id = 3
    AND EXTRACT(YEAR FROM start_date) = 2020;
```

**Results**
| customers_upgraded_to_annual |
| ---------------------------- |
| 195                          |

**Insights**
- A total of 195 customers upgraded to an annual plan in 2020

---


###  9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

**Query**
```sql
WITH trial_dates AS (
    SELECT 
        customer_id,
        start_date AS trial_date
    FROM subscriptions
    WHERE plan_id = 0
),
annual_dates AS (
    SELECT 
        customer_id,
        start_date AS annual_date
    FROM subscriptions
    WHERE plan_id = 3
)
SELECT 
    ROUND(AVG(ad.annual_date - td.trial_date), 0) AS avg_days_to_annual
FROM trial_dates td
JOIN annual_dates ad ON td.customer_id = ad.customer_id;
```

**Results**
| avg_days_to_annual |
| ------------------ |
| 105                |

**Insights**
- Customers take an average of 105 days (~3.5 months) to upgrade to annual plans, indicating they need substantial product experience before committing long-term

---


###  10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

**Query**
```sql
WITH trial_dates AS (
    SELECT 
        customer_id,
        start_date AS trial_date
    FROM subscriptions
    WHERE plan_id = 0
),
annual_dates AS (
    SELECT 
    customer_id,
    start_date AS annual_date
    FROM subscriptions
    WHERE plan_id = 3
),
days_to_annual AS (
    SELECT 
    td.customer_id,
    ad.annual_date - td.trial_date AS days_to_upgrade
    FROM trial_dates td
    JOIN annual_dates ad ON td.customer_id = ad.customer_id
)
SELECT 
    CASE 
        WHEN days_to_upgrade BETWEEN 0 AND 30 THEN '0-30 days'
        WHEN days_to_upgrade BETWEEN 31 AND 60 THEN '31-60 days'
        WHEN days_to_upgrade BETWEEN 61 AND 90 THEN '61-90 days'
        WHEN days_to_upgrade BETWEEN 91 AND 120 THEN '91-120 days'
        WHEN days_to_upgrade BETWEEN 121 AND 150 THEN '121-150 days'
        WHEN days_to_upgrade BETWEEN 151 AND 180 THEN '151-180 days'
        WHEN days_to_upgrade BETWEEN 181 AND 210 THEN '181-210 days'
        WHEN days_to_upgrade BETWEEN 211 AND 240 THEN '211-240 days'
        WHEN days_to_upgrade BETWEEN 241 AND 270 THEN '241-270 days'
        WHEN days_to_upgrade BETWEEN 271 AND 300 THEN '271-300 days'
        WHEN days_to_upgrade BETWEEN 301 AND 330 THEN '301-330 days'
        WHEN days_to_upgrade BETWEEN 331 AND 360 THEN '331-360 days'
        ELSE '360+ days'
    END AS period,
    COUNT(*) AS customer_count
FROM days_to_annual
GROUP BY period
ORDER BY MIN(days_to_upgrade);
```

**Results**
| period       | customer_count |
| ------------ | -------------- |
| 0-30 days    | 49             |
| 31-60 days   | 24             |
| 61-90 days   | 34             |
| 91-120 days  | 35             |
| 121-150 days | 42             |
| 151-180 days | 36             |
| 181-210 days | 26             |
| 211-240 days | 4              |
| 241-270 days | 5              |
| 271-300 days | 1              |
| 301-330 days | 1              |
| 331-360 days | 1              |

**Insights**
- The first 6 months capture a majority of all annual conversions
- There is a dramatic drop off of conversions after 210 days
- This demonstrates that there are 'quick deciders' as well as 'gradual converters' within the customer base

---


###  11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

**Query**
```sql
WITH customer_plans AS (
    SELECT 
        customer_id,
        plan_id,
        start_date,
        LAG(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS previous_plan
    FROM subscriptions
)
SELECT 
    COUNT(*) AS downgrade_count
FROM customer_plans
WHERE previous_plan = 2
    AND plan_id = 1
    AND EXTRACT(YEAR FROM start_date) = 2020;
```

**Results**
| downgrade_count |
| --------------- |
| 0               |

**Insights**
- This confirms customers who upgrade to pro find sufficient value to maintain their premium tier

---