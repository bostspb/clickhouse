# 7.1 Views

## Defining a normal view
```sql
CREATE VIEW my_view AS 
   SELECT 
      user_id, 
      splitByChar(':', message)[1] as error_code
   FROM log_messages;

SELECT uniq(error_code) 
FROM my_view;
```

In general, use a regular view when any of the following are true:
- The results of the view change often
- The results are not used often (relative to the rate at which the results change)
- The query is not resource-intensive, so it is not costly to re-run it
If your view doesn't change often and is resource intensive, consider using a materialized view instead.