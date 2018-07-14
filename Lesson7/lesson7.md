# Lesson 7 quizzes

### FULL OUTER JOIN:

1) Return each account who has a sales rep and each sales rep that has an account but also each account that does not have a sales rep and each sales rep that 
does not have an account. Are there any unmatched rows?

```sql
SELECT *
	FROM accounts AS a
	FULL OUTER JOIN sales_reps AS sr
		ON a.sales_rep_id = sr.id
	WHERE a.sales_rep_id IS NULL OR sr.id IS NULL
```

No, all rows are matched.

### JOIN with comparison operator:

1) Write a query that left joins the accounts table and the sales_reps tables on each sale rep's ID number and joins it using the < comparison operator on 
accounts.primary_poc and sales_reps.name. The query results should be a table with three columns: the account name (e.g. Johnson Controls), the primary 
contact name (e.g. Cammy Sosnowski), and the sales representative's name (e.g. Samuel Racine). 

```sql
SELECT a.name AS account_name, a.primary_poc, sr.name AS sales_rep_name
	FROM accounts AS a
    LEFT JOIN sales_reps AS sr
    	ON a.sales_rep_id = sr.id
        AND a.primary_poc < sr.name
```

### Self JOIN:

1) Select events from the web_events table that are made by the same account within one day of each other. Include the channel used in both events.

```sql
SELECT we1.id AS we1_id,
       we1.account_id AS we1_account_id,
       we1.occurred_at AS we1_occurred_at,
       we1.channel AS we1_channel,
       we2.id AS we2_id,
       we2.account_id AS we2_account_id,
       we2.occurred_at AS we2_occurred_at,
       we2.channel AS we2.channel
	FROM web_events we1
	LEFT JOIN web_events we2
		ON we1.account_id = we2.account_id
		AND we2.occurred_at > we1.occurred_at
		AND we2.occurred_at <= we1.occurred_at + INTERVAL '1 day'
	ORDER BY we1.account_id, we1.occurred_at
```

### UNION statement:

1) Write a query that uses UNION ALL on two instances (and selecting all columns) of the accounts table.

```sql
SELECT *
	FROM accounts AS a1
UNION ALL
SELECT *
	FROM accounts AS a2
```

2) Add a WHERE clause to each of the tables that you unioned in the query above, filtering the first table where name equals Walmart and filtering the second 
table where name equals Disney.

```sql
SELECT *
	FROM accounts AS a1
    WHERE name = 'Walmart'
UNION ALL
SELECT *
	FROM accounts AS a2
    WHERE name = 'Disney'
```

3) Perform the union in your first query in a common table expression and name it double_accounts. Then do a COUNT the number of times a name appears in the 
double_accounts table.

```sql
WITH double_accounts AS (
	SELECT *
		FROM accounts AS a1
	UNION ALL
	SELECT *
		FROM accounts AS a2 )

SELECT name, COUNT(*) AS num_name
	FROM double_accounts
	GROUP BY name
```