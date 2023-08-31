# Implementing Deduplication with ReplacingMergeTree

It is quite handy when you want to implement deduplication while inserting rows from multiple clients and/or threads.

```sql
CREATE TABLE hackernews_views_vcmt (
    id UInt32, 
    author String, 
    views UInt64,
    sign Int8,
    version UInt32
) 
ENGINE = VersionedCollapsingMergeTree(sign, version)
PRIMARY KEY (id, author)
```
- It deletes each pair of rows that have the same primary key and version and different sign
- The order that the rows were inserted does not matter
- Note that if the `version` column is not part of the primary key, ClickHouse adds it to the primary 
  key implicitly as the last field

```sql
INSERT INTO hackernews_views_vcmt VALUES 
   (1, 'ricardo', 0, 1, 1),
   (2, 'ch_fan', 0, 1, 1),
   (3, 'kenny', 0, 1, 1);

INSERT INTO hackernews_views_vcmt VALUES 
   (1, 'ricardo', 0, -1, 1),
   (1, 'ricardo', 50, 1, 2),
   (2, 'ch_fan', 0, -1, 1),
   (3, 'kenny', 0, -1, 1),
   (3, 'kenny', 1000, 1, 2);

SELECT
    id,
    author,
    sum(views * sign)
FROM hackernews_views_vcmt
GROUP BY (id, author)
HAVING sum(sign) > 0
ORDER BY id ASC;  
```
```
┌─id─┬─author──┬─sum(multiply(views, sign))─┐
│  1 │ ricardo │                         50 │
│  3 │ kenny   │                       1000 │
└────┴─────────┴────────────────────────────┘
```

```sql
-- We do not recommend forcing a merge of a table in production
OPTIMIZE TABLE hackernews_views_vcmt;

SELECT * FROM hackernews_views_vcmt;
```
```
┌─id─┬─author──┬─views─┬─sign─┬─version─┐
│  1 │ ricardo │    50 │    1 │       2 │
│  3 │ kenny   │  1000 │    1 │       2 │
└────┴─────────┴───────┴──────┴─────────┘
```