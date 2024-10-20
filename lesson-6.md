```
postgres@14902b0a13b9:~$ pg_lsclusters
Ver Cluster Port Status          Owner    Data directory               Log file
16  main    5432 online          postgres /var/lib/postgresql/16/main  /var/log/postgresql/postgresql-16-main.log
16  main2   5433 online,recovery postgres /var/lib/postgresql/16/main2 /var/log/postgresql/postgresql-16-main2.log

postgres=# SELECT
    application_name,
    state,
    sync_state,
    sync_priority
FROM pg_stat_replication;
 application_name |   state   | sync_state | sync_priority
------------------+-----------+------------+---------------
 16/main2         | streaming | async      |             0
(1 row)
```

синхронную реплику так и не удалось создать, час мучался, меняя параметры, добавляя application_name на реплике 😔

select
from main
```
postgres@14902b0a13b9:~$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres thai
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 910060
number of failed transactions: 0 (0.000%)
latency average = 0.088 ms
initial connection time = 4.413 ms
tps = 91043.327764 (without initial connection time)
```
from replica
```
postgres@14902b0a13b9:~$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -U postgres thai
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
starting vacuum...end.
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 903471
number of failed transactions: 0 (0.000%)
latency average = 0.089 ms
initial connection time = 4.305 ms
tps = 90383.660191 (without initial connection time)
```

insert
single
```
postgres@14902b0a13b9:~$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/insert.sql -n -U postgres -p 5432 postgres
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
transaction type: /var/lib/postgresql/insert.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 11042
number of failed transactions: 0 (0.000%)
latency average = 7.248 ms
initial connection time = 4.047 ms
tps = 1103.692302 (without initial connection time)
```

with replica
```
postgres@14902b0a13b9:~$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/insert.sql -U postgres
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
transaction type: /var/lib/postgresql/insert.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 4496
number of failed transactions: 0 (0.000%)
latency average = 17.804 ms
initial connection time = 4.725 ms
tps = 449.334578 (without initial connection time)
```