# Lab 1. Views

Create a new view for `crypto_prices` named `interesting_crypto_prices` that filters out all rows where `crptyo_name` 
is empty AND where `market_cap` is 0.

```sql
CREATE VIEW interesting_crypto_prices AS 
   SELECT *
   FROM crypto_prices
   WHERE
    crypto_name != ''
    AND market_cap > 0;
```

View the number of rows in your view - you should have 1,835,174
```sql
SELECT count() FROM interesting_crypto_prices;
-- 1835174
```

Using your view, write a query that returns both the maximum and minimum market caps for each cryptocurrency 
(for each `crypto_name`). Sort the results by the maximum market cap descending.
```sql
SELECT
    crypto_name,
    max(market_cap) max_market_cap,
    min(market_cap) min_market_cap
FROM interesting_crypto_prices
GROUP BY crypto_name
ORDER BY max_market_cap DESC;
```
```
Innovative Bioresearch Classic  29328328000000000   667.52
Bitcoin                         326502480000        5496598000
Ethereum                        135400735000        71176660
XRP                             130853470000        171445070
Bitcoin Cash                    66171060000         1354857600
...
```