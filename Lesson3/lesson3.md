# Lesson 3 quizzes

### SUM function:

1) Find the total amount of poster_qty paper ordered in the orders table.

```sql
SELECT SUM(poster_qty) AS poster_qty_sum
	FROM orders
```

2) Find the total amount of standard_qty paper ordered in the orders table.

```sql
SELECT SUM(standard_qty) AS standard_qty_sum
	FROM orders
```

3) Find the total dollar amount of sales using the total_amt_usd in the orders table.

```sql
SELECT SUM(total_amt_usd) AS total_sales
	FROM orders
```

4) Find the total amount spent on standard_amt_usd and gloss_amt_usd paper for each order in the orders table. This should give a dollar amount for each 
order in the table.

```sql
SELECT standard_amt_usd+gloss_amt_usd AS standard_gloss_spent
	FROM orders
```

5) Find the standard_amt_usd per unit of standard_qty paper. Your solution should use both an aggregation and a mathematical operator.

```sql
SELECT SUM(standard_amt_usd)/SUM(standard_qty) AS price_unit_standard
	FROM orders
```

### MIN, MAX and AVG functions:

1) When was the earliest order ever placed? You only need to return the date.

```sql
SELECT MIN(occurred_at) AS earliest_date
	FROM orders
```

2) Try performing the same query as in question 1 without using an aggregation function.

```sql
SELECT occurred_at AS earliest_date
	FROM orders
	ORDER BY occurred_at
	LIMIT 1
```

3) When did the most recent (latest) web_event occur?

```sql
SELECT MAX(occurred_at) AS most_recent_event
	FROM web_events
```

4) Try to perform the result of the previous query without using an aggregation function.

```sql
SELECT occurred_at AS most_recent_event
	FROM web_events
	ORDER BY occurred_at DESC
	LIMIT 1
```

5) Find the mean (AVERAGE) amount spent per order on each paper type, as well as the mean amount of each paper type purchased per order. Your final answer 
should have 6 values - one for each paper type for the average number of sales, as well as the average amount.

```sql
SELECT AVG(standard_qty) AS standard_avg_qty,
	   AVG(gloss_qty) AS gloss_avg_qty,
       AVG(poster_qty) AS poster_avg_qty,
       AVG(standard_amt_usd) AS standard_avg_usd,
	   AVG(gloss_amt_usd) AS gloss_avg_usd,
       AVG(poster_amt_usd) AS poster_avg_usd
	FROM orders
```

### GROUP BY function:

1) Which account (by name) placed the earliest order? Your solution should have the account name and the date of the order.

```sql
SELECT a.name, o.occurred_at
	FROM accounts a
	JOIN orders o
		ON a.id = o.account_id
	ORDER BY occurred_at
	LIMIT 1;
```

2) Find the total sales in usd for each account. You should include two columns - the total sales for each company's orders in usd and the company name.

```sql
SELECT a.name, SUM(total_amt_usd) AS total_sales
	FROM accounts AS a
	JOIN orders AS o
		ON a.id = o.account_id
	GROUP BY a.name
```

3) Via what channel did the most recent (latest) web_event occur, which account was associated with this web_event? Your query should return only three values 
- the date, channel, and account name.

```sql
SELECT we.occurred_at, we.channel, a.name
	FROM web_events we
	JOIN accounts a
		ON we.account_id = a.id 
	ORDER BY we.occurred_at DESC
	LIMIT 1;
```

4) Find the total number of times each type of channel from the web_events was used. Your final table should have two columns - the channel and the number of 
times the channel was used.

```sql
SELECT channel, COUNT(*) AS times_used
	FROM web_events
	GROUP BY channel
```

5) Who was the primary contact associated with the earliest web_event? 

```sql
SELECT a.primary_poc
	FROM accounts AS a
	JOIN web_events AS we
		ON a.id = we.account_id
	ORDER BY we.occurred_at
	LIMIT 1
```

6) What was the smallest order placed by each account in terms of total usd. Provide only two columns - the account name and the total usd. Order from smallest 
dollar amounts to largest.

```sql
SELECT a.name, MIN(total_amt_usd) AS min_usd_order
	FROM accounts AS a
	JOIN orders AS o
		ON a.id = o.account_id
	GROUP BY a.name
	ORDER BY min_usd_order
```

