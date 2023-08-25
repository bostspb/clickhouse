# 5.1 Writing Queries

## The Dataset
```sql
SHOW CREATE TABLE trips;

-- result --
CREATE TABLE default.trips
(
    `trip_id` UInt32,
    `pickup_datetime` DateTime,
    `dropoff_datetime` DateTime,
    `pickup_longitude` Nullable(Float64),
    `pickup_latitude` Nullable(Float64),
    `dropoff_longitude` Nullable(Float64),
    `dropoff_latitude` Nullable(Float64),
    `passenger_count` UInt8,
    `trip_distance` Float32,
    `fare_amount` Float32,
    `extra` Float32,
    `tip_amount` Float32,
    `tolls_amount` Float32,
    `total_amount` Float32,
    `payment_type` Enum8('CSH' = 1, 'CRE' = 2, 'NOC' = 3, 'DIS' = 4),
    `pickup_ntaname` LowCardinality(String),
    `dropoff_ntaname` LowCardinality(String)
)
ENGINE = SharedMergeTree('/clickhouse/tables/{uuid}/{shard}', '{replica}')
PRIMARY KEY (pickup_datetime, dropoff_datetime)
ORDER BY (pickup_datetime, dropoff_datetime)
SETTINGS index_granularity = 8192;
```

```sql
SELECT count() FROM trips;
-- 4000911
```

## Changing the Output Format 
Format `Vertical`
```sql
SELECT * FROM trips LIMIT 2 FORMAT Vertical
```
```
Row 1:
──────
trip_id:           1203745557
pickup_datetime:   2015-07-01 00:00:09
dropoff_datetime:  2015-07-01 00:06:27
pickup_longitude:  -73.97540283203125
pickup_latitude:   40.75189971923828
dropoff_longitude: -73.99105834960938
dropoff_latitude:  40.750728607177734
passenger_count:   5
trip_distance:     1.12
fare_amount:       6.5
extra:             0.5
tip_amount:        2.34
tolls_amount:      0
total_amount:      10.14
payment_type:      CSH
pickup_ntaname:    Turtle Bay-East Midtown
dropoff_ntaname:   Midtown-Midtown South

Row 2:
──────
trip_id:           1201746944
pickup_datetime:   2015-07-01 00:00:12
dropoff_datetime:  2015-07-01 00:08:33
pickup_longitude:  -73.9787368774414
pickup_latitude:   40.78765869140625
dropoff_longitude: -73.96562194824219
dropoff_latitude:  40.80792999267578
passenger_count:   1
trip_distance:     1.78
fare_amount:       8.5
extra:             0.5
tip_amount:        1.96
tolls_amount:      0
total_amount:      11.76
payment_type:      CSH
pickup_ntaname:    Upper West Side
dropoff_ntaname:   Morningside Heights
```

Format `JSONEachRow`
```sql
SELECT * FROM trips LIMIT 2 FORMAT JSONEachRow 
```
```
{"trip_id":1203745557,"pickup_datetime":"2015-07-01 00:00:09","dropoff_datetime":"2015-07-01 00:06:27","pickup_longitude":-73.97540283203125,"pickup_latitude":40.75189971923828,"dropoff_longitude":-73.99105834960938,"dropoff_latitude":40.750728607177734,"passenger_count":5,"trip_distance":1.12,"fare_amount":6.5,"extra":0.5,"tip_amount":2.34,"tolls_amount":0,"total_amount":10.14,"payment_type":"CSH","pickup_ntaname":"Turtle Bay-East Midtown","dropoff_ntaname":"Midtown-Midtown South"}
{"trip_id":1201746944,"pickup_datetime":"2015-07-01 00:00:12","dropoff_datetime":"2015-07-01 00:08:33","pickup_longitude":-73.9787368774414,"pickup_latitude":40.78765869140625,"dropoff_longitude":-73.96562194824219,"dropoff_latitude":40.80792999267578,"passenger_count":1,"trip_distance":1.78,"fare_amount":8.5,"extra":0.5,"tip_amount":1.96,"tolls_amount":0,"total_amount":11.76,"payment_type":"CSH","pickup_ntaname":"Upper West Side","dropoff_ntaname":"Morningside Heights"}
```

## Export Results to a File

```sql
SELECT * 
FROM trips 
LIMIT 10000 
INTO OUTFILE 'trips.parquet' 
FORMAT Parquet;

-- Query id: 2bf797b5-8db9-4903-b755-63c3b41d89cb
-- 10000 rows in set. Elapsed: 0.012 sec. Processed 24.58 thousand rows, 1.79 MB (2.00 million rows/s., 145.28 MB/s.)
-- Peak memory usage: 37.08 MiB.
```
If you name the file with the `.parquet` extension, ClickHouse will assume you mean the Parquet format 
and you can omit the `FORMAT` clause.

```sql
SELECT * 
FROM trips 
LIMIT 10000
INTO OUTFILE 'trips.tsv.gz';

-- Query id: 232b460c-1ab2-49c0-9365-08586c5bd0bc
-- 10000 rows in set. Elapsed: 0.015 sec. Processed 24.58 thousand rows, 1.79 MB (1.60 million rows/s., 116.34 MB/s.)
-- Peak memory usage: 37.08 MiB.
```

## Analyzing Data

```sql
SELECT
    pickup_datetime,
    passenger_count,
    trip_distance
FROM trips
ORDER BY trip_distance DESC
LIMIT 2
```
```
Query id: c1636aca-e4ae-4f8f-aa95-93db46bbd09f

┌─────pickup_datetime─┬─passenger_count─┬─trip_distance─┐
│ 2015-08-25 13:16:37 │               1 │     1670156.2 │
│ 2015-08-04 06:41:09 │               1 │      940000.5 │
└─────────────────────┴─────────────────┴───────────────┘

2 rows in set. Elapsed: 0.538 sec. Processed 4.00 million rows, 36.01 MB (7.43 million rows/s., 66.88 MB/s.)
Peak memory usage: 38.64 MiB.
```

## The any Function
The any function returns the first non-null value from the aggregated rows in the same group.
```sql
SELECT
    any(dropoff_ntaname),
    count()
FROM trips
GROUP BY pickup_ntaname
ORDER BY 2 DESC
LIMIT 10
```
```
Query id: c7a75421-1b10-434b-8530-a40b9e5ce33c

┌─any(dropoff_ntaname)───────────────────────┬─count()─┐
│ Lenox Hill-Roosevelt Island                │  702262 │
│ Midtown-Midtown South                      │  384396 │
│ SoHo-TriBeCa-Civic Center-Little Italy     │  280516 │
│ Turtle Bay-East Midtown                    │  262767 │
│ Clinton                                    │  246396 │
│ Midtown-Midtown South                      │  201946 │
│ Murray Hill-Kips Bay                       │  193297 │
│ Battery Park City-Lower Manhattan          │  184860 │
│ Upper East Side-Carnegie Hill              │  181154 │
│ Hudson Yards-Chelsea-Flatiron-Union Square │  173677 │
└────────────────────────────────────────────┴─────────┘

10 rows in set. Elapsed: 0.409 sec. Processed 4.00 million rows, 8.00 MB (9.77 million rows/s., 19.54 MB/s.)
Peak memory usage: 16.50 MiB.
```