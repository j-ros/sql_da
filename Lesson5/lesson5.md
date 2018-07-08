# Lesson 5 quizzes

### LEFT and RIGHT functions:

1) In the accounts table, there is a column holding the website for each company. The last three digits specify what type of web address they are using.
Pull these extensions and provide how many of each website type exist in the accounts table.

```sql
SELECT RIGHT(website, 3) AS web_address_type, COUNT(*) AS web_address_count
	FROM accounts
	GROUP BY 1
	ORDER BY 2 DESC
```

2) There is much debate about how much the name (or even the first letter of a company name) matters. Use the accounts table to pull the first letter of each 
company name to see the distribution of company names that begin with each letter (or number). 

```sql
SELECT LEFT(name, 1) AS first_letter, COUNT(*) AS first_letter_count
	FROM accounts
	GROUP BY 1
	ORDER BY 2 DESC
```

3) Use the accounts table and a CASE statement to create two groups: one group of company names that start with a number and a second group of those company names 
that start with a letter. What proportion of company names start with a letter?

```sql
SELECT CASE 
			WHEN LEFT(name, 1) IN ('0','1','2','3','4','5','6','7','8','9','0') THEN 1
			ELSE 0
	   END AS is_numeric,
	   COUNT(*) AS group_number
	FROM accounts
	GROUP BY 1
```

350 start with a letter and 1 with a number.

4) Consider vowels as a, e, i, o, and u. What proportion of company names start with a vowel, and what percent start with anything else? 

```sql
SELECT CASE 
			WHEN LOWER(LEFT(name, 1)) IN ('a','e','i','o','u') THEN 1
			ELSE 0
	   END AS is_vowel,
	   COUNT(*) AS group_number
	FROM accounts
	GROUP BY 1
```

80 start with vowel, 271 not.

### POSITION and STRPOS statements:

1) Use the accounts table to create first and last name columns that hold the first and last names for the primary_poc. 

```sql
SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') - 1) AS first_name,
       RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) AS last_name             
	FROM accounts
```

2) Now see if you can do the same thing for every rep name in the sales_reps table. Again provide first and last name columns.

```sql
SELECT LEFT(name, STRPOS(name, ' ') - 1) AS first_name,
       RIGHT(name, LENGTH(name) - STRPOS(name, ' ')) AS last_name             
FROM sales_reps
```

### CONCAT function:

1) Each company in the accounts table wants to create an email address for each primary_poc. The email address should be the first name of the primary_poc . 
last name primary_poc @ company name .com.

```sql
WITH t1 AS(
	SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') - 1) AS first_name,
           RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) AS last_name,
		   name
	FROM accounts
)

SELECT CONCAT(first_name,'.',last_name,'@',name,'.com') AS email
	FROM t1
```

2) You may have noticed that in the previous solution some of the company names include spaces, which will certainly not work in an email address. See if you 
can create an email address that will work by removing all of the spaces in the account name, but otherwise your solution should be just as in question 1. 

```sql
WITH t1 AS(
	SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') - 1) AS first_name,
           RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) AS last_name,
		   name
	FROM accounts
)

SELECT CONCAT(first_name,'.',last_name,'@',REPLACE(name, ' ', ''),'.com') AS email
	FROM t1
```

3) We would also like to create an initial password, which they will change after their first log in. The first password will be the first letter of the 
primary_poc's first name (lowercase), then the last letter of their first name (lowercase), the first letter of their last name (lowercase), the last letter of 
their last name (lowercase), the number of letters in their first name, the number of letters in their last name, and then the name of the company they are 
working with, all capitalized with no spaces. 

```sql
WITH t1 AS(
	SELECT LEFT(primary_poc, STRPOS(primary_poc, ' ') - 1) AS first_name,
           RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) AS last_name,
		   name
	FROM accounts
)

SELECT CONCAT(LOWER(LEFT(first_name,1)),
			  LOWER(RIGHT(first_name,1)),
			  LOWER(LEFT(last_name,1)),
			  LOWER(RIGHT(last_name,1)),
			  LENGTH(first_name),
			  LENGTH(last_name),
			  UPPER(REPLACE(name, ' ', ''))) AS password
	FROM t1
```

### CAST function:

1) Convert the date column in the sf_crime_data table to the correct DATE format in SQL. The column is in mm/dd/yyyy format.

```sql
SELECT (SUBSTR(date,7,4) || '-' || LEFT(date,2) || '-' || SUBSTR(date,4,2))::DATE AS new_date
	FROM sf_crime_data
```

Notice that CONCAT operations can be written with pipes '||' and that the CAST can be made with ()::DATE instead of CAST(... AS DATE)

### COALESCE function:

1) Use COALESCE to fill the accounts.id column with the account.id value, orders.account_id with the account.id and each of the qty and usd columns with 0.
Then count the number of id.

```sql
WITH filled_table AS(
	SELECT  COALESCE(a.id, a.id) AS filled_id,
			a.name, 
			a.website, 
			a.lat, 
			a.long, 
			a.primary_poc, 
			a.sales_rep_id, 
			COALESCE(o.account_id, a.id) AS filled_account_id, 
			o.occurred_at, 
			COALESCE(o.standard_qty,0) AS filled_standard_qty, 
			COALESCE(o.gloss_qty,0) AS filled_gloss_qty, 
			COALESCE(o.poster_qty,0) AS filled_poster_qty, 
			COALESCE(o.total,0) AS filled_total, 
			COALESCE(o.standard_amt_usd,0) AS filled_standard_usd, 
			COALESCE(o.gloss_amt_usd,0) AS filled_gloss_usd, 
			COALESCE(o.poster_amt_usd,0) AS filled_poster_usd, 
			COALESCE(o.total_amt_usd,0) AS filled_total_usd
	FROM accounts AS a
	LEFT JOIN orders AS o
	ON a.id = o.account_id
)

SELECT COUNT(*)
	FROM filled_table
```

6913 registers.