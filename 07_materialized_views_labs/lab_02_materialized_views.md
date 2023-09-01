# Lab 2. Materialized views

The number of times that a cryptocurrency went up in value 1,000% in a single day
```sql
SELECT count()
FROM crypto_prices
WHERE change_1_day > 10;

-- 1003
```

Create a table that keeps track of these anomalies using a materialized view.
```sql
CREATE TABLE big_changes (
    crypto_name String,
    trade_date Date,
    change_1_day Float32
)
ORDER BY (crypto_name, trade_date);
```

Define a materialized view that satisfies the following requirements:
- The name of the view is `big_changes_view`
- The source of the view is `crypto_prices` table
- An entry is added to the view table if the value of `change_1_day` is greater than 10
- The result of the view is stored in the `big_changes` table

```sql
CREATE MATERIALIZED VIEW big_changes_view
TO big_changes
AS
   SELECT 
      crypto_name,
      trade_date,
      change_1_day
    FROM crypto_prices
    WHERE change_1_day > 10;
```

Populate view
```sql
INSERT INTO big_changes
    SELECT 
       crypto_name,
       trade_date,
       change_1_day 
    FROM crypto_prices
       WHERE change_1_day > 10;

select count() from big_changes;
-- 1003

INSERT INTO crypto_prices (crypto_name, trade_date, change_1_day) VALUES
   ('ClickHouse Coin', now(), 50);

SELECT *
FROM big_changes_view 
WHERE crypto_name LIKE 'ClickHouse Coin';

--  ClickHouse Coin     2023-08-31      50
```