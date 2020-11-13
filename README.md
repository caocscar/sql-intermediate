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
<details>
  <summary>MySQL and PostgreSQL</summary>
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
</details>

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

Example: 
```SQL
SELECT REPLACE(County, "Washtenaw", "WTW")
FROM Covid
```
</details>

<details>
  <summary>PostgreSQL</summary>

Example: 
```SQL
UPDATE 
	Covid 
SET
	County = REPLACE(County, 'Washtenaw', 'WTW');
SELECT * FROM Covid;
```
</details>

## WINDOW FUNCTIONS


