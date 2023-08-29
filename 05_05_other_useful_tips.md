# Other Useful Tips

## Comments
- **SQL-style comments**: anything after `--`, `#!`, or `#` until the end of the line
- **C-style comments**: anything between `/*` and `*/`

## Query Parameters
- Query parameters are **defined** using the SET command that starts with `param_`, 
  followed by the name of the parameter
- Query parameters are **referenced** using curly braces and include the name of the parameter 
  and its datatype: `{<name>: <datatype>}`

```sql
SET param_x = 5;

SELECT
    product_id,
    max(helpful_votes) AS max_votes
FROM amazon_reviews
GROUP BY product_id
ORDER BY max_votes DESC
LIMIT {x: UInt32};
```

If you are using `clickhouse-client`, you are not able to send two statements in a single request, so the above 
commands will not work. Instead, define the parameters using the `--param` flag.
```bash
./clickhouse client \
   --param_x=10 \
   --query="SELECT
             product_id,
             max(helpful_votes) AS max_votes
            FROM amazon_reviews
            GROUP BY product_id
            ORDER BY max_votes DESC
            LIMIT {x: UInt32}"
```

## Optimizing Queries
- Primary Keys
- Projections
- Materialized Views
- Partitioning your data properly
- [Avoid the "Deadly Sins" of ClickHouse](https://clickhouse.com/blog/common-getting-started-issues-with-clickhouse)