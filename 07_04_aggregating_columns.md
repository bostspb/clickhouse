# 7.4 Aggregating columns

## The AggregatingMergeTree table engine
What's unique about `AggregatingMergeTree` are the data types used for storing the aggregations - they are ClickHouse 
**state functions**. State functions are created by appending `SimpleState` or `State` to the end of a ClickHouse 
function.
```sql
CREATE MATERIALIZED VIEW max_and_min
ENGINE = AggregatingMergeTree() 
ORDER BY (user_id)
POPULATE AS 
   SELECT
      user_id,
      maxSimpleState(metric1) AS max_metric1,
      minSimpleState(metric2) AS min_metric2
   FROM metrics
   GROUP BY user_id;

SELECT * FROM max_and_min;
```
```
┌─user_id─┬─max_metric1─┬─min_metric2─┐
│       1 │         500 │         100 │
│       2 │          80 │          75 │
│       3 │          90 │          89 │
└─────────┴─────────────┴─────────────┘
```

## Using SimpleState Combinators

A list of all the ClickHouse functions that can be appended with `SimpleState` for use 
in an `AggregatingMergeTree` table:
```
any
anyLast
min
max
sum
sumWithOverflow
groupBitAnd
groupBitOr
groupBitXor
groupArrayArray
groupUniqArrayArray
sumMap
minMap
maxMap
```
The values stored in these data types can be retrieved by simply selecting the column, 
which is not the case for `State` combinators, which require a special `Merge` combinator to view the data.

## Using State Combinators
- **uniq** - and the other functions for counting unique values (uniqExact, uniqCombined, uniqHLL12, and so on)
- **quantiles** - which includes quantiles, quantilesDeterministic, quantilesTiming and others
- **avg** - the average function

```sql
CREATE MATERIALIZED VIEW daily_totals
ENGINE = AggregatingMergeTree() 
ORDER BY day
POPULATE AS 
   SELECT
      toYYYYMMDD(timestamp) as day,
      avgState(metric1) AS avg_metric1,
      avgState(metric2) AS avg_metric2,
      uniqState(user_id) AS user_count
   FROM metrics
   GROUP BY day
```
- All rows in the metrics table from the same day will be aggregated into a single row in daily_totals
- avgState and uniqState are aggregate function data types that store more than a single value
  - an average has to be a sum of numbers along with the number of values
- You're grouping the results by day, so the materialized view has to be sorted by day
- The view was populated, and you can select the columns in daily_totals, but their results are not human-readable

```sql
SELECT * FROM states_example
```
```
┌──────day─┬─avg_metric1─┬─avg_metric2─┬─user_count─┐
│ 20221105 │             │            │ ���	,��4�e │
│ 20221106 │             │            │ ���	,��4�e │
│ 20221201 │             │ �            │ ,��4�e        │
│ 20221202 │             │ �            │ ���	,��4�e │
└──────────┴─────────────┴─────────────┴────────────┘
```

## Viewing state data types
```sql
SELECT
    avgMerge(avg_metric1),
    avgMerge(avg_metric2),
    uniqMerge(user_count)
FROM daily_totals
```
```
┌─avgMerge(avg_metric1)─┬─avgMerge(avg_metric2)─┬─uniqMerge(user_count)─┐
│    124.83333333333333 │    109.29166666666667 │                     3 │
└───────────────────────┴───────────────────────┴───────────────────────┘
```