7) Find the number of sales reps in each region. Your final table should have two columns - the region and the number of sales_reps. Order from fewest reps to 
most reps.

```sql
SELECT r.name, COUNT(*) AS sales_rep_number
	FROM region AS r
	JOIN sales_reps AS sr
		ON r.id = sr.region_id
	GROUP BY r.name
	ORDER BY sales_rep_number
```

### GROUP BY function (II):

1) For each account, determine the average amount of each type of paper they purchased across their orders. Your result should have four columns - one for the 
account name and one for the average quantity purchased for each of the paper types for each account.

```sql
SELECT a.name, 
	   AVG(o.standard_qty) AS standard_qty_avg,
	   AVG(o.gloss_qty) AS gloss_qty_avg, 
	   AVG(o.poster_qty) AS poster_qty_avg
	FROM accounts AS a
	JOIN orders as o
		ON o.account_id = a.id
	GROUP BY a.name
```

2) For each account, determine the average amount spent per order on each paper type. Your result should have four columns - one for the account name and one 
for the average amount spent on each paper type.

```sql
SELECT a.name, 
	   AVG(o.standard_amt_usd) AS standard_usd,
	   AVG(o.gloss_amt_usd) AS gloss_usd, 
	   AVG(o.poster_amt_usd) AS poster_usd
	FROM accounts AS a
	JOIN orders as o
		ON o.account_id = a.id
	GROUP BY a.name
```

3) Determine the number of times a particular channel was used in the web_events table for each sales rep. Your final table should have three columns - the 
name of the sales rep, the channel, and the number of occurrences. Order your table with the highest number of occurrences first.

```sql
SELECT sr.name, we.channel, COUNT(*) AS num_times
	FROM web_events AS we
	JOIN accounts AS a
		ON we.account_id = a.id
	JOIN sales_reps as sr
		ON sr.id = a.sales_rep_id
	GROUP BY r.name, we.channel
	ORDER BY num_times DESC
```

4) Determine the number of times a particular channel was used in the web_events table for each region. Your final table should have three columns - the region 
name, the channel, and the number of occurrences. Order your table with the highest number of occurrences first.

```sql
SELECT r.name, we.channel, COUNT(*) AS num_times
	FROM web_events AS we
	JOIN accounts AS a
		ON we.account_id = a.id
	JOIN sales_reps as sr
		ON sr.id = a.sales_rep_id
	JOIN region AS r
		ON sr.region_id = r.id
	GROUP BY r.name, we.channel
	ORDER BY num_times DESC
```

### DISTINCT function:

1) Use DISTINCT to test if there are any accounts associated with more than one region.

```sql
SELECT a.id, sr.region_id
	FROM accounts AS a
	JOIN sales_reps AS sr
		ON a.sales_rep_id = sr.id
```

```sql
SELECT DISTINCT id
	FROM accounts
```

Since both queries return 351 rows, there are no accounts associated with more than 1 region.

2) Have any sales reps worked on more than one account?

```sql
SELECT sr.id, COUNT(*) AS num_accounts
	FROM accounts AS a
	JOIN sales_reps AS sr
		ON a.sales_rep_id = sr.id
	GROUP BY sr.id
```

```sql
SELECT DISTINCT id
	FROM sales_reps
```

We see that both queries return 50 rows, so there are 50 sales reps. And all of them worked on more than 1 account.

### HAVING statement:

1) How many of the sales reps have more than 5 accounts that they manage?

```sql
SELECT sr.id, COUNT(*) AS num_accounts
	FROM accounts AS a
	JOIN sales_reps AS sr
		ON a.sales_rep_id = sr.id
	GROUP BY sr.id
    HAVING COUNT(*) > 5
```

2) How many accounts have more than 20 orders?

```sql
SELECT a.id, COUNT(*) AS order_num
	FROM accounts AS a
	JOIN orders AS o
		ON o.account_id = a.id
	GROUP BY a.id
	HAVING COUNT(*) > 20
```

3) Which account has the most orders?

