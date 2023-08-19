# 1.3 Inserting data

## Creating a database and a table
```sql
CREATE DATABASE my_database

CREATE TABLE my_database.hits
(
    UserID UInt32,
    URL String,
    EventTime DateTime
)
PRIMARY KEY (UserID, URL)
```

## Inserting data from a file
```sql
INSERT INTO my_database.hits SELECT
   c1 as UserID,
   c2 as URL,
   c3 as EventTime
FROM url('https://datasets-documentation.s3.eu-west-3.amazonaws.com/hits/hits.tsv.gz')
SETTINGS input_format_try_infer_datetimes=0
```

- Since we did not specify a schema for the incoming rows (which we could have done), 
  the `url` function simply names the columns `c1`, `c2`, and so on
- The file is streamed into the `hits` table (as opposed to being downloaded in its entirety locally 
  to the ClickHouse server)
- The `url` function is smart enough not only to figure out the data is tab-separated (from the file extension),  
  but also how to uncompress the file (based on the file extension as well)

To verify it worked, there should be 4,178,363 rows:
```sql
SELECT count() FROM my_database.hits
-- 4178363
```

## Links
- Table Functions - https://clickhouse.com/docs/en/sql-reference/table-functions
- Formats for Input and Output Data - https://clickhouse.com/docs/en/interfaces/formats