# Creating tables

## The CREATE TABLE command

```sql
CREATE TABLE my_db.hits (
   id UInt32,
   timestamp DateTime,
   url String,
   size Nullable(Float),
   params Array(String),
   event FixedString(4)
)
ENGINE = ReplicatedMergeTree
ORDER BY (timestamp, id)
COMMENT 'My table for storing hits';
```

- It will be in the `my_db` database
- The schema is explicitly defined and has 6 columns
- All tables in ClickHouse have an engine 
- In ClickHouse Cloud, the default table engine is `ReplicatedMergeTree`, 
  which means you can omit the ENGINE clause unless you are defining a special type of table
- For self-managed deployments, the default table engine is `MergeTree`

## Specifying a schema
- **Explicitly** - you list the column names and data types, as seen in the hits table above
- **The AS clause** - specify an existing table to copy the schema from 
- **Table functions** - the schema is passed into the table function explicitly, 
  or you can let ClickHouse infer the schema by analyzing a subset of rows

### The AS clause
If you want your new table to have the same schema as an existing table
```sql
CREATE TABLE hits_today AS my_db.hits;
```

View the actual CREATE TABLE query
```sql
SELECT create_table_query
FROM system.tables
WHERE name = 'hits_today'
```
```
CREATE TABLE default.hits_today (
   `id` UInt32, 
   `timestamp` DateTime, 
   `url` String, 
   `size` Nullable(Float32), 
   `params` Array(String), 
   `event` FixedString(4)
) 
ENGINE = ReplicatedMergeTree('/clickhouse/tables/{uuid}/{shard}', '{replica}') 
ORDER BY (timestamp, id) 
SETTINGS index_granularity = 8192
```

### Table functions
> https://clickhouse.com/docs/en/sql-reference/table-functions

- `file` - create a table from a file
- `url` - reads from an HTTP/HTTPS server 
- `s3` - for reading files stored in AWS S3
- `hdfs` - creates a table from a file in HDFS

### Inferring data types

```sql
SELECT *
FROM s3(
   'https://datasets-documentation.s3.eu-west-3.amazonaws.com/hits/hits.tsv.gz',
   'TSV'
)
LIMIT 100
SETTINGS input_format_try_infer_datetimes=0
```
```
┌─c1─────┬─c2──────────────────────────────────────────────────────────────────────────┬─c3──────────────────┐
│ 258382 │ http://lvov.live_key=5&boardId=617249287/dem                                │ 2022-03-21 06:51:27 │
│ 320182 │ https://mail.clickhouse.php?act                                             │ 2022-03-22 22:22:38 │
│ 321780 │ http://womanager                                                            │ 2022-03-22 05:52:38 │
│ 349295 │ http://public/?hash=8vIx7SAgLkksPiNPkhcmFtIjoicm9vbXNbXT0yJnN0YXJUVibmJE8gE │ 2022-03-20 07:59:39 │
│ 349295 │ http://public/?hash=8vIx7SAgLkksPiNPkhcmFtIjoicm9vbXNbXT0yJnN0YXJUVibmJE8gE │ 2022-03-20 07:59:54 │
│ 358841 │ https://mail.clickhouse?appkey                                              │ 2022-03-19 16:27:10 │
│ 358841 │ https://mail.clickhouse?appkey                                              │ 2022-03-19 16:27:25 │
│ 378846 │ http://fotki.clickhouse.com/radika_5_2/205870d37e4e13bdrive                 │ 2022-03-17 02:13:01 │
│ 378846 │ http://fotki.clickhouse.com/radika_5_2/205870d37e4e13bdrive                 │ 2022-03-17 02:13:37 │
│ 378846 │ http://fotki.clickhouse.com/radika_5_2021-1-0-92f0-05-58-                   │ 2022-03-17 02:09:55 │
└────────┴─────────────────────────────────────────────────────────────────────────────┴─────────────────────┘
```
Formats for Input and Output Data - https://clickhouse.com/docs/en/interfaces/formats

We can see the data types
```sql
DESCRIBE TABLE s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/hits/hits.tsv.gz', 'TSV')
```
```
┌─name─┬─type────────────────────┬─default_type─┬─default_expression─┬─comment─┬─codec_expression─┬─ttl_expression─┐
│ c1   │ Nullable(DateTime64(9)) │              │                    │         │                  │                │
│ c2   │ Nullable(String)        │              │                    │         │                  │                │
│ c3   │ Nullable(DateTime64(9)) │              │                    │         │                  │                │
└──────┴─────────────────────────┴──────────────┴────────────────────┴─────────┴──────────────────┴────────────────┘
```

