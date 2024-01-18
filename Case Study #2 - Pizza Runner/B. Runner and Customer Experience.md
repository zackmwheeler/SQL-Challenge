# Case Study #2 - Pizza Runner

## B. Runner and Customer Experience

### Questions and Solutions

#### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```
SELECT
	DATE_TRUNC('WEEK', registration_date) as registration_week
,	COUNT(runner_id) as number_registered
FROM pizza_runner.runners
GROUP BY registration_week
ORDER BY registration_week;
```

#### Answer:

| registration_week        | number_registered |
| ------------------------ | ----------------- |
| 2020-12-28T00:00:00.000Z | 2                 |
| 2021-01-04T00:00:00.000Z | 1                 |
| 2021-01-11T00:00:00.000Z | 1                 |

---

#### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```
SELECT
	r.runner_id
,	AVG(EXTRACT(MINUTE FROM(r.pickup_time::timestamp - c.order_time))) as min_to_pickup
FROM customer_orders1 c
INNER JOIN runner_orders1 r
	ON c.order_id = r.order_id
WHERE r.pickup_time IS NOT NULL
GROUP BY r.runner_id
ORDER BY r.runner_id;
```

#### Answer:

| runner_id | min_to_pickup      |
| --------- | ------------------ |
| 1         | 15.333333333333334 |
| 2         | 23.4               |
| 3         | 10                 |

---

#### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```
WITH prep_time_cte AS
(
  SELECT 
    c.order_id
,	COUNT(c.order_id) AS pizza_order
,	c.order_time
,	r.pickup_time
,	EXTRACT(MINUTE FROM(r.pickup_time::timestamp - c.order_time)) AS prep_time_minutes
  FROM customer_orders1 c
  JOIN runner_orders1 r
    ON c.order_id = r.order_id
  WHERE r.distance IS NOT NULL
  GROUP BY c.order_id, c.order_time, r.pickup_time
)

SELECT 
  pizza_order, 
  AVG(prep_time_minutes) AS avg_prep_time_minutes
FROM prep_time_cte
WHERE prep_time_minutes > 1
GROUP BY pizza_order;
```

#### Answer:

| pizza_order | avg_prep_time_minutes |
| ----------- | --------------------- |
| 3           | 29                    |
| 2           | 18                    |
| 1           | 12                    |

On average a single pizza takes 12 minutes to prepare.

When ordering 3 pizzas it takes 29 minutes to make a pizza, which is 9.67 minutes per pizza making it quicker than the 12 minutes for a single order.

When ordering 2 pizzas it only takes 18 minutes, which is 9 minutes per piza making it the most efficient.

---

#### 4. What was the average distance travelled for each customer?

```
SELECT 
	c.customer_id
,	AVG(r.distance) AS avg_distance
FROM customer_orders1 c
JOIN runner_orders1 r
  ON c.order_id = r.order_id
WHERE r.duration IS NOT NULL
GROUP BY c.customer_id;
```

#### Answer:

| customer_id | avg_distance |
| ----------- | ------------ |
| 101         | 20           |
| 102         | 16.733333333 |
| 103         | 23.4         |
| 104         | 10           |
| 105         | 25           |

---

#### 5. What was the difference between the longest and shortest delivery times for all orders?

```
SELECT
    MAX(CAST(duration AS INT)) - MIN(CAST(duration AS INT)) AS delivery_time_diff
FROM runner_orders1;
```

#### Answer:

| delivery_time_diff |
| ------------------ |
| 30                 |

---
#### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

```
SELECT 
	runner_id
,	AVG(CAST(distance AS DECIMAL(3,1)))
,	AVG(CAST(duration AS INT))
FROM runner_orders1
GROUP BY runner_id;
```

#### Answer:

| runner_id | avg                 | avg                 |
| --------- | ------------------- | ------------------- |
| 3         | 10.0000000000000000 | 15.0000000000000000 |
| 2         | 23.9333333333333333 | 26.6666666666666667 |
| 1         | 15.8500000000000000 | 22.2500000000000000 |

---

#### 7. What is the successful delivery percentage for each runner?

```
SELECT 
	runner_id
,	CAST(SUM(CASE
        WHEN pickup_time IS NULL THEN 0
        ELSE 1
    END) as DECIMAL(3,0)) / COUNT(order_id)*100 as successful_order_percentage
FROM runner_orders1
GROUP BY runner_id;
```

#### Answer

| runner_id | successful_order_percentage |
| --------- | --------------------------- |
| 3         | 50                          |
| 2         | 75                          |
| 1         | 100                         |

---