```sql
SELECT a.id, COUNT(*) AS order_num
	FROM accounts AS a
	JOIN orders AS o
		ON o.account_id = a.id
	GROUP BY a.id
	ORDER BY order_num DESC
	LIMIT 1
```

4) How many accounts spent more than 30,000 usd total across all orders?

```sql
SELECT a.id, SUM(o.total_amt_usd) AS total_usd
	FROM accounts AS a
	JOIN orders AS o
		ON o.account_id = a.id
	GROUP BY a.id
	HAVING SUM(o.total_amt_usd) > 30000
```

5) How many accounts spent less than 1,000 usd total across all orders?

```sql
SELECT a.id, SUM(o.total_amt_usd) AS total_usd
	FROM accounts AS a
	JOIN orders AS o
		ON o.account_id = a.id
	GROUP BY a.id
	HAVING SUM(o.total_amt_usd) < 1000
```

6) Which account has spent the most with us?

```sql
SELECT a.id, SUM(o.total_amt_usd) AS total_usd
	FROM accounts AS a
	JOIN orders AS o
		ON o.account_id = a.id
	GROUP BY a.id
	ORDER BY total_usd DESC
	LIMIT 1
```

7) Which account has spent the least with us?

```sql
SELECT a.id, SUM(o.total_amt_usd) AS total_usd
	FROM accounts AS a
	JOIN orders AS o
		ON o.account_id = a.id
	GROUP BY a.id
	ORDER BY total_usd
	LIMIT 1
```

8) Which accounts used facebook as a channel to contact customers more than 6 times?

```sql
SELECT a.id, COUNT(*) AS num_fb
	FROM accounts AS a
	JOIN web_events AS we
		ON a.id = we.account_id
		AND we.channel = 'facebook'
	GROUP BY a.id
	HAVING COUNT(*) > 6
```

9) Which account used facebook most as a channel? 

```sql
SELECT a.id, COUNT(*) AS num_fb
	FROM accounts AS a
	JOIN web_events AS we
		ON a.id = we.account_id
		AND we.channel = 'facebook'
	GROUP BY a.id
	ORDER BY num_fb DESC
	LIMIT 1
```

10) Which channel was most frequently used by most accounts?

```sql
SELECT a.id, we.channel, COUNT(*) AS times_used
	FROM web_events AS we
	JOIN accounts AS a
		ON a.id = we.account_id
	GROUP BY a.id, we.channel
	ORDER BY times_used DESC
	LIMIT 10
```

All 10 are direct.

### DATE functions:

1) Find the sales in terms of total dollars for all orders in each year, ordered from greatest to least. Do you notice any trends in the yearly sales totals?

```sql
SELECT DATE_PART('year',occurred_at) AS year, SUM(total_amt_usd) AS total_sales
	FROM orders
	GROUP BY 1
	ORDER BY 2 DESC
```

Sales increase from 2013 to 2016 then dropped 2017. We notice that we only have sales for one month in 2013 and 2017.

2) Which month did Parch & Posey have the greatest sales in terms of total dollars? Are all months evenly represented by the dataset?

```sql
SELECT DATE_PART('month',occurred_at) AS month, SUM(total_amt_usd) AS total_sales
	FROM orders
	WHERE occurred_at BETWEEN '2014-01-01' and '2017-01-01'
	GROUP BY 1
	ORDER BY 2 DESC
```

We exclude year 2013 and 2017 as per the observation above in all questions from here. Greatest sales occur in December (12).

3) Which year did Parch & Posey have the greatest sales in terms of total number of orders? Are all years evenly represented by the dataset?

```sql
SELECT DATE_PART('year',occurred_at) AS year, SUM(total) AS total_qty
	FROM orders
	GROUP BY 1
	ORDER BY 2 DESC
```

Again 2016 is the best year, considering 2013 and 2017 are not evenly represented.

4) Which month did Parch & Posey have the greatest sales in terms of total number of orders? Are all months evenly represented by the dataset?

```sql
SELECT DATE_PART('month',occurred_at) AS month, SUM(total) AS total_qty
	FROM orders
	WHERE occurred_at BETWEEN '2014-01-01' and '2017-01-01'
	GROUP BY 1
	ORDER BY 2 DESC
```

Again December is best month.

