# sql-intermediate

## More SQL Syntax

## IF
<details>
  <summary>MySQL</summary>
Similar to the excel functionality for IF
  
`IF(expression, true expression, false expression)`

```SQL
SELECT County, 
IF (LENGTH(County) < 8, 'short', 'long') AS StringLength
FROM Covid
```
</details>

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
WITH cte AS
(
  SELECT Day, SUM(Deaths) AS DeathTotal
  FROM Covid
  WHERE county = 'Wayne'
  GROUP BY Day
)

SELECT *, RANK() OVER (ORDER BY DeathTotal DESC) as Rank
FROM cte
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
