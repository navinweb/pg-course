```
SELECT
    fk_employee,
    amount,
    from_date,
    to_date,
    fk_grade,
    COALESCE(amount - LAG(amount) OVER (PARTITION BY fk_employee ORDER BY from_date), 0) AS salary_increase
FROM
    salary
ORDER BY
    fk_employee, from_date;
```


| fk\_employee | amount | from\_date | to\_date | fk\_grade | salary\_increase |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 100000 | 2024-01-01 | 2024-01-31 | 1 | 0 |
| 1 | 200000 | 2024-02-01 | 2024-02-29 | 2 | 100000 |
| 1 | 300000 | 2024-03-01 | 2099-12-31 | 3 | 100000 |
| 2 | 200000 | 2023-01-01 | 2024-01-31 | 2 | 0 |
| 3 | 200000 | 2024-03-01 | 2024-01-31 | 2 | 0 |
