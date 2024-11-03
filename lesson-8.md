```
CREATE TABLE sales
(
    id        SERIAL PRIMARY KEY,
    sale_date DATE           NOT NULL,
    amount    NUMERIC(10, 2) NOT NULL,
    product   TEXT           NOT NULL
);

INSERT INTO sales (sale_date, amount, product)
VALUES ('2024-01-10', 100.00, 'Product A'),
       ('2024-01-15', 150.50, 'Product B'),
       ('2024-01-20', 200.00, 'Product C'),
       ('2024-02-05', 300.00, 'Product D'),
       ('2024-02-15', 250.75, 'Product E'),
       ('2024-03-01', 120.00, 'Product F'),
       ('2024-03-10', 180.50, 'Product G'),
       ('2024-04-20', 400.00, 'Product H'),
       ('2024-05-05', 500.25, 'Product I'),
       ('2024-05-15', 600.00, 'Product J'),
       ('2024-06-01', 150.75, 'Product K'),
       ('2024-07-10', 220.30, 'Product L'),
       ('2024-08-01', 90.00, 'Product M'),
       ('2024-08-15', 130.50, 'Product N'),
       ('2024-09-05', 350.00, 'Product O'),
       ('2024-09-20', 80.25, 'Product P'),
       ('2024-10-01', 310.00, 'Product Q'),
       ('2024-10-15', 450.00, 'Product R'),
       ('2024-11-01', 70.90, 'Product S'),
       ('2024-12-10', 600.50, 'Product T');

CREATE OR REPLACE FUNCTION get_sales_report(report_date DATE DEFAULT NULL)
    RETURNS TABLE
            (
                id        INTEGER,
                sale_date DATE,
                amount    NUMERIC,
                product   TEXT
            )
AS
$$
DECLARE
    quarter INTEGER;
    start_month INTEGER;
    end_month INTEGER;
BEGIN
    report_date := COALESCE(report_date, CURRENT_DATE);

    quarter := FLOOR((EXTRACT(MONTH FROM report_date) - 1) / 3) + 1;

    start_month := (quarter - 1) * 3 + 1;
    end_month := quarter * 3;

    RETURN QUERY
        SELECT *
        FROM sales s
        WHERE EXTRACT(MONTH FROM s.sale_date) BETWEEN start_month AND end_month;
END
$$ LANGUAGE plpgsql;

SELECT * FROM get_sales_report();
SELECT * FROM get_sales_report('2024-01-01');
SELECT * FROM get_sales_report('2024-04-01');
SELECT * FROM get_sales_report('2024-09-01');
```
