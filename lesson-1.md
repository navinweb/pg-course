Количество поездок - 5185505

```
postgres=# \c postgres
You are now connected to database "postgres" as user "postgres".
postgres=# \c thai
You are now connected to database "thai" as user "postgres".
thai=# \dn
      List of schemas
  Name  |       Owner
--------+-------------------
 book   | postgres
 public | pg_database_owner
(2 rows)

thai=#  select count(*) from book.tickets;
  count
---------
 5185505
(1 row)
```