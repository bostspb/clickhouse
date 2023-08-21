# Creating databases

## The CREATE DATABASE command

Predefined databases
```sql
SHOW DATABASES

┌─name───────────────┐
│ INFORMATION_SCHEMA │
│ default            │
│ information_schema │
│ system             │
└────────────────────┘
```
- When you create a new table and do not specify a database, it gets created in the `default` database
- The `system` database contains a wealth of information about your ClickHouse database, 
  from metadata details, metrics, logs, server states and processes, and [much more](https://clickhouse.com/docs/en/operations/system-tables)
- The `INFORMATION_SCHEMA` (with an alias `information_schema`) database contains a collection of ANSI-standard 
  views containing information about the metadata of database objects. 
  These views read data from columns in the `system.columns`, `system.databases` and `system.tables` tables

To define your own database and create a table
```sql
CREATE DATABASE my_db;

CREATE TABLE my_db.my_table (
   …
);
```

## CREATE DATABASE clauses

- **IF NOT EXISTS** - avoids an exception getting thrown if the specified database is already defined
- **COMMENT** - provide a comment about your database
- **ENGINE** - a database engine

```sql
CREATE DATABASE IF NOT EXISTS my_db 
ENGINE = Replicated 
COMMENT 'My first database'
```

## Database engines
- The default database engine in ClickHouse Cloud is `Replicated`
- The default database engine for a self-managed deployment is `Atomic`

* **Atomic** - supports non-blocking `DROP TABLE` and `RENAME TABLE` queries and atomic `EXCHANGE TABLES` queries.
* **Replicated** - based on the Atomic engine, it supports replication of metadata via DDL that gets executed 
  on all of the replicas for a given database.
* **Lazy** - keeps tables in RAM (only works with [Log table engines](https://clickhouse.com/docs/en/engines/table-engines/log-family)).
* **External DB Enines** - for connecting to external databases:
  * `PostgreSQL`
  * `MaterializedPostgreSQL`
  * `MySQL`
  * `MaterializedMySQL`
  * `SQLite`