# Case Study #2: Pizza Runner - C. Ingredient Optimization Questions and Solutions

###  1. What are the standard ingredients for each pizza?

**Query**
```sql
SELECT
    pn.pizza_name,
    STRING_AGG(pt.topping_name, ', ') AS standard_ingredients
FROM pizza_names pn
JOIN pizza_recipes pr ON pn.pizza_id = pr.pizza_id
CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(pr.toppings, ',')) AS topping_id_text
JOIN pizza_toppings pt ON pt.topping_id = topping_id_text::INTEGER
GROUP BY pn.pizza_name
ORDER BY pn.pizza_name;
```

**Results**
| pizza_name | standard_ingredients                                                  |
| ---------- | --------------------------------------------------------------------- |
| Meatlovers | BBQ Sauce, Pepperoni, Cheese, Salami, Chicken, Bacon, Mushrooms, Beef |
| Vegetarian | Tomato Sauce, Cheese, Mushrooms, Onions, Peppers, Tomatoes            |

**Insights**
- Meatlovers is significantly more complex with 8 ingredients compared to Vegetarian's 6 ingredients
- Five meat proteins (Pepperoni, Salami, Chicken, Bacon, Beef) are present within the Meatlovers pizza
- Only two ingredients overlap between pizzas: Cheese and Mushrooms 

---


###  2. What was the most commonly added extra?

**Query**
```sql
SELECT 
    pt.topping_name,
    COUNT(*) AS times_added_as_extra
FROM customer_orders_clean co
CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(co.extras, ',')) AS extra_id
JOIN pizza_toppings pt ON pt.topping_id = TRIM(extra_id)::INTEGER
WHERE co.extras IS NOT NULL
GROUP BY pt.topping_name
ORDER BY times_added_as_extra DESC;
```

**Results**
| topping_name | times_added_as_extra |
| ------------ | -------------------- |
| Bacon        | 4                    |
| Chicken      | 1                    |
| Cheese       | 1                    |

**Insights**
- Bacon is the most popular extra topping, added 4 times compared to just 1 time each for Chicken and Cheese
---


###  3. What was the most common exclusion?

**Query**
```sql
SELECT 
    pt.topping_name,
    COUNT(*) AS times_as_exclusion
FROM customer_orders_clean co
CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(co.exclusions, ',')) AS exclusion_id
JOIN pizza_toppings pt ON pt.topping_id = TRIM(exclusion_id)::INTEGER
WHERE co.exclusions IS NOT NULL
GROUP BY pt.topping_name
ORDER BY times_as_exclusion DESC;
```

**Results**
| topping_name | times_as_exclusion |
| ------------ | ------------------ |
| Cheese       | 4                  |
| Mushrooms    | 1                  |
| BBQ Sauce    | 1                  |

**Insights**
- Cheese is by far the most excluded ingredient, removed 4 times
- Only 3 toppings are ever excluded, suggesting most customers accept the standard recipes
- High Cheese exclusion rate suggests offering a dairy-free or reduced-cheese option to appeal to customers

---


###  4. Generate an order item for each record in the **customers_orders** table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

**Query**
```sql
WITH order_details AS(
    SELECT
        co.order_id,
        co.customer_id,
        co.pizza_id,
        pn.pizza_name,
        co.extras,
        co.exclusions,
        (SELECT STRING_AGG(pt.topping_name, ', ')
            FROM UNNEST(STRING_TO_ARRAY(co.exclusions, ', ')) AS exclusion_id
            JOIN pizza_toppings pt ON pt.topping_id = TRIM(exclusion_id)::INT)
        AS exclusions_name,
        (SELECT STRING_AGG(pt.topping_name, ', ')
            FROM UNNEST(STRING_TO_ARRAY(co.extras, ', ')) AS extras_id
            JOIN pizza_toppings pt ON pt.topping_id = TRIM(extras_id)::INT)
        AS extras_name
    FROM customer_orders_clean co
    JOIN pizza_names pn ON co.pizza_id = pn.pizza_id)
SELECT 
    order_id,
    customer_id,
    pizza_id,
    CASE 
        WHEN exclusions IS NULL AND extras IS NULL THEN pizza_name
        WHEN exclusions IS NOT NULL AND extras IS NULL THEN pizza_name || ' - Exclude ' || exclusions_name
        WHEN exclusions IS NULL AND extras IS NOT NULL THEN pizza_name || ' - Extra ' || extras_name
        ELSE pizza_name || ' - Exclude ' || exclusions_name || ' - Extra ' || extras_name
    END AS order_item
FROM order_details
ORDER BY order_id;
```

**Results**
| order_id | customer_id | pizza_id | order_item                                                      |
| -------- | ----------- | -------- | --------------------------------------------------------------- |
| 1        | 101         | 1        | Meatlovers                                                      |
| 2        | 101         | 1        | Meatlovers                                                      |
| 3        | 102         | 2        | Vegetarian                                                      |
| 3        | 102         | 1        | Meatlovers                                                      |
| 4        | 103         | 1        | Meatlovers - Exclude Cheese                                     |
| 4        | 103         | 1        | Meatlovers - Exclude Cheese                                     |
| 4        | 103         | 2        | Vegetarian - Exclude Cheese                                     |
| 5        | 104         | 1        | Meatlovers - Extra Bacon                                        |
| 6        | 101         | 2        | Vegetarian                                                      |
| 7        | 105         | 2        | Vegetarian - Extra Bacon                                        |
| 8        | 102         | 1        | Meatlovers                                                      |
| 9        | 103         | 1        | Meatlovers - Exclude Cheese - Extra Bacon, Chicken              |
| 10       | 104         | 1        | Meatlovers                                                      |
| 10       | 104         | 1        | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |

