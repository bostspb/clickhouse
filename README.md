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
- [5.1 Writing Queries](05_01_writing_queries.md)
- [5.2 Using Functions](05_02_functions.md)
  - [Lab 2](05_functions_labs/lab_02_functions.md)
- [5.3 Aggregate Function Combinators](05_03_aggregate_function_combinators.md)
  - [Lab 3](05_functions_labs/lab_03_aggregate_function_combinators.md)
- [5.4 Common Table Expressions](05_04_common_table_expressions.md)
  - [Lab 4](05_functions_labs/lab_04_common_table_expressions.md)
- [5.5 Other Useful Tips](05_05_other_useful_tips.md)

## 6. Deduplication
- 6.1 Overview of Deduplication
  - Deduplication is not immediate - it is **eventual**
    - At any moment in time your table can still have duplicates (rows with the same sorting key).
    - The actual removal of duplicate rows occurs during the merging of parts.
    - Your queries need to allow for the possibility of duplicates.
  - Options for deduplication
    - With **ReplacingMergeTree** table engine, duplicate rows with the same sorting key
      are removed during merges. ReplacingMergeTree is a good option for emulating upsert behavior 
      (where you want queries to return the last row inserted).<br> 
      For a `ReplacingMergeTree` table, the last row inserted is the row that is not deleted.
    - The **CollapsingMergeTree** and **VersionedCollapsingMergeTree** table engines use a logic where 
      an existing row is "canceled" and a new row is inserted. <br> 
      For a `CollapsingMergeTree` table, a "sign" column is used of type `Int8` that is set to either 1 or -1. 
      To cancel a row, you insert a row with the same sort key but change the sign column to -1.
- [6.2 Implementing Deduplication with ReplacingMergeTree](06_02_replacing_merge_tree.md)
  - [Lab 2](06_deduplication_labs/lab_02_replacing_merge_tree.md)
- [6.3 Implementing Deduplication with CollapsingMergeTree](06_03_collapsing_merge_tree.md)
  - [Lab 3](06_deduplication_labs/lab_03_collapsing_merge_tree.md)
- [6.4 Implementing Deduplication with VersionedCollapsingMergeTree](06_04_versioned_collapsing_merge_tree.md)  

## 7. Materialized Views
- [7.1 Views](07_01_views.md)
  - [Lab 1](07_materialized_views_labs/lab_01_views.md)
- [7.2 Materialized views](07_02_materialized_views.md)
  - [Lab 2](07_materialized_views_labs/lab_02_materialized_views.md) 
- [7.3 Summing columns](07_03_summing_columns.md)
  - [Lab 3](07_materialized_views_labs/lab_03_summing_columns.md)
- [7.4 Aggregating columns](07_04_aggregating_columns.md)
  - [Lab 4](07_materialized_views_labs/lab_04_aggregating_columns.md)
- 7.5 Best Practices
  - Use the `TO` Clause
  - Avoid using the `POPULATE` Clause
  - Pausing Inserts
    - simply stop inserting rows into your source table,
    - wait until your materialized view is populated,
    - verify it worked,
    - then resume inserts.
  - Using a Timestamp Column
    - Add a `WHERE` clause to the view that contains a moment in time in the near future.
    - Wait for that moment in time to arrive, and your view will start being populated with all the new rows coming in.
    - Write a query that populates your view with all the rows whose timestamp is prior to the moment in time you 
      selected in the view's definition.

## 8. Projections
- 8.1 Overview of Projections
  - projection stores the same data of a single table but in a different sort 
    order (or in some other way that you define)
  - the projected data is stored at the part level and is actually saved in a subfolder of each table's part
    a single table can have multiple projections
  - you do not need to worry about or specify when to use a projection - ClickHouse determines at query time 
    if a query will benefit from using a projection vs. the original sort order (by determining which option 
    processes the fewest granules)
  - your data is stored twice, so there is a tradeoff between storage and the gain in performance
  - inserts can take slightly longer since the data is written twice
- [8.2 Defining Projections](08_02_defining_projections.md)

## 9. TTL: Managing Old Data
