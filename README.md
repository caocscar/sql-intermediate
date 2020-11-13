# sql-intermediate

## More SQL Syntax

## IF
<details>
  <summary>MySQL</summary>
Similar to the excel functionality for IF
  
`IF(expression, true expression, false expression)`

```
SELECT County, 
IF (LENGTH(County) < 8, 'short', 'long') AS StringLength
FROM Covid
```
</details>

## CASE
<details>
  <summary>MySQL and PostgreSQL</summary>
Used to convert a continuous variable into a categorical or ordinal variable. It is equivalent to if else if statements or switch case statements in programming.

```
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
