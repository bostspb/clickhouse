# Aggregate Function Combinators
You can append an aggregate function with one or more of the available combinators to add or 
alter the behavior of the aggregation.

## The If Combinator
```sql
SELECT 
    maxIf(helpful_votes, star_rating = 5)
FROM amazon_reviews;
```
The max function is only applied to rows in `amazon_reviews` where `star_rating` equals 5.

## The Array Combinator
```sql
CREATE TABLE array_demo (
    x UInt32,
    y Array(UInt32)
)
ENGINE = MergeTree
ORDER BY x;

INSERT INTO array_demo VALUES 
    (10, [100,150,200]),
    (20, [200, 250]),
    (10, [250, 300, 350,400]);
```

```sql
SELECT avg(x) 
FROM array_demo;

-- 13.333333333333334
```

```sql
SELECT avg(y)
FROM array_demo;

-- Code: 43. DB::Exception: Illegal type Array(UInt32) of argument for aggregate function avg. 
-- (ILLEGAL_TYPE_OF_ARGUMENT) (version 23.8.1.41498 (official build))
```

```sql
SELECT avgArray(y)
FROM array_demo;

-- 244.44444444444446
```

## Combining Combinators
You can append `-Array` and `-If` to any aggregate function, so that the incoming argument is an array,
but you also provide a conditional. Creating a function all like `avgArrayIf` is perfectly valid in ClickHouse, 
even though you will not see this function in the documentation or in the `system.functions` table.
```sql
SELECT avgArrayIf(y, length(y) >= 3)
FROM array_demo

-- 250
```

```sql
CREATE TABLE array_demo2 (
    z Array(Array(UInt32))
)
ENGINE = MergeTree
ORDER BY tuple();

INSERT INTO array_demo2 VALUES 
    ([[100,150,200],[200, 250]]),
    ([[250, 300, 350,400],[400, 500]]);

SELECT avgArrayArray(z)
FROM array_demo2

-- 281.8181818181818
```

```sql
SELECT avgArrayArrayIf(z, has(z[1], 100))
FROM array_demo2

-- 180
```
The `has(z[1], 100)` conditional means that the first array of `z` must contain the value 100 or that row is skipped. 
The answer is 180 because only the first row matches the conditional, and `(100+150+200+200+250)/5 = 180`.

## The Distinct Combinator
```sql
SELECT avgDistinct(x)
FROM array_demo

-- 15
```

## The Other Combinators
Aggregate Function Combinators - https://clickhouse.com/docs/en/sql-reference/aggregate-functions/combinators