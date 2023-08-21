# Running queries

## Does ClickHouse support SQL?
Supported queries include the following:
- GROUP BY
- ORDER BY
- subqueries in FROM
- JOIN clause
- IN operator 
- window functions
- scalar subqueries 

More detail - [ANSI SQL Compatibility](https://clickhouse.com/docs/en/sql-reference/ansi

## Querying the hits table 
How many unique users we have
```sql
SELECT 
   UserID, 
   count() as total 
FROM hits 
GROUP BY UserID
ORDER BY total DESC
LIMIT 100;
```
```
┌─────UserID─┬─total─┐
│ 3731416266 │ 11700 │
│ 2054361600 │  9206 │
│ 2115636007 │  9065 │
│ 3013373937 │  7135 │
│ 4247098161 │  6888 │
│ 1502897340 │  5877 │
│ 4029406099 │  4789 │
│ 2459550954 │  4495 │

100 rows in set. 
Elapsed: 0.381 sec. 
Processed 4.18 million rows, 16.71 MB (22.14 million rows/s., 88.56 MB/s.)
```

The top 10 URLs visited by user `UserID = 3731416266`
```sql
SELECT 
   URL, 
   count(URL) as total
FROM hits
WHERE UserID = 3731416266
GROUP BY URL
ORDER BY total DESC
LIMIT 10;
```
```
┌─URL─────────────────────────────────────────────────────────────────────┬─total─┐
│ http://image&lr=42&text=rokes=&msectionsmedia.mobile=0                  │  4325 │
│ http://image&lr=42&text=roketa.mail.bing?rdrnd=43                       │  4273 │
│ http://image&lr=42825458_75401186SXQ?sira=12                            │  2849 │
│ http://doc/00003713844324&education.html?logi-38-rasstreferer_id        │    24 │
│ http://wotlauncher.html#/battle/ffffdf499594b053RjFDa0k3aG1YMHNTWDRi1jR │    14 │
│ http://astroitelnoe_pomogut-shared                                      │    12 │
│ http://public_search                                                    │    11 │
│ http://q-ec.bstatistic                                                  │    10 │
│ http://q-ec.bstanii_otret-otline_number                                 │     9 │
│ http://astration38/product_type=images.clickhouse?appkey                │     5 │
└─────────────────────────────────────────────────────────────────────────┴───────┘

10 rows in set. 
Elapsed: 0.242 sec. 
Processed 49.15 thousand rows, 3.32 MB (151.31 thousand rows/s., 10.23 MB/s.)
```
Granules are processed by a query: 49,152 / 8,192 = 6

Execution plan of a query
```sql
EXPLAIN indexes = 1
SELECT 
   URL, 
   count(URL) as total
FROM hits
WHERE UserID = 3731416266
GROUP BY URL
ORDER BY total DESC
LIMIT 10;
```
```
┌─explain───────────────────────────────────────────────────────────┐
│ Expression (Projection)                                           │
│   Limit (preliminary LIMIT (without OFFSET))                      │
│     Sorting (Sorting for ORDER BY)                                │
│       Expression (Before ORDER BY)                                │
│         Aggregating                                               │
│           Expression (Before GROUP BY)                            │
│             Filter (WHERE)                                        │
│               ReadFromMergeTree (default.hits)                    │
│                 PrimaryKey                                        │
│                   Keys:                                           │
│                     UserID                                        │
│                   Condition: (UserID in [3731416266, 3731416266]) │
│                   Parts: 4/4                                      │
│                   Granules: 6/513                                 │
└───────────────────────────────────────────────────────────────────┘
```

Let's analyze some of the details about the URLs.
```sql
SELECT
   URL,
   count() as total
FROM hits
GROUP BY URL
ORDER BY total DESC
LIMIT 10;
```
```
┌─URL──────────────────────────────────────────────────────────────┬──total─┐
│ http://public_search                                             │ 311119 │
│ http://bravoslava-230v                                           │  32907 │
│ http://doc/00003713844324&education.html?logi-38-rasstreferer_id │  21145 │
│ http://search?win=11&pos=22&img_url=http:%2F%2Fcs411276          │  19060 │
│ http://videovol-9-sezon                                          │  18982 │
│ http://search?clid=1784%26nid                                    │  17862 │
│ http://saechkala/avtomobile=0&ads                                │  15635 │
│ http://fontact Power[1                                           │  12922 │
│ http:%2F%2F2011&groups[]                                         │  11064 │
│ http://xofx.net%2F63857&secret-oper=reply&id=0&extras]           │   9763 │
└──────────────────────────────────────────────────────────────────┴────────┘

10 rows in set. Elapsed: 0.244 sec. Processed 4.18 million rows, 
277.12 MB (17.14 million rows/s., 1.14 GB/s.)
```

Count the number of unique URLs
```sql
SELECT
   uniq(URL)
FROM hits;

-- 4178363
```

Time of day had the most hits
```sql
SELECT
   EventTime,
   count() AS hits
FROM hits
GROUP BY EventTime
ORDER BY hits DESC
LIMIT 10;
```
```
┌───────────EventTime─┬─hits─┐
│ 2022-03-23 17:27:35 │  123 │
│ 2022-03-18 17:03:19 │  110 │
│ 2022-03-20 05:35:42 │   96 │
│ 2022-03-17 03:09:14 │   86 │
│ 2022-03-17 11:05:45 │   81 │
│ 2022-03-17 07:47:03 │   80 │
│ 2022-03-17 07:47:04 │   80 │
│ 2022-03-17 23:35:56 │   78 │
│ 2022-03-18 16:46:14 │   77 │
│ 2022-03-20 03:54:50 │   75 │
└─────────────────────┴──────┘

10 rows in set. Elapsed: 0.214 sec. Processed 4.18 million rows, 16.71 
MB (19.48 million rows/s., 77.92 MB/s.)
```