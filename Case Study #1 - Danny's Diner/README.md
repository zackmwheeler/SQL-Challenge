<h1>Case Study #1 - Danny's Diner</h1>

<a href = "https://8weeksqlchallenge.com/case-study-1/"><img alt="Danny's Diner" width="500" src="https://8weeksqlchallenge.com/images/case-study-designs/1.png"></a>  

<h3>Business Task</h3>
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite.

<h3>Business Questions & Solutions</h3>

<h4>1. What is the total amount each customer spent at the restaurant?</h4>

```
Select 
	sales.customer_id
,	SUM(menu.price) as total_sales
From dannys_diner.sales
INNER JOIN dannys_diner.menu
	on sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```
<h4>Steps Used:</h4>
  <ul>
    <li>Started by using <b>JOIN</b> on <tt>dannys_diner.sales</tt> and <tt>dannys_diner.menu</tt> to have access to the two tables needed, <tt>sales.customer_id</tt> and <tt>menu.price</tt>.</li>
    <li>Used <b>SUM</b> to calculate total sales.</li>
    <li>Finally used <b>GROUP BY</b> and <b>ORDER BY</b> to combine and order results.</li>
  </ul>
<h4>Answer:</h4>

customer_id | total_sales
--- | --- 
A | 76
B | 74
C | 36

---

<h4>2. How many days has each customer visited the restaurant?</h4>

```
Select 
	sales.customer_id
,	COUNT(DISTINCT sales.order_date) as count_visits
From dannys_diner.sales
GROUP BY sales.customer_id;
```
<h4>Steps Used:</h4>
  <ul>
    <li>Started by using <tt>dannys_diner.sales</tt> since it has both <tt>sales.customer_id</tt> and <tt>sales.order_date</tt>.</li>
    <li>In order to find unique values for different days visited I used the <b>COUNT(DISTINCT <tt>sales.order_date</tt>)</b>.</li>
  </ul>

  <h4>Answer:</h4>

customer_id | count_visits
--- | --- 
A | 4
B | 6
C | 2

---

<h4>3. What was the first item from the menu purchased by each customer?</h4>

```
WITH cte_ranked as (
  Select 
  	sales.customer_id
  ,	sales.order_date
  ,	menu.product_name
  ,	dense_rank() over(
          partition by sales.customer_id
          order by
              sales.order_date) as rank
  From dannys_diner.sales
  INNER JOIN dannys_diner.menu
      on sales.product_id = menu.product_id
)
  
Select
	customer_id
,	product_name
From cte_ranked
where rank = 1
group by
	customer_id
,	product_name;
```

<h4>Steps Used:</h4>
  <ul>
    <li>Started by creating a CTE(<tt>cte_ranked</tt>) to create a new column(<tt>rank</tt>).  In the new column I used the <b>DENSE_RANK()</b> function to number the rows by the <tt>sales.order_date</tt></li>
    <li>In the outerquery I <b>SELECT</b> <tt>customer_id</tt> and <tt>produyct_name</tt> from the new cte table I made(<tt>cte_ranked</tt>).  I then used the <b>GROUP BY</b> function on both <tt>customer_id</tt> and <tt>produyct_name</tt>.  With the <b>WHERE</b> clause will pull the product that is ranked '1'.</li>
  </ul>

<h4>Answer:</h4>
  
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

We can see that customer 'A' bought two items the first time, which shows here.  Customers 'B' and 'C' ordered one each.

---

<h4>4. What is the most purchased item on the menu and how many times was it purchased by all customers?</h4>

```
Select
	menu.product_name
,	COUNT(sales.product_id) as number_sold
From dannys_diner.sales
INNER JOIN dannys_diner.menu
	on sales.product_id = menu.product_id
Group by menu.product_name
Order by number_sold DESC
LIMIT 1;
```

<h4>Steps Used:</h4>
  <ul>
    <li>I used the <b>COUNT</b> function on <tt>sales.product_id</tt> to count the amount of each item was sold(<tt>number_sold</tt>).</li>
    <li>Then I used <b>ORDER BY DESC</b> to show the most to least sold.</li>
    <li>Finally used <b>LIMIT 1</b> to show only the product that sold the most.</li>
  </ul>

<h4>Answer:</h4>

| product_name | number_sold |
| ------------ | ----------- |
| ramen        | 8           |

---

<h4>5. Which item was the most popular for each customer?</h4>

```
WITH cte_ranked as (
  Select 
  	sales.customer_id
  ,	menu.product_name
  ,	COUNT(sales.product_id) as number_sold
  ,	dense_rank() over(
          partition by sales.customer_id
          order by
              COUNT(sales.customer_id) desc
  	) as rank
  From dannys_diner.sales
  INNER JOIN dannys_diner.menu
      on sales.product_id = menu.product_id
  group by
  	sales.customer_id
  ,	menu.product_name
)
  
Select
	customer_id
,	product_name
,	number_sold
From cte_ranked
where rank = 1;
```

