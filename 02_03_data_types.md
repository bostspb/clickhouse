# 2.3 ClickHouse data types

## Overview of ClickHouse data types

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
COMMENT 'My table for storing hits'
```

- `UInt32` is short for unsigned 32-bit integer, so nonnegative integers
- `DateTime` stores an instant in time with second precision
- `String` is for strings. SQL types like `VARCHAR`, `CHAR`, `TEXT` and so on are all aliases for the String type
- Notice the `size` column is `Nullable`, meaning any missing values for it will be `NULL` (instead of the default value for Float, which is 0.0)
- The `params` column is an array of strings, so each row can contain multiple values
- A `FixedString` is a fixed-length string of bytes

> The data types in your tables map to a C++ data type

Available data types
```sql
SELECT * FROM system.data_type_families;
```
few examples
```
┌─name────────────────────────────┬─case_insensitive─┬─alias_to────┐
│ bool                            │                1 │ Bool        │
│ BINARY                          │                1 │ FixedString │
│ boolean                         │                1 │ Bool        │
│ TEXT                            │                1 │ String      │
│ VARCHAR2                        │                1 │ String      │
│ DOUBLE PRECISION                │                1 │ Float64     │
│ VARCHAR                         │                1 │ String      │
│ CHAR                            │                1 │ String      │
│ TIMESTAMP                       │                1 │ DateTime    │
│ FIXED                           │                1 │ Decimal     │
│ NUMERIC                         │                1 │ Decimal     │
│ TIME                            │                1 │ Int64       │
│ FLOAT                           │                1 │ Float32     │
│ CLOB                            │                1 │ String      │
│ BLOB                            │                1 │ String      │
│ INTEGER                         │                1 │ Int32       │
│ INT                             │                1 │ Int32       │
│ DOUBLE                          │                1 │ Float64     │
│ CHARACTER                       │                1 │ String      │
│ BYTE                            │                1 │ Int8        │
└─────────────────────────────────┴──────────────────┴─────────────┘
```

## Nullable vs default values
How Nullable works:
- A missing value is NULL, or
- A missing value takes on the default value of whatever data type it is

```sql
CREATE TABLE null_vs_default (
   id1 UInt32,
   id2 Nullable(UInt32),
   url_1 String,
   url_2 Nullable(String)
)
ORDER BY tuple();

INSERT INTO null_vs_default (id1, id2) VALUES (1,2);

SELECT * FROM null_vs_default;
```
```
┌─id1─┬─id2─┬─url1─┬─url2─┐
│   1 │   2 │      │ ᴺᵁᴸᴸ │
└─────┴─────┴──────┴──────┘
```
```sql
INSERT INTO null_vs_default (url_1, url_2) VALUES ('just','testing');

SELECT * FROM null_vs_default;
```
```
┌─id1─┬─id2─┬─url1─┬─url2─┐
│   1 │   2 │      │ ᴺᵁᴸᴸ │
└─────┴─────┴──────┴──────┘
┌─id1─┬──id2─┬─url1─┬─url2────┐
│   0 │ ᴺᵁᴸᴸ │ just │ testing │
└─────┴──────┴──────┴─────────┘
```

## Numeric types
- **Integers**. You have lots of choices for integers, between signed vs. unsigned, and how many bytes they consume. 
  They include `UInt8`, `UInt16`, `UInt32`, `UInt64`, `UInt128`, `UInt256`, `Int8`, `Int16`, `Int32`, `Int64`, 
  `Int128`, and `Int256`.
- **Floating point numbers**. `Float32` and `Float64` for IEEE 754 standard floating point values.
- **The Decimal family**. Signed fixed-point numbers that keep precision during add, subtract and multiply operations 
  by storing the value as multiple signed integers. They include the types `Decimal(P,S)`, `Decimal32(S)`, 
  `Decimal64(S)`, `Decimal128(S)`, `Decimal256(S)`, where `P` and `S` are the precision and scale.

```sql
SELECT toDecimal128(9.123456789, 12);
-- 9.123456789

SELECT toDecimal128(9.123456789, 5);
-- 9.12345
```

Example
```sql
CREATE TABLE items_for_sale
(
    id UInt32,
    description String,
    rating Float32,
    quantity_in_stock Int32,
    quantity_sold UInt16,
    current_price Decimal64(2),
    sale_price Decimal64(2)
) 
ORDER BY id;

