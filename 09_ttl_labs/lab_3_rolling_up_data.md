# 9.3 Rolling Up Data

Add a TTL rule to `crypto_prices` that satisfies the following requirements:
- It rolls up the sum of the `volume` column for each `crypto_name` and stores it in a new column 
  named `sum_volume` of type `Float32`.
- It rolls up the maximum price for each `crypto_name` and stores it in a new column named `max_price` of type `Float32`.
- The TTL for each row is 1 day past the `trade_date`.

```sql
ALTER TABLE crypto_prices
   ADD COLUMN sum_volume Float32 DEFAULT volume,
   ADD COLUMN max_price Float32 DEFAULT price,
   MODIFY TTL 
      trade_date + INTERVAL 1 DAY
      GROUP BY crypto_name
      SET sum_volume = sum(sum_volume),
          max_price = max(max_price);

SELECT *
FROM crypto_prices
WHERE crypto_name = 'Bitcoin';
```
```
┌─trade_date─┬─crypto_name─┬───volume─┬──price─┬─market_cap─┬─change_1_day─┬─────sum_volume─┬─max_price─┐
│ 2016-01-01 │ Bitcoin     │ 36278900 │ 434.33 │ 6529299500 │            0 │ 18179162000000 │   19497.4 │
└────────────┴─────────────┴──────────┴────────┴────────────┴──────────────┴────────────────┴───────────┘
```