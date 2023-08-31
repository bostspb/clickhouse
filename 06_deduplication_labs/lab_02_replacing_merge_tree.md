# Lab 2. Implementing Deduplication with ReplacingMergeTree
Suppose you have a use case where you want the rows of a table to contain the latest (most recent) set
of values for each unique user.

## Task 1
Your `message` table should satisfy the following requirements:
- The first column is an integer `id` that is the primary key
- The second column is a DateTime `timestamp`
- The third column is a string for the `message`
- If a new row is inserted into the `messages` table with the same primary key, it replaces any existing 
  rows with a matching primary key

```sql
CREATE TABLE messages (
    id UInt32,
    timestamp DateTime,
    message String
)
ENGINE = ReplacingMergeTree(timestamp)
ORDER BY id;

INSERT INTO messages VALUES 
   (1, now(), 'Message #1'),
   (2, now(), 'Message #2'),
   (3, now(), 'Message #3');
   
INSERT INTO messages VALUES 
   (1, now() + 10, 'New message #1');
```
```
1   2023-08-30 11:00:35     Message #1
2   2023-08-30 11:00:35     Message #2
3   2023-08-30 11:00:35     Message #3
1   2023-08-30 11:01:24     New message #1
```
```sql
SELECT * FROM messages FINAL;
```
```
2   2023-08-30 11:00:35     Message #2
3   2023-08-30 11:00:35     Message #3
1   2023-08-30 11:01:24     New message #1
```