unix socket
```
postgres@14902b0a13b9:/root$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres thai
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 832224
number of failed transactions: 0 (0.000%)
latency average = 0.096 ms
initial connection time = 4.826 ms
tps = 83260.933160 (without initial connection time)
```

localhost
```
postgres@14902b0a13b9:/root$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -h localhost thai
Password:
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 623062
number of failed transactions: 0 (0.000%)
latency average = 0.128 ms
initial connection time = 22.797 ms
tps = 62446.611205 (without initial connection time)
```

pgbouncer
```
postgres@14902b0a13b9:/root$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 6432 -h localhost thai
Password:
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 385873
number of failed transactions: 0 (0.000%)
latency average = 0.207 ms
initial connection time = 24.450 ms
tps = 38680.675150 (without initial connection time)
```

session
```
pgbouncer=# show pools;
 thai      | postgres  |         8 |          0 |                    0 |                     0 |         8 |                0 |                 0 |       0 |       0 |         0 |        0 |       0 |
0 | session
```
```
postgres@14902b0a13b9:/root$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 6432 -h localhost thai
Password:
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 378307
number of failed transactions: 0 (0.000%)
latency average = 0.211 ms
initial connection time = 25.644 ms
tps = 37926.551774 (without initial connection time)
```
но мог и tps = 39000.651596 (without initial connection time) выдать

transaction
```
pgbouncer=# show pools;
thai      | postgres  |         8 |          0 |                    0 |                     0 |         5 |                0 |                 0 |       3 |       0 |         0 |        0 |       0 |
0 | transaction
```
```
postgres@14902b0a13b9:/root$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 6432 -h localhost thai
Password:
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 383174
number of failed transactions: 0 (0.000%)
latency average = 0.208 ms
initial connection time = 24.447 ms
tps = 38409.109431 (without initial connection time)
```

statement
```
pgbouncer=# show pools;
 thai      | postgres  |         8 |          0 |                    0 |                     0 |         6 |                0 |                 0 |       2 |       0 |         0 |        0 |       0 |
0 | statement
```
```
postgres@14902b0a13b9:/root$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 6432 -h localhost thai
Password:
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 377211
number of failed transactions: 0 (0.000%)
latency average = 0.212 ms
initial connection time = 24.555 ms
tps = 37812.875630 (without initial connection time)
```
