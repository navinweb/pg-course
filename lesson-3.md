1.
```
postgres=# Begin;
CREATE TABLE user_attr ( id SERIAL PRIMARY KEY, random_text TEXT );
INSERT INTO user_attr (random_text) SELECT md5(random()::text) FROM generate_series(1, 1000000);
Commit;
BEGIN
CREATE TABLE
INSERT 0 1000000
COMMIT
```

2.
```
postgres=# SELECT pg_size_pretty(pg_total_relation_size('user_attr')) AS table_size;
 table_size
------------
 87 MB
(1 row)
```

3.
```
postgres=# CREATE OR REPLACE FUNCTION update_table_multiple_times(update_count INT)
RETURNS VOID AS $$
DECLARE
    i INT;
BEGIN
    FOR i IN 1..update_count LOOP
        UPDATE user_attr
        SET random_text = random_text || '!';
        RAISE NOTICE 'Step %', i;
    END LOOP;
END $$ LANGUAGE plpgsql;
CREATE FUNCTION

postgres=# SELECT update_table_multiple_times(5);
NOTICE:  Step 1
NOTICE:  Step 2
NOTICE:  Step 3
NOTICE:  Step 4
NOTICE:  Step 5
 update_table_multiple_times
-----------------------------

(1 row)


```

4.
```
postgres=# SELECT relname AS table_name, n_dead_tup AS dead_tuples, last_autovacuum FROM pg_stat_all_tables WHERE relname = 'user_attr';
 table_name | dead_tuples |       last_autovacuum
------------+-------------+------------------------------
 user_attr  |     5000000 | 2024-10-12 13:26:34.86781+00
(1 row)
```
5.
```
postgres=# SELECT relname AS table_name, n_dead_tup AS dead_tuples, last_autovacuum FROM pg_stat_all_tables WHERE relname = 'user_attr';
 table_name | dead_tuples |        last_autovacuum
------------+-------------+-------------------------------
 user_attr  |           0 | 2024-10-12 13:39:37.442455+00
(1 row)
```

6.
```
postgres=# SELECT update_table_multiple_times(5);
NOTICE:  Step 1
NOTICE:  Step 2
NOTICE:  Step 3
NOTICE:  Step 4
NOTICE:  Step 5
 update_table_multiple_times
-----------------------------

(1 row)
```

7.
```
postgres=# SELECT pg_size_pretty(pg_total_relation_size('user_attr')) AS table_size;
 table_size
------------
 524 MB
(1 row)
```

8.
```
ALTER TABLE user_attr
SET (autovacuum_enabled = false);
```

9.
```
postgres=# SELECT update_table_multiple_times(10);
NOTICE:  Step 1
NOTICE:  Step 2
NOTICE:  Step 3
NOTICE:  Step 4
NOTICE:  Step 5
NOTICE:  Step 6
NOTICE:  Step 7
NOTICE:  Step 8
NOTICE:  Step 9
NOTICE:  Step 10
 update_table_multiple_times
-----------------------------

(1 row)
```

10.
```
postgres=# SELECT pg_size_pretty(pg_total_relation_size('user_attr')) AS table_size;
 table_size
------------
 1008 MB
(1 row)
```

11. Размер таблицы увеличился, потому что мертвые строчки теперь не будут удаляться автоматически, тк мы отключили autovacuum. Старые версии строки остаются при обновлении.

12.
```
ALTER TABLE user_attr
SET (autovacuum_enabled = true);
```

после включения обратно пришел autovacuum и всё почистил снова, при этом размер размер таблицы не изменился.
```
postgres=# SELECT relname AS table_name, n_dead_tup AS dead_tuples, last_autovacuum FROM pg_stat_all_tables WHERE relname = 'user_attr';
 table_name | dead_tuples |        last_autovacuum
------------+-------------+-------------------------------
 user_attr  |           0 | 2024-10-12 13:48:21.160197+00
(1 row)

postgres=# SELECT pg_size_pretty(pg_total_relation_size('user_attr')) AS table_size;
 table_size
------------
 1008 MB
(1 row)
```

Запустил `VACUUM FULL user_attr;` для эксперимента и получил
```
postgres=# SELECT pg_size_pretty(pg_total_relation_size('user_attr')) AS table_size;
 table_size
------------
 1008 MB
(1 row)

postgres=# VACUUM FULL user_attr;
VACUUM
postgres=# SELECT pg_size_pretty(pg_total_relation_size('user_attr')) AS table_size;
 table_size
------------
 110 MB
(1 row)
```