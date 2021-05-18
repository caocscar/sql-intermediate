# Intermediate SQL Workshop <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->
- [SQL for Beginners](#sql-for-beginners)
- [DB Fiddle](#db-fiddle)
- [Dataset](#dataset)
- [SQL Order of Operations](#sql-order-of-operations)
- [Part I: More SQL Syntax](#part-i-more-sql-syntax)
- [Set Operations (UNION, INTERSECT, EXCEPT)](#set-operations-union-intersect-except)
- [IF](#if)
- [CASE](#case)
- [COALESCE](#coalesce)
- [Practice 1](#practice-1)
- [ROLLUP](#rollup)
- [Practice 2](#practice-2)
- [REPLACE](#replace)
- [FILTER](#filter)
- [EXTRACT](#extract)
- [TO_CHAR](#to_char)
- [Working with time zones](#working-with-time-zones)
- [WINDOW FUNCTIONS](#window-functions)
	- [OVER](#over)
	- [Non-Aggregate Function Example: RANK](#non-aggregate-function-example-rank)
	- [DENSE_RANK](#dense_rank)
	- [WINDOW](#window)
	- [PARTITION BY](#partition-by)
	- [RANGE (ROWS) BETWEEN](#range-rows-between)
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
- [UPDATE](#update)
- [ON CONFLICT DO UPDATE](#on-conflict-do-update)
- [EXPLAIN](#explain)
- [Miscellaneous Commands](#miscellaneous-commands)
- [PostgreSQL](#postgresql)
	- [Show Postgres Version](#show-postgres-version)
	- [List all public tables in database](#list-all-public-tables-in-database)
	- [List table schema](#list-table-schema)
	- [Get Table Size](#get-table-size)
	- [Get Database Size](#get-database-size)
	- [Get TOAST information](#get-toast-information)
	- [Create materialized view](#create-materialized-view)
	- [List materialized views](#list-materialized-views)
	- [Create Role and User for Read Only](#create-role-and-user-for-read-only)
	- [Create Index](#create-index)
	- [List Indexes in Database](#list-indexes-in-database)
	- [Toast (The Oversized-Attribute Storage Technique)](#toast-the-oversized-attribute-storage-technique)
- [Types of SQL Commands](#types-of-sql-commands)
	- [DDL](#ddl)
	- [DQL](#dql)
	- [DML](#dml)
	- [DCL](#dcl)
	- [TCL](#tcl)
- [PSQL](#psql)
	- [Connecting from the terminal](#connecting-from-the-terminal)
	- [Common psql Commands](#common-psql-commands)
	- [Queries](#queries)
	- [Changing Autovacuum Settings (Table Storage Parameters)](#changing-autovacuum-settings-table-storage-parameters)
	- [Reset Autovacuum Settings (Table Storage Parameters)](#reset-autovacuum-settings-table-storage-parameters)

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
4. aggregate functions
5. HAVING
6. window functions
7. SELECT
8. DISTINCT
9. set operations
10. ORDER BY
11. OFFSET
12. LIMIT

## Part I: More SQL Syntax

## Set Operations (UNION, INTERSECT, EXCEPT)
These three set operations cover the union (`UNION`), intersection (`INTERSECT`) and difference (`EXCEPT`) between two sets (i.e. queries). Recall, from part I, that the queries must have the same number of columns and data types. Duplicates are removed unless `ALL` is used too.

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

## FILTER
If we wanted to bin and count student grades, we could use a CTE with `CASE`.
```SQL
WITH cte AS (
SELECT *,
	CASE
    	WHEN Marks BETWEEN 40 AND 60 THEN 'C'
		WHEN Marks BETWEEN 60 AND 80 THEN 'B'
		WHEN Marks BETWEEN 80 AND 100 THEN 'A'
        ELSE 'F'
    END AS grades
FROM students
)
SELECT grades, count(*) as ct
FROM cte
GROUP BY grades
```
Alternatively, we can use the `FILTER` clause with `COUNT`. I also included a scalar subquery as another alternative.
```SQL
SELECT COUNT(*) AS total 
	,COUNT(*) FILTER (WHERE Marks BETWEEN 40 AND 59) AS C
	,COUNT(1) FILTER (WHERE Marks BETWEEN 60 AND 79) AS B
	,(SELECT COUNT(*) FROM students WHERE Marks BETWEEN 80 AND 100) AS A
FROM students;
```

## EXTRACT
Retrieves a datetime field (e.g. year, day) from a `timestamp`, `time`, or `interval` datatype. The example shows the difference in seconds between two dates.
```SQL
SELECT EXTRACT(EPOCH FROM TIMESTAMP '2021-04-17 15:45:14' - TIMESTAMP '2021-04-16 15:45:14')
SELECT EXTRACT(EPOCH FROM '2021-04-26 08:55:15'::timestamp - '2021-04-16 15:45:14'::timestamp)
```
Reference: https://www.postgresql.org/docs/10/functions-datetime.html

## TO_CHAR
Format `timestamp` as a string
```SQL
SELECT TO_CHAR('2021-04-26 18:34:56' AT TIME ZONE 'America/Chicago', 'YYYY-MM-DD HH24:MI:SS')
```

## Working with time zones
Cast `timestamp with time zone` to local time `timestamp without time zone`
```SQL
SELECT utcz_time AT TIME ZONE 'America/Detroit' AS local_time
```

To convert `timestamp without time zone` to `timestamp with time zone` (back to UTC)
```SQL
SELECT local_time AT TIME ZONE 'America/Detroit'
```

Cast `timestamp with time zone` to `timestamp without time zone` without changing the time value
```SQL
SELECT utcz_time::timestamp
```

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

### RANGE (ROWS) BETWEEN
Suppose we wanted to get the most recent number of daily cases per county. We could filter by date and show the cases for the most recent day but what if some counties did not report cases the day. We would want the most recent data they had. We could use the `LAST_VALUE` window function in conjunction with `RANGE BETWEEN` to do so. The latter function defines the *frame clause* in the partition. Here it goes from the first row to the last row because of the `UNBOUNDED` syntax.
```SQL
With cte AS (
SELECT LAST_VALUE(Cases) OVER (PARTITION BY County ORDER BY Day ASC RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS recent_cases
FROM Covid
)
```
Here the frame_clause is `RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`. It usually takes the form of  
`{ RANGE | ROWS } BETWEEN frame_start AND frame_end`  
OR  
`{ RANGE | ROWS } frame_start`  
The default frame clause is `RANGE UNBOUNDED PRECEDING`, which is the same as `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`

If you wanted to create a window that included the past 5 entries for a 5-day moving average, for example, your frame clause would be `ROWS BETWEEN 4 PRECEDING AND CURRENT ROW`.  Likewise, if you wanted to create a window for a median filter of size 7, your syntax would be
`ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING`

Reference: https://www.postgresql.org/docs/10/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS

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
The `WITH` clause allows you separate your subqueries to make your code more modular and easier to read. It is also known as a **Common Table Expression (CTE)** or subquery factoring. It is a temporary result set that you can reference within another SELECT, INSERT, UPDATE, or DELETE statement

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

## UPDATE
To update an existing table and return updated rows
```SQL
UPDATE vehicle_logs
SET start_time::timestamp AT TIME ZONE 'America/Chicago'
WHERE RIGHT(log_name, 8) = TO_CHAR(start_time, 'HH24-MI-SS')
RETURNING *
```

## ON CONFLICT DO UPDATE
The idea is that when you insert a new row into the table, PostgreSQL will update the row if it already exists, otherwise, it will insert the new row. The `UPDATE` or `INSERT` operation is a.k.a. **UPSERT**. The examples specifies that if the `date` is suppose to be constrained or unique in the table, then literally do nothing.
```SQL
INSERT INTO Covid (date, Cases)
VALUES('2021-04-01', '6036') 
ON CONFLICT (date) 
DO NOTHING;
```
Alternatively, we can update the value (by prepending the new value in this case)
```SQL
INSERT INTO Covid (date, Cases)
VALUES('2021-04-01', '6036') 
ON CONFLICT (date) 
DO UPDATE
SET Cases = EXCLUDED.Cases || ' (' || Covid.Cases || ')';
```
`(date)` is considered the conflict target and performs all conflicts with usable constraints and unique indexes.

Reference: https://www.postgresql.org/docs/10/sql-insert.html

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
SELECT *
FROM information_schema.columns
WHERE table_name = 'vehicle_logs';
```
Reference: https://www.postgresql.org/docs/10/infoschema-columns.html

### Get Table Size
Query the total size of each table (including indexes and toast tables) in the current database (use `pg_relation_size` to exclude indexes (`pg_indexes_size`) and toast tables)
```SQL
SELECT relname AS relation
	,pg_size_pretty(pg_total_relation_size(C.oid)) AS total_size
FROM pg_class C
LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
WHERE nspname NOT IN ('pg_catalog', 'information_schema')
AND C.relkind <> 'i'
AND nspname !~ '^pg_toast'
ORDER BY pg_total_relation_size (C.oid) DESC
```
Reference: https://www.postgresql.org/docs/10/functions-admin.html
https://stackoverflow.com/questions/41991380/whats-the-difference-between-pg-table-size-pg-relation-size-pg-total-relatio

### Get Database Size
Query the size of each database in the current database server
```SQL
SELECT pg_database.datname
	,pg_size_pretty(pg_database_size(pg_database.datname)) AS size
FROM pg_database;
```

Reference: https://www.postgresqltutorial.com/postgresql-database-indexes-table-size/

### Get TOAST information
Query to find out all tables with TOAST tables
```SQL
SELECT oid
	,oid::regclass
	,reltoastrelid
    ,reltoastrelid::regclass
    ,pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_size
FROM pg_class
WHERE relkind = 'r'
AND reltoastrelid <> 0
ORDER BY toast_size DESC;
```
Reference: https://www.postgresql.org/docs/9.3/catalog-pg-class.html
https://dba.stackexchange.com/questions/264691/understanding-where-pg-toast-is-coming-from
https://stackoverflow.com/questions/41991380/whats-the-difference-between-pg-table-size-pg-relation-size-pg-total-relatio

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

### Toast (The Oversized-Attribute Storage Technique)
PostgreSQL uses a fixed page size (commonly 8 kB), and does not allow tuples to span multiple pages. Therefore, it is not possible to store very large field values directly. To overcome this limitation, large field values are compressed and/or broken up into multiple physical rows. The technique is affectionately known as TOAST. The TOAST infrastructure is also used to improve handling of large data values in-memory.

Reference: https://www.postgresql.org/docs/10/storage-toast.html

## Types of SQL Commands
- DDL - Data Definition Language
- DQL - Data Query Language
- DML - Data Manipulation Language
- DCL - Data Control Language
- TCL - Transaction Control Language

### DDL
Consists of the SQL commands that can be used to define the database schema. It simply deals with descriptions of the database schema and is used to create and modify the structure of database objects in the database.
- CREATE
- ALTER
- DROP
- RENAME
- TRUNCATE
- COMMENT

### DQL
DQL statements are used for performing queries on the data within schema objects. The purpose of DQL Command is to get some schema relation based on the query passed to it.
- SELECT

### DML
The SQL commands that deals with the manipulation of data present in the database belong to DML or Data Manipulation Language and this includes most of the SQL statements.
- INSERT
- UPDATE
- DELETE
- MERGE
- CALL
- EXPLAIN PLAN
- LOCK TABLE

### DCL
DCL mainly deals with the rights, permissions and other controls of the database system.
- GRANT
- REVOKE

### TCL
TCL commands deals with the transaction within the database.
- COMMIT
- ROLLBACK
- SAVEPOINT
- SET TRANSACTION
  
Reference: https://www.geeksforgeeks.org/sql-ddl-dql-dml-dcl-tcl-commands/

## PSQL
**psql** is a terminal-based front-end to PostgreSQL. You can use a local instance of the psql command line utility to connect to a PostgreSQL DB instance. You need either PostgreSQL or the psql client installed on your client computer.

### Connecting from the terminal
```Shell
psql --host=<hostname> --username=<user> --password --dbname=<database-name> --port=5432 
psql -h <hostname> -U <user> -W -d <database-name> --port=5432
```
Password prompt will appear after command is entered.

### Common psql Commands
Command|Description
---|---
`\l`|Show databases
`\c <database>`|Switch to different database closing previous connection
`\dt`|Show tables in database
`\dt+`|Show tables in database with more info
`\d+`|Describe a table with details
`\du`|Show users and their roles
`\g`|Execute previous command
`\s`|Command history
`\i <filename>`|Execute psql commands from filename
`\?`|Help
`\h <COMMAND>`|Specific Help
`\timing`|Turn on query execution time
`\q`|Quit psql

REFERENCE: https://www.postgresqltutorial.com/psql-commands/

### Queries
You can write multi-line queries directly from the prompt. Terminate lines with <kbd>Enter</kbd>. Terminate queries with a `;`.

### Changing Autovacuum Settings (Table Storage Parameters)
```SQL
ALTER TABLE vehicle_dash_image
SET (
	autovacuum_vacuum_threshold=120
	,autovacuum_vacuum_cost_delay=20
)
```

### Reset Autovacuum Settings (Table Storage Parameters)
```SQL
ALTER TABLE vehicle_dash_image
RESET (
	autovacuum_vacuum_threshold
	,autovacuum_vacuum_cost_delay
)
```

References:
- https://www.postgresql.org/docs/10/runtime-config-autovacuum.html
- https://www.postgresql.org/docs/10/runtime-config-resource.html#RUNTIME-CONFIG-RESOURCE-VACUUM-COST