<h4>Steps Used:</h4>
  <ul>
    <li>Started by creating a CTE(<tt>cte_ranked</tt>).</li>
    <li>In the CTE I used <b>DENSE_RANK()</b> again to calculate the ranking of each <tt>sales.customer_id</tt> partition by the number of orders of <b>COUNT(<tt>sales.customer_id</tt>)</b> in descending order.</li>
    <li>In the outer query is where I chose the 3 columns to be used and used the <b>WHERE</b> clause to choose the products that are only ranked 1.</li>
  </ul>

<h4>Answer:</h4>

| customer_id | product_name | number_sold |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| B           | ramen        | 2           |
| B           | curry        | 2           |
| B           | sushi        | 2           |
| C           | ramen        | 3           |

Here we see customer 'B' with all 3 items ranked the same, which means they choose them all the same amount.  Customers 'A' and 'C' both have ramen as their favorite.

---

<h4>6. Which item was purchased first by the customer after they became a member?</h4>

```
WITH cte_became_member as (
  Select 
  	members.customer_id
  ,	sales.product_id
  ,	ROW_NUMBER() over(
          partition by members.customer_id
          order by
              sales.order_date
  	) as rank
  From dannys_diner.members
  INNER JOIN dannys_diner.sales
  	on members.customer_id = sales.customer_id
  	and sales.order_date > members.join_date
)
  
Select
	customer_id
,	menu.product_name
From cte_became_member
INNER JOIN dannys_diner.menu
	on cte_became_member.product_id = menu.product_id
where rank = 1
order by customer_id;
```

<h4>Steps Used:</h4>
  <ul>
    <li>Started by creating a CTE(<tt>cte_became_member</tt>).</li>
    <li>In the CTE I used <b>ROW_NUMBER()</b> to order each <tt>members.customer_id</tt> partition by the order date <tt>sales.order_date</tt>.  Then used <b>INNER JOIN</b> on <tt>dannys_diner.members</tt> and <tt>dannys_diner.sales</tt> on <tt>customer_id</tt> of both and any <tt>sales.order_date</tt> that was after <tt>members.join_date</tt>.</li>
    <li>In the outer query I used <b>INNER JOIN</b> on the CTE table we just created (<tt>cte_became_member</tt>) and <tt>dannys_diner.menu</tt> in order to pull <tt>menu.product_name</tt>.  With the <b>WHERE</b> clause I made it so that it will only grab the first date.  Then finally I ordered it by <tt>customer_id</tt>.</li>
  </ul>

<h4>Answer:</h4>

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

---

<h4>7. Which item was purchased just before the customer became a member?</h4>

```
WITH cte_before_member as (
  Select 
  	members.customer_id
  ,	sales.product_id
  ,	ROW_NUMBER() over(
          partition by members.customer_id
          order by
              sales.order_date desc
  	) as rank
  From dannys_diner.members
  INNER JOIN dannys_diner.sales
  	on members.customer_id = sales.customer_id
  	and sales.order_date < members.join_date
)
  
Select
	customer_id
,	menu.product_name
From cte_before_member
INNER JOIN dannys_diner.menu
	on cte_before_member.product_id = menu.product_id
where rank = 1
order by customer_id;
```

<h4>Steps Used:</h4>
  <ul>
    <li>Started by creating a CTE(<tt>cte_before_member</tt>).</li>
    <li>In the CTE I used <b>ROW_NUMBER()</b> to order each <tt>members.customer_id</tt> partition by the order date <tt>sales.order_date</tt> in descending order.  Then used <b>INNER JOIN</b> on <tt>dannys_diner.members</tt> and <tt>dannys_diner.sales</tt> on <tt>customer_id</tt> of both and any <tt>sales.order_date</tt> that was before <tt>members.join_date</tt>.</li>
    <li>In the outer query I used <b>INNER JOIN</b> on the CTE table we just created (<tt>cte_before_member</tt>) and <tt>dannys_diner.menu</tt> in order to pull <tt>menu.product_name</tt>.  With the <b>WHERE</b> clause I made it so that it will only grab the date the is first in descending order(or the last one before becoming a member).  Then finally I ordered it by <tt>customer_id</tt>.</li>
  </ul>

<h4>Answer:</h4>

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | sushi        |

---

<h4>8. What is the total items and amount spent for each member before they became a member?</h4>

```
Select 
	members.customer_id
  ,	COUNT(sales.product_id)
  ,	SUM(menu.price)
From dannys_diner.members
INNER JOIN dannys_diner.sales
	on members.customer_id = sales.customer_id
  	and sales.order_date < members.join_date
INNER JOIN dannys_diner.menu
	on sales.product_id = menu.product_id
group by
	members.customer_id
order by
	members.customer_id;
```

