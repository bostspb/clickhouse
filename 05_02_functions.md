# 5.2 Using Functions

## Types of Functions
There are about 1,300 functions.
- **[Regular functions](https://clickhouse.com/docs/en/sql-reference/functions).** 
  Also just called functions, regular functions work as if they are applied to each row separately 
  (for each row, the result of the function does not depend on the other rows).
- **[Aggregated functions](https://clickhouse.com/docs/en/sql-reference/aggregate-functions).** 
  They accumulate a set of values from multiple rows.
- **[Windows functions](https://clickhouse.com/docs/en/sql-reference/window-functions).** 
  ClickHouse supports the standard SQL grammar for defining windows and window functions.
- **[Table functions](https://clickhouse.com/docs/en/sql-reference/table-functions).** 
  Functions that create tables, typically from an external resource like a file or another database.

## How many functions does ClickHouse have?
```sql
SELECT count()
FROM system.functions

-- 1,416
```

## The max and argMax Functions
```sql
SELECT
    max(total_votes),
    argMax(review_headline, total_votes)
FROM amazon_reviews
```
```
┌─max(total_votes)─┬─argMax(review_headline, total_votes)──────┐
│            48362 │ Why and how the Kindle changes everything │
└──────────────────┴───────────────────────────────────────────┘
```

## An Example Using sum and any
```sql
SELECT
    product_id,
    any(product_title),
    sum(total_votes) AS sum_of_votes
FROM amazon_reviews
GROUP BY product_id
ORDER BY sum_of_votes DESC
LIMIT 10
```
```
┌─product_id─┬─any(product_title)──────────────────────────────────────────────────┬─sum_of_votes─┐
│ B00992CF6W │ Minecraft                                                           │       525672 │
│ B004F9QBE6 │ BIC Cristal For Her Ball Pen, 1.0mm, Black, 16ct (MSLP16-Blk)       │       246945 │
│ B000FI73MA │ Kindle: Amazon's Original Wireless Reading Device (1st generation)  │       232395 │
│ B0051VVOB2 │ Kindle Fire (Previous Generation - 1st)                             │       202407 │
│ B009UX2YAC │ Subway Surfers                                                      │       177207 │
│ B001B0CTMU │ Avery Durable View Binder                                           │       163086 │
│ 1892112000 │ To Train Up a Child                                                 │       142984 │
│ B00154JDAI │ Kindle Wireless Reading Device (6" Display, U.S. Wireless)          │       136806 │
│ 0895260174 │ Unfit For Command: Swift Boat Veterans Speak Out Against John Kerry │       134058 │
│ B00CMEN95U │ Samsung UN85S9 Framed 85-Inch 4K Ultra HD 3D Smart LED TV           │       120933 │
└────────────┴─────────────────────────────────────────────────────────────────────┴──────────────┘

10 rows in set. Elapsed: 29.325 sec. Processed 202.32 million rows, 22.04 GB (6.90 million rows/s., 751.43 MB/s.)
```

## The uniq Function
The `uniq` function calculates the number of different values of a column.
```sql
SELECT formatReadableQuantity(uniq(product_id))
FROM amazon_reviews

-- 21.33 million
```

## The quantiles Function
A quantile is where a sample is divided into equal-sized groups and a value is computed 
for each group based on the number of rows in the group. For example, the 90% percentile 
refers to the quantile value at 90%, where 90% of all values are less than that 90th percentile value. 

```sql
SELECT quantile(0.9)(helpful_votes)
FROM amazon_reviews
-- 4
-- 90% of Amazon reviews receive 4 or fewer helpful votes
```

If you need the exact value of a quantile, which computes its value using every row, use the quantileExact function, 
which in our example is actually the same value:
```sql
SELECT quantileExact(0.9)(helpful_votes)
FROM amazon_reviews
-- 4
```

```sql
SELECT quantiles(0.1, 0.5, 0.9, 0.99)(helpful_votes)
FROM amazon_reviews
-- [0,0,4,25.090000000000146]     
```

## Window Functions
```sql
SELECT
    product_id,
    review_date,
    helpful_votes,
    sum(helpful_votes) OVER (
         PARTITION BY product_id, review_date 
         ORDER BY review_date ASC
    ) AS daily_total
FROM amazon_reviews
WHERE (product_id = 'B00AXIZ4TQ') AND (helpful_votes > 0)
ORDER BY review_date ASC
```

```
┌─product_id─┬─review_date─┬─helpful_votes─┬─daily_total─┐
│ B00AXIZ4TQ │  2013-05-15 │             2 │         839 │
│ B00AXIZ4TQ │  2013-05-15 │            10 │         839 │
│ B00AXIZ4TQ │  2013-05-15 │            42 │         839 │
│ B00AXIZ4TQ │  2013-05-15 │            67 │         839 │
│ B00AXIZ4TQ │  2013-05-15 │             2 │         839 │
│ B00AXIZ4TQ │  2013-05-15 │             6 │         839 │
│ B00AXIZ4TQ │  2013-05-15 │           710 │         839 │
│ B00AXIZ4TQ │  2013-05-16 │             1 │          63 │
│ B00AXIZ4TQ │  2013-05-16 │             6 │          63 │
│ B00AXIZ4TQ │  2013-05-16 │             9 │          63 │
│ B00AXIZ4TQ │  2013-05-16 │             2 │          63 │
│ B00AXIZ4TQ │  2013-05-16 │             6 │          63 │
│ B00AXIZ4TQ │  2013-05-16 │             3 │          63 │
...
```