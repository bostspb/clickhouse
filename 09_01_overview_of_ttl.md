# 9.1 Overview of TTL

## Use case for TTL
TTL (time-to-live) can be applied to entire tables or specific columns.

Use cases:
- **Removing old data**: you can delete rows or columns after a specified time interval
- **Moving data between disks**: after a certain amount of time, you can move data between storage 
  volumes - useful for deploying a hot/warm/cold architecture
- **Data rollup**: rollup your older data into various useful aggregations and computations before deleting it

## TTL Syntax 
```
CREATE TABLE table_name
(
    name1 [type1] [TTL expr1],
    name2 [type2] [TTL expr2],
    ...
) 
ENGINE = MergeTree()
ORDER BY expr
[TTL expr
    [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx' [, ...] ]
    [WHERE conditions]
    [GROUP BY key_expr [SET v1 = aggr_func(v1) [, v2 = aggr_func(v2) ...]] ] ]
```
```sql
CREATE TABLE example1 (
   timestamp DateTime,
   x UInt32 TTL now() + INTERVAL 1 MONTH,
   y String TTL timestamp + INTERVAL 1 DAY,
   z String
)
ENGINE = MergeTree
ORDER BY tuple()
```
- The `x` column has a time to live of 1 month from now
- The `y` column has a time to live of 1 day from the `timestamp` column

```sql
SHOW CREATE TABLE example1;

CREATE TABLE default.example1
(
    `timestamp` DateTime,
    `x` UInt32 TTL now() + toIntervalMonth(1),
    `y` String TTL timestamp + toIntervalDay(1),
    `z` String
)
ENGINE = MergeTree
ORDER BY tuple()
SETTINGS index_granularity = 8192 
```

## Altering or Adding a TTL Rule
```sql
ALTER TABLE example1 
MODIFY TTL
   timestamp + INTERVAL 2 MONTH
```

A new TTL rule is not applied to the existing rows in the table. 
If you want your TTL rule to be applied to the existing rows, you need to materialize it.
```sql
ALTER TABLE example1 MATERIALIZE TTL
```

The actual deletion of rows does not happen until its parts get merged.

## Triggering TTL Events
The deleting or aggregating of expired rows is not immediate - it only occurs during table merges. 
If you have a table that's not actively merging (for whatever reason), there are two settings that trigger TTL events:
- `merge_with_ttl_timeout`: the minimum delay in seconds before repeating a merge with delete TTL. 
  The default is 14400 seconds (4 hours).
- `merge_with_recompression_ttl_timeout`: the minimum delay in seconds before repeating a merge with 
  recompression TTL (rules that roll up data before deleting). Default value: 14400 seconds (4 hours).

> `OPTIMIZE` initializes an unscheduled merge of the parts of your table, 
> and `FINAL` forces a reoptimization if your table is already a single part. 
> (not recommended for frequent use)