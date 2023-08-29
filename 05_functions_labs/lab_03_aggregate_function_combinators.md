# Lab 3. Aggregate Function Combinators

## Task 1
Write a query that returns the sum of all helpful votes, and also returns the average of only those helpful 
votes for products that have a star rating equal to 1.
> Dataset `amazon_reviews` was broken down, so I used `reddit` and change a task.
```sql
CREATE TABLE reddit
(
    subreddit LowCardinality(String),
    subreddit_id LowCardinality(String),
    subreddit_type Enum('public' = 1, 'restricted' = 2, 'user' = 3, 'archived' = 4, 'gold_restricted' = 5, 'private' = 6),
    author LowCardinality(String),
    body String CODEC(ZSTD(6)),
    created_date Date DEFAULT toDate(created_utc),
    created_utc DateTime,
    retrieved_on DateTime,
    id String,
    parent_id String,
    link_id String,
    score Int32,
    total_awards_received UInt16,
    controversiality UInt8,
    gilded UInt8,
    collapsed_because_crowd_control UInt8,
    collapsed_reason Enum('' = 0, 'comment score below threshold' = 1, 'may be sensitive content' = 2, 'potentially toxic' = 3, 'potentially toxic content' = 4),
    distinguished Enum('' = 0, 'moderator' = 1, 'admin' = 2, 'special' = 3),
    removal_reason Enum('' = 0, 'legal' = 1),
    author_created_utc DateTime,
    author_fullname LowCardinality(String),
    author_patreon_flair UInt8,
    author_premium UInt8,
    can_gild UInt8,
    can_mod_post UInt8,
    collapsed UInt8,
    is_submitter UInt8,
    _edited String,
    locked UInt8,
    quarantined UInt8,
    no_follow UInt8,
    send_replies UInt8,
    stickied UInt8,
    author_flair_text LowCardinality(String)
)
ENGINE = MergeTree
ORDER BY (subreddit, created_date, author);

-- insert 2005 - 2009 years
INSERT INTO reddit
    SELECT *
    FROM s3Cluster(
        'default',
        'https://clickhouse-public-datasets.s3.eu-central-1.amazonaws.com/reddit/original/RC_2005*',
        'JSONEachRow'
    )
    SETTINGS zstd_window_log_max = 31;
```

```sql
SELECT 
    sum(score),
    avgIf(score, subreddit='1000words')
from reddit;

-- 297798034    2.018324607329843
```

## Task 2
Suppose we have the following table and rows:
```sql
CREATE TABLE orders (
    id String,
    prices Array(Decimal32(2))
)
ENGINE = MergeTree
ORDER BY id;

INSERT INTO orders VALUES
    ('a', [9.99, 24.00, 100.00]),
    ('b', [12.50]),
    ('c', [100.00, 5.99, 30.00, 85.00]);
```

Write a query that sums the `prices` column for each `id`. 
```sql
SELECT
    id, 
    sumArray(prices)
FROM orders
GROUP BY id;
```
```
b   12.5
c   220.99
a   133.99
```

## Task 3
Write a query on the `orders` table that returns the following:
a. the sum of all orders for each id (the same query as the previous step)
b. the number of items sold
```sql
SELECT
    id, 
    sumArray(prices),
    countArray(prices)
FROM orders
GROUP BY id;
```
```
b   12.5    1
c   220.99  4
a   133.99  3
```

## Task 4
Write a query on the `orders` table that returns the following: 
a. the sum of all orders for each `id` and the number of items sold (the same query as the previous step)
b. the average price of items sold, if they purchased more than one item
```sql
SELECT
    id, 
    sumArray(prices),
    countArray(prices),
    avgArrayIf(prices, length(prices) > 1)
FROM orders
GROUP BY id;
```
```
b   12.5    1
c   220.99  4   55.2475
a   133.99  3   44.663333333333334
```
