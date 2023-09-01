# 8.2 Defining Projections

## Defining a projection in a new table
```sql
CREATE TABLE hackernews2 (
    id UInt32, 
    type LowCardinality(String), 
    author String, 
    timestamp DateTime, 
    comment String, 
    children Array(UInt32),
    PROJECTION author_projection (
      SELECT * ORDER BY author
    )
) 
ENGINE = MergeTree 
PRIMARY KEY (type, toYYYYMMDD(timestamp))
```

## Add a project to an existing table
```sql
ALTER TABLE hackernews2
ADD PROJECTION author_count_projection
   (
      SELECT 
         author, 
         count() AS count
      GROUP BY
         author
   )
```

**The projection is not materialized**, so only new rows being inserted will appear in the projection. 
To materialize the existing rows, use the `MATERIALIZE PROJECTION` command:
```sql
ALTER TABLE hackernews2 MATERIALIZE PROJECTION author_count_projection
```

```sql
SELECT
    table,
    formatReadableSize(sum(bytes)) AS size
FROM system.parts
WHERE active AND (table LIKE 'hackernews%')
GROUP BY table;

-- hackernews2  690.60 MiB
-- hackernews   343.84 MiB
```