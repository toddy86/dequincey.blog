---
layout: post
title: SQL Cheat Sheet
categories: resources
---

A library of SQL notes and code snippets.

<!-- more -->
## **General Overview**

***References***  
The following are notes from a variety of sources, including, but not limited to:
* [Mode SQL Tutorial](https://mode.com/resources/sql-tutorial/)

The intellectual property of the below notes and resources belong to the sources which they were derived. 

## SQL

### Basics

#### Query Clause Order

```
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
```

#### HAVING

```
SELECT year,
       month,
       MAX(high) AS month_high
FROM table
GROUP BY year, month
HAVING MAX(high) > 400
ORDER BY year, month
```

HAVING is the “clean” way to filter a query that has been aggregated, but this is also commonly done using a subquery.

Note: You need to re-state the aggregation (e.g. MAX(col), COUNT(*) etc).

#### LIKE

```
SELECT *
FROM table
WHERE "group" LIKE 'Snoop%'
```

* % is a wildcard value
* Instead of using the single quotes, the double quotes around "group" are to indicate that "group" is the name of a column and not the group function in SQL. 

### Union, Intersect and Except

![alt text](https://i.stack.imgur.com/phA2q.jpg)

### Joins

![alt text](https://i.stack.imgur.com/1Tfy0.jpg)

### Sub-Queries

#### Overview

* Also known as inner or nested queries
* Breaks down a large SQL statement into individual steps which are run sequentially (i.e. the inner query is run first)
* Inner queries **MUST** be valid SQL queries, as the database will treat it like an independent query
* Once the inner queries have run, the results of that query are then used
* Sub-queries **MUST** be named

#### Aggregation in multiple stages

```
SELECT LEFT(sub.date, 2) AS cleaned_month,
       sub.day_of_week,
       AVG(sub.incidents) AS average_incidents  # Aggregation of COUNT from inner query
  FROM (
        SELECT day_of_week,
               date,
               COUNT(incidnt_num) AS incidents
          FROM tutorial.sf_crime_incidents_2014_01
         GROUP BY 1,2
       ) sub  # Inner queries must be named
 GROUP BY 1,2
 ORDER BY 1,2
 ```

#### Conditional Logic

Sub-queries can be used in conditional logic with WHERE, JOIN/ON or CASE.

Sub-query selecting a single cell.

```
SELECT *
FROM table
WHERE Date = (
              SELECT MIN(date)
              FROM table
              )
```

Must use IN if our sub-query returns multiple results.

```
SELECT *
FROM table
WHERE Date IN (
                SELECT date
                table
                ORDER BY date
                LIMIT 5
              )
```

Note you do **NOT** include an alias when you write a subquery in a conditional statement. This is because the subquery is treated as an individual value (or set of values in the IN case) rather than as a table.

#### Joining Sub-Queries

Write a query that displays all rows from the three categories with the fewest incidents reported.

Using a sub-query in the WHERE clause can add complexity, as only one result can be returned for use in the WHERE. 
```
SELECT *
FROM table
WHERE category IN (
    SELECT 
      sub.category
    FROM (
          SELECT 
            category, 
            COUNT(category) as counter
          FROM table
          GROUP BY category
          ORDER BY counter
          LIMIT 3
          ) sub
  )
```

Alternatively, the same can be achieved with less code using the INNER JOIN function.

Using an INNER JOIN is particularly useful when combined with aggregations, as multiple columns can be returned.

```
SELECT * 
FROM table as main
INNER JOIN (
          SELECT
            category, 
            COUNT(category) as counter
          FROM table
          GROUP BY category
          ORDER BY counter
          LIMIT 3
) sub
ON main.category = sub.category
```

Further, using the INNER JOIN allows us to use the results of the inner query in the outer query. 

```
SELECT * 
FROM table as main
INNER JOIN (
          SELECT
            category, 
            COUNT(category) as counter
          FROM table
          GROUP BY category
          ORDER BY counter
          LIMIT 3
) sub
ON main.category = sub.category
ORDER BY sub.counter  -- Results can be ordered by the inner query results
```

#### Sub-queries and UNION

```
SELECT COUNT(*) AS total_rows
  FROM (
        SELECT *
          FROM table_part1

         UNION ALL

        SELECT *
          FROM table_part2
       ) sub
```

### Dates & Times

#### Extracting Dates

To extract dates and times from a datetime field, use the MySQL EXTRACT option.

```
 SELECT 
        EXTRACT('year' FROM date) AS year,
        EXTRACT('quarter' FROM date) AS quarter,
        EXTRACT('month' FROM date) AS month,
        EXTRACT('week' FROM date) AS week,
        EXTRACT('day' FROM date) AS day,
        EXTRACT('hour' FROM date) AS hour,
        EXTRACT('minute' FROM date) AS minute,
        EXTRACT('second' FROM date) AS second,
        EXTRACT('day_hour' FROM date) AS day_hour,
        EXTRACT('year_month' FROM date) AS year_month
     
FROM table_name
 ```

#### Calculating Intervals

```
SELECT name,
       department,
       start_date::timestamp + INTERVAL '1 week' AS plus_one_week,
       NOW() - start_date::timestamp AS start_time_ago
  FROM students
```

The interval is defined using plain-English terms like ‘10 seconds’ or ‘5 months’. 

Adding or subtracting a date column and an interval column results in another date column.

NOW() gets the current datetime.

#### Current Date & Time

```
SELECT CURRENT_DATE AS date,
       CURRENT_TIME AS time,
       CURRENT_TIMESTAMP AS timestamp,
       LOCALTIME AS localtime,
       LOCALTIMESTAMP AS localtimestamp,
       NOW() AS now
```

The above calculates the current date/time based on the timezone of the server. 

The below specifies the datetime to use when calculating.

```
SELECT 
    CURRENT_TIME AS time,
    CURRENT_TIME AT TIME ZONE 'PST' AS time_pst
```

#### CAST

```
SELECT incidnt_num,
       date,
       CAST((SUBSTR(date, 7, 4) || '-' || LEFT(date, 2) ||
        '-' || SUBSTR(date, 4, 2)) AS date) AS cleaned_date
FROM table
```
OR 
```
SELECT incidnt_num,
       date,
       CAST(
            (
            CONCAT
                  (
                  SUBSTR(date, 7, 4), 
                  '-', 
                  LEFT(date, 2),
                  '-',
                  SUBSTR(date, 4, 2)
                  )
            ) AS date
        ) AS cleaned_date
FROM table
```


Convert an incorrectly formatted date string into the correct format and then cast it as a date.

**Cast Operators**  
* ::date
* ::datetime
* ::time
* ::char

#### DATE_TRUNC

```
SELECT
    DATE_TRUNC('week', cleaned_date)::date as week,
    COUNT(incidnt_num) as incid_count
FROM table
GROUP BY week
```

DATE_TRUNC allows you to round dates to the nearest unit of measurement. 

The display value is the first value in the period (i.e. the first day of the week). 

```
SELECT cleaned_date,
       DATE_TRUNC('year', cleaned_date) AS year,
       DATE_TRUNC('month', cleaned_date) AS month,
       DATE_TRUNC('week', cleaned_date) AS week,
       DATE_TRUNC('day', cleaned_date) AS day,
       DATE_TRUNC('hour', cleaned_date) AS hour,
       DATE_TRUNC('minute', cleaned_date) AS minute,
       DATE_TRUNC('second', cleaned_date) AS second,
       DATE_TRUNC('decade', cleaned_date) AS decade
FROM table
```

### Strings

Most of the functions presented in this lesson are specific to certain data types. However, using a particular function will, in many cases, change the data to the appropriate type. LEFT, RIGHT, and TRIM are all used to select only certain elements of strings, but using them to select elements of a number or date will treat them as strings for the purpose of the function.

#### LEFT, RIGHT, LENGTH & SUBSTR

**LEFT**  

Left will get the n number of characters starting from the left.
```
SELECT incidnt_num,
       date,
       LEFT(date, 10) AS cleaned_date
FROM table
```

**RIGHT & LENGTH**  

Right will get the n number of characters starting from the right. Right is often combined with LENGTH to set a start and end point. 

```
SELECT incidnt_num,
       date,
       LEFT(date, 10) AS cleaned_date,
       RIGHT(date, LENGTH(date) - 11) AS cleaned_time
FROM table
```

**SUBSTR**  
Instead of using RIGHT & LENGTH to get a mid-start posint, can use SUBSTR.

```
SELECT incidnt_num,
       date,
       SUBSTR(date, 4, 2) AS day
FROM table
```

#### TRIM

The TRIM function is used to remove characters from the beginning and end of a string. 

```
SELECT location,
       TRIM(both '()' FROM location)
FROM table
```

Takes 3 args:  
* You have to specify whether you want to remove characters from the beginning (‘leading’), the end (‘trailing’), or both
* Specify all characters to be trimmed. Any characters included in the single quotes will be removed from both beginning, end, or both sides of the string.
* Must specify the string to trim

#### POSITION & STRPOS

```
SELECT incidnt_num,
       descript,
       POSITION('A' IN descript) AS a_position
FROM table
  
SELECT incidnt_num,
       descript,
       STRPOS(descript, 'A') AS a_position
FROM table
```

* POSITION and STRPOS allows you to specify a substring, then returns a numerical value equal to the character number (counting from left) where that substring first appears. 
* Indexing starts at 1
* Case sensitive

#### CONCAT

```
SELECT incidnt_num,
       day_of_week,
       LEFT(date, 10) AS cleaned_date,
       CONCAT(day_of_week, ', ', LEFT(date, 10)) AS day_and_date
FROM table
```

Alternatively, you can also use 2x pipes to do a concatenation.
```
SELECT incidnt_num,
       day_of_week,
       LEFT(date, 10) AS cleaned_date,
       day_of_week || ', ' || LEFT(date, 10) AS day_and_date
FROM table
```

#### UPPER & LOWER

```
SELECT incidnt_num,
       address,
       UPPER(address) AS address_upper,
       LOWER(address) AS address_lower
FROM table

```

#### COALESCE

```
SELECT incidnt_num,
       descript,
       COALESCE(descript, 'No Description')
FROM table
ORDER BY descript DESC
```

If a dataset contains nulls that you’d prefer to contain actual values, use COALESCE to cocerce null into the desired value (e.g. 0). 


## Resources

#### SQL
**Books** 
* [Effective SQL: 61 Specific Ways to Write Better SQL](https://www.amazon.co.uk/Effective-SQL-Specific-Software-Development/dp/0134578899)

**Web Resources**
* [Mode: The SQL Tutorial for Data Analysis](https://mode.com/sql-tutorial/introduction-to-sql/)
* 