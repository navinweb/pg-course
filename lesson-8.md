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

drop function get_sales_report_by_quarter;

CREATE OR REPLACE FUNCTION get_sales_report_by_quarter(quarter INTEGER)
    RETURNS TABLE
            (
                id        INTEGER,
                sale_date DATE,
                amount    NUMERIC,
                product   TEXT
            )
AS
$$
BEGIN
    IF quarter IS NULL THEN
        RAISE NOTICE 'Параметр quarter равен NULL. Используйте 1, 2, 3 или 4.';
        RETURN;
    ELSIF quarter < 1 OR quarter > 4 THEN
        RAISE NOTICE 'Параметр quarter неверный. Используйте 1, 2, 3 или 4.';
        RETURN;
    END IF;

    DECLARE
        start_month INTEGER := (quarter - 1) * 3 + 1;
        end_month   INTEGER := quarter * 3;
    BEGIN
        RETURN QUERY
            SELECT s.id,
                   s.sale_date,
                   s.amount,
                   s.product
            FROM sales s
            WHERE EXTRACT(MONTH FROM s.sale_date) BETWEEN start_month AND end_month;
    END;
END
$$ LANGUAGE plpgsql;

SELECT * FROM get_sales_report_by_quarter(1);
SELECT * FROM get_sales_report_by_quarter(2);
SELECT * FROM get_sales_report_by_quarter(3);
SELECT * FROM get_sales_report_by_quarter(4);
SELECT * FROM get_sales_report_by_quarter(NULL);
SELECT * FROM get_sales_report_by_quarter(5);