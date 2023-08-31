# Implementing Deduplication with ReplacingMergeTree

## Updating numeric columns frequently
```sql
CREATE TABLE hackernews_views (
    id UInt32,
    author String, 
    views UInt64,
    -- the sign column can only be 1 or -1
    -- the `Int8` data type is required
    sign Int8  
) 
ENGINE = CollapsingMergeTree(sign)  
PRIMARY KEY (id, author);

INSERT INTO hackernews_views VALUES 
   (123, 'ricardo', 0, 1);
   
INSERT INTO hackernews_views VALUES 
   (123, 'ricardo', 0, -1),
   (123, 'ricardo', 150, 1);

SELECT * FROM hackernews_views;
```
```
┌──id─┬─author──┬─views─┬─sign─┐
│ 123 │ ricardo │     0 │   -1 │
│ 123 │ ricardo │   150 │    1 │
│ 123 │ ricardo │     0 │    1 │
└─────┴─────────┴───────┴──────┘
```
```sql
SELECT * FROM hackernews_views FINAL;
```
```
┌──id─┬─author──┬─views─┬─sign─┐
│ 123 │ ricardo │   150 │    1 │
└─────┴─────────┴───────┴──────┘
```

## Query the data
How do you retrieve the current state row without using `FINAL`?
```sql
INSERT INTO hackernews_views VALUES 
   (123, 'ricardo', 150, -1),
   (123, 'ricardo', 400, 1);

SELECT * FROM hackernews_views;
``` 
```
┌──id─┬─author──┬─views─┬─sign─┐
│ 123 │ ricardo │     0 │   -1 │

│ 123 │ ricardo │   150 │    1 │
│ 123 │ ricardo │     0 │    1 │

│ 123 │ ricardo │   150 │   -1 │
│ 123 │ ricardo │   400 │    1 │
└─────┴─────────┴───────┴──────┘
```
```sql
SELECT
   id,
   author,
   sum(views*sign)
FROM hackernews_views
GROUP BY (id, author)
HAVING sum(sign) > 0;
``` 
```
┌──id─┬─author──┬─sum(multiply(views, sign))─┐
│ 123 │ ricardo │                        400 │
└─────┴─────────┴────────────────────────────┘
```