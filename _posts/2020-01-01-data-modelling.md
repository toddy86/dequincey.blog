---
layout: post
title: Kimball Data Modelling Concepts
categories: resources
---

A summary of key Kimball method data modelling concepts for data engineering.

<!-- more -->

*Note: These notes are a work in progress*


## Attributes of Successful Data Warehouses

The data warehouse and the accompanying business intelligence tool(s) must:
- make information easily accessible to the business users.
- be accurate, credible and consistent. If business users have no faith in the data, or it provides inconsistent results then it is worthless.
- be able to gracefully handle change as and when the business needs change.
- make information available in a timely fashion. Depending on the business needs this could be within a day, hours, minutes or even seconds of new data being available.
- be secure and accessible only to those with the correct levels of permission.
- must serve as the authoritive and trustworthy source of truth for making data-based business decisions.
- must be embraced and accepted by the business users.

*Source: The Datawarehouse Toolkit*

## Dimensional Models
- Critical that DW projects focus on **business processes** and NOT departments (e.g. focus on purchase orders, customer returns process vs focusing on the sales department)
- If tasked with drafting the DW/BI system’s data architecture, you need to wrap your head around the organization’s processes, along with the associated master descriptive dimension data. The prime deliverable for this activity, the enterprise data warehouse bus matrix.
- The enterprise data warehouse bus matrix is central to the entire Kimball Methodology. This matrix helps eliminate data redundancy, provide a high-level development plan and architecture guide (especially usefull where agile methodologies are being used)  

## 3NF Models vs Dimensional Models

- Dimensional modelling is widely accepted as the best technique for deliverying analytics data as:
	- It provides business users with understandable data to assist in the decision making process
	- It provides fast, reliable, accurate and consistent query performance
- Dimensional models are often instantiated in relational databases (e.g. MySQL, Potgresql)
- However, unlike databases for OLTP or other consumer / user facing applications, they do not ordinarily follow the rules of normalisation to the third normal form (3NF) which remove data redundancies
- Under the rules of normalisation the 3NF, data is divided into separate entities (to become tables in the database) to remove all data redundancies
- Data models normalised to the 3NF whilst reduce data redundancy, require more complicated SQL joins and manipulation to derive information for analytics platforms. Further, query performance (due to the complicated SQL) can be slow. 3NF data bases are excellent for transactional processing systems, as updates and inserts to the data base are only completed in one spot (ie. no data redundancy), but are too complicated for BI solutions. Finally, 3NF data models can be extremely difficult to read, interpret and understand, due to the normalisation process. 
- Data models for analytics are perferred to be denormalised to improve query speed, SQL complexity and "human understandability" of the models
- A dimensional model contains the same information as a normalized model, but packages the data in a format that delivers user understandability, query performance, and resilience to change *Source: The Data Warehouse Toolkit*


## Star Schemas
- Comprised of a single fact table and multiple dimension tables
- Are simple, which benefits the business users as it is easier to understand and navigate
- As there are fewer table joins required to get insights, the query performance is significantly improved over a 3NF model
- Are very extensible to accommodate change within the business, whereas a model in the 3NF is significantly harder to change.

## Fact Tables
### General
- Stores the performance measurements of a business
- Each row in a fact table corresponds to a measurement event (e.g. a sale, support call)
- The data on each row is at a specific level of details, referred to as the *grain* (e.g. one row per product sold in a sales transaction)
- A key principle of dimensional modelling is that ALL rows in a fact table MUST have the same grain. Required to avoid double counting and erronous data
- All fact table grains fall into one of three categories:
	- Transaction (most common)
	- Periodic snapshot
	- Accumulating snapshot
- The most usefull facts are numeric
- Facts can either be additive, semi-additive or non-additive
	- Additive: Can be summed up to represent a valid total (e.g. sum of the sales for a month)
	- Semi-additive: Can be summed up for some dimensions (e.g. a specific point in time), but cannot be summed across all dimensions (e.g. time). For example, cannot sum the account balance of receivables over the course of a week
	- Non-additive: Cannot be summed to make a logical value (e.g. cannot sum a series of product prices, but can do counts, averages etc).
