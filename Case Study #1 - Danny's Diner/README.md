<h1>Case Study #1 - Danny's Diner</h1>
<img alt="Danny's Diner" width="500" src="https://8weeksqlchallenge.com/images/case-study-designs/1.png">
<a href = "https://8weeksqlchallenge.com/case-study-1/">Link to the challenge</a>  

<h3>Business Task</h3>
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite.

<h3>Business Questions & Solutions</h3>

<h4>1. What is the total amount each customer spent at the restaurant?</h4>

```
Select 
	sales.customer_id,
    SUM(menu.price) as total_sales
From dannys_diner.sales
INNER JOIN dannys_diner.menu
	on sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```
<h4>Steps Used:</h4>
  <ul>
    <li>Started by using <b>JOIN</b> on `dannys_diner.sales` and `dannys_diner.menu` to have access to the two tables needed, `sales.customer_id` and `menu.price`.</li>
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
	sales.customer_id,
    COUNT(DISTINCT sales.order_date) as count_visits
From dannys_diner.sales
GROUP BY sales.customer_id;
```
<h4>Steps Used:</h4>
  <ul>
    <li>Started by using `dannys_diner.sales` since it has both `sales.customer_id` and `sales.order_date`.</li>
    <li>In order to find unique values for different days visited I used the <b>COUNT(DISTINCT `sales.order_date`)</b>.</li>
  </ul>

  <h4>Answer:</h4>

customer_id | count_visits
--- | --- 
A | 4
B | 6
C | 2

---

<h4>3. What was the first item from the menu purchased by each customer?</h4>

