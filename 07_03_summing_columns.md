# 7.3 Summing columns
How to create a materialized view that keeps a running total of numeric columns.

## The SummingMergeTree table engine
```
┌─user_id─┬─message──────────────────────┬───────────timestamp─┬─metric─┐
│       1 │ INFO: This is just a message │ 2022-10-06 00:00:00 │     10 │
│       1 │ WARNING: Something seems off │ 2022-10-06 00:00:00 │     30 │
│       1 │ INFO: Things are going well  │ 2022-10-07 00:00:00 │     15 │
│       1 │ ERROR: Something went wrong  │ 2022-10-07 00:00:00 │     60 │
│       2 │ ERROR: Something went wrong  │ 2022-10-06 00:00:00 │     20 │
│       2 │ ERROR: Another problem       │ 2022-10-07 00:00:00 │     50 │
│       2 │ INFO: Things are going well  │ 2022-10-07 00:00:00 │     25 │
│       2 │ INFO: Things are going well  │ 2022-10-07 00:00:00 │     50 │
│       2 │ ERROR: Another problem       │ 2022-10-08 00:00:00 │     35 │
│       3 │ INFO: Things are going well  │ 2022-10-06 00:00:00 │    123 │
│       3 │ ERROR: Something went wrong  │ 2022-10-06 00:00:00 │    456 │
└─────────┴──────────────────────────────┴─────────────────────┴────────┘
```

Sum metrics via query
```sql
SELECT user_id, sum(metric) 
FROM log_messages 
GROUP BY user_id 
ORDER BY user_id
```
```
┌─user_id─┬─sum(metric)─┐
│       1 │         115 │
│       2 │         180 │
│       3 │         579 │
└─────────┴─────────────┘
```

Sum metrics via `SummingMergeTree` table engine
```sql
CREATE MATERIALIZED VIEW my_sums
ENGINE = SummingMergeTree
ORDER BY user_id
POPULATE AS 
   SELECT
      user_id,
      sum(metric) as metric
   FROM log_messages
   GROUP BY user_id;

SELECT * FROM my_sums;
```
```
┌─user_id─┬─sum(metric)─┐
│       1 │         115 │
│       2 │         180 │
│       3 │         579 │
└─────────┴─────────────┘
```

```sql
INSERT INTO log_messages VALUES
   (1, 'INFO: This is just a message', '2022-10-08', 50),
   (2, 'ERROR: Something went wrong', '2022-10-08', 200),
   (3, 'WARNING: Something seems off', '2022-10-08', 300),
   (1, 'ERROR: Another problem', '2022-10-08', 50);

SELECT * FROM my_sums;
```
```
┌─user_id─┬─metric─┐
│       1 │    100 │
│       2 │    200 │
│       3 │    300 │
└─────────┴────────┘
┌─user_id─┬─metric─┐
│       1 │    115 │
│       2 │    180 │
│       3 │    579 │
└─────────┴────────┘
```
```sql
SELECT
    user_id,
    sum(metric)
FROM my_sums
GROUP BY user_id
```
```
┌─user_id─┬─sum(metric)─┐
│       3 │         879 │
│       2 │         380 │
│       1 │         215 │
└─────────┴─────────────┘
```
`SELECT` query on `my_sums` is only adding two numbers together per `user_id`. If `log_messages` had millions of rows
and `my_sums` only had a dozen parts, it is much faster to add together a dozen numbers instead of millions.


## Summing by date ranges (histograms)

Metrics source table
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

The table for storing the daily sums of `metric1` and `metric2`, along with the count of each metric
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
- The primary key is what makes this work as a daily histogram: we are summing all the numeric values that have 
  the same `day` and the same `user_id`
- We do not need to partition by `day`, but it seems logical for our use case

```sql
CREATE MATERIALIZED VIEW daily_totals_view
TO daily_totals
AS
   SELECT
      user_id,
      toDate(timestamp) AS day,
      sum(metric1) AS metric1_sum,
      count(metric1) AS metric1_count,
      sum(metric2) AS metric2_sum,
      count(metric2) AS metric2_count
   FROM metrics
   GROUP BY day, user_id
```
- The `TO` clause is used, so this is an example of a materialized view where we define the underlying table ourselves
  (in this case, the `daily_totals` table)
- In the `SELECT`, it is important to use aliases for our selected columns so that they match the column 
  names of `daily_totals`
- It is also important that the `GROUP BY` in the `SELECT` query of `daily_totals_view` matches
  the `ORDER BY` of `daily_totals`
- The `timestamp` in `metrics` is of type `DateTime`, but our `daily_totals` only needs to be a `Date`, 
  so we use the `toDate` function to make the conversion

```sql
INSERT INTO metrics VALUES
   (1, '2022-12-01', 5, 100),
   (2, '2022-12-01', 1, 75),
   (1, '2022-12-01', 3, 120),
   (2, '2022-12-01', 7, 90),
   (1, '2022-12-02', 2, 140),
   (2, '2022-12-02', 8, 110),
   (1, '2022-12-02', 4, 150),
   (3, '2022-12-02', 6, 90);

SELECT * FROM daily_totals ORDER BY (day, user_id);
```
```
┌─user_id─┬────────day─┬─metric1_sum─┬─metric1_count─┬─metric2_sum─┬─metric2_count─┐
│       1 │ 2022-12-01 │           8 │             2 │         220 │             2 │
│       2 │ 2022-12-01 │           8 │             2 │         165 │             2 │
└─────────┴────────────┴─────────────┴───────────────┴─────────────┴───────────────┘
┌─user_id─┬────────day─┬─metric1_sum─┬─metric1_count─┬─metric2_sum─┬─metric2_count─┐
│       1 │ 2022-12-02 │           6 │             2 │         290 │             2 │
│       2 │ 2022-12-02 │           8 │             1 │         110 │             1 │
│       3 │ 2022-12-02 │           6 │             1 │          90 │             1 │
└─────────┴────────────┴─────────────┴───────────────┴─────────────┴───────────────┘
```

```sql
SELECT     
   user_id,
   day,  
   sum(metric1_sum),
   sum(metric1_count),
   sum(metric2_sum),
   sum(metric2_count)
FROM daily_totals
GROUP BY day, user_id
ORDER BY (day, user_id)
```
```
┌─user_id─┬────────day─┬─sum(metric1_sum)─┬─sum(metric1_count)─┬─sum(metric2_sum)─┬─sum(metric2_count)─┐
│       1 │ 2022-12-01 │                8 │                  2 │              220 │                  2 │
│       2 │ 2022-12-01 │                8 │                  2 │              165 │                  2 │
│       1 │ 2022-12-02 │                6 │                  2 │              290 │                  2 │
│       2 │ 2022-12-02 │                8 │                  1 │              110 │                  1 │
│       3 │ 2022-12-02 │                6 │                  1 │               90 │                  1 │
└─────────┴────────────┴──────────────────┴────────────────────┴──────────────────┴────────────────────┘
```