# 1.2 Understanding primary keys in ClickHouse

- Primary keys in ClickHouse are **not unique** for each row in a table
- The **primary key** of a ClickHouse table determines **how the data is sorted** when written to disk
- The index created by the primary key does not contain an entry for every row but instead contains 
  an entry for every 8,192 rows (referred to as **index granularity**)

## Defining a primary key
Step 1
```sql
CREATE TABLE helloworld
(
    x UInt32,
    y FixedString(1),
    PRIMARY KEY(x)
)
```

Step 2
```sql
INSERT INTO helloworld
   SELECT 
      number AS x, 
      char(modulo(rand(),26) + 97) AS y
   FROM numbers(10000000)
```

Step 3
```sql
SELECT * FROM helloworld LIMIT 10
```
```
0	u
1	n
2	z
3	i
4	h
5	d
6	n
7	m
8	f
9	j
```

Step 4
```sql
SELECT y FROM helloworld WHERE x = 20000
```
```
┌─y─┐
│ q │
└───┘

1 rows in set. Elapsed: 0.211 sec. 
Processed 8.19 thousand rows, <-------- !!!
40.97 KB (38.87 thousand rows/s., 194.39 KB/s.)
```

Step 5
```sql
SELECT count() FROM helloworld WHERE y = 'c'
```
```
┌─count()─┐
│  385070 │
└─────────┘

1 rows in set. Elapsed: 0.228 sec. 
Processed 10.00 million rows, <--------  !!!
10.00 MB (43.91 million rows/s., 43.92 MB/s.)
```

To demonstrate the improvement in performance, suppose you choose `y` as the primary key instead of `x`:
```sql
CREATE TABLE helloworld2
(
    x UInt32,
    y FixedString(1),
    PRIMARY KEY(y)
)
```
```sql
SELECT *
FROM helloworld2
LIMIT 10
```
```
┌───x─┬─y─┐
│  45 │ a │
│  56 │ a │
│  61 │ a │
│  68 │ a │
│ 132 │ a │
│ 145 │ a │
│ 148 │ a │
│ 193 │ a │
│ 202 │ a │
│ 210 │ a │
└─────┴───┘
```
```sql
SELECT count()
FROM helloworld2
WHERE y = 'c'
```
```
┌─count()─┐
│  385778 │
└─────────┘

1 rows in set. Elapsed: 0.596 sec. 
Processed 417.79 thousand rows, <------- !!!
417.86 KB (701.51 thousand rows/s., 701.62 KB/s.)
```