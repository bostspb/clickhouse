# Lab 3. Inserting data from a file
> In this lab, you populate your `crypto_prices` table with data stored in a CSV file in S3.

## Step 1. CSV example
```
"trade_date","volume","price","market_cap","change_1_day","crypto_name"
"2016-01-01",36278900,434.33,6529299500,0,"Bitcoin"
"2016-01-02",30096600,433.44,6517390300,-0.0020491332,"Bitcoin"
"2016-01-03",39633800,430.01,6467430000,-0.007913437,"Bitcoin"
"2016-01-04",38477500,433.09,6515713500,0.007162624,"Bitcoin"
"2016-01-05",34522600,431.96,6500393500,-0.0026091575,"Bitcoin"
"2016-01-06",34042500,429.11,6458942000,-0.0065978332,"Bitcoin"
"2016-01-07",87562200,458.05,6896279000,0.06744192,"Bitcoin"
"2016-01-08",56993000,453.23,6825700400,-0.0105228685,"Bitcoin"
"2016-01-09",32278000,447.61,6742767000,-0.012399885,"Bitcoin"
```

## Step 2. Inserting
Using the s3 table function, insert all the rows from the file - which is located 
at https://learn-clickhouse.s3.us-east-2.amazonaws.com/crypto_prices.csv - into your `crypto_prices` table.

```sql
INSERT INTO crypto_prices
   SELECT * 
   FROM s3(
    'https://learn-clickhouse.s3.us-east-2.amazonaws.com/crypto_prices.csv',
    'CSVWithNames'
);
```

## Step 3. Verify rows count
Verify it worked by retrieving the number of rows - you should have 2,382,643 rows in your `crypto_prices` table
```sql
SELECT count() FROM crypto_prices;
-- 2 382 643
```

## Step 4. Query
Run the following query, which shows how many prices we have for each type of cryptocurrency:
```sql
SELECT 
   count() AS count,
   crypto_name
FROM crypto_prices
GROUP BY crypto_name
ORDER BY crypto_name;

-- 158849	
-- 84	Governance
-- 449	HBZ coin
-- 1730	Maxcoin
-- 704	Token
-- 602	#MetaHash
-- 42	01coin
-- 209	0cash
-- ...
```
- 158,849 - rows that are not associated with a cryptocurrency 
- The largest quantity of prices per cryptocurrency is about 1,750, 
  which represents about 6 years of daily prices for each crypto