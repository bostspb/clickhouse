# Lab 3. ClickHouse data types

## Overview of ClickHouse data types
The names of the columns based on the header row of the CSV file 
`https://learn-clickhouse.s3.us-east-2.amazonaws.com/crypto_raw.csv.gz`
```
trade_date,
volume,
price_usd,
price_btc,
market_cap,
capitalization_change_1_day,
USD_price_change_1_day,
BTC_price_change_1_day,
crypto_name,
crypto_type,
ticker,
max_supply,
site_url,
github_url,
minable,
platform_name,
industry_name
```

```sql
SELECT * 
FROM s3(
   'https://learn-clickhouse.s3.us-east-2.amazonaws.com/crypto_raw.csv.gz',
   'CSVWithNames'
)
LIMIT 100;
-- Without errors

SELECT *
FROM s3(
   'https://learn-clickhouse.s3.us-east-2.amazonaws.com/crypto_raw.csv.gz',
   'CSVWithNames'
)
-- With error at 48514 row
```
```
Code: 27. DB::ParsingException: Cannot parse input: expected ',' before: '.31,0.000395,0.000000041964981020142,72674823,0.019815795349645,0.020671834625323,0.0014503552050896,Bytecoin,0,BCN,184470000000,https://bytecoin.org/,https://g': 
Row 48514:
Column 0,   name: trade_date,                  type: Nullable(Date),    parsed text: "2019-11-03"
Column 1,   name: volume,                      type: Nullable(Int64),   parsed text: "5789"
Column 2,   name: price_usd,                   type: Nullable(Float64), parsed text: "0.000387"
Column 3,   name: price_btc,                   type: Nullable(Float64), parsed text: "0.00000004190420503825"
Column 4,   name: market_cap,                  type: Nullable(Int64),   parsed text: "71262696"
Column 5,   name: capitalization_change_1_day, type: Nullable(Float64), parsed text: "0.0069347570243618"
Column 6,   name: USD_price_change_1_day,      type: Nullable(Float64), parsed text: "0.0078124999999999"
Column 7,   name: BTC_price_change_1_day,      type: Nullable(Float64), parsed text: "0.017565048969449"
Column 8,   name: crypto_name,                 type: Nullable(String),  parsed text: "Bytecoin"
Column 9,   name: crypto_type,                 type: Nullable(Int64),   parsed text: "0"
Column 10,  name: ticker,                      type: Nullable(String),  parsed text: "BCN"
Column 11,  name: max_supply,                  type: Nullable(Int64),   parsed text: "184470000000"
Column 12,  name: site_url,                    type: Nullable(String),  parsed text: "https://bytecoin.org/"
Column 13,  name: github_url,                  type: Nullable(String),  parsed text: "https://github.com/bcndev"
Column 14,  name: minable,                     type: Nullable(Int64),   parsed text: "1"
Column 15,  name: platform_name,               type: Nullable(String),  parsed text: "Wanchain"
Column 16,  name: industry_name,               type: Nullable(String),  parsed text: "Privacy"
```

Ignore errors.  it stops at error number 101
```sql
SELECT *
FROM s3(
   'https://learn-clickhouse.s3.us-east-2.amazonaws.com/crypto_raw.csv.gz',
   'CSVWithNames'
)
SETTINGS input_format_allow_errors_num=100;
```
```
Code: 27. DB::ParsingException: Cannot parse input: expected ',' before: '.84,0.000497,0.00000005130883998662,91511901,-0.067701279071788,-0.067542213883677,-0.075046974043822,Bytecoin,0,BCN,184470000000,https://bytecoin.org/,https://': (Already have 101 errors out of 48623 rows, which is 0.0020772062604117393 of all rows): 
Row 48622:
Column 0,   name: trade_date,                  type: Nullable(Date),    parsed text: "2020-02-20"
Column 1,   name: volume,                      type: Nullable(Int64),   parsed text: "16247"
ERROR: garbage after Nullable(Int64): ".15,0.0005"
: While executing ParallelParsingBlockInputFormat: While executing S3: (at row 48623)
. (CANNOT_PARSE_INPUT_ASSERTION_FAILED) (version 23.8.1.41455 (official build))
```
So.. the issue is actually not with our dataset

```sql
SELECT *
FROM s3(
   'https://learn-clickhouse.s3.us-east-2.amazonaws.com/crypto_raw.csv.gz',
   'CSVWithNames'
)
LIMIT 50000
SETTINGS schema_inference_hints='volume Float32, market_cap Float32';

-- Without errors
```

## Task
Create a new table named `crypto_raw` that contains the 17 columns listed earlier. 
You can basically use any data types you prefer, but here are some tips to follow:
- Make `crypto_type` a `Nullable(UInt8)`, since it is either 0, 1 or null
- Make `minable` a `UInt8` since it is only 0 or 1
- Use `LowCardinality` for the `crypto_name`, `ticker`, `platform_name`, and `industry_name` columns
- Pick a primary key that makes sense for the types of queries you want to write

```sql
CREATE TABLE crypto_raw (
    trade_date Date,
    volume Float32,
    price_usd Float32,
    price_btc Float32,
    market_cap Float32,
    capitalization_change_1_day Float32,
    USD_price_change_1_day Float32,
    BTC_price_change_1_day Float32,
    crypto_name LowCardinality(String),
    crypto_type Nullable(UInt8),
    ticker LowCardinality(String),
    max_supply Float32,
    site_url LowCardinality(String),
    github_url LowCardinality(String),
    minable UInt8,
    platform_name LowCardinality(String),
    industry_name LowCardinality(String)
)
PRIMARY KEY (crypto_name, trade_date);
```

Insert all of the rows from the CSV file in S3 into crypto_raw
```sql
INSERT INTO crypto_raw 
   SELECT *
   FROM s3(
      'https://learn-clickhouse.s3.us-east-2.amazonaws.com/crypto_raw.csv.gz',
      'CSVWithNames'
   )
   SETTINGS schema_inference_hints='volume Float32, market_cap Float32';

SELECT count() FROM crypto_raw;
-- 2382643
```

Which cryptocurrencies have had the overall highest price
```sql
SELECT 
   crypto_name,
   max(price_usd) AS m,
   max(price_btc)
FROM crypto_raw
GROUP BY crypto_name
ORDER BY m DESC;
```
```
Travel1Click                15104564000     1472360.6
Project-X                   2300740         595.36896
LOOPREX                     270822          23.436916
42-coin                     127515          11.145734
Robonomics Web Services     107552          9.42279
...
```

Which platforms trade the most cryptocurrencies
```sql
SELECT 
   platform_name,
   uniq(crypto_name) AS count
FROM crypto_raw
WHERE platform_name != ''
GROUP BY platform_name
ORDER BY count DESC;
```
```
XRP         352
Ontology    167
EOS         153
VITE        139
INT Chain   128
...
```

Counts the number of days that a price dropped, for each crypto
```sql
SELECT 
   crypto_name,
   count() AS count
FROM crypto_raw
WHERE USD_price_change_1_day < 0
AND crypto_name != ''
GROUP BY crypto_name
ORDER BY count DESC;
```
```
Yocoin          942
TransferCoin    942
Feathercoin     942
Expanse         936
XRP             933
```

The number of times a cryptocurrency lost at least half of its value in a single day
```sql
SELECT 
   crypto_name,
   count() AS count
FROM crypto_raw
WHERE USD_price_change_1_day < -0.5
AND crypto_name != ''
GROUP BY crypto_name
ORDER BY count DESC;
```
```
TRONCLASSIC 138
SuperCoin   120
Mooncoin    101
PopularCoin 101
Dimecoin    100
```
