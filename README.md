# Noon-Food-orders
Personal Project
**-- Find Top 3 Resturants in Noon and their cuisine**.
WITH CTE AS (Select Cuisine, Restaurant_id, count(*) AS no_of_orders
FROM orders
group by Cuisine, Restaurant_id)
SELECT * FROM (
SELECT*, 
ROW_NUMBER () OVER (PARTITION by Cuisine order by no_of_orders DESC) AS rn
FROM CTE) a
Limit 3;

**-- Find the no of new customer since the launch date.**

WITH CTE AS(
Select
 Customer_code, CAST(Min(Placed_at) AS date )AS First_order_date
 FROM orders
 GROUP BY Customer_code)
 SELECT first_order_date, count(*) AS no_of_new_customers
 FROM CTE
 group by first_order_date
ORDER BY first_order_date
;

**-- Find the no of acquired customer in Jan who placed one order and didn't return**

SELECT 
Customer_code, count(*) as no_of_orders
FROM Orders 
WHERE month(Placed_at) = 1 and year(Placed_at) =2025 AND Customer_code Not In 
(
SELECT distinct Customer_code FROM Orders
WHERE Not (month(Placed_at) = 1 and year(Placed_at) =2025)
)
GROUP BY Customer_code
HAVING count(*) = 1
;

-- List all the customer with zero orders that were acquired in the past month through promo
WITH CTE AS(
SELECT Customer_code, min(Placed_at) AS first_order_date, Max(Placed_at) AS latest_order_date
FROM Orders
GROUP BY Customer_code)
SELECT CTE.*, Orders.Promo_code_name As first_order
FROM CTE 
inner JOIN Orders 
on CTE.Customer_code = Orders.Customer_code
AND CTE.first_order_date = Orders.Placed_at
WHERE latest_order_date < DATE_SUB(NOW(), INTERVAL 7 DAY)
AND first_order_date < DATE_SUB(CURDATE(), INTERVAL 1 MONTH)
AND Promo_code_name is not null
; 
-- Target customer who have placed 3 orders and offer promo

WITH CTE AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY Customer_code ORDER BY placed_at) AS order_no
  FROM Orders
)
SELECT *
FROM CTE
WHERE order_no % 3 = 0
  AND CAST(placed_at AS date) = cast(getdate() AS date)
  ;
-- List all customer that placed more than 1 order all through promo

SELECT Customer_code, count(*) AS no_of_orders, count(promo_code_name) AS promo_orders
FROM orders
GROUP BY customer_code
HAVING count(*)>1 AND count(promo_code_name) =  count(*)
;

--percentage of customers acquired in Jan-2025 and placed their first order without promo code

WITH CTE AS
(
SELECT*, row_number () Over( partition by customer_code order by placed_at) AS rn
FROM orders
WHERE MONTH(placed_at) =1
)
SELECT count(CASE When rn = 1 AND promo_code_name is null THEN customer_code end)*100
/ COUNT(distinct Customer_code)
FROM CTE
;

