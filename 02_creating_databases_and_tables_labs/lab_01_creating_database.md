# Lab 1. Creating databases

## Step 1
```sql
CREATE DATABASE training;
```
```
CREATE succeeded
```

## Step 2
```sql
SHOW CREATE DATABASE training;
```
```
CREATE DATABASE training
ENGINE = Replicated('/clickhouse/databases/9de88f6d-5eaf-4b7b-a0d7-1dda3a2bf4ed', '{shard}', '{replica}')
```

## Step 3
```sql
DROP DATABASE training;
```
```
DROP succeeded
```