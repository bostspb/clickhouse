# Lab 3. Implementing Deduplication with CollapsingMergeTree

```sql
DROP TABLE messages;

CREATE TABLE messages (
    id UInt32,
    timestamp DateTime,
    message String,
    sign Int8
)
ENGINE = CollapsingMergeTree(sign)
ORDER BY id;

INSERT INTO messages VALUES 
   (1, now(), 'Message #1', 1),
   (2, now(), 'Message #2', 1),
   (3, now(), 'Message #3', 1);

INSERT INTO messages VALUES 
   (2, null, null, -1),
   (2, now(), 'New message #2', 1);

SELECT * FROM messages;
```
```
1   2023-08-30 12:04:40     Message #1       1
2   2023-08-30 12:04:40     Message #2       1
3   2023-08-30 12:04:40     Message #3       1
2   1970-01-01 00:00:00                     -1
2   2023-08-30 12:04:40     New message #2   1
```

```sql
SELECT * FROM messages FINAL
```
```
3   2023-08-30 12:04:40     Message #3      1
1   2023-08-30 12:04:40     Message #1      1
2   2023-08-30 12:04:40     New message #2  1
```