<h4>Steps Used:</h4>
  <ul>
    <li>Decided what columns would be needed. <tt>members.customer_id</tt> then I needed the amount of items sold (<b>COUNT(<tt>sales.product_id</tt>)</b>.  Finally needed the total spent on items which is in the <b>SUM(<tt>menu.price</tt>)</b>.</li>
    <li>In order to get all the columns I needed I had to join all 3 tables together.  Since I needed to find the amount spent by members before becoming they became member I needed to <b>INNER JOIN</b> <tt>dannys_diner.members</tt> and <tt>dannys_diner.sales</tt> on <tt>customer_id</tt> on both and any sales before they became a member.</li>
    <li>Finally I needed the prices which is in the <tt>dannys_diner.menu</tt> table so I used <b>INNER JOIN</b> with <tt>dannys_diner.sales</tt>.</li>
  </ul>

<h4>Answer:</h4>

| customer_id | count | sum |
| ----------- | ----- | --- |
| A           | 2     | 25  |
| B           | 3     | 40  |

---

<h4>9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?</h4>

```
WITH cte_points as (
  Select
  	menu.product_id
  ,	CASE
  		WHEN menu.product_id = 1 THEN price * 20
  		ELSE price * 10 
  		END as points
  From dannys_diner.menu
)

Select
	sales.customer_id
,	SUM(cte_points.points) as total_points
From dannys_diner.sales
INNER JOIN cte_points
	on sales.product_id = cte_points.product_id
Group by sales.customer_id
Order by sales.customer_id;
```

<h4>Steps Used:</h4>
  <ul>
    <li>Created a CTE (<tt>cte_points</tt>) where it takes the <tt>menu.product_id</tt> and multiplies the price by the amount of points earned ( 10 or 20).  Since sushi has the <tt>menu.product_id</tt> of 1 we mulyiplied it by 20 and all others would be by 10.</li>
    <li>In the outer query I calculated the <tt>total_points</tt> and <b>GROUP BY</b> <tt>sales.customer_id</tt>.</li>
  </ul>

<h4>Answer:</h4>

| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

---

<h4>10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?</h4>

```
WITH cte_points as (
  Select
  	menu.product_id
  ,	sales.customer_id
  ,	CASE
  		WHEN menu.product_id = 1 THEN menu.price * 20
  		WHEN sales.order_date BETWEEN members.join_date and (members.join_date + interval '6 day') THEN menu.price * 20
  		ELSE menu.price * 10
  		END as points
  From dannys_diner.menu
  INNER JOIN dannys_diner.sales
  	on menu.product_id = sales.product_id
  INNER JOIN dannys_diner.members
  	on sales.customer_id = members.customer_id
  WHERE sales.order_date <= '2021-01-31'
)

Select
	customer_id
,	SUM(cte_points.points) as total_points
From cte_points
Group by customer_id
Order by customer_id;
```

<h4>Steps Used:</h4>
  <ul>
    <li>I started with the previous CTE from the question before.  This time I used <b>INNER JOIN</b> on both <tt>dannys_diner.sales</tt> and <tt>dannys_diner.members</tt> to have access to the columns needed.</li>
    <li>I added a new part to the <b>CASE</b> function where it looks at the <tt>sales.order_date</tt> and checks if it is <b>BETWEEN</b> <tt>members.join_date</tt> and <tt>members.join_date</tt> + <b>INTERVAL '6 day'</b>.  If it is, then they get double points, just as they would with sushi.  If not then they get the normal amount of points.</li>
    <li>Finally I added a <b>WHERE</b> clause to the end of the CTE so it will only check up to <b>'2021-01-31'</b> on the <tt>sales.order_date</tt> to find all points up until the end of January.</li>
  </ul>

<h4>Answer:</h4>

| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 820          |

---

<h4>Bonus Question: Join All The Things</h4>

```
Select
	sales.customer_id
,	sales.order_date
,	menu.product_name
,	menu.price
,	CASE
  		WHEN sales.order_date >= members.join_date THEN 'Y'
  		ELSE 'N' END as member
From dannys_diner.sales
INNER JOIN dannys_diner.menu
	on sales.product_id = menu.product_id
LEFT JOIN dannys_diner.members
	on sales.customer_id = members.customer_id
Order by 
	sales.customer_id
,	sales.order_date;
```

<h4>Answer:</h4>

| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |

---

<h4>Bonus Question: Rank All The Things</h4>

```
WITH cte_customers as (
	Select
		sales.customer_id
	,	sales.order_date
	,	menu.product_name
	,	menu.price
	,	CASE
	  		WHEN sales.order_date >= members.join_date THEN 'Y'
	  		ELSE 'N' END as member       
	From dannys_diner.sales
	INNER JOIN dannys_diner.menu
		on sales.product_id = menu.product_id
	LEFT JOIN dannys_diner.members
		on sales.customer_id = members.customer_id
	Order by 
		sales.customer_id
	,	sales.order_date
)

Select
	*
,	CASE
		WHEN member = 'N' THEN NULL
        ELSE RANK() OVER (
          Partition by customer_id, member
          Order by order_date
    ) END as ranking
From cte_customers;
```

<h4>Answer:</h4>

| customer_id | order_date               | product_name | price | member | ranking |
| ----------- | ------------------------ | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |         |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |         |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      | 1       |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |         |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |         |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |         |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |         |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |         |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |         |


<h2>Thank you for checking out my solutions to Case Study #1 - Danny's Diner from the 8 Week SQL Challenge!
If you would like to do it yourself, <a href = "https://8weeksqlchallenge.com/case-study-1/">Click here!</a></h2>
