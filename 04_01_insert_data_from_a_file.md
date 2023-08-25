# 4.2 Insert data from a file
> [Table Functions](https://clickhouse.com/docs/en/sql-reference/table-functions)

## Table functions and inferring schema
The number of rows processed defaults to **25,000**, 
but you can specify the number of rows using the `input_format_max_rows_to_read_for_schema_inference` setting.

```sql
SELECT * 
FROM url(
   'https://datasets-documentation.s3.eu-west-3.amazonaws.com/nyc-taxi/trips_0.gz',
   'TabSeparatedWithNames',
   'auto'  -- default: infer the column names and data types based on the data
) 
LIMIT 100
```

View the schema of the inferred table
```sql
DESCRIBE url(
   'https://datasets-documentation.s3.eu-west-3.amazonaws.com/nyc-taxi/trips_0.gz',
   'TabSeparatedWithNames',
   'auto'
)
```
```
┌─name──────────────────┬─type───────────────┐
│ trip_id               │ Nullable(Float64)  │
│ vendor_id             │ Nullable(Float64)  │
│ pickup_date           │ Nullable(String)   │
│ pickup_datetime       │ Nullable(String)   │
│ dropoff_date          │ Nullable(String)   │
│ dropoff_datetime      │ Nullable(String)   │
│ store_and_fwd_flag    │ Nullable(Float64)  │
│ rate_code_id          │ Nullable(Float64)  │
│ pickup_longitude      │ Nullable(Float64)  │
...
```
The numeric fields are set to `Nullable(Float64)`, and all other fields are `Nullable(String)`

## Create the table
```sql
CREATE TABLE trips (
    trip_id             UInt32,
    pickup_datetime     DateTime,
    dropoff_datetime    DateTime,
    pickup_longitude    Nullable(Float64),
    pickup_latitude     Nullable(Float64),
    dropoff_longitude   Nullable(Float64),
    dropoff_latitude    Nullable(Float64),
    passenger_count     UInt8,
    trip_distance       Float32,
    fare_amount         Float32,
    extra               Float32,
    tip_amount          Float32,
    tolls_amount        Float32,
    total_amount        Float32,
    payment_type        Enum('CSH' = 1, 'CRE' = 2, 'NOC' = 3, 'DIS' = 4),
    pickup_ntaname      LowCardinality(String),
    dropoff_ntaname     LowCardinality(String)
)
PRIMARY KEY (pickup_datetime, dropoff_datetime)
```

## Populate the table
We are only selecting and inserting the columns from the file that we actually want inserted into our trips table.
```sql
INSERT INTO trips  
SELECT 
    trip_id,
    pickup_datetime,
    dropoff_datetime,
    pickup_longitude,
    pickup_latitude,
    dropoff_longitude,
    dropoff_latitude,
    passenger_count,
    trip_distance,
    fare_amount,
    extra,
    tip_amount,
    tolls_amount,
    total_amount,
    payment_type,
    pickup_ntaname,
    dropoff_ntaname 
FROM 
   url(
      'https://datasets-documentation.s3.eu-west-3.amazonaws.com/nyc-taxi/trips_0.gz',
      'TabSeparatedWithNames'
   );   
   
SELECT count() FROM trips;
-- 1000660 
```

You can also use curly braces to specify a range of numbers in file names. 
For example, the following format reads three files: `trips_1.gz`, `trips_2.gz` and `trips_3.gz`.

```sql
INSERT INTO trips  
SELECT 
    trip_id,
    pickup_datetime,
    dropoff_datetime,
    pickup_longitude,
    pickup_latitude,
    dropoff_longitude,
    dropoff_latitude,
    passenger_count,
    trip_distance,
    fare_amount,
    extra,
    tip_amount,
    tolls_amount,
    total_amount,
    payment_type,
    pickup_ntaname,
    dropoff_ntaname 
FROM 
   url(
      'https://datasets-documentation.s3.eu-west-3.amazonaws.com/nyc-taxi/trips_{1..3}.gz',
      'TabSeparatedWithNames'
   );

SELECT count() FROM trips;
-- 4000911
```


## Query the table
How quickly you can calculate
```sql
SELECT avg(fare_amount) FROM trips;

-- 13.12109255293824

-- Elapsed: 0.093s    Read: 4,000,911 rows (16.00 MB)
```