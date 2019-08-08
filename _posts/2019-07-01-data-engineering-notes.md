---
layout: post
title: Data Engineering Notes and Resources
categories: resources
---

A living and ever changing library of notes, resources useful technology and libraries for data engineering.

<!-- more -->
## **General Overview**

***References***  
The following are notes from a variety of sources, including, but not limited to:
* [A Beginners Guide to Data Engineering - Part 1](https://medium.com/@rchang/a-beginners-guide-to-data-engineering-part-i-4227c5c457d7)
* [A Beginners Guide to Data Engineering - Part 2](https://medium.com/@rchang/a-beginners-guide-to-data-engineering-part-ii-47c4e7cbda71)
* [A Beginners Guide to Data Engineering - Part 3](https://medium.com/@rchang/a-beginners-guide-to-data-engineering-the-series-finale-2cc92ff14b0)
* [Mode SQL Tutorial](https://mode.com/resources/sql-tutorial/)

The intellectual property of the below notes and resources belong to the sources which they were derived. 

**Example ETL use cases** 

*  Extracts a series of CRUD operations from a production database and derive business events such as a user deactivation.

* Take in some experiment configuration file, compute the relevant metrics for that experiment, and finally output p-values and confidence intervals in a UI to inform us whether the product change is preventing from user churn. 
* Batch ETL job that computes features for a machine learning model on a daily basis to predict whether a user will churn in the next few days. 



**Common Technology and Libraries**   
* Airlflow
* Luigi
* Pinball
* Apache Superset (web based dashboarding)
* [Jinja](http://jinja.pocoo.org/docs/2.10/) 

**Considerations when choosing a framework**  
* Configuration - how is the ETL authored. Is it configured on a UI, a domain specific language or code? Configuration as code is gaining popularity because it allows users to expressively build pipelines programmatically that are customisable. 
* UI, Monitoring and Alerts - Long running batch processes run into errors (e.g. cluster failure). Monitoring or altering are critical. How well does a framework provide a visual UI of progress on a batch? Are there alerts? Are the alerts timely? 
* Backfilling: Once a data pipeline built, we often need to go back in time and re-process the historical data. Ideally, we do not want to build two separate jobs, one for backfilling historical data and another for computing current or future metrics. 

## Data Modelling

There is a difference in the goals of OLTP systems (i.e. CRUD to capture transactions) vs OLAP (i.e. efficiently query and create statistics)

**Normalised Tables**   
* Simpler schemas
* More standardized data
* Carry less redundancy. 
* The volume of smaller tables also means that: 
** Tracking data relations requires more diligence
** Querying patterns become more complex (more JOINs)
** There are more ETL pipelines to maintain.

**Denormalised Tables**  
* Are wide tables, making them easier to query 
* All of the metrics and dimensions are already pre-joined. 
* The width of the denormalised tables means that:
** Data processing is slower and involves more upstream dependencies. 
** Can make maintenance of ETL pipelines more difficult because the unit of work is not as modular.





### Star Schemas

This design focuses on building normalized tables, specifically fact and dimension tables. When needed, denormalized tables can be built from these smaller normalized tables. This design strives for a balance between ETL maintainability and ease of analytics.

![alt text](https://miro.medium.com/max/700/1*keaNlycSUfgMGHawlcWnzw.png)

**Fact Tables**  
* Typically contain point-in-time transactional data
* Can have multiple fact tables to track for example bookings, resverations, alterations, cancellations etc

**Dimension Tables**  
* Typically show categorical data or slowly changing attributes of specific entities
* Can be organised in a hierarchial structure
* Always joined to a fact table(s), provided there is a foreign key in the fact table

**QUESTIONS**  
* Are star schemas normalised or denormalised? Conflicting information on the internet

#### Example Code

**Joining Star Schema Tables**  
```
SELECT
    b.dim_market
  , SUM(a.m_bookings) AS m_bookings
FROM (
  SELECT
      id_listing
    , 1          AS m_bookings
    , m_a        # not used (for illustration only)
    , m_b        # not used (for illustration only)
    , m_c        # not used (for illustration only)
  FROM
    fct_bookings
  WHERE
    ds BETWEEN '{{ last_sunday }}' AND '{{ this_saturday }}'
) a 
JOIN (
  SELECT
      id_listing
    , dim_market
    , dim_x      # not used (for illustration only)
    , dim_y      # not used (for illustration only)
    , dim_z      # not used (for illustration only)
  FROM
    dim_listings
  WHERE
    ds BETWEEN '{{ latest_ds }}'
) b
ON (a.id_listing = b.id_listing)
GROUP BY
  b.dim_market
;
```


## Data Partitioning & Backfilling Historical Data



Due to growth of database / DW over time,  running queries and analytics can become inefficient. 

In addition to SQL best practices, an effective technique to improve query performance is to partition data.

**SQL Best Practices**  
* Filter early and often
* Project only the fields that are needed 

### Data Partitioning by Datestamp


* Instead of storing data all in a single chunk / DW, split it up between indepndent, self-contained chunks
* Data from the same chunk will be assigned with a unique partition key
* A common partition key to use is datestamp (ds for short)
* ds is commonly used as:
** Raw data is often stored and organised by datestamp and stored in time-labeled dictionaries
** The unit of work for a batch ETL is typically one day, meaning a new partition is created for each day of work
** Many analytical questions involve counting events occurring for time ranges, so querying a datestamp is common
** Using the ds as the partitioning key also makes backfilling easier
* This technique can significantly improve query performance

```
       Column       |  Type   |      Extra      |    Comment
--------------------+---------+-----------------+-------------------------------------------
 id_listing         | bigint  |                 | Unique id of the listing.
 id_host            | bigint  |                 | Unique id of the host for the listing
 dim_market         | varchar |                 | The market in which the listing is located
 ds                 | varchar |  partition key  |
 ```


### Backfilling Historical Data

* When an ETL pipeline is built, it computes metrics and dimensions forward, not backward
* To revisit historical trends and movements, we would need to compute metric and dimensions in the past. This process is called backfilling. 
* Back filling is so common, that Hive has built in dynamic partitions to perform multiple SQL insertions at once

**Hive Dynamic Partitions Backfilling Example**  
```
INSERT OVERWRITE TABLE bookings_summary PARTITION (ds)
SELECT
      dim_market
    , SUM(m_bookings) AS m_bookings
    , ds              # For Hive to know we are using dynamic partitions
FROM
    fct_bookings
WHERE
    ds BETWEEN '{{ earliest_ds }}' AND '{{ latest_ds }}'
GROUP BY
      dim_market
    , ds
;

```
Notice the extra ds in the SELECT and GROUP BY clause, the expanded range in the WHERE clause, and how we changed the syntax from PARTITION (ds= '{{ds}}') to PARTITION (ds) . The beauty of dynamic partitions is that we wrap all the same work that is needed with a GROUP BY ds and insert the results into the relevant ds partitions all at once. 


## Airflow

### Directed Acyclic Graph (DAG)

DAGs describe **how** to run a data pipeline. 

A node in a graph represents a task, and an arrow represents the dependency of one task on another. 

Given that data only needs to be computed once on a given task and the computation then carries forward, the graph is directed and forms part of a cycle. 

This is why Airflow jobs are commonly referred to as “DAGs” (Directed Acyclic Graphs).

![alt text](https://miro.medium.com/max/1400/1*sTUUer1dRmEeHuc1h0023g.png)

### Operators: Sensors, Operators, and Transfers

Operators describe **what** to do in a data pipeline.

Typically there are three broad categories of operator :


1.   **Sensors**: waits for a certain time, external file, or upstream data source
2.   **Operators**: triggers a certain actions and transformations (e.g. run a bash command, execute a python function, or execute a Hive query, etc)
3.   **Transfers**: moves (loads) data from one location to another

Sensors = **E**xtract: Times the extraction of data

Operators = **T**ransforms

Transfers = **L**oads



#### Example DAG Definition File

```
"""
A DAG docstring might be a good way to explain at a high level
what problem space the DAG is looking at.
Links to design documents, upstream dependencies etc
are highly recommended.
"""
from datetime import datetime, timedelta
from airflow.models import DAG  # Import the DAG class
from airflow.operators.sensors import NamedHivePartitionSensor
from airflow.operators.hive_operator import HiveOperator

### You can import more operators as you see fit!
# from airflow.operators.bash_operator import BashOperator
# from airflow.operators.python_operator import PythonOperator

# setting some default arguments for the DAG
default_args = {
    'owner': 'you',
    'depends_on_past': False,
    'start_date': datetime(2018, 2, 9),
}

# Instantiate the Airflow DAG
dag = DAG(
    dag_id='anatomy_of_a_dag',
    description="This describes my DAG",
    default_args=default_args,
    schedule_interval=timedelta(days=1))   # This is a daily DAG.

# Put upstream dependencies in a dictionary
wf_dependencies = {
    'wf_upstream_table_1': 'upstream_table_1/ds={{ ds }}',
    'wf_upstream_table_2': 'upstream_table_2/ds={{ ds }}',
    'wf_upstream_table_3': 'upstream_table_3/ds={{ ds }}',
}

# Define the sensors for upstream dependencies
for wf_task_id, partition_name in wf_dependencies.iteritems():
    NamedHivePartitionSensor(
        task_id=wf_task_id,
        partition_names=[partition_name],
        dag=dag
    )

# Put the tasks in a list
tasks = [
    ('hql', 'task_1'),
    ('hql', 'task_2'),
]

# Define the operators in the list above
for directory, task_name in tasks:
    HiveOperator(
        task_id=task_name,
        hql='{0}/{1}.hql'.format(directory, task_name),
        dag=dag,
    )

# Put the dependencies in a map
deps = {
    'task_1': [
        'wf_upstream_table_1',
        'wf_upstream_table_2',
    ],
    'task_2': [
        'wf_upstream_table_1',
        'wf_upstream_table_2',
        'wf_upstream_table_3',
    ],
}

# Explicitly define the dependencies in the DAG
for downstream, upstream_list in deps.iteritems():
    for upstream in upstream_list:
        dag.set_dependency(upstream, downstream)
 ```
 
 ![alt text](https://miro.medium.com/max/1400/1*OMp-WLUdleDWVlBImO4OJA.png)

### Airflow ETL Best Practices

#### Partition Data Tables
Especially usefull when dealing with large-size tables with a long history. 

When data is partitioned using datestamps, we can leverage dynamic partitions to parallelize backfilling.

```
CREATE TABLE IF NOT EXISTS fct_bookings (
    id_listing BIGINT COMMENT 'Unique ID of the listing'
  , id_host    BIGINT COMMENT 'Unique ID of the host who owns the listing'
  , m_bookings BIGINT COMMENT 'Denoted 1 if a booking transaction occurred'
)
PARTITION BY ( -- this is how we define partition keys
  ds STRING
);
```

#### Load Data Incrementally
This will make your ETLs more modular and manageable, especially when building dimension tables from fact tables. 

In each run of the ETL, we only need to append new transactions to the dimension table from the previous date partition instead of scanning the entire fact history. 

```
-- Not Recommended Approach: Scan the entire table and rebuild everyday
INSERT OVERWRITE TABLE dim_total_bookings PARTITION (ds = '{{ ds }}')
SELECT
     dim_market
  ,  SUM(m_bookings) AS m_bookings
FROM
  fct_bookings
WHERE
  ds <= '{{ ds }}' -- this is expensive, and can quickly run into scale issue
GROUP BY
  dim_market
;

-- Recommended Approach: Incremental Load
INSERT OVERWRITE TABLE dim_total_bookings PARTITION (ds = '{{ ds }}')
SELECT
    dim_market
  , SUM(m_bookings) AS m_bookings
FROM (
  SELECT
      dim_market
    , m_bookings
  FROM
    dim_total_bookings            -- a dim table
  WHERE
    ds = DATE_SUB('{{ ds }}', 1)  -- from the previous ds

  UNION
  
  SELECT
      dim_market
    , SUM(m_bookings) AS m_bookings
  FROM
    fct_bookings                  -- a fct table
  WHERE
    ds = '{{ ds }}'               -- from the current ds
  GROUP BY
     dim_market
) a
GROUP BY
  dim_market
;
```

#### Enforce Idempotency
Idempotency means that the underlying source table should not be mutable (changeable) as time progresses. Otherwise, when running the same query against the same business logic and time range would return different results. 

#### Parameterise Workflow
Jinja can be used in conjunction with SQL to template/parameterise ETLs. 
```
{% raw  %}
{%- if backfill %}
INSERT OVERWRITE TABLE bookings_summary PARTITION (ds)
{%- else %}
INSERT OVERWRITE TABLE bookings_summary PARTITION (ds = '{{ ds }}')
{%- endif %}
SELECT
    dim_market
  , SUM(m_bookings) AS m_bookings
  {%- if backfill %}
  , ds
  {%- endif %}
FROM
  fct_bookings
WHERE
{%- if backfill %}
  ds BETWEEN '{{ start_date }}' AND '{{ end_date }}'
{%- else %}
  ds = '{{ ds }}'
{%- endif %}
GROUP BY
    dim_market
{%- if backfill %}
  , ds
{%- endif %}
{% endraw  %}
;
```


#### Add Data Checks Early and Often
It is useful to write data into staging tables, check the data quality and then if passing testing on staging load into the final production table. At AirBnB they call this *stage-check-exchange*.

Checks can be as simple as counting if the total number of records is greater than 0 or more complex such as anomaly detection. 
```
# Define the CREATE TABLE statement here
{% raw  %}
{%- macro create_table() %}
...
{%- endmacro %}

# Main ETL logic, insert the results into a STAGING table
{%- macro main() %}
...
{%- endmacro %}

# A series of simple presto CHECKS on the staging table
{%- macro health_checks() %}
...
{%- endmacro %}
{% endraw  %}
# Finally, EXCHANGE the staging table with the prod table
```


#### Building Useful Alerts and Monitoring Systems
Use EmailOperators to send alerts for jobs which do not meet SLA targets, or where errors exceed certain statistical thresholds etc. 


### Design Patterns for Data Engineering Frameworks

There are generally three layers of a well-designed data engineering framework: the **input** layer, the **data processing** layer and the **output** layer.

**Input:** This is where an end user specifies how their DAGs should be configured. User experience really matters here. Typically, the input could be a static configuration file (e.g. YAML or HOCON), or it could be something as elaborate as a web UI. The goal here is to capture user needs.

**Data Processing:** This is the core of any data engineering framework, where ETL pipelines are instantiated dynamically. The code that achieves this is generally referred to as a DAG factory, which whimsically captures the notion that DAGs are being created one at a time, like in a factory.

**Output:** The DAGs generated from the previous step create derived data, and the output is often saved in a downstream Hive table, presented in a well-designed UI / visualization layer, or simply consumed by downstream pipelines or frameworks.

![alt text](https://miro.medium.com/max/1400/1*iDfX-IBcwGu8R0PbiXw-Bg.png)

#### Incremental Computation Framework

It is quite common for data scientists to calculate computationally intensive metrics like a cumulative sum or the time since the first or last event. For example, we might wish to report the total number of users who ever engaged with a new product or to compute a histogram of days since users have last returned. The naive approach would be to query a fact table and take the sum, max, or min over all date partitions in order to calculate these desired metrics. However, this querying pattern is rather inefficient.

Why? This solution violates the ETL principle of load data incrementally since the required computation scans the entire fact table. Ideally, we would build a summary table to pre-compute these metrics so an end-user only needs to reference the metric in a single or latest date partition of the summary table. This pattern is so common that our data engineer built a framework called Incremental Computation Framework.

**Input:** A HOCON configuration file where a user specifies which metrics or events to pre-compute, which subject key to group by, and which fact table to query from in order to build the summary table.

**Data Processing:** An Airflow script that builds the summary table incrementally: namely, union the summary table from the previous date partition with today’s fact table to update the expensive metrics: cumsum_metric_today = f(cumsum_metric_yesterday, metric_today), where f can be a sum, min/max, or any other aggregation functions.

**Output:** Optimized summary table where cumulative sum, days since first / last event or other expensive metrics can be queried from one and only one single date partition from the summary table.

![alt text](https://miro.medium.com/max/1400/1*MrZeunAzDfm2eHsyZXk0EA.png)

#### Backfill Framework

**Input:** A simple UI where users can specify the job name, the start_date and end_date of the backfill job, how many processes we want to parallelize the backfill for, and how many days each process should backfill for.

**Data Processing:** Once a user specifies how the backfill job would be run, the framework creates a Airflow pipeline that automatically parallelizes the backfill tasks, performs sanity-checks, and swaps staging tables with production tables.

**Output:** A fully backfilled table ready for consumption.

![alt text](https://miro.medium.com/max/700/1*eR0I__mi5VXU5_BkbFyL6g.png)

**What workflow does this framework automate?**

It automates away many of the ad-hoc backfilling scripts people have to run on their own machines. It automates quality assurance by setting up automatic comparison. Finally, it swaps the staging table with the production table after QA tests.

#### Global Metrics Framework

**Input:** A much more involved HOCON configuration file that specifies one or more metrics in an atomic fact table, dimension sets that one wishes to include in the final table, primary keys and foreign keys to be used for joins, and a slew of other useful information to track table creations.

**Data Processing:** The framework identifies the metrics and the dimensional cuts that it needs to aggregate and cut by, joins the dimension tables with the atomic fact tables to create the denormalized tables automatically.

**Output:** One or more Hive tables with the same set of metrics but possibly different sets of dimensions are created. This means that one or more denormalized tables can be created on the fly, and all these data sources are further made available in Druid for visualization in Superset.

**What workflow does this framework automate?**

It automates away the common data engineering work that is required for the creation of denormalized tables that can be later used for dashboards, analytics, and more. The


#### Experimentation Reporting Framework

**Input:** Instead of a simple configuration file or a simple UI, a fully fledged UI is built here so users can specify the type of experiment to run, which target or secondary metrics to track, what are the experiment buckets and their relative sizes, etc. Anything that is relevant for launching and computing the experiment data is captured in this step.

**Data Processing:** The metrics pipeline that computes, for each experiment, the subject-level metrics and their corresponding dimensions. The sheer combinations of these metrics and dimensions are what makes the computation super complex. In fact, it is often the case that experimentation pipelines are the most complex ETL job at a company.

**Output:** Instead of a simple output table, there is a lot of downstream processing involved in this step. For example, statistics such as p-value, confidence interval, significance, and minimum detectable effect are calculated here. Depending on the maturity of the reporting framework, users might be able to do metrics capping or variance reduction. Each step requires a separate calculation before being served in the final UI.

![alt text](https://miro.medium.com/max/700/1*1bJye64wHKTX81-AD2DBEA.png)







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


#### General Data Engineering

* [A Beginners Guide to Data Engineering - Part 1](https://medium.com/@rchang/a-beginners-guide-to-data-engineering-part-i-4227c5c457d7 )
* [A Beginners Guide to Data Engineering - Part 2](https://medium.com/@rchang/a-beginners-guide-to-data-engineering-part-ii-47c4e7cbda71)
* [A Beginners Guide to Data Engineering - Part 3](https://medium.com/@rchang/a-beginners-guide-to-data-engineering-the-series-finale-2cc92ff14b0)

#### Airflow

* [Awesome Apache Airflow - list of resoures](https://github.com/jghoman/awesome-apache-airflow)
* [ETL Best Practices with Aitflow ](https://gtoonstra.github.io/etl-with-airflow/index.html)
* [Airflow Documentation](https://airflow.apache.org/index.html)

#### SQL

**Books** 
* [Effective SQL: 61 Specific Ways to Write Better SQL](https://www.amazon.co.uk/Effective-SQL-Specific-Software-Development/dp/0134578899)

**Web Resources**
* [Mode: The SQL Tutorial for Data Analysis](https://mode.com/sql-tutorial/introduction-to-sql/)
* 

#### Python

* [Effective Python: 59 Specific Ways to Write Better Python](https://www.amazon.co.uk/Effective-Python-Specific-Software-Development/dp/0134034287/ref=sr_1_1?crid=3C1VT02ANXTZU&keywords=effective+python+59+specific+ways+to+write+better+python&qid=1564046565&s=books&sprefix=59+eff%2Cstripbooks%2C326&sr=1-1)