- You should NOT store textual data in a fact table. Every effort should be made to put text based data into a dimension table. You should not store redundant text based data in fact tables, unless the text is unique for every row in the fact table. Otherwise textual data belongs in a dimension table. Textual data in a fact table is very rare.
- Fact tables tend to be long (many rows), but narrow (not many columns)
- All fact tables have 2 or more FK, which correspond to the dimension table PK
- You access the fact table via the dimension tables joined to it
- The fact table generally has its own primary key composed of a subset of the foreign keys. This key is often called a composite key. Every table that has a composite key is a fact table. Fact tables express many-to-many relationships. All others are dimension tables. *Source: The Data Warehouse Toolkit*
- As the fact data is the largest set of data in a dimensional model, it should NOT be replicated in multiple places for each business function (e.g. sales, marketing, support)
- Providing only a single, centralised fact data repository for business users to query ensures that the data provided is consistent throughout the organisation

### Techniques
#### Table Structure
- Contains the numeric measures produced from business events in the real world
- At the lowest, atomic grain, each row corresponds to a measurement event
- Fundamentally based on physical activity (e.g. a sales order) and is NOT influenced by eventual reports to be produced
- Always contains a FK to each of the dimension tables
- Optionally can include date/time stamps, and degenerate dimension keys (dimensions which do not have a separate dimension table, as they have a 1:1 relationship with each row in the fact table)

#### Additive, Semi-additive and Non-Additive Facts
- The most useful facts are fully additive
- A good approach for non-additive facts is, where possible, to store the fully additive components of the non-additive measure and sum these components into the final answer set before calculating the final non-additive fact. For example, a non-additive fact of a ratio, it would be better to store the additive facts in the fact table and then use these additive facts to calculate the end ratio in the BI layer.

See above for more details

#### Nulls in Fact Tables
- Nulls are fine to be included in fact tables
- But nulls MUST NOT be included in any of the FK, as these nulls would cause a referential integrity violation
- Inplace of NULL values in FK columns, the associated dimension table must have a default row (and surrogate key - i.e. incremental integer key) representing the unknown or not applicable condition.

#### Transaction vs Periodic Snapshot Fact Tables

