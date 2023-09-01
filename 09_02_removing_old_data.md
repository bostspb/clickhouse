# 9.2 Removing Old Data

```sql
CREATE TABLE customers (
   timestamp DateTime,
   name String,
   balance Int32,
   address String 
)
ENGINE = MergeTree
ORDER BY timestamp
TTL timestamp + INTERVAL 15 SECOND;

INSERT INTO customers VALUES 
   (now(), 'Ricardo', 123, 'New York'),
   (now(), 'Susanna', 456, 'London'),
   (now() + INTERVAL 2 MINUTE, 'Manish', 789, 'Madrid');

SELECT * FROM customers; -- afrer 15 sec and after 2 min
```
```
┌───────────timestamp─┬─name────┬─balance─┬─address──┐
│ 2023-01-21 18:21:28 │ Ricardo │     123 │ New York │
│ 2023-01-21 18:21:28 │ Susanna │     456 │ London   │
│ 2023-01-21 18:23:28 │ Manish  │     789 │ Madrid   │
└─────────────────────┴─────────┴─────────┴──────────┘
```

```sql
OPTIMIZE TABLE customers FINAL;

SELECT * FROM customers;

0 rows in set. Elapsed: 0.007 sec.
```

## Removing Columns
```sql
ALTER TABLE customers
   MODIFY TTL timestamp + INTERVAL 1 HOUR;
   
INSERT INTO customers VALUES 
   (now() + INTERVAL 1 MINUTE, 'Ricardo', 123, 'New York'),
   (now() + INTERVAL 2 MINUTE, 'Susanna', 456, 'London'),
   (now() + INTERVAL 1 HOUR, 'Manish', 789, 'Madrid');   

ALTER TABLE customers
MODIFY COLUMN balance Int32 TTL timestamp + INTERVAL 15 SECOND,
MODIFY COLUMN address String TTL timestamp + INTERVAL 15 SECOND;

ALTER TABLE customers MATERIALIZE TTL;   
```

After 1 minute and 15 seconds (from the time the rows were inserted), the first row will have expired columns. 
Similarly, after 2 minutes and 15 seconds the second row will have expired columns. 
Notice the Int32 column is 0 and the String column is an empty string after 2 minutes and 15 seconds:
```sql
SELECT * FROM customers
```
```
┌───────────timestamp─┬─name────┬─balance─┬─address─┐
│ 2023-01-21 19:23:57 │ Ricardo │       0 │         │
│ 2023-01-21 19:24:57 │ Susanna │       0 │         │
│ 2023-01-21 20:22:57 │ Manish  │     789 │ Madrid  │
└─────────────────────┴─────────┴─────────┴─────────┘
```