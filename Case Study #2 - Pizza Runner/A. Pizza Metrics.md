# Case Study #2 - Pizza Runner

## A. Pizza Metrics

### Questions and Solutions

#### 1. How many pizzas were ordered?

```
SELECT
	COUNT(*) num_pizzas_ordered
FROM
	customer_orders1;
```

#### Answer:

| num_pizzas_ordered |
| ------------------ |
| 14                 |

---

#### 2. How many unique customer orders were made?

```
SELECT
	COUNT(DISTINCT order_id) unique_pizza_count
FROM
	customer_orders1;
```

#### Answer:

| unique_pizza_count |
| ------------------ |
| 10                 |

---

#### 3. How many successful orders were delivered by each runner?

```
SELECT
	runner_id
,	COUNT(order_id) as completed_orders
FROM
	runner_orders1
WHERE distance IS NOT NULL
GROUP BY runner_id
ORDER BY completed_orders DESC;
```

#### Answer:

| runner_id | completed_orders |
| --------- | ---------------- |
| 1         | 4                |
| 2         | 3                |
| 3         | 1                |

---

#### 4. How many of each type of pizza was delivered?

```
SELECT
	p.pizza_name
,	COUNT(c.pizza_id) as num_ordered
FROM
	customer_orders1 c
INNER JOIN pizza_names1 p
	ON c.pizza_id = p.pizza_id
INNER JOIN runner_orders1 r
	ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL
GROUP BY p.pizza_name
ORDER BY num_ordered DESC;
```

#### Answer:

| pizza_name | num_ordered |
| ---------- | ----------- |
| Meatlovers | 9           |
| Vegetarian | 3           |

---

#### Q5. How many Vegetarian and Meatlovers were ordered by each customer?

```
SELECT
	c.customer_id
,	p.pizza_name
,	COUNT(c.pizza_id) as num_ordered
FROM
	customer_orders1 c
INNER JOIN pizza_names1 p
	ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY c.customer_id;
```

#### Answer:

| customer_id | pizza_name | num_ordered |
| ----------- | ---------- | ----------- |
| 101         | Meatlovers | 2           |
| 101         | Vegetarian | 1           |
| 102         | Meatlovers | 2           |
| 102         | Vegetarian | 1           |
| 103         | Meatlovers | 3           |
| 103         | Vegetarian | 1           |
| 104         | Meatlovers | 3           |
| 105         | Vegetarian | 1           |

---

#### 6. What was the maximum number of pizzas delivered in a single order?

```
SELECT
	c.order_id
,	COUNT(c.pizza_id) as total_delivered
FROM
	customer_orders1 c
INNER JOIN runner_orders1 r
	ON c.order_id = r.order_id
GROUP BY c.order_id
ORDER BY total_delivered DESC
LIMIT 1;
```

#### Answer:

| order_id | total_delivered |
| -------- | --------------- |
| 4        | 3               |

---

#### Q7. For each customer, how many delivered pizzas had at least 1 change, and how many had no changes?

```
SELECT
	c.customer_id
,	SUM(
		CASE WHEN c.exclusions IS NOT NULL OR c.extras IS NOT NULL THEN 1
  		ELSE 0
		END) as more_than_1_change
,	SUM(
		CASE WHEN c.exclusions IS NULL AND c.extras IS NULL THEN 1
  		ELSE 0
		END) as no_changes
FROM
	customer_orders1 c
INNER JOIN runner_orders1 r
	ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY c.customer_id
ORDER BY c.customer_id;
```

#### Answer:

| customer_id | more_than_1_change | no_changes |
| ----------- | ------------------ | ---------- |
| 101         | 0                  | 2          |
| 102         | 0                  | 3          |
| 103         | 3                  | 0          |
| 104         | 2                  | 1          |
| 105         | 1                  | 0          |

---

#### Q7. 8. How many pizzas were delivered that had both exclusions and extras?

```
SELECT
	SUM(
		CASE WHEN c.exclusions IS NOT NULL AND c.extras IS NOT NULL THEN 1
  		ELSE 0
		END) as extras_and_exclusions
FROM
	customer_orders1 c
INNER JOIN runner_orders1 r
	ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
;
```

#### Answer:

| extras_and_exclusions |
| --------------------- |
| 1                     |

---

#### 9. What was the total volume of pizzas ordered for each hour of the day?

```
SELECT
	DATE_PART('HOUR', c.order_time) as hour
,	COUNT(c.order_id) as pizzas_ordered
FROM customer_orders1 c
GROUP BY hour
ORDER BY pizzas_ordered DESC;
```

#### Answer:

| hour | pizzas_ordered |
| ---- | -------------- |
| 18   | 3              |
| 23   | 3              |
| 21   | 3              |
| 13   | 3              |
| 11   | 1              |
| 19   | 1              |

---

#### 10. What was the volume of orders for each day of the week?

```
WITH cte_day_name AS (
	SELECT
        DATE_PART('dow', order_time) as dow
    ,	COUNT(order_id) as pizzas_ordered
    FROM customer_orders1 
    GROUP BY dow
)

SELECT
	pizzas_ordered
,	CASE
		WHEN dow = 0 THEN 'Sunday'
		WHEN dow = 1 THEN 'Monday'
        WHEN dow = 2 THEN 'Tuesday'
        WHEN dow = 3 THEN 'Wednesday'
        WHEN dow = 4 THEN 'Thursday'
        WHEN dow = 5 THEN 'Friday'
        WHEN dow = 6 THEN 'Saturday'
	END as day_of_week
FROM cte_day_name
ORDER BY pizzas_ordered DESC;
```

#### Answer:

| pizzas_ordered | day_of_week |
| -------------- | ----------- |
| 5              | Wednesday   |
| 5              | Saturday    |
| 3              | Thursday    |
| 1              | Friday      |

