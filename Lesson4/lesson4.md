# Lesson 4 quizzes

### Subqueries:

1) Find the average number of events per day for each channel.

```sql
SELECT channel, AVG(num_events) AS avg_events
	FROM
		(SELECT DATE_TRUNC('day',occurred_at) AS day,
				channel,
				COUNT(*) AS num_events
			FROM web_events
			GROUP BY day, channel
		) AS sub
	GROUP BY channel
```

2) Find the average quantity sold for each type of paper and the total usd amount during the first month with orders placed on the ordes table.

```sql
SELECT AVG(standard_qty) AS avg_standard,
	   AVG(gloss_qty) AS avg_gloss,
       AVG(poster_qty) AS avg_poster,
	   SUM(total_amt_usd) AS total_usd
	FROM orders
	WHERE DATE_TRUNC('month',occurred_at) =
		( SELECT DATE_TRUNC('month',MIN(occurred_at)) AS min_month
		FROM orders )
```

### Subqueries (II):

1) Provide the name of the sales_rep in each region with the largest amount of total_amt_usd sales.

```sql
SELECT t3.sales_rep, t3.region, t3.total_usd_region_sales_rep
	FROM
		(SELECT t1.region, MAX(t1.total_usd_region_sales_rep) AS max_region_usd
			FROM
				(SELECT sr.name AS sales_rep, r.name AS region, SUM(total_amt_usd) AS total_usd_region_sales_rep
					FROM orders AS o
					JOIN accounts AS a
					ON o.account_id = a.id
					JOIN sales_reps AS sr
					ON a.sales_rep_id = sr.id
					JOIN region AS r
					ON sr.region_id = r.id
					GROUP BY 1,2) AS t1
			GROUP BY t1.region) AS t2
	JOIN
		(SELECT sr.name AS sales_rep, r.name AS region, SUM(total_amt_usd) AS total_usd_region_sales_rep
			FROM orders AS o
			JOIN accounts AS a
			ON o.account_id = a.id
			JOIN sales_reps AS sr
			ON a.sales_rep_id = sr.id
			JOIN region AS r
			ON sr.region_id = r.id
			GROUP BY 1,2) AS t3
	ON t3.region = t2.region AND t3.total_usd_region_sales_rep = t2.max_region_usd
```

2) For the region with the largest (sum) of sales total_amt_usd, how many total (count) orders were placed? 

```sql
SELECT r.name, COUNT(*) AS total_orders
	FROM orders AS o
	JOIN accounts AS a
	ON o.account_id = a.id
	JOIN sales_reps AS sr
	ON a.sales_rep_id = sr.id
	JOIN region AS r
	ON sr.region_id = r.id
	GROUP BY 1
	HAVING SUM(o.total_amt_usd) =
		(SELECT MAX(t1.total_usd_region) AS max_region_usd
			FROM
				(SELECT r.name AS region, SUM(o.total_amt_usd) AS total_usd_region
					FROM orders AS o
					JOIN accounts AS a
					ON o.account_id = a.id
					JOIN sales_reps AS sr
					ON a.sales_rep_id = sr.id
					JOIN region AS r
					ON sr.region_id = r.id
					GROUP BY 1
				) AS t1
		)
```

3) For the name of the account that purchased the most (in total over their lifetime as a customer) standard_qty paper, how many accounts still had more in 
total purchases? 

```sql
SELECT COUNT(*) AS num
	FROM
		(SELECT a.name
			FROM orders AS o
			JOIN accounts AS a
			ON o.account_id = a.id
			GROUP BY 1
			HAVING SUM(o.total) >
				(SELECT sum_total 
					FROM
						(SELECT a.name, SUM(o.standard_qty) AS sum_standard_qty, SUM(o.total) AS sum_total
							FROM orders AS o
							JOIN accounts AS a
							ON o.account_id = a.id
							GROUP BY 1
							ORDER BY 2 DESC
							LIMIT 1
						) AS t1
				)
		) AS t2
```

4) For the customer that spent the most (in total over their lifetime as a customer) total_amt_usd, how many web_events did they have for each channel?

```sql
SELECT a.name, we.channel, COUNT(*) AS num_web_events
	FROM web_events AS we
	JOIN accounts AS a
	ON we.account_id = a.id AND account_id =
		(SELECT id
			FROM
				(SELECT a.id, SUM(o.total_amt_usd)
					FROM orders AS o
					JOIN accounts AS a
					ON o.account_id = a.id
					GROUP BY 1
					ORDER BY 2 DESC
					LIMIT 1
				) AS t1
		)
	GROUP BY 1,2
```

5) What is the lifetime average amount spent in terms of total_amt_usd for the top 10 total spending accounts?

