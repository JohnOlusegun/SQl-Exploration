-- Selecting the database to use

USE learning_with_bro_code;

-- show the information about the tables in the database
Select * from transactions;

Select * from customers;

-- How many customers do we have
SELECT 
    COUNT(DISTINCT (customer_id)) AS total_customer
FROM
    customers;

-- Update the Transaction table for a trnsaction not recoreded for customer_i 4
UPDATE transactions 
SET 
    customer_id = 4
WHERE
    transaction_id = '1004';

--- Customers that have purchased
SELECT 
    c.customer_id, t.transaction_id
FROM
    customers c
        JOIN
    transactions t ON c.customer_id = t.customer_id
ORDER BY c.customer_id;

-- Customer that spent the highest amount and name
WITH spending_rank AS (
    SELECT *,
           RANK() OVER (ORDER BY x.total_amount_spent DESC) AS rnk
    FROM (
        SELECT t.customer_id,
               CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
               SUM(t.amount) AS total_amount_spent
        FROM transactions t
        JOIN customers c ON t.customer_id = c.customer_id
        GROUP BY t.customer_id, c.first_name, c.last_name
    ) AS x
)
SELECT customer_name,
       total_amount_spent
FROM spending_rank
WHERE rnk = 1;


-- Ratio of amount per customer spent to total
WITH customer_t AS (
    SELECT 
        t.customer_id,
        CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
        SUM(t.amount) AS total_amount_spent
    FROM
        transactions t
    JOIN
        customers c ON t.customer_id = c.customer_id
    GROUP BY 
        t.customer_id, c.first_name, c.last_name
)
SELECT 
    customer_id,
    customer_name,
    total_amount_spent,
    SUM(total_amount_spent) OVER () AS total_spent,
    concat(round(total_amount_spent / SUM(total_amount_spent) OVER (),2),'%') AS ratio_of_total
FROM 
    customer_t
ORDER BY 
    total_amount_spent DESC;
