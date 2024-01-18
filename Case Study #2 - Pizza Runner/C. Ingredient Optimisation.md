# Case Study #2 - Pizza Runner

## C. Ingredient Optimisation

### Questions and Solutions

#### 1. What are the standard ingredients for each pizza?

```
SELECT 
	pr.pizza_id 
,	pt.topping_name
FROM pizza_recipes pr
LEFT JOIN pizza_toppings pt 
    ON pr.toppings = pt.topping_id
```

#### Answer:

| pizza_id | topping_name |
| -------- | ------------ |
| 1        | Bacon        |
| 1        | BBQ Sauce    |
| 1        | Beef         |
| 1        | Cheese       |
| 1        | Chicken      |
| 1        | Mushrooms    |
| 1        | Pepperoni    |
| 1        | Salami       |
| 2        | Cheese       |
| 2        | Mushrooms    |
| 2        | Onions       |
| 2        | Peppers      |
| 2        | Tomatoes     |
| 2        | Tomato Sauce |

This means that Pizza 1 (Meatlovers) included the toppings:
  baccon, BBQ sauce, beef, cheese, chicken, mushrooms, pepperoni, salami
Pizza 2 (Vegitarian) included the toppings:
  cheese, mushrooms, onions, peppers, tomatoes, tomato sauce

---

#### 2. What was the most commonly added extra?

```
SELECT 
	extras
,	COUNT(extras) AS extras_counted
FROM customer_orders1
WHERE extras LIKE '%'
GROUP BY extras;
```

#### Answer:

| extras | extras_counted |
| ------ | -------------- |
| 1, 5   | 1              |
| 1, 4   | 1              |
| 1      | 2              |

---

#### 3. What was the most common exclusion?

```
SELECT
	exclusions
,	COUNT(exclusions) AS exclusions_count
FROM customer_orders1
WHERE exclusions LIKE '%'
GROUP BY exclusions;
```

#### Answer:

| exclusions | exclusions_count |
| ---------- | ---------------- |
| 2, 6       | 1                |
| 4          | 4                |

---

#### 4.Generate an order item for each record in the customers_orders table in the format of one of the following:

### Meat Lovers
```
SELECT order_id
FROM customer_orders1
WHERE pizza_id = 1
GROUP BY order_id;
```

| order_id |
| -------- |
| 1        |
| 2        |
| 3        |
| 4        |
| 5        |
| 8        |
| 9        |
| 10       |

### Meat Lovers - Exclude Beef
```
SELECT order_id
FROM customer_orders1
WHERE pizza_id = 1 AND exclusions LIKE '%3%'
GROUP BY order_id;
```
No results were generated.  There were no orders that had beef as an exclusion.

### Meat Lovers - Extra Bacon
```
SELECT order_id
FROM customer_orders1
WHERE pizza_id = 1 AND extras LIKE '%1%'
GROUP BY order_id;
```

| order_id |
| -------- |
| 5        |
| 7        |
| 9        |
| 10       |

### Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
```
WITH exc_ext_counter AS (
SELECT
	order_id
,	CASE
		WHEN exclusions LIKE '%1%' OR exclusions LIKE '%4%' THEN 1
		WHEN extras LIKE '%6%' OR extras LIKE '%9%' THEN 1
	END AS exc_ext_count
FROM customer_orders1
WHERE pizza_id = 1
)

SELECT order_id
FROM exc_ext_counter
WHERE exc_ext_count = 1
GROUP BY order_id;
```

| order_id |
| -------- |
| 4        |
| 9        |

---

#### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients.

```

```
