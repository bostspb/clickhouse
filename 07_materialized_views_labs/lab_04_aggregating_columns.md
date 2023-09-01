# Lab 4. Aggregating columns

Define a materialized view that satisfies the following requirements:
- The name of the materialized view is `market_caps_view`
- The materialized view contains each `crypto_name` (that is not an empty string) from the `crypto_prices` table, 
  along with the maximum and minimum values of the `market_cap` column for each cryptocurrency
- The materialized view is populated with the existing data in `crypto_price`

```sql
CREATE MATERIALIZED VIEW market_caps_view 
ENGINE = AggregatingMergeTree()
ORDER BY (crypto_name) 
POPULATE AS
SELECT
    crypto_name,
    maxSimpleState(market_cap) AS max_market_cap,
    minSimpleState(market_cap) AS min_market_cap
FROM
    crypto_prices
WHERE
    crypto_name != ''
GROUP BY
    crypto_name;
    

SELECT count() FROM market_caps_view;
-- 4138

SELECT * 
FROM market_caps_view
ORDER BY max_market_cap DESC;
```
```
Innovative Bioresearch Classic  29328328000000000 0
Bitcoin                         326502480000        0
Ethereum                        135400735000        71176660
XRP                             130853470000        171445070
Bitcoin Cash                    66171060000         0
...
```

```sql
SELECT
   max(market_cap),
   min(market_cap)
FROM crypto_prices
WHERE crypto_name = 'Bitcoin';

-- 326502480000   0
```

```sql
INSERT INTO crypto_prices VALUES
   ('2022-12-01', 'Bitcoin', 123, 0.0, 0.0, -1);
   
   
SELECT
    max_market_cap,
    min_market_cap
FROM
    market_caps_view
WHERE
    crypto_name = 'Bitcoin';
    
-- 326502480000   0
-- 0              0    


SELECT
    max_market_cap,
    min_market_cap
FROM
    market_caps_view FINAL
WHERE
    crypto_name = 'Bitcoin';
  
-- 326502480000   0
```