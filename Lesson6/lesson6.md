# Lesson 6 quizzes

### Window Functions:

1) create a running total of standard_amt_usd (in the orders table) over order time with no date truncation. Your final table should have two columns: one with 
the amount being added for each new row, and a second with the running total.

```sql
SELECT standard_amt_usd,
	   SUM(standard_amt_usd) OVER (ORDER BY occurred_at) AS running_total
FROM orders
```

2) Now, modify your query from the previous quiz to include partitions. Still create a running total of standard_amt_usd (in the orders table) over order time, 
but this time, date truncate occurred_at by year and partition by that same year-truncated occurred_at variable. Your final table should have three columns: One 
with the amount being added for each row, one for the truncated date, and a final columns with the running total within each year.

```sql
SELECT standard_amt_usd,
	   DATE_TRUNC('year', occurred_at) AS year,
	   SUM(standard_amt_usd) OVER (PARTITION BY DATE_TRUNC('year', occurred_at) ORDER BY occurred_at) AS running_total
FROM orders
```

### ROW_NUMBER and RANK functions:

1) Select the id, account_id, and total variable from the orders table, then create a column called total_rank that ranks this total amount of paper ordered (from 
highest to lowest) for each account using a partition. Your final table should have these four columns.

```sql
SELECT id, account_id, total,
	   RANK() OVER (PARTITION BY account_id ORDER BY total DESC) AS total_rank
FROM orders
```

### Aliasing window functions

1) Use an alias to simplify the following SQL expression.

```sql
SELECT id,
       account_id,
       DATE_TRUNC('year',occurred_at) AS year,
       DENSE_RANK() OVER account_year_window AS dense_rank,
       total_amt_usd,
       SUM(total_amt_usd) OVER account_year_window AS sum_total_amt_usd,
       COUNT(total_amt_usd) OVER account_year_window AS count_total_amt_usd,
       AVG(total_amt_usd) OVER account_year_window AS avg_total_amt_usd,
       MIN(total_amt_usd) OVER account_year_window AS min_total_amt_usd,
       MAX(total_amt_usd) OVER account_year_window AS max_total_amt_usd
FROM orders
WINDOW account_year_window AS (PARTITION BY account_id ORDER BY DATE_TRUNC('year',occurred_at))                                                                         
```

### LAG and LEAD functions:

1) Imagine you're an analyst at Parch & Posey and you want to determine how the current order's total revenue (i.e. from sales of all types of paper) revenue 
compares to the next order's total revenue. You'll need to use occurred_at and total_amt_usd in the orders table along with LEAD to do so. In your query results,
there should be four columns: occurred_at, total_amt_usd, lead, and lead_difference.

```sql
SELECT occurred_at,
       total_sum,
       LEAD(total_sum) OVER (ORDER BY occurred_at) - total_sum AS revenue_comparison
	FROM (
		SELECT occurred_at,
			   SUM(total_amt_usd) AS total_sum
			FROM orders 
			GROUP BY 1
	) sub
```

### NTILE function:

1) Use the NTILE functionality to divide the accounts into 4 levels in terms of the amount of standard_qty for their orders. Your resulting table should have 
the account_id, the occurred_at time for each order, the total amount of standard_qty paper purchased, and one of four levels in a standard_quartile column.

```sql
SELECT account_id, occurred_at, standard_qty,
	   NTILE(4) OVER (PARTITION BY account_id ORDER BY standard_qty) AS standard_quartile
	FROM orders
```

2) Use the NTILE functionality to divide the accounts into two levels in terms of the amount of gloss_qty for their orders. Your resulting table should have 
the account_id, the occurred_at time for each order, the total amount of gloss_qty paper purchased, and one of two levels in a gloss_half column.

```sql
SELECT account_id, occurred_at, gloss_qty,
	   NTILE(2) OVER (PARTITION BY account_id ORDER BY gloss_qty) AS gloss_half
	FROM orders
```

3) Use the NTILE functionality to divide the orders for each account into 100 levels in terms of the amount of total_amt_usd for their orders. Your resulting 
table should have the account_id, the occurred_at time for each order, the total amount of total_amt_usd paper purchased, and one of 100 levels in a 
total_percentile column.

```sql
SELECT account_id, occurred_at, total_amt_usd,
	   NTILE(100) OVER (PARTITION BY account_id ORDER BY total_amt_usd) AS total_percentile
	FROM orders
```