```sql
SELECT AVG(total_usd)
	FROM
		(SELECT account_id, SUM(total_amt_usd) AS total_usd
			FROM orders
			GROUP BY 1
			ORDER BY 2 DESC
			LIMIT 10
		) AS t1
```

6) What is the lifetime average amount spent in terms of total_amt_usd for only the companies that spent more than the average of all orders.

```sql
SELECT AVG(avg_usd)
	FROM
		(SELECT account_id, AVG(total_amt_usd) as avg_usd
			FROM orders
			GROUP BY 1
			HAVING AVG(total_amt_usd) >
				(SELECT AVG(total_amt_usd)
					FROM orders
				)
		) AS t1
```

### WITH statement:

1) Provide the name of the sales_rep in each region with the largest amount of total_amt_usd sales.

```sql
WITH t1 AS 
	(SELECT sr.name AS sales_rep, r.name AS region, SUM(total_amt_usd) AS total_usd_region_sales_rep
		FROM orders AS o
		JOIN accounts AS a
		ON o.account_id = a.id
		JOIN sales_reps AS sr
		ON a.sales_rep_id = sr.id
		JOIN region AS r
		ON sr.region_id = r.id
		GROUP BY 1,2
	),
	 t2 AS
	(SELECT t1.region, MAX(t1.total_usd_region_sales_rep) AS max_region_usd
		FROM t1
		GROUP BY t1.region
	)

SELECT t1.sales_rep, t1.region, t1.total_usd_region_sales_rep
	FROM t2
	JOIN t1
	ON t1.region = t2.region AND t1.total_usd_region_sales_rep = t2.max_region_usd
```

2) For the region with the largest (sum) of sales total_amt_usd, how many total (count) orders were placed? 

```sql
WITH t1 AS
	(SELECT r.name AS region, SUM(o.total_amt_usd) AS total_usd_region
		FROM orders AS o
		JOIN accounts AS a
		ON o.account_id = a.id
		JOIN sales_reps AS sr
		ON a.sales_rep_id = sr.id
		JOIN region AS r
		ON sr.region_id = r.id
		GROUP BY 1
	),
	 t2 AS
	(SELECT MAX(t1.total_usd_region) AS max_region_usd
		FROM t1
	)
	
SELECT r.name, COUNT(*) AS total_orders
	FROM orders AS o
	JOIN accounts AS a
	ON o.account_id = a.id
	JOIN sales_reps AS sr
	ON a.sales_rep_id = sr.id
	JOIN region AS r
	ON sr.region_id = r.id
	GROUP BY 1
	HAVING SUM(o.total_amt_usd) = (SELECT * FROM t2)
```

3) For the name of the account that purchased the most (in total over their lifetime as a customer) standard_qty paper, how many accounts still had more in 
total purchases? 

```sql
WITH t1 AS
	(SELECT a.name, SUM(o.standard_qty) AS sum_standard_qty, SUM(o.total) AS sum_total
		FROM orders AS o
		JOIN accounts AS a
		ON o.account_id = a.id
		GROUP BY 1
		ORDER BY 2 DESC
		LIMIT 1
	),
	 t2 AS
	(SELECT a.name
		FROM orders AS o
		JOIN accounts AS a
		ON o.account_id = a.id
		GROUP BY 1
		HAVING SUM(o.total) > (SELECT sum_total FROM t1)
	)
SELECT COUNT(*) AS num
	FROM t2
```

4) For the customer that spent the most (in total over their lifetime as a customer) total_amt_usd, how many web_events did they have for each channel?

```sql
WITH t1 AS
	(SELECT a.id, SUM(o.total_amt_usd)
		FROM orders AS o
		JOIN accounts AS a
		ON o.account_id = a.id
		GROUP BY 1
		ORDER BY 2 DESC
		LIMIT 1
	)
	
SELECT a.name, we.channel, COUNT(*) AS num_web_events
	FROM web_events AS we
	JOIN accounts AS a
	ON we.account_id = a.id AND account_id = (SELECT id FROM t1)
	GROUP BY 1,2
```

5) What is the lifetime average amount spent in terms of total_amt_usd for the top 10 total spending accounts?

```sql
WITH t1 AS
	(SELECT account_id, SUM(total_amt_usd) AS total_usd
		FROM orders
		GROUP BY 1
		ORDER BY 2 DESC
		LIMIT 10
	)
	
SELECT AVG(total_usd)
	FROM t1
```

6) What is the lifetime average amount spent in terms of total_amt_usd for only the companies that spent more than the average of all orders.

```sql
WITH t1 AS
	(SELECT account_id, AVG(total_amt_usd) as avg_usd
		FROM orders
		GROUP BY 1
		HAVING AVG(total_amt_usd) > (SELECT AVG(total_amt_usd) FROM orders)
	)
	
SELECT AVG(avg_usd)
	FROM t1
```