# Case Study #2 - Pizza Runner

<a href = "https://8weeksqlchallenge.com/case-study-2/"><img alt="Pizza Runner" width="500" src="https://8weeksqlchallenge.com/images/case-study-designs/2.png"></a>

### Business Task
Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

### Data Cleaning

First I needed to clean the available data since there are blanks, NULLS, and 'nulls' throughout the the datasets.  So let's look at each table individually.

#### Table: customer_orders

```
CREATE TEMP TABLE customer_orders1 as
SELECT
	order_id
,	customer_id
,	pizza_id
,	CASE
		WHEN exclusions = '' THEN NULL
        	WHEN exclusions = 'null' THEN NULL
		ELSE exclusions
        	END as exclusions
,	CASE
		WHEN extras = '' THEN NULL
        	WHEN extras = 'null' THEN NULL
        	ELSE extras
        	END as extras
,	order_time
FROM pizza_runner.customer_orders;
```
#### Table: runner_orders

```
CREATE TEMP TABLE runner_orders1 as
SELECT 
	order_id
,	runner_id
,	CASE
		WHEN pickup_time = 'null' THEN NULL
        	ELSE pickup_time
        	END as pickup_time
,	CASE
		WHEN distance = 'null' THEN NULL
        	ELSE TRIM('km ' from distance)
        	END as distance
,	CASE
		WHEN duration = 'null' THEN NULL
        	ELSE TRIM('minutes ' from duration)
        	END as duration
,	CASE
		WHEN cancellation = '' THEN NULL
        	WHEN cancellation = 'null' THEN NULL
        	ELSE cancellation
        	END as cancellation
FROM pizza_runner.runner_orders;
```

For the `runner_orders` table, I also needed to change a few columns into a different data type rather than __VARCHAR__

```
ALTER TABLE runner_orders1
	ALTER COLUMN pickup_time DATETIME
,	ALTER COLUMN distance FLOAT
,	ALTER COLUMN duration INT;
```

Now it should be good to start working on it!



### Case Study Questions

Case study questions are split into 5 categories.  Links below will access each of them:

<a href = "https://github.com/zackmwheeler/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/A.%20Pizza%20Metrics.md">**A. Pizza Metrics**</a>

<a href = "https://github.com/zackmwheeler/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/B.%20Runner%20and%20Customer%20Experience.md">**B. Runner and Customer Experience**</a>

<a href = "https://github.com/zackmwheeler/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/C.%20Ingredient%20Optimisation.md">**C. Ingredient Optimisation**</a>

<a href = "https://github.com/zackmwheeler/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/D.%20Pricing%20and%20Ratings.md">**D. Pricing and Ratings**</a>

<a href = "https://github.com/zackmwheeler/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/E.%20Bonus%20Questions.md">**E. Bonus Questions**</a>
