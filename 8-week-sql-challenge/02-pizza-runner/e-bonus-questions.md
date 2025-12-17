###  If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an `INSERT` statement to demonstrate what would happen if a new `Supreme` pizza with all the toppings was added to the Pizza Runner menu?

**Query**
```sql
INSERT INTO pizza_names (pizza_id, pizza_name)
VALUES (3, 'Supreme');

INSERT INTO pizza_recipes (pizza_id, toppings)
VALUES (3, '1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12');

SELECT 
    pn.pizza_name,
    STRING_AGG(pt.topping_name, ', ' ORDER BY pt.topping_name) AS all_toppings
FROM pizza_recipes pr
JOIN pizza_names pn ON pr.pizza_id = pn.pizza_id
CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(pr.toppings, ',')::INTEGER[]) AS t(topping_id)
JOIN pizza_toppings pt ON t.topping_id = pt.topping_id
GROUP BY pn.pizza_name;
```

**Results**
| pizza_name | all_toppings                                                                                                   |
| ---------- | -------------------------------------------------------------------------------------------------------------- |
| Meatlovers | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami                                          |
| Supreme    | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Onions, Pepperoni, Peppers, Salami, Tomato Sauce, Tomatoes |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                                                     |

**Insights**
- The original data design does not change with the addition of a supreme pizza as only one row needs to be added to `pizza_names` and `pizza_recipes` each
- Therefore, no schema updates or changes to existing data are needed
---