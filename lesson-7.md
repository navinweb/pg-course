Изначальный запрос
```
thai=# EXPLAIN ANALYZE
WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    GROUP BY s.fkbus
),
order_place AS (
    SELECT count(t.id) as order_place, t.fkride
    FROM book.tickets t
    GROUP BY t.fkride
)
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,
      t.order_place, st.all_place
FROM book.ride r
JOIN book.schedule as s
      ON r.fkschedule = s.id
JOIN book.busroute br
      ON s.fkroute = br.id
JOIN book.busstation bs
      ON br.fkbusstationfrom = bs.id
JOIN order_place t
      ON t.fkride = r.id
JOIN all_place st
      ON r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place, st.all_place
ORDER BY r.startdate
LIMIT 10;
```
Aнализ
```
Limit  (cost=954045.33..954045.34 rows=5 width=56) (actual time=77156.244..77156.338 rows=10 loops=1)
Planning Time: 0.681 ms
 JIT:
   Functions: 74
   Options: Inlining true, Optimization true, Expressions true, Deforming true
   Timing: Generation 2.939 ms, Inlining 92.628 ms, Optimization 160.787 ms, Emission 127.988 ms, Total 384.342 ms
 Execution Time: 77193.604 ms
```

---

Добавим ключи
```
CREATE INDEX idx_seat_fkbus ON book.seat(fkbus); 
CREATE INDEX idx_tickets_fkride ON book.tickets(fkride); 
CREATE INDEX idx_ride_fkschedule ON book.ride(fkschedule); 
CREATE INDEX idx_schedule_fkroute ON book.schedule(fkroute); 
CREATE INDEX idx_busroute_fkbusstationfrom ON book.busroute(fkbusstationfrom);
```

Анализ
```
Limit  (cost=3439036.72..3439036.75 rows=10 width=56) (actual time=10465.461..10465.595 rows=10 loops=1)
Planning Time: 0.938 ms
 JIT:
   Functions: 88
   Options: Inlining true, Optimization true, Expressions true, Deforming true
   Timing: Generation 3.660 ms, Inlining 87.233 ms, Optimization 166.640 ms, Emission 122.689 ms, Total 380.221 ms
 Execution Time: 10515.085 ms
```