# SQLPractice-- 

Chapter 2 Exercises

-- 1. Write a query against the Sales.Orders table that returns orders placed in June 2015
SELECT * 
FROM Sales.Orders
WHERE MONTH(orderdate) = 6;

-- 2. Write a query against the Sales.Orders table that returns orders placed on the last day of the month
SELECT *
FROM Sales.Orders
WHERE orderdate = EOMONTH(orderdate);

-- 3. Write a query against the HR.Employees table that returns employees with a last name containing the letter e twice or more
SELECT empid, firstname, lastname
FROM HR.Employees
WHERE LEN(LastName) - LEN(REPLACE(LastName, 'e', '')) >= 2;

-- 4. Write a query against the Sales.OrdersDetails table that returns orders with a total value greater than 10,000, sorted by total value
SELECT orderid, SUM(qty * unitprice) AS totalvalue
FROM Sales.OrderDetails
GROUP BY orderid
HAVING (SUM(qty * unitprice) > 10000)
ORDER BY totalvalue DESC;

-- 5. To check the validity of the data, write a query against the HR.Employees table that returns employees with a last name that starts with a lowercase English letter in the range a through z.
SELECT empid, lastname
FROM HR.Employees
WHERE lastname COLLATE Latin1_General_CS_AS LIKE N'[abcdefghijklmnopqrstuvwxyz]%';

-- 6. Explain the difference between the following 2 queries:

-- Query 1
--SELECT empid, COUNT(*) AS numorders
--FROM Sales.Orders
--WHERE orderdate <  '20160501'
--GROUP BY empid;

-- Query 2
--SELECT empid, COUNT(*) AS numorders
--FROM Sales.Orders
--GROUP BY empid
--HAVING MAX(orderdate) < '20160501';

-- The first query groups all orders by empid that takes place before the given date. The second query does the same, but also filters out employees who have not placed an order after the given date, which is why the first query gives 9 rows of data, to the other's four

-- 7. Write a query against the Sales.Orders table that returns the three shipped-to countries with the highest average freight in 2015
SELECT TOP 3 shipcountry, AVG(freight) AS avgfreight
FROM Sales.Orders
WHERE YEAR(orderdate) = 2015
GROUP BY shipcountry
ORDER BY avgfreight DESC;

-- 8. Write a query against the Sales.Orders table that calculates row numbers for orders based on order date ordering (using the order ID as the tiebreaker) for each customer seperately

SELECT custid, orderdate, orderid, ROW_NUMBER() OVER(PARTITION BY custid ORDER BY orderdate, orderid) AS rownum
FROM Sales.Orders;

-- 9. Using the HR.Employees table, write a SELECT statement that returns for each employee the gender based on the title of courtesy. For 'Ms.' and 'Mrs.' return 'Female'; for 'Mr.' return 'Male' and in all other cases return 'Unknown'

SELECT empid, firstname, lastname, titleofcourtesy, 
	CASE WHEN  titleofcourtesy = 'Ms.' THEN  'Female'
		 WHEN  titleofcourtesy = 'Mrs.' THEN 'Female'
		 WHEN  titleofcourtesy = 'Mr.' THEN  'Male'
		 ELSE								 'Unknown'
	END AS gender
FROM HR.Employees;

-- 10. Write a query against the Sales.Customers table that returns for each customer the customer ID and region. Sort the rows in the output by region, having NULLs sort last (after non-NULL values).

SELECT custid, region
FROM Sales.Customers
ORDER BY 
	CASE WHEN REGION IS NULL THEN 1 ELSE 0 END, region;
