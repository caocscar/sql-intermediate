# Intermediate SQL Workshop <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->
- [SQL for Beginners](#sql-for-beginners)
- [DB Fiddle](#db-fiddle)
- [Dataset](#dataset)
- [SQL Order of Operations](#sql-order-of-operations)
- [Part I: More SQL Syntax](#part-i-more-sql-syntax)
- [IF](#if)
- [CASE](#case)
- [COALESCE](#coalesce)
- [Practice 1](#practice-1)
- [ROLLUP](#rollup)
- [Practice 2](#practice-2)
- [REPLACE](#replace)
- [WINDOW FUNCTIONS](#window-functions)
	- [OVER](#over)
	- [Non-Aggregate Function Example: RANK](#non-aggregate-function-example-rank)
	- [DENSE_RANK](#dense_rank)
	- [WINDOW](#window)
	- [PARTITION BY](#partition-by)
- [Practice 3](#practice-3)
- [SUBQUERIES](#subqueries)
- [CTE (Common Table Expression)](#cte-common-table-expression)
- [Scalar Subquery](#scalar-subquery)
- [Practice 4](#practice-4)
- [LATERAL JOIN](#lateral-join)
- [Nested SELECT](#nested-select)
- [CROSS JOIN](#cross-join)
- [SELF JOIN](#self-join)
- [USING and NATURAL](#using-and-natural)
- [UNNEST](#unnest)
- [EXPLAIN](#explain)
- [Miscellaneous Commands](#miscellaneous-commands)
- [PostgreSQL](#postgresql)
	- [Show Postgres Version](#show-postgres-version)
	- [List all public tables in database](#list-all-public-tables-in-database)
	- [List table schema](#list-table-schema)
	- [Create materialized view](#create-materialized-view)
	- [List materialized views](#list-materialized-views)
	- [Create Role and User for Read Only](#create-role-and-user-for-read-only)
	- [Create Index](#create-index)
	- [List Indexes in Database](#list-indexes-in-database)

## SQL for Beginners
My Intro to SQL workshop can be found [here](https://github.com/caocscar/workshops/tree/master/sql) as a Jupyter Notebook slide deck. This workshop builds off of that material.

## DB Fiddle
- DB Fiddle is an online environment testing SQL code
- DB Fiddle is like [w3schools](https://www.w3schools.com/sql/) but has more functionality
- You can create tables and query the tables in the browser
- In the left-hand window, you create the schema (columns and data types) and the data for a table
- In the right-hand window, you query the tables
- Link to db-fiddle: https://www.db-fiddle.com/
- Switch the current database to `MySQL v8.0` (which is a common database in the wild) in the top-left corner
- Switch the current database to `Postgres v10.0` (for Mayniacs) in the top-left corner

## Dataset
- We’ll be using Michigan COVID-19 public data. I filtered it to include records since the re-opening.
- You can find it here at this gist https://gist.github.com/caocscar/b9a1418e5fd9c2cd69bb6f9d67fbc05a
- Click on the `Raw` button in the top-right corner. This will bring up a new page.
- Copy the entire contents (21472 Records + Header Row)
- In the DB Fiddle window, click the `Text to DDL` button in lower-left
- Give it a table name of `Covid`
- Paste the contents into the `Formatted Text` window
- Click `Append to Schema`

## SQL Order of Operations
SQL queries are not executed sequentially when you write it. It is executed in the following order
1. FROM / JOIN
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. DISTINCT
7. ORDER BY
8. LIMIT / OFFSET

## Part I: More SQL Syntax

## IF
<details>
  <summary>MySQL</summary>
Similar to the Microsoft Excel functionality for IF
  
```SQL
IF(expression, true expression, false expression)
```

```SQL
SELECT County, 
IF (LENGTH(County) < 8, 'short', 'long') AS StringLength
FROM Covid
```
</details>

**Note**: PostgreSQL does not support `IF` statement in `SELECT`

## CASE
Used to convert a continuous variable into a categorical or ordinal variable. It is equivalent to if else if statements or switch case statements in programming.

```SQL
SELECT County, Day, Cases,
    CASE
        WHEN Cases = 0 THEN 'Good'
        WHEN Cases <= 5 THEN 'Not Bad'
        WHEN Cases <= 20 THEN 'Opposite of good'
        ELSE 'Stay at home'
    END AS Outlook
FROM Covid
WHERE County = 'Washtenaw' AND CP = 'Confirmed'
```

## COALESCE
Used to return the first non-null value in a list. We can use it to return a default value instead of `null` in the query. Here we return zero instead of `null`.
```SQL
SELECT COALESCE(SUM(Deaths), 0)
FROM Covid
WHERE County = 'Keweenaw'
```

## Practice 1
Create a variable called deathIndex that takes on the following values: 
- -1 if deathIndex is 0  
- 0 if deathIndex is equal to 1  
- log(N) if deathIndex is greater than 1 (where N = Deaths)

## ROLLUP
<details>
  <summary>MySQL</summary>
	
Suppose you want to add a total row to the bottom of a table. Based on the intro class, one way to do this is using `UNION`.

```SQL
SELECT CP, SUM(Cases) AS CaseTotal
FROM Covid
WHERE County = 'Washtenaw'
GROUP BY CP
UNION
SELECT NULL, SUM(Cases) AS CaseTotal
FROM Covid
WHERE County = 'Washtenaw'
```

A better way to code this and easier to understand is using the `WITH ROLLUP` syntax

```SQL
SELECT CP, SUM(Cases) AS CaseTotal
FROM Covid
WHERE County = 'Washtenaw'
GROUP BY CP WITH ROLLUP
```

If you want to substitute meaningful labels instead of null, use the `GROUPING` function with the `IF` function
returns 1 when null occurs in a super-aggregate row, otherwise 0

```SQL
SELECT IF(GROUPING(CP), 'Total', CP),
SUM(Cases) AS CaseTotal
FROM Covid
WHERE County = 'Washtenaw'
GROUP BY CP WITH ROLLUP
```
</details>

<details>
  <summary>PostgreSQL</summary>
	
Suppose you want to add a total row to the bottom of a table. Based on the intro class, one way to do this is using `UNION`.

```SQL
SELECT CP, SUM(Cases) AS CaseTotal
FROM Covid
WHERE County = 'Washtenaw'
GROUP BY CP
UNION
SELECT 'Total', SUM(Cases) AS CaseTotal
FROM Covid
WHERE County = 'Washtenaw'
ORDER BY CP
```

A better way to code this and easier to understand is using the `WITH ROLLUP` syntax

```SQL
SELECT CP, SUM(Cases) AS CaseTotal
FROM Covid
WHERE County = 'Washtenaw'
GROUP BY ROLLUP(CP)
ORDER BY CP
```

If you want to substitute meaningful labels instead of null, use the `GROUPING` function with the `IF` function
returns 1 when null occurs in a super-aggregate row, otherwise 0
```SQL
SELECT 
	CASE
		WHEN GROUPING(CP) = 0 THEN CP
        ELSE 'Total'
    END AS CP,
SUM(Cases) AS CaseTotal
FROM Covid
WHERE County = 'Washtenaw'
GROUP BY ROLLUP(CP)
ORDER BY CP
```
</details>

## Practice 2
Create a table that shows the total confirmed + probable deaths for each county. Include a row that has the total for Michigan. Only include counties that have a death.

Expand the table above by CP status. What is different in the table from the result above?

## REPLACE
To replace values within a column: 
```SQL
REPLACE(column, substring_to_replace, replacement_substring)
```

<details>
  <summary>MySQL</summary>

```SQL
SELECT REPLACE(County, "Washtenaw", "WTW")
FROM Covid
```
</details>

<details>
  <summary>PostgreSQL</summary>

```SQL
UPDATE 
	Covid 
SET
	County = REPLACE(County, 'Washtenaw', 'WTW');
SELECT * FROM Covid;
```
</details>

## WINDOW FUNCTIONS

A window function performs an aggregate-like operation on a set of query rows. However, whereas an aggregate operation groups query rows into a single result row, a window function produces a result for each row.

```SQL
SELECT *,
	SUM(Cases) OVER() AS Total
FROM Covid
WHERE County = 'Genesee'
```
**Note**: MySQL requires single quotes around the alias like so `SUM(Cases) OVER() AS 'Total'`

### OVER
You can use aggregate functions as window functions. They require the `OVER` clause to specify whether it is a window function or not. Some examples we saw in the Intro Class are:
```
SUM()
COUNT()
MIN()
MAX()
AVG()
```

You can also use non-aggregate functions that are used as window functions. They always require the `OVER` clause.
```
CUME_DIST()
DENSE_RANK()
FIRST_VALUE()
LAG()
LAST_VALUE()
LEAD()
NTH_VALUE()
NTILE()
PERCENT_RANK()
RANK()
ROW_NUMBER()
```

### Non-Aggregate Function Example: RANK
If you want to rank rows, use the `RANK() OVER` function
We use `ORDER BY` to determine the rank order

```SQL
SELECT County, 
	Cases, 
	RANK() OVER (ORDER BY Cases) AS Rank
FROM Covid
WHERE Day IN ('2020-06-01','2020-09-25') AND CP = 'Confirmed'
```
**Note**: MySQL requires single quotes around the alias like so `RANK() OVER (ORDER BY Cases) AS 'Rank'`

### DENSE_RANK
Notice ties have the same rank. If you don't want the ranking to skip any numbers, use `DENSE_RANK` instead

```SQL
SELECT County, 
	Cases, 
	DENSE_RANK() OVER (ORDER BY Cases DESC) AS Rank
FROM Covid
WHERE Day IN ('2020-06-01','2020-09-25') AND CP = 'Confirmed'
```

### WINDOW
Alternatively use the `WINDOW` syntax
Use case: when you use the same window more than once

```SQL
SELECT County, 
	Cases,
	RANK() OVER w AS Rank,
	DENSE_RANK() OVER w AS Rank2
FROM Covid
WHERE Day IN ('2020-06-01','2020-09-25') AND CP = 'Confirmed'
WINDOW w AS (ORDER BY Cases DESC)
```

### PARTITION BY
Works like `GROUP BY` but with the `OVER` clause
If we wanted to have the ranking be within a single day, we can use `PARTITION BY`

```SQL
SELECT Day, 
	County, 
	Cases, 
	RANK() OVER (PARTITION BY Day ORDER BY Cases) AS Rank
FROM Covid
WHERE Day IN ('2020-06-01','2020-09-25') AND CP = 'Confirmed'
```

## Practice 3
Rank county confirmed cases for each day in the last week (Sep 24-30th). Only rank counties starting with S. (i.e for each day, rank the “S” counties). Spell out the word “St” where applicable.

Create a variable that has the number of confirmed cases from a week ago for Wayne county. Show the most recent dates on top (Hint: Use the `LAG()` function).

## SUBQUERIES
In the intro workshop, we saw how subqueries worked. For example,

```SQL
SELECT *, RANK() OVER (ORDER BY DeathTotal DESC) as Rank
FROM (
	SELECT Day, SUM(Deaths) AS DeathTotal
	FROM Covid
	WHERE county = 'Wayne'
	GROUP BY Day
) AS Alex
```

## CTE (Common Table Expression)
The `WITH` clause allows you separate your subqueries to make your code more modular and easier to read. It is also known as a **Common Table Expression (CTE)** or subquery factoring.

```SQL
WITH cte AS (
  SELECT Day, SUM(Deaths) AS DeathTotal
  FROM Covid
  WHERE county = 'Wayne'
  GROUP BY Day
)

SELECT *, RANK() OVER (ORDER BY DeathTotal DESC) as Rank
FROM cte
```

Use a comma and omit the subsequent `WITH` keyword for multiple CTEs.
```SQL
WITH cte1 AS (
...
),
cte2 AS (
...
)

SELECT *
FROM cte1
JOIN cte2
ON cte1.pk = cte2.pk
```

## Scalar Subquery
A scalar subquery is an ordinary `SELECT` query in parentheses that returns exactly **one row with one column**.
```SQL
SELECT name
	,(SELECT max(pop) FROM cities WHERE cities.state = states.name)
FROM states;
```

## Practice 4
Return a table of Michigan cases (confirmed and probable) showing the max daily numbers for each calendar week (Hint: Use the `WEEK()` function and/or the `EXTRACT()` function).

This dataset
```CSV
Week,Day,Cases
1,Sun,10
1,Mon,20
2,Sun,30
2,Mon,40
```
Should be transformed into this
```
Week,Max Daily Cases
1,20
2,40
```

## LATERAL JOIN
`Lateral` joins allow you to reference table columns in the preceding left table of the join. They serve as an alternative to using CTEs. Looks like window functions do not work with lateral joins since they require more than one row to operate properly.

```SQL
SELECT orders.orderid
	,statuslow
	,statushigh 
FROM orders
JOIN LATERAL (
	SELECT orderlinestatus.name AS statuslow
        ,orderlines.orderid 
	FROM orderlines 
	JOIN orderlinestatus USING (orderlinestatusid)
	WHERE orderlines.orderid = orders.orderid 
	GROUP BY orderlines.orderid, orderlinestatus.name, orderlinestatus.weight
	ORDER BY orderlinestatus.weight LIMIT 1
) AS subquery_statuslow ON (subquery_statuslow.orderid=orders.orderid),
JOIN LATERAL (
	SELECT orderlinestatus.name AS statushigh
	,orderlines.orderid 
	FROM orderlines 
	JOIN orderlinestatus USING (orderlinestatusid)
	WHERE orderlines.orderid = orders.orderid 
	GROUP BY orderlines.orderid, orderlinestatus.name, orderlinestatus.weight
	ORDER BY orderlinestatus.weight DESC LIMIT 1
) AS subquery_statushigh ON (subquery_statushigh.orderid=orders.orderid)
```
Reference: https://www.davici.nl/blog/postgresql-lateral-join

## Nested SELECT
```SQL
SELECT
  salesperson.name,
  -- find maximum sale size for this salesperson
  (SELECT MAX(amount) AS amount
    FROM all_sales
    WHERE all_sales.salesperson_id = salesperson.id)
  AS amount,
  -- find customer for this maximum size
  (SELECT customer_name
    FROM all_sales
    WHERE all_sales.salesperson_id = salesperson.id
    AND all_sales.amount =
         -- find maximum size, again
         (SELECT MAX(amount) AS amount
           FROM all_sales
           WHERE all_sales.salesperson_id = salesperson.id))
  AS customer_name
FROM
  salesperson;
```
Reference: https://dev.mysql.com/doc/refman/8.0/en/lateral-derived-tables.html

## CROSS JOIN
Performs a cartesian product of the two tables (combination of the two table rows). The result will be `rows(table1) * rows(table2)`
```SQL
SELECT source, nombre
FROM suits
CROSS JOIN locale
```
OR equivalently you can the comma notation
```SQL
SELECT source, nombre
FROM suits, locale
```

## SELF JOIN
Join a table with itself
```SQL
SELECT a.name, b.name, a.state
FROM company a, company b
WHERE a.state=b.state AND a.name != b.name
```
Reference: https://www.w3resource.com/sql/joins/join-a-table-to-itself.php

## USING and NATURAL
Here is a standard join statement
```SQL
SELECT customers.*, orders.*
FROM customers
JOIN orders ON customers.custid = orders.custid
```
`USING` with parentheses `()` can be used when the column names are identical in each table (explicit)
```SQL
SELECT customers.*, orders.*
FROM customers
JOIN orders USING (custid)
```
`NATURAL` can also be used when the column names are identical in each table (implicit)
```SQL
SELECT customers.*, orders.*
FROM customers
NATURAL JOIN orders
```
Reference: https://www.postgresql.org/docs/10/queries-table-expressions.html

## UNNEST
Equivalent to pandas `explode` method expanding an array into a set of rows.
```SQL
SELECT log_name
	,unnest(edge_id) AS edgeid
	,unnest(lat) AS lat
	,unnest(lon) AS lon
	,unnest(autonomy_count) AS auto
	,unnest(healthy_count) AS healthy
FROM vehicle_autonomy_edges AS vae
```

OR if you want to add the array index number to the result
```SQL
SELECT vae.log_name
	,a.edgeid
	,a.lat
	,a.lon
	,a.auto
	,a.healthy
	,a.nr
FROM vehicle_autonomy_edges AS vae
CROSS JOIN LATERAL UNNEST(vae.edge_id, vae.lat, vae.lon, vae.autonomy_count, vae.healthy_count) WITH ORDINALITY AS a(edgeid, lat, lon, auto, healthy, nr)
```
Reference: https://www.postgresql.org/docs/10/functions-array.html
https://stackoverflow.com/questions/8760419/postgresql-unnest-with-element-number

## EXPLAIN
Profiling queries for bottlenecks and optimization
```SQL
EXPLAIN SELECT COUNT(*)
FROM Covid
```
```SQL
EXPLAIN (ANALYZE, COSTS, VERBOSE, BUFFERS, FORMAT JSON) SELECT COUNT(*)
FROM Covid
```
Reference: https://www.postgresql.org/docs/10/sql-explain.html


## Miscellaneous Commands

## PostgreSQL
### Show Postgres Version
```SQL
SELECT VERSION()
```

### List all public tables in database
```SQL
SELECT *
FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY table_name;
```
Other `table_schema` types are `information_schema` and `pg_catalog`.

Reference: https://www.postgresql.org/docs/10/infoschema-tables.html

Alternatively,
```SQL
SELECT *
FROM pg_catalog.pg_tables
WHERE schemaname != 'pg_catalog' AND schemaname != 'information_schema';
```
Reference: https://www.postgresql.org/docs/10/view-pg-tables.html

### List table schema
```SQL
SELECT table_name
	,column_name
	,data_type 
FROM information_schema.columns
WHERE table_name = 'vehicle_logs';
```
Reference: https://www.postgresql.org/docs/10/infoschema-columns.html

### Create materialized view
```SQL
CREATE MATERIALIZED VIEW matview AS (
	SELECT *
	FROM table
)
```

### List materialized views
```SQL
SELECT *
FROM pg_matviews
```

### Create Role and User for Read Only
```SQL
-- Create a group
CREATE ROLE readaccess;

-- Grant access to existing tables
GRANT USAGE ON SCHEMA public TO readaccess;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readaccess;

-- Grant access to future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readaccess;

-- Create a final user with password
CREATE USER alex WITH PASSWORD 'cao';
GRANT readaccess TO alex;
```

Alternatively for tables, sequences, functions. Ref: https://wiki.postgresql.org/images/d/d1/Managing_rights_in_postgresql.pdf
```SQL
CREATE ROLE readonly LOGIN PASSWORD 'some_pass';
-- Existing objects
GRANT CONNECT ON DATABASE the_db TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;

GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA public TO readonly;
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO readonly;
-- New objects
ALTER DEFAULT PRIVILEGES FOR ddl_user IN SCHEMA public GRANT SELECT ON TABLES TO readonly;
ALTER DEFAULT PRIVILEGES FOR ddl_user IN SCHEMA public GRANT SELECT ON SEQUENCES TO readonly;
ALTER DEFAULT PRIVILEGES FOR ddl_user IN SCHEMA public GRANT EXECUTE ON FUNCTIONS TO readonly;
```

### Create Index
```SQL
CREATE INDEX name_idx ON table (column);
```

### List Indexes in Database
```SQL
SELECT *
FROM pg_indexes
WHERE schemaname = 'public'
```
Reference: https://www.postgresql.org/docs/10/view-pg-indexes.html
