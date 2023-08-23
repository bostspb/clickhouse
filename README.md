# ClickHouse Developer: Learning Path
> https://learn.clickhouse.com/visitor_catalog_class/show/1049584/ClickHouse-Developer-Learning-Path

## 1. Getting Started
- 1.1 Start a new ClickHouse Service
  - [Lab 1](01_getting_started_labs/lab_01_start_new_clickhouse_service.md)
- [1.2 Understanding primary keys in ClickHouse](01_02_primary_keys.md)
  - [Lab 2](01_getting_started_labs/lab_02_create_table.md)
- [1.3 Inserting data](01_03_inserting_data.md)
  - [Lab 3](01_getting_started_labs/lab_03_insert_from_file.md)
- [1.4 Running queries](01_04_running_queries.md)
  - [Lab 4](01_getting_started_labs/lab_04_queries.md)

## 2. Creating Databases and Tables
- [2.1 Creating databases](02_01_creating_databases.md)
  - [Lab 1](02_creating_databases_and_tables_labs/lab_01_creating_database.md)
- [2.2 Creating tables](02_02_creating_tables.md)
  - [Lab 2](02_creating_databases_and_tables_labs/lab_02_creating_tables.md)
- [2.3 ClickHouse data types](02_03_data_types.md)
  - [Lab 3](02_creating_databases_and_tables_labs/lab_03_data_types.md)

## 3. ClickHouse Architecture
- **3.1 Data storage**
  -	**granule** - a logical breakdown of rows inside an uncompressed block; default is 8,192 rows
  - **primary key** - the sort order of a table
  -	**primary index** - an in-memory index containing the values of the primary keys of the first row of each granule
  -	**part** - a folder of files consisting of the column files and index file of a subset of a table's data
- **3.2 Choosing a primary key**
  - PK can be defined PRIMARY KEY inside, outside and via ORDER BY
  - Sorting is a superset of the primary key
  - Good candidates for PK columns are:
    - Lots of queries on a column
    - Ordered by cardinality in ascending order
  - Primary indexes must fit in memory
  - Options for creating additional primary indexes
    - Create two tables
    - Use a projection
    - Use a materialized view
    - Define a skipping index
  - Focus on defining a good primary key instead partitioning if you want to improve query performance
- **3.3 Partitions**
  - A partition is a logical combination of records in a table by a specified criterion
  - In most cases, you don't need a partition key
  - Why create a partition - for data management
  - Partitions are only available for MergeTree family engines
  - Only parts that have the same partition key are merged

## 4. Data Ingestion
- [4.1 Insert data from a file](04_01_insert_data_from_a_file.md)
- [4.2 Insert data from an external database](04_02_insert_data_from_from_an_external_database.md)
- 4.3 Using Table Functions and Engines
  - If you are going to write **a single ad-hoc query** of some data sitting in external source, 
    then the **table function** is perfect for that use case.
  - If you are going to run **ad-hoc queries on a regular basis**, consider using the **table engine**, 
    which stores all the connection details and credentials so you don't have to enter them every time/
  - If you are going to be **analyzing data** continuously, then you should use the table function 
    or table engine to **move the data into ClickHouse** and store it in a MergeTree table.

## 5. Analyzing Data
- 5.1 Writing Queries
  - Lab 1
- 5.2 Using Functions
  - Lab 2
- 5.3 Aggregate Function Combinators
  - Lab 3
- 5.4 Common Table Expressions
  - Lab 4
- 5.5 Other Useful Tips

## 6. Deduplication
## 7. Materialized Views
## 8. Projections
## 9. TTL: Managing Old Data