5) In which month of which year did Walmart spend the most on gloss paper in terms of dollars?

```sql
SELECT DATE_TRUNC('month',o.occurred_at) AS month_year, SUM(o.gloss_amt_usd) AS gloss_usd_total
	FROM orders AS o
	JOIN accounts AS a
		ON o.account_id = a.id
	AND a.name = 'Walmart'
	GROUP BY 1
	ORDER BY 2 DESC
	LIMIT 1
```

In May 2016.

### CASE statement:

1) We would like to understand 3 different levels of customers based on the amount associated with their purchases. The top branch includes anyone with a 
Lifetime Value (total sales of all orders) greater than 200,000 usd. The second branch is between 200,000 and 100,000 usd. The lowest branch is anyone under 
100,000 usd. Provide a table that includes the level associated with each account. You should provide the account name, the total sales of all orders for the 
customer, and the level. Order with the top spending customers listed first.

```sql
SELECT a.name, SUM(o.total_amt_usd) AS total_sales,
	   CASE 
       		WHEN SUM(o.total_amt_usd) > 200000 THEN 'greater than 200,000'
        	WHEN SUM(o.total_amt_usd) <= 200000 AND SUM(o.total_amt_usd) >= 100000 THEN '200,000 and 100,000'
        	ELSE 'under 100,000' 
       END AS level
	FROM accounts AS a
	JOIN orders AS o
	ON a.id = o.account_id
	GROUP BY a.name
	ORDER BY total_sales DESC
```

2) We would now like to perform a similar calculation to the first, but we want to obtain the total amount spent by customers only in 2016 and 2017. Keep the 
same levels as in the previous question. Order with the top spending customers listed first.

```sql
SELECT a.name, SUM(o.total_amt_usd) AS total_sales,
	   CASE 
       		WHEN SUM(o.total_amt_usd) > 200000 THEN 'greater than 200,000'
        	WHEN SUM(o.total_amt_usd) <= 200000 AND SUM(o.total_amt_usd) > 100000 THEN '200,000 and 100,000'
        	ELSE 'under 100,000' 
       END AS level
	FROM accounts AS a
	JOIN orders AS o
		ON a.id = o.account_id
		AND o.occurred_at BETWEEN '2016-01-01' and '2018-01-01'
	GROUP BY a.name
	ORDER BY total_sales DESC
```

3) We would like to identify top performing sales reps, which are sales reps associated with more than 200 orders. Create a table with the sales rep name, the 
total number of orders, and a column with top or not depending on if they have more than 200 orders. Place the top sales people first in your final table.

```sql
SELECT sr.name, COUNT(*) AS total_orders,
	   CASE
       		WHEN COUNT(*) > 200 THEN 'top'
            ELSE 'not'
       END AS top_performer
	FROM orders AS o
	JOIN accounts AS a
		ON o.account_id = a.id
	JOIN sales_reps AS sr
		ON a.sales_rep_id = sr.id
	GROUP BY sr.name
	ORDER BY total_orders DESC
```

4) The previous didn't account for the middle, nor the dollar amount associated with the sales. Management decides they want to see these characteristics 
represented as well. We would like to identify top performing sales reps, which are sales reps associated with more than 200 orders or more than 750000 in total 
sales. The middle group has any rep with more than 150 orders or 500000 in sales. Create a table with the sales rep name, the total number of orders, total sales
across all orders, and a column with top, middle, or low depending on this criteria. Place the top sales people based on dollar amount of sales first in your 
final table. You might see a few upset sales people by this criteria!

```sql
SELECT sr.name, COUNT(*) AS total_orders, SUM(o.total_amt_usd) AS total_sales,
	   CASE
       		WHEN COUNT(*) > 200 OR SUM(o.total_amt_usd) > 750000 THEN 'top'
            WHEN COUNT(*) > 150 OR SUM(o.total_amt_usd) > 500000 THEN 'middle'
            ELSE 'low'
       END AS top_performer
	FROM orders AS o
	JOIN accounts AS a
		ON o.account_id = a.id
	JOIN sales_reps AS sr
		ON a.sales_rep_id = sr.id
	GROUP BY sr.name
	ORDER BY total_sales DESC
```