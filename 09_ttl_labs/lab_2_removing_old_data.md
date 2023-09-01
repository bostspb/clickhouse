# Lab 2. Removing Old Data

```sql
DROP TABLE IF EXISTS crypto_prices;

CREATE TABLE crypto_prices
(
    trade_date Date,
    crypto_name String,
    volume Float32,
    price Float32,
    market_cap Float32,
    change_1_day Float32
)
PRIMARY KEY (crypto_name, trade_date);

INSERT INTO crypto_prices
   SELECT * 
   FROM s3(
    'https://learn-clickhouse.s3.us-east-2.amazonaws.com/crypto_prices.csv',
    'CSVWithNames'
);

SELECT count() FROM crypto_prices
WHERE crypto_name = '';
-- 158849
```

Add a TTL rule to the crypto_prices table that removes rows after 24 hours from the `trade_date` column 
where `crypto_name` is an empty string.
```sql
ALTER TABLE crypto_prices
   MODIFY TTL trade_date + INTERVAL 24 HOUR WHERE crypto_name = '';

SELECT count() 
FROM crypto_prices
WHERE crypto_name = '';
-- 0
```