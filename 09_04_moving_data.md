# 9.4 Moving Data

## Implementing a hot/warm/cold architecture 
The TO DISK and TO VOLUME options refer to the names of disks or volumes defined in your ClickHouse configuration files. 
Create a new file named `my_system.xml` that defines your disks, then define volumes that use your disks:

```
<clickhouse>
    <storage_configuration>
        <disks>
           <hot_disk>
              <path>./data/hot/</path>
           </hot_disk>
           <warm_disk>
              <path>./data/warm/</path>
           </warm_disk>
           <cold_disk>
              <path>./data/cold/</path>
           </cold_disk>
        </disks>
        <policies>
            <my_policies>
                <volumes>
                    <hot_volume>
                        <disk>hot_disk</disk>
                    </hot_volume>
                    <warm_volume>
                        <disk>warm_disk</disk>
                    </warm_volume>
                    <cold_volume>
                        <disk>cold_disk</disk>
                    </cold_volume>
                </volumes>
            </my_policies>
        </policies>
    </storage_configuration>
</clickhouse>
```

view the disks
```sql
SELECT name, path, free_space, total_space
FROM system.disks
```
```
┌─name────────┬─path───────────┬───free_space─┬──total_space─┐
│ cold_disk   │ ./data/cold/   │ 179143311360 │ 494384795648 │
│ default     │ ./             │ 179143311360 │ 494384795648 │
│ hot_disk    │ ./data/hot/    │ 179143311360 │ 494384795648 │
│ medium_disk │ ./data/medium/ │ 179143311360 │ 494384795648 │
└─────────────┴────────────────┴──────────────┴──────────────┘
```

verify the volumes
```sql
SELECT
    volume_name,
    disks
FROM system.storage_policies
```
```
┌─volume_name─┬─disks─────────┐
│ default     │ ['default']   │
│ hot_volume  │ ['hot_disk']  │
│ warm_volume │ ['warm_disk'] │
│ cold_volume │ ['cold_disk'] │
└─────────────┴───────────────┘
```

```sql
ALTER TABLE crypto_prices
   MODIFY TTL 
      trade_date TO VOLUME 'hot_volume',
      trade_date + INTERVAL 2 YEAR TO VOLUME 'warm_volume',
      trade_date + INTERVAL 4 YEAR TO VOLUME 'cold_volume';

ALTER TABLE crypto_prices MATERIALIZE TTL;

SELECT
    name,
    disk_name
FROM system.parts
WHERE (table = 'crypto_prices') AND (active = 1);

-- ┌─name────────┬─disk_name─┐
-- │ all_1_3_1_5 │ warm_disk │
-- └─────────────┴───────────┘
```

```sql
INSERT INTO crypto_prices (trade_date, crypto_name, price) VALUES
   (now(), 'MyWorthlessCrypto', 100.00);

SELECT
    name,
    disk_name
FROM system.parts
WHERE (table = 'crypto_prices') AND (active = 1)

-- ┌─name────────┬─disk_name─┐
-- │ all_1_3_1_5 │ warm_disk │
-- │ all_2_2_0   │ hot_disk  │
-- └─────────────┴───────────┘  
```

```sql
INSERT INTO crypto_prices (trade_date, crypto_name, price) VALUES
   (now() - INTERVAL 8 YEAR, 'MyWorthlessCrypto', 1.00)

SELECT
    name,
    disk_name
FROM system.parts
WHERE (table = 'crypto_prices') AND (active = 1)
```
```
┌─name────────┬─disk_name─┐
│ all_1_3_1_5 │ warm_disk │
│ all_2_2_0   │ hot_disk  │
│ all_3_3_0   │ cold_disk │
└─────────────┴───────────┘
```