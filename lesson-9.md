## Блоатинг TOAST
```
CREATE TABLE t AS
SELECT i AS id, (SELECT jsonb_object_agg(j, j) FROM generate_series(1, 1000) j) js
FROM generate_series(1, 10000) i;

SELECT oid::regclass AS heap_rel,
       pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
       reltoastrelid::regclass AS toast_rel,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
FROM pg_class WHERE relname = 't';
```

```
[
  {
    "heap_rel": "t",
    "heap_rel_size": "512 kB",
    "toast_rel": "pg_toast.pg_toast_81983",
    "toast_rel_size": "78 MB"
  }
]
```

---

```
SELECT pg_current_wal_lsn(); -- 4/EB285758
UPDATE t SET id = id + 1; -- 32 ms
SELECT pg_current_wal_lsn(); -- 4/EB3FF448
SELECT pg_size_pretty(pg_wal_lsn_diff('4/EB285758','4/EB3FF448')) AS wal_size; -- 1511 kB


SELECT pg_current_wal_lsn(); -- 4/EB413DF0
UPDATE t SET js = js::jsonb || '{"a":1}'; -- 8s
SELECT pg_current_wal_lsn(); -- 4/F35A1A20
SELECT pg_size_pretty(pg_wal_lsn_diff('4/EB413DF0','4/F35A1A20')) AS wal_size; -- 130 MB


SELECT oid::regclass AS heap_rel,
       pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
       reltoastrelid::regclass AS toast_rel,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
FROM pg_class WHERE relname = 't';
```

```
[
  {
    "heap_rel": "t",
    "heap_rel_size": "1528 kB",
    "toast_rel": "pg_toast.pg_toast_81983",
    "toast_rel_size": "156 MB"
  }
]
```

---

```
VACUUM t;
SELECT * FROM t;
SELECT oid::regclass AS heap_rel,
       pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
       reltoastrelid::regclass AS toast_rel,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
FROM pg_class WHERE relname = 't';
```

```
[
  {
    "heap_rel": "t",
    "heap_rel_size": "1528 kB",
    "toast_rel": "pg_toast.pg_toast_81983",
    "toast_rel_size": "156 MB"
  }
]
```
не поменялось

---

```
VACUUM FULL t;
SELECT oid::regclass AS heap_rel,
       pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
       reltoastrelid::regclass AS toast_rel,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
FROM pg_class WHERE relname = 't';
```

```
[
  {
    "heap_rel": "t",
    "heap_rel_size": "512 kB",
    "toast_rel": "pg_toast.pg_toast_81983",
    "toast_rel_size": "78 MB"
  }
]
```
очистилось

---

`VACUUM FULL pg_toast.pg_toast_81983;`

```
[
  {
    "heap_rel": "t",
    "heap_rel_size": "512 kB",
    "toast_rel": "pg_toast.pg_toast_81983",
    "toast_rel_size": "78 MB"
  }
]
```
очистилось

---

также сделал
```
ALTER TABLE t SET (toast_tuple_target = 8160);
UPDATE t SET js = js::jsonb || '{"a":1}';
```

```
[
  {
    "heap_rel": "t",
    "heap_rel_size": "156 MB",
    "toast_rel": "pg_toast.pg_toast_81983",
    "toast_rel_size": "0 bytes"
  }
]
```

---

## Блоатинг индексов

```
CREATE INDEX idx_js ON t USING gin (js);
SELECT
    pg_size_pretty(pg_relation_size('idx_js')); -- 63 MB

UPDATE t SET js = js::jsonb || '{"a":1}';
SELECT
    pg_size_pretty(pg_relation_size('idx_js')); -- 98 MB

REINDEX INDEX idx_js;
SELECT pg_size_pretty(pg_relation_size('idx_js')); -- 63 MB
```