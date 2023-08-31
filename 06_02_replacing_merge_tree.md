# Implementing Deduplication with ReplacingMergeTree

## An example using upserts
```sql
CREATE TABLE hackernews_rmt (
    id UInt32,
    author String, 
    comment String,
    views UInt64
) 
ENGINE = ReplacingMergeTree 
PRIMARY KEY (author, id);
```
```sql
INSERT INTO hackernews_rmt VALUES 
   (1, 'ricardo', 'This is post #1', 0),
   (2, 'ch_fan', 'This is post #2', 0);
```
```sql
INSERT INTO hackernews_rmt VALUES 
   (1, 'ricardo', 'This is post #1', 100),
   (2, 'ch_fan', 'This is post #2', 200);
```

Result
```sql
SELECT *
FROM hackernews_rmt
```
```
┌─id─┬─author──┬─comment─────────┬─views─┐
│  2 │ ch_fan  │ This is post #2 │     0 │
│  1 │ ricardo │ This is post #1 │     0 │
│  2 │ ch_fan  │ This is post #2 │   200 │
│  1 │ ricardo │ This is post #1 │   100 │
└────┴─────────┴─────────────────┴───────┘
```

Final result (logical result, not physical)
```sql
SELECT *
FROM hackernews_rmt
FINAL
```
```
┌─id─┬─author──┬─comment─────────┬─views─┐
│  2 │ ch_fan  │ This is post #2 │   200 │
│  1 │ ricardo │ This is post #1 │   100 │
└────┴─────────┴─────────────────┴───────┘
```
>Using **FINAL** works OK if you have a small amount of data. If you are dealing with a large amount of data,
>using **FINAL** is probably not the best option.

## Avoiding merges
```sql
INSERT INTO hackernews_rmt VALUES 
   (1, 'ricardo', 'This is post #1', 150),
   (2, 'ch_fan', 'This is post #2', 250);

SELECT * FROM hackernews_rmt;
```
```
┌─id─┬─author──┬─comment─────────┬─views─┐
│  2 │ ch_fan  │ This is post #2 │   200 │
│  1 │ ricardo │ This is post #1 │   100 │
│  2 │ ch_fan  │ This is post #2 │     0 │
│  1 │ ricardo │ This is post #1 │     0 │
│  2 │ ch_fan  │ This is post #2 │   250 │
│  1 │ ricardo │ This is post #1 │   150 │
└────┴─────────┴─────────────────┴───────┘
```
```sql
SELECT
    id,
    author,
    comment,
    max(views)  -- you can use `timestamp` colomn as well
FROM hackernews_rmt
GROUP BY (id, author, comment)
```
```
┌─id─┬─author──┬─comment─────────┬─max(views)─┐
│  2 │ ch_fan  │ This is post #2 │        250 │
│  1 │ ricardo │ This is post #1 │        150 │
└────┴─────────┴─────────────────┴────────────┘
```
Grouping as shown in the query above can actually be more efficient (in terms of query performance) 
than using the `FINAL` keyword. The `FINAL` keyword forces a query to run in a single thread, 
while `GROUP BY` executes in parallel.


## Specify a version column
```sql
CREATE TABLE hackernews_rmt_version (
    id UInt32,
    author String, 
    comment String,
    views UInt64
) 
ENGINE = ReplacingMergeTree(views)  -- `ver` parameter, represents a version column
PRIMARY KEY (author, id)
```
The `ver` parameter must be a column of type unsigned integer, date, or timestamp.

```sql
INSERT INTO hackernews_rmt_version VALUES 
   (1, 'ricardo', 'This is post #1',5),
   (2, 'ch_fan', 'This is post #2', 10),
   (3, 'kenny', 'This is post #3', 20);

INSERT INTO hackernews_rmt_version VALUES 
   (1, 'ricardo', 'This is post #1',105),
   (2, 'ch_fan', 'This is post #2', 110),
   (3, 'kenny', 'This is post #3', 1);

SELECT * FROM hackernews_rmt_version FINAL;
```
```
┌─id─┬─author──┬─comment─────────┬─views─┐
│  2 │ ch_fan  │ This is post #2 │   110 │
│  3 │ kenny   │ This is post #3 │    20 │
│  1 │ ricardo │ This is post #1 │   105 │
└────┴─────────┴─────────────────┴───────┘
```

A better option than using `FINAL` would be to group the results and use an aggregate function like `max`:
```sql
SELECT
    id,
    author,
    max(views)
FROM hackernews_rmt_version
GROUP BY (id, author)
```
```
┌─id─┬─author──┬─max(views)─┐
│  1 │ ricardo │        105 │
│  3 │ kenny   │         20 │
│  2 │ ch_fan  │        110 │
└────┴─────────┴────────────┘
```

> A `ReplacingMergeTree` table is often used in materialized views, where deduplication happens on rows and 
> columns specific to the problem you want to solve.