With `input_format_try_infer_datetimes = 0`
```sql
DESCRIBE TABLE s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/hits/hits.tsv.gz', 'TSV')
SETTINGS input_format_try_infer_datetimes = 0
```
```
┌─name─┬─type─────────────┬─default_type─┬─default_expression─┬─comment─┬─codec_expression─┬─ttl_expression─┐
│ c1   │ Nullable(String) │              │                    │         │                  │                │
│ c2   │ Nullable(String) │              │                    │         │                  │                │
│ c3   │ Nullable(String) │              │                    │         │                  │                │
└──────┴──────────────────┴──────────────┴────────────────────┴─────────┴──────────────────┴────────────────┘
```

### Specifying the schema

```sql
SELECT *
FROM s3(
   'https://datasets-documentation.s3.eu-west-3.amazonaws.com/hits/hits.tsv.gz',
   'TSV',
   'id UInt64,
    url String,
    timestamp DateTime64
   '
)
LIMIT 10
```
```
┌─────id─┬─url─────────────────────────────────────────────────────────────────────────┬───────────────timestamp─┐
│ 258382 │ http://lvov.live_key=5&boardId=617249287/dem                                │ 2022-03-21 06:51:27.000 │
│ 320182 │ https://mail.clickhouse.php?act                                             │ 2022-03-22 22:22:38.000 │
│ 321780 │ http://womanager                                                            │ 2022-03-22 05:52:38.000 │
│ 349295 │ http://public/?hash=8vIx7SAgLkksPiNPkhcmFtIjoicm9vbXNbXT0yJnN0YXJUVibmJE8gE │ 2022-03-20 07:59:39.000 │
│ 349295 │ http://public/?hash=8vIx7SAgLkksPiNPkhcmFtIjoicm9vbXNbXT0yJnN0YXJUVibmJE8gE │ 2022-03-20 07:59:54.000 │
│ 358841 │ https://mail.clickhouse?appkey                                              │ 2022-03-19 16:27:10.000 │
│ 358841 │ https://mail.clickhouse?appkey                                              │ 2022-03-19 16:27:25.000 │
│ 378846 │ http://fotki.clickhouse.com/radika_5_2/205870d37e4e13bdrive                 │ 2022-03-17 02:13:01.000 │
│ 378846 │ http://fotki.clickhouse.com/radika_5_2/205870d37e4e13bdrive                 │ 2022-03-17 02:13:37.000 │
│ 378846 │ http://fotki.clickhouse.com/radika_5_2021-1-0-92f0-05-58-                   │ 2022-03-17 02:09:55.000 │
└────────┴─────────────────────────────────────────────────────────────────────────────┴─────────────────────────┘
```

### Populating a table from a table function

```sql
CREATE TABLE hits (
   id UInt64,
   url String,
   timestamp DateTime64
)
PRIMARY KEY (id, timestamp)
```
```sql
INSERT INTO hits 
SELECT *
FROM s3(
   'https://datasets-documentation.s3.eu-west-3.amazonaws.com/hits/hits.tsv.gz',
   'TSV',
   'id UInt64,
    url String,
    timestamp DateTime64
   '
)
```
```sql
SELECT COUNT()
FROM hits
-- 4178363
```

Functions for reading from other databases
```sql
CREATE TABLE ch_table
AS 
   SELECT * 
   FROM mysql(
      `mysqlhost:3306`, 
      'mysql_database', 
      'mysql_table', 
      'user', 
      'password'
)
```

Table with 1M rows, values will be from 0 to 999999
```sql
CREATE TABLE one_million 
AS numbers(1000000);
```

## Table engines
- **MergeTree**. And the entire [MergeTree family of table engines](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family), 
  which are the core of ClickHouse data storage capabilities. They provide the most features for 
  resilience and high-performance data retrieval.
- **Special types of engines**. Like `View`, `MaterializedView`, `Dictionary`, and [more](https://clickhouse.com/docs/en/engines/table-engines/special).
- **Integrations**. For tables that connect to external resources such as Kafka, S3, HDFS, and [other databases](https://clickhouse.com/docs/en/engines/table-engines/integrations).

```sql
CREATE TABLE daily_totals (
   user_id UInt32,
   day Date,
   metric1_sum UInt64,
   metric1_count UInt32,
   metric2_sum UInt64,
   metric2_count UInt32
)
ENGINE = SummingMergeTree
PARTITION BY day
ORDER BY (day, user_id)
```

```sql
CREATE TABLE hits_on_S3 (
   id UInt64,
   url String,
   timestamp DateTime64
)
Engine = S3(
   'https://datasets-documentation.s3.eu-west-3.amazonaws.com/hits/hits.tsv.gz',
   'TSV'
)
```

A table engine that doesn't store any of your data
```sql
CREATE TABLE logs_raw (
   message String
)
ENGINE = Null;

-- Any rows inserted into logs_raw will be ignored, 
-- but Null can be very useful when creating materialized views
```