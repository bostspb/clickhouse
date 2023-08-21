# Lab 2. Creating a table
> We have about 2.3M crypto prices stored in a .zip file in S3.
> In this short lab, you are going to simply create a table to store these prices.
> (You will populate the table with data in the next lab.)

## Task 1
Create a new table that satisfies the following requirements:
- The name of the table is `crypto_prices`
- It contains 6 columns:
  - `trade_date` of type Date
  - `crypto_name` of type String
  - 4 columns of type Float32 named `volume`, `price`, `market_cap`, and `change_1_day`
- The primary key is the `crypto_name` first, followed by the `trade_date`

```sql
CREATE TABLE crypto_prices (
   trade_date Date,
   crypto_name String,
   volume Float32,
   price Float32,
   market_cap Float32,
   change_1_day Float32
)
PRIMARY KEY(crypto_name, trade_date);
```
Verify it worked by "describing" the table
```sql
DESC crypto_prices;
```
```
trade_date    Date
crypto_name   String
volume        Float32
price         Float32
market_cap    Float32
change_1_day  Float32
```