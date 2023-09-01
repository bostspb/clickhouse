# 7.2 Materialized views

## Defining a materialized view
```sql
CREATE MATERIALIZED VIEW just_testing
ENGINE = MergeTree
ORDER BY (user_id, error_code)
AS 
   SELECT 
      user_id, 
      splitByChar(':', message)[1] as error_code
   FROM log_messages
```

- You need to specify the table engine of the underlying table
- The schema is determined by the results of the SELECT query
- The name of the underlying table is generated automatically and will look like `.inner.just_testing`
- This view is empty because we did not tell ClickHouse to populate it with 
  the existing data in `log_messages` using the `POPULATE` command

```sql
SELECT * FROM just_testing
-- 0 rows returned
```

When you insert rows into log_messages, the new rows are also sent to the view.
```sql
INSERT INTO log_messages VALUES
   (3, 'INFO: Things are going well', '2022-10-06', 123),
   (3, 'ERROR: Something went wrong', '2022-10-06', 456);
   
SELECT * FROM just_testing
```
```
┌─user_id─┬─error_code─┐
│       3 │ ERROR      │
│       3 │ INFO       │
└─────────┴────────────┘   
```

## Populating a view
```sql
CREATE MATERIALIZED VIEW error_codes
ENGINE = MergeTree
ORDER BY (user_id, error_code)
POPULATE AS 
   SELECT 
      user_id, 
      splitByChar(':', message)[1] as error_code
   FROM log_messages;

SELECT * FROM error_codes;
```
```
┌─user_id─┬─error_code─┐
│       1 │ INFO       │
│       1 │ INFO       │
│       1 │ WARNING    │
│       2 │ ERROR      │
│       2 │ ERROR      │
│       2 │ ERROR      │
│       2 │ INFO       │
│       3 │ ERROR      │
│       3 │ INFO       │
└─────────┴────────────┘
```
```sql
INSERT INTO log_messages VALUES
   (2, 'INFO: Things are going well', '2022-10-07', 50),
   (1, 'ERROR: Something went wrong', '2022-10-07', 60);

SELECT * FROM error_codes ORDER BY user_id;
```
```
┌─user_id─┬─error_code─┐
│       1 │ ERROR      │
│       1 │ INFO       │
│       1 │ INFO       │
│       1 │ WARNING    │
│       2 │ INFO       │
│       2 │ ERROR      │
│       2 │ ERROR      │
│       2 │ ERROR      │
│       2 │ INFO       │
│       3 │ ERROR      │
│       3 │ INFO       │
└─────────┴────────────┘
```

## Defining a view table
Source table
```sql
CREATE TABLE metrics 
(
   user_id UInt32,
   timestamp DateTime,
   metric1 UInt64,
   metric2 Uint64
)
ENGINE = MergeTree
ORDER BY (user_id, timestamp)
```

The table for view
```sql
CREATE TABLE max_metrics
(
   max_metric UInt64
)
ENGINE = MergeTree
ORDER BY tuple()
```

Materialized view
```sql
CREATE MATERIALIZED VIEW max_metrics_view
TO max_metrics
AS
   SELECT
      greatest(metric1, metric2) AS max_metric
   FROM metrics
```
- The `TO` clause only requires a table name - the schema and table engine are already defined in the `max_metrics` table.
- When using the `TO` clause, you cannot use `POPULATE AS`. If you want your new view to materialize the existing rows
  of the source table, you will have to write a query yourself.

Materialized views are particularly useful when:
- Query results contain a small number of rows and/or columns relative to the base table (the table on which the view 
  is defined)
- Query results contain results that require significant processing, including:
- Analysis of semi-structured data
- Aggregates that take a long time to calculate
- The query is on an external table (that is, data sets stored in files in an external stage), which might have slower 
  performance compared to querying native database tables
- The view's base table does not change frequently