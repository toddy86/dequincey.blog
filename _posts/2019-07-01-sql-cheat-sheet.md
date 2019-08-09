---
layout: post
title: SQL Cheat Sheet
categories: resources
---

A library of SQL notes and code snippets.

<!-- more -->
<h1>Table of Contents<span class="tocSkip"></span></h1>
<div class="toc"><ul class="toc-item"><li><span><a href="#Basics" data-toc-modified-id="Basics-1"><span class="toc-item-num">1&nbsp;&nbsp;</span>Basics</a></span><ul class="toc-item"><li><span><a href="#Query-Clause-Order" data-toc-modified-id="Query-Clause-Order-1.1"><span class="toc-item-num">1.1&nbsp;&nbsp;</span>Query Clause Order</a></span></li><li><span><a href="#Order-of-operation" data-toc-modified-id="Order-of-operation-1.2"><span class="toc-item-num">1.2&nbsp;&nbsp;</span>Order of operation</a></span></li><li><span><a href="#HAVING" data-toc-modified-id="HAVING-1.3"><span class="toc-item-num">1.3&nbsp;&nbsp;</span>HAVING</a></span></li><li><span><a href="#LIKE" data-toc-modified-id="LIKE-1.4"><span class="toc-item-num">1.4&nbsp;&nbsp;</span>LIKE</a></span></li><li><span><a href="#CASE" data-toc-modified-id="CASE-1.5"><span class="toc-item-num">1.5&nbsp;&nbsp;</span>CASE</a></span></li></ul></li><li><span><a href="#Union,-Intersect-and-Except" data-toc-modified-id="Union,-Intersect-and-Except-2"><span class="toc-item-num">2&nbsp;&nbsp;</span>Union, Intersect and Except</a></span><ul class="toc-item"><li><span><a href="#UNION" data-toc-modified-id="UNION-2.1"><span class="toc-item-num">2.1&nbsp;&nbsp;</span>UNION</a></span></li></ul></li><li><span><a href="#Joins" data-toc-modified-id="Joins-3"><span class="toc-item-num">3&nbsp;&nbsp;</span>Joins</a></span><ul class="toc-item"><li><span><a href="#INNER-JOIN" data-toc-modified-id="INNER-JOIN-3.1"><span class="toc-item-num">3.1&nbsp;&nbsp;</span>INNER JOIN</a></span></li></ul></li><li><span><a href="#Sub-Queries" data-toc-modified-id="Sub-Queries-4"><span class="toc-item-num">4&nbsp;&nbsp;</span>Sub-Queries</a></span><ul class="toc-item"><li><span><a href="#Overview" data-toc-modified-id="Overview-4.1"><span class="toc-item-num">4.1&nbsp;&nbsp;</span>Overview</a></span></li><li><span><a href="#Aggregation-in-multiple-stages" data-toc-modified-id="Aggregation-in-multiple-stages-4.2"><span class="toc-item-num">4.2&nbsp;&nbsp;</span>Aggregation in multiple stages</a></span></li><li><span><a href="#Conditional-Logic" data-toc-modified-id="Conditional-Logic-4.3"><span class="toc-item-num">4.3&nbsp;&nbsp;</span>Conditional Logic</a></span></li><li><span><a href="#Joining-Sub-Queries" data-toc-modified-id="Joining-Sub-Queries-4.4"><span class="toc-item-num">4.4&nbsp;&nbsp;</span>Joining Sub-Queries</a></span></li><li><span><a href="#Sub-queries-and-UNION" data-toc-modified-id="Sub-queries-and-UNION-4.5"><span class="toc-item-num">4.5&nbsp;&nbsp;</span>Sub-queries and UNION</a></span></li></ul></li><li><span><a href="#Dates-&amp;-Times" data-toc-modified-id="Dates-&amp;-Times-5"><span class="toc-item-num">5&nbsp;&nbsp;</span>Dates &amp; Times</a></span><ul class="toc-item"><li><span><a href="#Extracting-Dates" data-toc-modified-id="Extracting-Dates-5.1"><span class="toc-item-num">5.1&nbsp;&nbsp;</span>Extracting Dates</a></span></li><li><span><a href="#Calculating-Intervals" data-toc-modified-id="Calculating-Intervals-5.2"><span class="toc-item-num">5.2&nbsp;&nbsp;</span>Calculating Intervals</a></span></li><li><span><a href="#Current-Date-&amp;-Time" data-toc-modified-id="Current-Date-&amp;-Time-5.3"><span class="toc-item-num">5.3&nbsp;&nbsp;</span>Current Date &amp; Time</a></span></li><li><span><a href="#CAST" data-toc-modified-id="CAST-5.4"><span class="toc-item-num">5.4&nbsp;&nbsp;</span>CAST</a></span></li><li><span><a href="#DATE_TRUNC" data-toc-modified-id="DATE_TRUNC-5.5"><span class="toc-item-num">5.5&nbsp;&nbsp;</span>DATE_TRUNC</a></span></li></ul></li><li><span><a href="#Strings" data-toc-modified-id="Strings-6"><span class="toc-item-num">6&nbsp;&nbsp;</span>Strings</a></span><ul class="toc-item"><li><span><a href="#LEFT,-RIGHT,-LENGTH-&amp;-SUBSTR" data-toc-modified-id="LEFT,-RIGHT,-LENGTH-&amp;-SUBSTR-6.1"><span class="toc-item-num">6.1&nbsp;&nbsp;</span>LEFT, RIGHT, LENGTH &amp; SUBSTR</a></span></li><li><span><a href="#TRIM" data-toc-modified-id="TRIM-6.2"><span class="toc-item-num">6.2&nbsp;&nbsp;</span>TRIM</a></span></li><li><span><a href="#POSITION-&amp;-STRPOS" data-toc-modified-id="POSITION-&amp;-STRPOS-6.3"><span class="toc-item-num">6.3&nbsp;&nbsp;</span>POSITION &amp; STRPOS</a></span></li><li><span><a href="#CONCAT" data-toc-modified-id="CONCAT-6.4"><span class="toc-item-num">6.4&nbsp;&nbsp;</span>CONCAT</a></span></li><li><span><a href="#UPPER-&amp;-LOWER" data-toc-modified-id="UPPER-&amp;-LOWER-6.5"><span class="toc-item-num">6.5&nbsp;&nbsp;</span>UPPER &amp; LOWER</a></span></li><li><span><a href="#COALESCE" data-toc-modified-id="COALESCE-6.6"><span class="toc-item-num">6.6&nbsp;&nbsp;</span>COALESCE</a></span></li></ul></li><li><span><a href="#Performance-Tuning" data-toc-modified-id="Performance-Tuning-7"><span class="toc-item-num">7&nbsp;&nbsp;</span>Performance Tuning</a></span><ul class="toc-item"><li><span><a href="#Reducing-Table-Size" data-toc-modified-id="Reducing-Table-Size-7.1"><span class="toc-item-num">7.1&nbsp;&nbsp;</span>Reducing Table Size</a></span></li><li><span><a href="#Making-joins-less-complicated" data-toc-modified-id="Making-joins-less-complicated-7.2"><span class="toc-item-num">7.2&nbsp;&nbsp;</span>Making joins less complicated</a></span></li><li><span><a href="#EXPLAIN" data-toc-modified-id="EXPLAIN-7.3"><span class="toc-item-num">7.3&nbsp;&nbsp;</span>EXPLAIN</a></span></li></ul></li></ul></div>