Transaction fact tables:
- Correspond to a measurement event at a point in space and time
- Atomic transaction grain measurement are the most dimensional and expressive fact tables, as they can be sliced, diced and aggregated in the highest number of ways
- May be dense or sparse, as they only contain rows where a measurement event has taken place (i.e. they aren't filled with 0 values for the sake of recording 0 values)
- Contain a specific date and time stamp

Periodic snapshot tables: 
- A row summarises / aggregates many measurement events for a standard measure of time (e.g. purchase orders by day, by week, by month)
- The grain is the period (e.g. by day), NOT the individual transaction
- Often contain many facts, as any measurement event consistent with the fact table grain is allowable
- Are often dense, as even if a measurement event hasn't taken place during the period, a row is usually inserted to contain NULL or 0 values for each fact (e.g. 0 sales for the day)

#### Aggregate Fact Tables
- Are simple numeric rollups of atomic fact data built soley to accelerate query performance
- The aggregate tables should be available at the BI layer at the same time as the atomic data
- A properly designed set of aggregates should function like database indexes in that it speeds up the query performance, but does not require any intervention by the BI user

#### Consolidated Fact Tables
- Two or more fact tables that are consolidated together
- Consolidated fact tables add burden to the ETL process, but ease analytics burden on BI applications. This helps reduce friction for adoption by the business users

## Dimension Tables
### General
- Contain the textual context associated with the measurement event recorded in the fact table. 
- Dimension tables are denormalised
- Dimensions describe the who, what, where, when, how and why of the fact data.
- Dimension tables tend to be shorter (fewer rows), but wide (many columns)
- Dimension tables typically are highly denormalized with flattened many-to-one relationships within a single dimension table
- Each dimension is defined by a SINGLE PK.
- Attributes in dimension tables act as the primary query constraints (e.g. GROUP BY, WHERE, labels). 
- Attributes in dimension tables should NOT be abbreviations or codes. They should be the full name of the attribute to avoid confusion. Even if the business users are used to speaking in codes, ALWAYS have an dimensional attribute which has the full, human readable name. For example, if all of the business users know that the first two digits (ie. 14) of 147553 represents Google Ads, do not force them to use substrings and other data manipulation to extract the code. Have three attributes for the full code (147553), the short hand code (14) and the full textual name (Google Ads). The more time spent providing attributes with verbose business terminology, the better. Robust dimension attributes deliver robust analytic slicing-and-dicing capabilities.
- For numeric data where it is unclear if the data is a fact or a dimension:
	- Fact --> Column takes on a lot of measurements and can be used in calculations
	- Dimension --> Discrete numeric values drawn from lists (e.g. product code) which are more or less constant and can be used in constraints (e.g. GROUP BY) or as a row label
	- Either or --> Always need to think about the situation. A product cost would ordinarily be considered a dimension, but if it changes so frequently, it may be considered a fact
- ALWAYS repeat values in dimension tables as necessary (e.g. for 100 products which are all of the same brand, repeat the brand name 100 times in the dimension table). Resist the tempation to normalise the data by having e.g. a separate table for the brand name. If you normalise this, it is called snowflaking. As dimension tables ordinarily are significantly smaller than fact tables, and often only represent circa 10% of the data storage of a dimensional model, normalising will not have any significant storage efficiency gains. You should almost ALWAYS trade off table space for simplicity and accessibility.

### Techniques
#### Table Structure
- Every dimension table has a SINGLE primary key (which cannot be a natural key see surrogate keys below), which is embedded in any associated fact table as a foreign key
- While operational codes and indicators can be treated as attributes, the most powerful dimension attributes are populated with verbose descriptions.

#### Dimension Surrogate Keys
- The (single) primary key cannot be a natural key, as the use of a natural key will not faciliate the tracking of changes to the dimension over time (i.e. slow changing dimensions). Further, the natrual keys are administered by external systems and may not be entirely reliable
- Therefore, the DW needs to take control over the PK of all dimensions by using surrogate keys (autoincrementing integers). The exclusion to this rule is date dimensions.
- Why use surrogate keys:
	- Buffer the data warehouse from operational changes (e.g. the code 14 may have been used in the business for the Google Ads platform, but the business reuses this code and reassigns it to Facebook. If we had of use the natural key of the code, we will encounter problems)
	- Integrate multiple source systems
	- Improve performance (smaller FK in the fact tables vs bulky text keys)
	- Handle null or unknown conditions
	- Support dimension attribute change tracking. Some may argue to use a pseudo surrogate key (created by simply gluing together the natural key with a time stamp), but this leads to multiple joins between the dimension and fact tables, which can adversely impact performance and ease of use.

**IMPORTANT NOTE: Every join between dimension and fact tables in the data warehouse should be based on meaningless integer surrogate keys. You should avoid using a natural key as the dimension table’s primary key. *Source: The Data Warehouse Toolkit, page 99***

#### Null Attributes
- Null valued attributes may occur in a dimensional table where the value is not applicable or has not been fully loaded
- Rather than null, it is better to have a descriptive text such as "Unknown" or "Not applicable". This is recommended as 
	1. It is more explicit than Null
	2. Different databases handle grouping and contraining on nulls differently

#### Flags and Indicators
- Abbreviations, true/false flags and other operational indicators and shorthands should be supplemented with full text words which have a real world, human meaning

#### Hierarchies
- Avoid the temptation to normalise the data into a snowflake schema
- Many hierarchies can coexist on a single dimension table without concern

#### Date Dimensions
- Attached to virtually every fact table to allow for navigation through dates, months, periods etc
- Typically has a number of attributes describing different aspects of the date (e.g. day in the year, month, abbreviated month name, fiscal month, fiscal year, calendar year etc)
- PK of a date dimension table is usually an integer representation of the date (e.g. 20200329)
- When additional precision is needed, a separate datetimestamp column can be added to the FACT table. This column is NOT a FK to the date dimension table - it is a standalone column, typically with a unique(ish) value for each row
- If business users regularly segregate / report on time of day, then a SEPARATE time-of-day dimension table should be added
 
#### Slow Changing Dimension Attributes
There are several ways of handling slow changing dimension attributes. Very often, the method used for these slow changing attributes will be different from attribute-to-attribute within a single dimension table.

Methods include:

1. Retain the original value so that it never changes. This is appropriate for any attribute which is labelled as "original". For example, the "original credit score"
2. Overwrite the value with the latest version. Probably the easiest to implement, but it removes all history, does not create additional dimension table rows and you must be careful that aggregate fact tables are recomputed with the latest values
3. Add a new row to the dimension table with the new attributes. This is faciliated by the use of surrogate primary keys. Each time a new dimension row is added, a new surrogate key is added and used as the FK in all fact tables from the moment of update. At a MINIMUM, three additional columns should be added to the dimension row:
	- Row effective date or datetime
	- Row expiration date or datetime
	- Current row indicator

## Data Warehouse Architecture
There are many methods to designing the architecture of a data warehouse / business intelligence solution. Three of the most dominant methods are:
1. The Kimball Method
2. Independent Data Marts
3. Hub-and-Spoke Corportate Information Factory

### Kimball Method
![png](/assets/images/data_modelling/kimball_data_architecture.png)
*Source: The Data Warehouse Toolkit*

Key concepts:
- Data is to be presented, stored and accessed in dimensional schemas - either star schemas or OLAP cubes
- The presentation area (i.e. where the BI application and end users access the data) must contain detailed, **atomic** data. Atomic data is a must to ensure that it can meet all of the adhoc queries and requirements of the business. The presentation area may contain aggreated tables as well, but the grainular data MUST also be provided alongside it. The most finely grained data must be available in the presentation area so that users can ask the most precise questions possible. Or to answer the most unpredicable of questions of the data.
- The presentation area must adhere to the bus architecture. The bus architecture mandates that all dimensions be conformed dimensions. Conformed dimensions are common, standardized, master dimensions that are managed once in the extract, transformation, and load (ETL) system and then reused by multiple fact tables. 
- Without the bus architecture (ie. without shared, conformed dimensions), a dimensional model becomes a stand alone application. When dimensional models have been designed with conformed dimensions, they can be readily combined and used together. Using the bus architecture is the secret to building distributed DW/BI systems. 
- Conformed dimensions removes the need for normalisation to resolve data inconsistencies
- Organised by **business processes** (e.g. retail sales, issue purchase order, customer returns etc)


**Example Bus Matrix**
![png](/assets/images/data_modelling/kimball_bus_matrix.png)


*[Source: Kimball Data Warehouse Bus Architecture](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/kimball-data-warehouse-bus-architecture/)*


**IMPORTANT NOTE: Data in the queryable presentation area of the DW/BI system must be dimensional, atomic (complemented by performance-enhancing aggregates), busi- ness process-centric, and adhere to the enterprise data warehouse bus architecture. The data must not be structured according to individual departments’ interpreta- tion of the data.** *Source: The Data Warehouse Toolkit*


### Independent Data Marts
![png](/assets/images/data_modelling/data_mart_architecture.png)
*Source: The Data Warehouse Toolkit*

Key concepts:
- Analytics data are deployed on a departmental basis without concern about sharing and integrating data from and across different departments
- Separate data base is created for each department to meet their unique needs (i.e. data needs, labelling needs, business rules etc)
- Can lead to significant duplication of effort and data where departments have similar data needs (e.g. sales department and the finance departments are both interested in sales data)
- Can lead to confusion and unaligned data between departments, depending on how each department has setup their database etc
- Almost no industry leaders recommend the use of data marts, these setups are very prevalent in large corporations, usually a result of how these organisations fund their IT projects and the potential lack of cross-department communication
- Data marts are much cheaper and quicker to develop in the short-term, but this comes at the long term cost (i.e. redundant data storage, reconciliation of cross-departmental figures etc)
- Requires a second ETL process to delivery the data to the presentation area (i.e. data marts)

### Hub-and-Spoke	
![png](/assets/images/data_modelling/hub_and_spoke_architecture.png)
*Source: The Data Warehouse Toolkit*

Key concepts:
- Approach advocated by Bill Inmon and several other industry leaders
- The atomic data extracted through the ETL process is stored in a 3NF database, often referred to as the Enterprise Data Warehouse (EDW) and is a mandatory construct in the Hub-and-Spoke setup
- Like the Kimball Method, the H&S approach advocates enterprise wide data coordination and integration (through this normalised EDW). Whereas, the Kimball Method stresses the importance of an enterprise bus with conformed dimensions.
- Frequently departmentally structured (vs business process focused in the Kimball Method) and often populated with summary / aggregated data (vs atomic data in the Kimball Method)

## Kimball Dimensional Modelling Techniques
### Business Requirements
- Before doing any modelling, you MUST spend adequate time to understand the business, its processes, the business data needs and the underlying data sources available.
- Document KPIs, business issues / problems that, decision making processes and data / analytics needs of business users.

### Four-step Dimensional Design Process
1. Select the business process which the data warehouse should cover 
	- what are the needs of the business 
	- what is the quality and quantity of the data available
	- does this data meet these business needs
	- Choosing the process is important because it defines a specific design target and allows the grain, dimen- sions, and facts to be declared
	- Each business process corresponds to a row in the enterprise data warehouse bus matrix
2. Declare the grain
	- Critical step in dimensional design
	- The grain establishes exactly what a single fact table row represents
	- Must declare the grain before choosing facts or dimensions, as the grain will determine what facts and dimensions will be used
	- Atomic grain refers to the lowest level at which data is captured by a given business process and it is HIGHLY RECOMMENDED that the model be focused on atomic-grained data to ensure the data is flexible enough to withstand changing business and query needs
	- A single fact table MUST only contain one grain. If other grains are needed, then these must be stored in separate fact tables
3. Identify the dimensions
	- Provide the who, what, where, when, why and how for slicing and dicing the fact data
	- With the grain of the fact table now set, it is now possible to identify all of the possible dimensions
	- A disproportionate amount of time and effort should be put into designing dimension tables, as they are the drivers of the user’s BI experience.
4. Identify the facts
	- A single fact table row has a one-to-one relationship to a measurement event as described by the fact table’s grain
	- Thus a fact table corresponds to a physical observable event, and not to the demands of a particular report.
	- 

### Dimensional Model Extensions
Dimensional models are resilient when data relationships change. All the following changes can be implemented without altering any existing BI query or application, and without any change in query results.
	- Facts consistent with the grain of an existing fact table can be added by creat- ing new columns.
	- Dimensions can be added to an existing fact table by creating new foreign key columns, presuming they don’t alter the fact table’s grain.
	- Attributes can be added to an existing dimension table by creating new columns.
	- The grain of a fact table can be made more atomic by adding attributes to an exist- ing dimension table, and then restating the fact table at the lower grain, being careful to preserve the existing column names in the fact and dimension tables.

*Source: The Data Warehouse Toolkit*

## References
The above notes are from a variety of sources, including, but not limited to:
* [The Data Warehouse Toolkit: The Definitive Guide to Dimensional Modeling, 3rd Edition](https://www.amazon.co.uk/Data-Warehouse-Toolkit-Definitive-Dimensional/dp/1118530802/ref=pd_sbs_14_t_0/260-8697327-9127526?_encoding=UTF8&pd_rd_i=1118530802&pd_rd_r=fa6e740a-7d6f-47cb-a583-df6ac1d5b3f3&pd_rd_w=9t239&pd_rd_wg=VUxZY&pf_rd_p=e44592b5-e56d-44c2-a4f9-dbdc09b29395&pf_rd_r=7YZTSDT3PMM7V0FBGMJQ&psc=1&refRID=7YZTSDT3PMM7V0FBGMJQ)


The intellectual property of the below notes and resources belong to the sources which they were derived. 