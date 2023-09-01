# 9.3 Rolling Up Data

## Implementing a Rollup
```sql
CREATE TABLE hits (
   timestamp DateTime,
   id String,
   hits Int32
)
ENGINE = MergeTree
PRIMARY KEY (id, timestamp);
```

```sql
CREATE TABLE hits (
   timestamp DateTime,
   id String,
   hits Int32,
   max_hits Int32 DEFAULT hits,
   sum_hits Int64 DEFAULT hits
)
ENGINE = MergeTree
PRIMARY KEY (id, toStartOfHour(timestamp), timestamp)
TTL timestamp + INTERVAL 15 SECOND
    GROUP BY id, toStartOfHour(timestamp)
    SET 
        max_hits = max(max_hits),
        sum_hits = sum(sum_hits);
```
- Obviously 15 seconds is too short of a TTL, but we are setting it low so we can verify our rollups are working.
- The `GROUP BY` columns in the `TTL` clause must be a prefix of the `PRIMARY KEY`, and we want to group our results by 
  the start of each hour. Therefore, `toStartOfHour(timestamp)` was added to the primary key.
- We added two fields to store the aggregated results: `max_hits` and `sum_hits`.
- Setting the default value of `max_hits` and `sum_hits` to `hits` is necessary for our logic to work, based on 
  how the `SET` clause is defined.

```sql
INSERT INTO hits (timestamp, id, hits) VALUES
   (now(), 'home', 100),
   (now(), 'home', 300),
   (now() + INTERVAL 1 MINUTE, 'home', 200),
   (now(), 'contactus', 5),
   (now(), 'contactus', 15),
   (now() + INTERVAL 1 MINUTE, 'contactus', 20);

SELECT * FROM hits
```
```
┌───────────timestamp─┬─id────────┬─hits─┬─max_hits─┬─sum_hits─┐
│ 2023-01-22 07:17:27 │ contactus │    5 │        5 │        5 │
│ 2023-01-22 07:17:27 │ contactus │   15 │       15 │       15 │
│ 2023-01-22 07:18:27 │ contactus │   20 │       20 │       20 │
│ 2023-01-22 07:17:27 │ home      │  100 │      100 │      100 │
│ 2023-01-22 07:17:27 │ home      │  300 │      300 │      300 │
│ 2023-01-22 07:18:27 │ home      │  200 │      200 │      200 │
└─────────────────────┴───────────┴──────┴──────────┴──────────┘

After 15 seconds
┌───────────timestamp─┬─id────────┬─hits─┬─max_hits─┬─sum_hits─┐
│ 2023-01-22 07:18:27 │ contactus │   20 │       20 │       20 │
│ 2023-01-22 07:17:27 │ contactus │    5 │       15 │       20 │
│ 2023-01-22 07:18:27 │ home      │  200 │      200 │      200 │
│ 2023-01-22 07:17:27 │ home      │  100 │      300 │      400 │
└─────────────────────┴───────────┴──────┴──────────┴──────────┘
```

```sql
-- After 1 minute
OPTIMIZE TABLE hits FINAL;

SELECT * FROM hits;
```
```
┌───────────timestamp─┬─id────────┬─hits─┬─max_hits─┬─sum_hits─┐
│ 2023-01-22 07:18:27 │ contactus │   20 │       20 │       40 │
│ 2023-01-22 07:18:27 │ home      │  200 │      300 │      600 │
└─────────────────────┴───────────┴──────┴──────────┴──────────┘
```

When it comes time to analyze the rows after lots of data is inserted over time, 
you will use a query that groups by your TTL grouping
```sql
SELECT
    id,
    toStartOfHour(timestamp),
    max(max_hits),
    sum(sum_hits)
FROM hits
GROUP BY
    id,
    toStartOfHour(timestamp)

┌─id────────┬─toStartOfHour(timestamp)─┬─max(max_hits)─┬─sum(sum_hits)─┐
│ home      │      2023-01-22 07:00:00 │           300 │           600 │
│ contactus │      2023-01-22 07:00:00 │            20 │            40 │
└───────────┴──────────────────────────┴───────────────┴───────────────┘
```