# SQL

***References***  
The following are notes from a variety of sources, including, but not limited to:
* [Mode SQL Tutorial](https://mode.com/resources/sql-tutorial/)
* [Effective SQL: 61 Specific Ways to Write Better SQL](https://www.amazon.co.uk/Effective-SQL-Specific-Software-Development/dp/0134578899)

The intellectual property of the below notes and resources belong to the sources which they were derived. 

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

#### Order of operation

!['Alt Text'](https://assets.website-files.com/589e47d231ee752554896f1f/59322015595b4e56313a0409_Screen%20Shot%202017-06-02%20at%207.32.53%20PM.png)

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

#### CASE

```
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    WHEN conditionN THEN resultN
    ELSE result
END
```

Used as an IF-THEN-ELSE statement for SQL

### Union, Intersect and Except

![alt text](https://i.stack.imgur.com/phA2q.jpg)

#### UNION

```
SELECT *
FROM table_part1

UNION  -- OR UNION ALL

SELECT *
FROM table_part2
```

* UNION appends one dataset to another
* UNION **excludes** any duplicate rows
* UNION ALL will keep any duplicate rows 
* Rules for appending data:  
** Both tables must have the **same number of columns** (The column names do NOT need to be the same, but often are)  
** The columns of both tables must have the **same data type**  and in the **same order**  


### Joins

![alt text](https://i.stack.imgur.com/1Tfy0.jpg)

#### INNER JOIN

Inner joins eliminate rows from both tables that do not satisfy the join condition set forth in the ON statement. In mathematical terms, an inner join is the intersection of the two tables.











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

### Performance Tuning

#### Reducing Table Size

Filtering the data to include only the observations you need can dramatically improve query speed. 

```
SELECT *
FROM table
WHERE 
    event_date >= '2014-03-01'
AND event_date <  '2014-04-01'
```

It can be good practice to do 'exploratory' SQL queries, to ensure that the results are correct before running the query on the full dataset. The full query may still take a long time to run, but you will have some assurance that the query results will be correct. 

**LIMIT** 
* LIMIT is a way to easily reduce the query size. 
* Note: **LIMIT does NOTHING for the speed of aggregations**, as the aggregation (e.g. COUNT) is performed BEFORE the LIMIT. 
* To effectively use LIMIT with aggregations, you must use LIMIT in a sub-query first.
* LIMIT is useful to test SQL query logic. But it will **NOT** produce accurate results. It should only be used in this context for improved query speed when checking logic.
* LIMIT should be used as early as possible in the query (i.e. in the sub-query, not the outer query).

```
SELECT 
    COUNT(*)
FROM (
    SELECT *
    FROM benn.sample_event_table
    LIMIT 100  -- This will speed up the COUNT aggregation
     ) sub
```

#### Making joins less complicated

It is better to reduce the table size **before** performing joins

**Slow join**  
```
SELECT teams.conference AS conference,
       players.school_name,
       COUNT(1) AS players
  FROM table1 as players
  JOIN table2 as teams
    ON teams.school_name = players.school_name
 GROUP BY conference, players.schoolname
``` 

**Faster join**  
```
SELECT teams.conference,
       sub.*
FROM (
    SELECT 
        players.school_name,
        COUNT(*) AS players
    FROM table1 as players
    GROUP BY players.school_name
      ) sub
JOIN table2 as teams
ON teams.school_name = sub.school_name
```

#### EXPLAIN

Add the key word **EXPLAIN** before a query to get a rough approximation as to how long the query will take to run, along with the query plan. 

```
EXPLAIN
SELECT teams.conference AS conference,
       players.school_name,
       COUNT(1) AS players
  FROM table1 as players
  JOIN table2 as teams
    ON teams.school_name = players.school_name
 GROUP BY conference, players.schoolname
```
