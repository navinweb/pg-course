## read committed
```
iso=# BEGIN;
CREATE TABLE users(id int);
INSERT INTO users(id) VALUES (1);
INSERT INTO users(id) VALUES (2);
COMMIT;
iso=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
```

Не видим до коммита транзакции из первого окна, потому что по умолчанию уровень изоляции read committed
```
iso=# BEGIN;
BEGIN
iso=*# SELECT * FROM users;
 id
----
  1
  2
(2 rows)
```

После коммита запись видно во втором окне, как и предполагает уровень изоляции read committed

---
## repeatable read
записи не видно при уровне изоляции repeatable read для закоммиченных и не закомиченных данных, он будет читать те же данные что и на момент начала транзакции.

```
iso=# BEGIN transaction isolation level repeatable read;
BEGIN
iso=*# INSERT INTO users(id) VALUES (4);
INSERT 0 1
iso=*# COMMIT;
```

в обоих случаях будет
```
iso=*# SELECT * FROM users;
 id
----
  1
  2
  3
(3 rows)
```

---
