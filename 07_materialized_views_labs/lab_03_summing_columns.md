# Lab 3. Summing columns

Create a new materialized view that satisfies the following requirements:
- The name of the view is `sum_of_volumes`
- The source table of the view is `crypto_prices`
- The `sum_of_volumes` view has two columns: `crypto_name` and `total_volume`, 
  which is a running total of the `volume` column of each `crypto_name`
- The view is populated with the existing data in `crypto_prices` 

```sql
CREATE MATERIALIZED VIEW sum_of_volumes
ENGINE = SummingMergeTree()
ORDER BY crypto_name
POPULATE AS
    SELECT
       crypto_name,
       volume AS total_volume
    FROM crypto_prices;

SELECT 
   crypto_name,
   sum(total_volume) AS total_volume
FROM sum_of_volumes 
GROUP BY crypto_name
ORDER BY total_volume DESC
LIMIT 10;
```
```
Tether            39485760864256
Bitcoin           36358320029696
Ethereum          15375675686912
Litecoin          4459883134976
EOS               3659922669568
Bitcoin Cash      3539194871808
TRON              1446042664960
XRP               1429454585856
Ethereum Classic  1421308067840
Bitcoin SV        1244662726656
```

```sql
INSERT INTO crypto_prices (crypto_name, volume) VALUES
   ('Tether', 1),
   ('Bitcoin', 1);
```
```
Tether            39485760864257
Bitcoin           36358320029697
Ethereum          15375675686912
Litecoin          4459883134976
EOS               3659922669568
Bitcoin Cash      3539194871808
TRON              1446042664960
XRP               1429454585856
Ethereum Classic  1421308067840
Bitcoin SV        1244662726656
```