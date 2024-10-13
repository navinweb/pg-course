1.
```
postgres=# CREATE TABLE accounts ( id INTEGER PRIMARY KEY, amount NUMERIC );
INSERT INTO accounts (id, amount) VALUES (1, 100);
INSERT INTO accounts (id, amount) VALUES (2, 200);
CREATE TABLE
INSERT 0 1
INSERT 0 1
```

2.
terminal 1
```
postgres=# BEGIN;
UPDATE accounts SET amount = amount + 50 WHERE id = 1;
BEGIN
UPDATE 1
```

terminal 2
```
postgres=# BEGIN;
UPDATE accounts SET amount = amount - 50 WHERE id = 2;
BEGIN
UPDATE 1
postgres=*# UPDATE accounts SET amount = amount - 30 WHERE id = 1;
```

terminal 1
```
postgres=*# UPDATE accounts SET amount = amount + 30 WHERE id = 2;
ERROR:  deadlock detected
DETAIL:  Process 331 waits for ShareLock on transaction 1027; blocked by process 319.
Process 319 waits for ShareLock on transaction 1026; blocked by process 331.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,2) in relation "accounts"
```


3.
```
2024-10-13 11:15:34.933 UTC [331] postgres@postgres ERROR:  deadlock detected
2024-10-13 11:15:34.933 UTC [331] postgres@postgres DETAIL:  Process 331 waits for ShareLock on transaction 1027; blocked by process 319.
        Process 319 waits for ShareLock on transaction 1026; blocked by process 331.
        Process 331: UPDATE accounts SET amount = amount + 30 WHERE id = 2;
        Process 319: UPDATE accounts SET amount = amount - 30 WHERE id = 1;
2024-10-13 11:15:34.933 UTC [331] postgres@postgres HINT:  See server log for query details.
2024-10-13 11:15:34.933 UTC [331] postgres@postgres CONTEXT:  while updating tuple (0,2) in relation "accounts"
2024-10-13 11:15:34.933 UTC [331] postgres@postgres STATEMENT:  UPDATE accounts SET amount = amount + 30 WHERE id = 2;
```

```
postgres=# SELECT pid, query, state FROM pg_stat_activity;
 pid |                         query                          |             state
-----+--------------------------------------------------------+----------
  26 |                                                        |
  27 |                                                        |
  41 | select * from pgrowlocks(thai)                         | idle
 319 | UPDATE accounts SET amount = amount - 30 WHERE id = 1; | idle in transaction
 331 | UPDATE accounts SET amount = amount + 30 WHERE id = 2; | idle in transaction (aborted)
 353 | SELECT pid, query, state FROM pg_stat_activity;        | active
  23 |                                                        |
  22 |                                                        |
  25 |                                                        |
```