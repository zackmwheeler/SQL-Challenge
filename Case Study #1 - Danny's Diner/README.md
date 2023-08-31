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
          partition by customer_id
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


