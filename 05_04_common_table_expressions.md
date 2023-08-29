# 5.4 Common Table Expressions

## Common Table Expressions (CTE)
There are two ways to define CTE.

1. Set an expression or value equal to an identifier:
```sql
WITH <expression> AS <identifier>
```

2. Set an identifier equal to the result of a query:
```sql
WITH <identifier> AS <subquery expression>
```

```sql
WITH 
   10000 AS min_votes
SELECT 
   count()
FROM amazon_reviews
WHERE total_votes >= min_votes;

-- 58
```

## Global Aggregations
```sql
WITH (
        SELECT avg(helpful_votes)
        FROM amazon_reviews
    ) AS avg_helpful_votes
SELECT count()
FROM amazon_reviews
WHERE helpful_votes >= avg_helpful_votes;

-- 30051020
```

```sql
WITH (
        SELECT max(helpful_votes)
        FROM amazon_reviews
    ) AS max_helpful_votes
SELECT
    product_title,
    helpful_votes / max_helpful_votes AS percent
FROM amazon_reviews
WHERE percent >= 0.6
```
```
┌─product_title─────────────────────────────────────────┬────────────percent─┐
│ The Mountain Kids 100% Cotton Three Wolf Moon T-Shirt │ 0.8685716690514267 │
└───────────────────────────────────────────────────────┴────────────────────┘
┌─product_title─────────────────────────────────────────────────┬───────────percent─┐
│ BIC Cristal For Her Ball Pen, 1.0mm, Black, 16ct (MSLP16-Blk) │ 0.870991499032068 │
└───────────────────────────────────────────────────────────────┴───────────────────┘
┌─product_title───────────────────────────┬────────────percent─┐
│ Kindle Fire (Previous Generation - 1st) │ 0.6020326571837388 │
└─────────────────────────────────────────┴────────────────────┘
┌─product_title───────────────────────────────────┬────────────percent─┐
│ Kindle Fire HD 7", Dolby Audio, Dual-Band Wi-Fi │ 0.6610765087113879 │
└─────────────────────────────────────────────────┴────────────────────┘
┌─product_title─────────────────────────────────────────┬────────────percent─┐
│ Kindle Keyboard 3G, Free 3G + Wi-Fi, 6" E Ink Display │ 0.6717448026260415 │
└───────────────────────────────────────────────────────┴────────────────────┘
┌─product_title──────────────────────────────────────────────────────┬─percent─┐
│ Kindle: Amazon's Original Wireless Reading Device (1st generation) │       1 │
└────────────────────────────────────────────────────────────────────┴─────────┘

6 rows in set. Elapsed: 0.975 sec. Processed 301.91 million rows, 1.21 GB (309.63 million rows/s., 1.24 GB/s.)
```

## GROUP BY Aggregations
```sql
WITH 
    sum(helpful_votes) AS sum
SELECT
    product_category,
    formatReadableQuantity(sum)
FROM amazon_reviews
GROUP BY product_category
ORDER BY sum DESC;
```
```
┌─product_category─────────┬─formatReadableQuantity(sum)─┐
│ Books                    │ 75.08 million               │
│ Digital_Ebook_Purchase   │ 18.02 million               │
│ Video DVD                │ 14.98 million               │
│ Mobile_Apps              │ 13.26 million               │
│ Music                    │ 12.65 million               │
│ Health & Personal Care   │ 12.27 million               │
│ Kitchen                  │ 10.96 million               │
│ Home                     │ 10.73 million               │
│ PC                       │ 10.25 million               │
│ Beauty                   │ 8.74 million                │
│ Wireless                 │ 8.01 million                │
│ Toys                     │ 7.18 million                │
...
```

## Multiple CTEs
```sql
WITH
    (
        SELECT sum(bytes)
        FROM system.parts
        WHERE active
    ) AS total_disk_usage,
    (
        SELECT max(bytes)
        FROM system.parts
        WHERE active
    ) AS max_bytes
SELECT
    (sum(bytes) / total_disk_usage) * 100 AS table_disk_usage,
    (max(bytes) / max_bytes) * 100 AS max_bytes_percent,
    table
FROM system.parts
GROUP BY table
ORDER BY table_disk_usage DESC;
```

A view can be defined from a query that uses a CTE.
```sql
CREATE VIEW test AS
WITH
    (
        SELECT sum(bytes)
        FROM system.parts
        WHERE active
    ) AS total_disk_usage,
    ( SELECT max(bytes)
        FROM system.parts
        WHERE active
    ) AS max_bytes
SELECT
    (sum(bytes) / total_disk_usage) * 100 AS table_disk_usage,
    (max(bytes) / max_bytes) * 100 AS max_bytes_percent,
    table
FROM system.parts
GROUP BY table;
```