INSERT INTO items_for_sale VALUES
   (1, 'baseball', 4.2, 100, 5000, 6.50, 5.25)
   (2, 'bicycle', 3.99, -3, 21, 599.99, 559.99)
   (3, 'just testing', 4.6666666, 0.0, -10, 5.99999, 4.555555);

SELECT * FROM items_for_sale;
```
```
    ┌─id─┬─description──┬────rating─┬─quantity_in_stock─┬─quantity_sold─┬─current_price─┬─sale_price─┐
    │  1 │ baseball     │       4.2 │               100 │          5000 │           6.5 │       5.25 │
    │  2 │ bicycle      │      3.99 │                -3 │            21 │        599.99 │     559.99 │
!>> │  3 │ just testing │ 4.6666665 │                 0 │         65526 │          5.99 │       4.55 │
    └────┴──────────────┴───────────┴───────────────────┴───────────────┴───────────────┴────────────┘
                         !! ^^^^^ !!
```

## Strings
A String and a FixedString column:
```sql
CREATE TABLE string_demo 
(
   city String,
   state FixedString(2)
)
ORDER BY tuple();

INSERT INTO string_demo VALUES
   ('Albany', 'NY'),
   ('Salem', 'OR'),
   ('Just testing', 'X');

SELECT * FROM string_demo;
```
```
┌─city─────────┬─state─┐
│ Albany       │ NY    │
│ Salem        │ OR    │
│ Just testing │ X     │
└──────────────┴───────┘
```

Insert a string longer than 2 into a FixedString(2) column:
```sql
INSERT INTO string_demo VALUES
   ('Just testing', 'XYZ')

-- Exception on client:
-- Code: 131. DB::Exception: String too long for type FixedString(2): ...
```

String is used as an alias for a long list of SQL types!

## Dates and times
- `Date` A date stored in two bytes as the number of days since 1970-01-01 up to the year 2148
- `Date32` A date stored in four bytes as the number of days since 1900-01-01 up to the year 2299
- `DateTime` An instant in time, with precision up to a second, since 1970-01-01 up to the year 2106
- `DateTime64` An instant in time, with precision up to nanoseconds, since 1900-01-01 up to the year 2299

```sql
CREATE TABLE date_demo (
   a Date,
   b Date32,
   c DateTime('US/Mountain'),
   d DateTime64(3, 'Europe/London')
) ORDER BY tuple();

INSERT INTO date_demo VALUES
(1667181600, '2022-10-31', '2022-10-31 09:45:27', '2022-10-31 09:45:27.123');

SELECT * FROM date_demo
```

Functions for working with dates and times - https://clickhouse.com/docs/en/sql-reference/functions/date-time-functions

## Low cardinality columns
If you have a column that has a relatively small number of unique values:
- Use the `Enum` data type to define an enumeration
- Wrap the column in a `LowCardinality` type
- Prefer `LowCardinality` over `Enum` unless you only have a few unique values. 
  It is nice to be able to add new values, and `LowCardinality` can give you a nice performance boost in both queries and storage.

### Enumerations
```sql
CREATE TABLE enum_demo
(
   device_id UInt32,
   device_type Enum('server' = 1, 'container' = 2, 'router' = 3)
)
ORDER BY tuple();

INSERT INTO enum_demo VALUES 
   (123, 'router'),
   (456, 'container');

INSERT INTO enum_demo VALUES (789, 'pod');
-- Code: 36. DB::Exception: Unknown element 'pod' for enum: While executing ValuesBlockInputFormat: 
-- data for INSERT was parsed from query. (BAD_ARGUMENTS)

SELECT *
FROM enum_demo
WHERE device_type = 'container'

-- ┌─device_id─┬─device_type─┐
-- │       456 │ container   │
-- └───────────┴─────────────┘
```

> !! You can not add values to them without an `ALTER TABLE`

### LowCardinality
- `LowCardinality` can improve performance on very large datasets, even if a column has up to 10,000 unique values
- It is typically used for strings, but you can also use it on `Date`, `DateTime`, and numeric data types.

```sql
CREATE TABLE lowcardinality_demo
(
   x UInt64,
   y LowCardinality(String)
)
ORDER BY x;

INSERT INTO lowcardinality_demo
   SELECT 
      number AS x, 
      randomPrintableASCII(1) AS y
   FROM numbers(10000000);
   
SELECT count()
FROM lowcardinality_demo
WHERE y = 'A'
-- 105157
-- the `y` values are stored as integers!   
```