**Insights**
- 7 out of 14 pizzas show customizations, showing customer demand for personalization
- Customer 103 consistently excludes cheese across all 3 pizzas ordered, strongly suggesting a dietary restriction
- The most complex customization is Order 10 with multiple exclusions and extras added

---


###  5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

**Query**
```sql
WITH standard_with_extras AS (
    SELECT
    co.order_id,
    co.pizza_id,
    pt.topping_id,
    CASE
        WHEN pt.topping_id = ANY(STRING_TO_ARRAY(co.extras, ',')::INTEGER[]) 
        THEN '2x' || pt.topping_name
        ELSE pt.topping_name
    END AS ingredient
    FROM customer_orders_clean co
    CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(
    (SELECT toppings FROM pizza_recipes WHERE pizza_id = co.pizza_id), 
    ',')::INTEGER[]) AS t(topping_id)
    JOIN pizza_toppings pt ON t.topping_id = pt.topping_id
    WHERE pt.topping_id != ALL(
    COALESCE(STRING_TO_ARRAY(co.exclusions, ',')::INTEGER[], ARRAY[]::INTEGER[])
    )
    
    UNION ALL
    
    SELECT
    co.order_id,
    co.pizza_id,
    e.topping_id,
    pt.topping_name AS ingredient
    FROM customer_orders_clean co
    CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(co.extras, ',')::INTEGER[]) AS e(topping_id)
    JOIN pizza_toppings pt ON e.topping_id = pt.topping_id
    WHERE e.topping_id != ALL(
    STRING_TO_ARRAY((SELECT toppings FROM pizza_recipes WHERE pizza_id = co.pizza_id), ',')::INTEGER[]
    )
),
all_ingredients AS (
    SELECT 
    order_id,
    pizza_id,
    STRING_AGG(DISTINCT ingredient, ', ' ORDER BY ingredient) AS ingredients
    FROM standard_with_extras
    GROUP BY order_id, pizza_id
)
SELECT 
    ai.order_id,
    CONCAT(pn.pizza_name, ': ', ai.ingredients) AS order_description
FROM all_ingredients ai 
JOIN pizza_names pn ON pn.pizza_id = ai.pizza_id
ORDER BY ai.order_id;
```

**Results**
| order_id | order_description                                                                                    |
| -------- | ---------------------------------------------------------------------------------------------------- |
| 1        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami                    |
| 2        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami                    |
| 3        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami                    |
| 3        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                               |
| 4        | Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami                            |
| 4        | Vegetarian: Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                                       |
| 5        | Meatlovers: 2xBacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami                  |
| 6        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                               |
| 7        | Vegetarian: Bacon, Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                        |
| 8        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami                    |
| 9        | Meatlovers: 2xBacon, 2xChicken, BBQ Sauce, Beef, Mushrooms, Pepperoni, Salami                        |
| 10       | Meatlovers: 2xBacon, 2xCheese, BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |

**Insights**
- Alphabetical ordering makes ingredient lists easy to scan and compare across orders
- The "2x" notation clearly distinguishes doubled ingredients from standard ones 

---


###  6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

**Query**
```sql
WITH delivered_orders AS (
    SELECT 
    co.*,
    ROW_NUMBER() OVER (ORDER BY co.order_id, co.pizza_id) AS pizza_row_id
    FROM customer_orders_clean co
    JOIN runner_orders_clean ro ON co.order_id = ro.order_id
    WHERE ro.cancellation IS NULL
),
standard_ingredients AS (
    SELECT
    dlo.pizza_row_id,
    dlo.order_id,
    dlo.pizza_id,
    t.topping_id
    FROM delivered_orders dlo
    CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(
    (SELECT toppings FROM pizza_recipes WHERE pizza_id = dlo.pizza_id), 
    ',')::INTEGER[]) AS t(topping_id)
),
exclusions_parsed AS (
    SELECT 
    dlo.pizza_row_id,
    e.topping_id
    FROM delivered_orders dlo
    CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(dlo.exclusions, ',')::INTEGER[]) AS e(topping_id)
    WHERE dlo.exclusions IS NOT NULL
),
extras_parsed AS (
    SELECT 
    dlo.pizza_row_id,
    e.topping_id
    FROM delivered_orders dlo
    CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(dlo.extras, ',')::INTEGER[]) AS e(topping_id)
    WHERE dlo.extras IS NOT NULL
),
all_used_ingredients AS (
    SELECT si.topping_id
    FROM standard_ingredients si
    LEFT JOIN exclusions_parsed ex 
    ON si.pizza_row_id = ex.pizza_row_id 
    AND si.topping_id = ex.topping_id
    WHERE ex.topping_id IS NULL
    
    UNION ALL
    
    SELECT ep.topping_id
    FROM extras_parsed ep
)
SELECT 
    pt.topping_name,
    COUNT(*) AS times_used
FROM all_used_ingredients aui
JOIN pizza_toppings pt ON aui.topping_id = pt.topping_id
GROUP BY pt.topping_name
ORDER BY times_used DESC, pt.topping_name;
```

**Results**
| topping_name | times_used |
| ------------ | ---------- |
| Bacon        | 12         |
| Mushrooms    | 11         |
| Cheese       | 10         |
| Beef         | 9          |
| Chicken      | 9          |
| Pepperoni    | 9          |
| Salami       | 9          |
| BBQ Sauce    | 8          |
| Onions       | 3          |
| Peppers      | 3          |
| Tomato Sauce | 3          |
| Tomatoes     | 3          |

**Insights**
- Bacon is by far the most used ingredient in all delivered pizzas, compared to all the other meats which are used 9 times
- This data confirms inventory management should prioritize Bacon, Mushrooms, and Cheese as the most critical ingredients

---