# T-SQL Fundamentals Exercises



## Chapter 2 Exercises: Single-Table Queries

**-- 1. Write a query against the Sales.Orders table that returns orders placed in June 2015**

SELECT * 

FROM Sales.Orders

WHERE MONTH(orderdate) = 6;


**-- 2. Write a query against the Sales.Orders table that returns orders placed on the last day of the month**

SELECT *

FROM Sales.Orders

WHERE orderdate = EOMONTH(orderdate);

**-- 3. Write a query against the HR.Employees table that returns employees with a last name containing the letter e twice or more**

SELECT empid, firstname, lastname

FROM HR.Employees

WHERE LEN(LastName) - LEN(REPLACE(LastName, 'e', '')) >= 2;

**-- 4. Write a query against the Sales.OrdersDetails table that returns orders with a total value greater than 10,000, sorted by total value**

SELECT orderid, SUM(qty * unitprice) AS totalvalue

FROM Sales.OrderDetails

GROUP BY orderid

HAVING (SUM(qty * unitprice) > 10000)

ORDER BY totalvalue DESC;

**-- 5. To check the validity of the data, write a query against the HR.Employees table that returns employees with a last name that starts with a lowercase English letter in the range a through z.**

SELECT empid, lastname

FROM HR.Employees

WHERE lastname COLLATE Latin1_General_CS_AS LIKE N'[abcdefghijklmnopqrstuvwxyz]%';

**-- 6. Explain the difference between the following 2 queries:**

**-- Query 1**

--SELECT empid, COUNT(*) AS numorders

--FROM Sales.Orders

--WHERE orderdate <  '20160501'

--GROUP BY empid;

**-- Query 2**

--SELECT empid, COUNT(*) AS numorders

--FROM Sales.Orders

--GROUP BY empid

--HAVING MAX(orderdate) < '20160501';

-- The first query groups all orders by empid that takes place before the given date. The second query does the same, but also filters out employees who have not placed an order after the given date, which is why the first query gives 9 rows of data, to the other's four

**-- 7. Write a query against the Sales.Orders table that returns the three shipped-to countries with the highest average freight in 2015**

SELECT TOP 3 shipcountry, AVG(freight) AS avgfreight

FROM Sales.Orders

WHERE YEAR(orderdate) = 2015

GROUP BY shipcountry

ORDER BY avgfreight DESC;

**-- 8. Write a query against the Sales.Orders table that calculates row numbers for orders based on order date ordering (using the order ID as the tiebreaker) for each customer seperately**

SELECT custid, orderdate, orderid, ROW_NUMBER() OVER(PARTITION BY custid ORDER BY orderdate, orderid) AS rownum

FROM Sales.Orders;

**-- 9. Using the HR.Employees table, write a SELECT statement that returns for each employee the gender based on the title of courtesy. For 'Ms.' and 'Mrs.' return 'Female'; for 'Mr.' return 'Male' and in all other cases return 'Unknown'**

SELECT empid, firstname, lastname, titleofcourtesy, 

 CASE WHEN  titleofcourtesy = 'Ms.' THEN  'Female'

   WHEN  titleofcourtesy = 'Mrs.' THEN    'Female'

   WHEN  titleofcourtesy = 'Mr.' THEN     'Male'

   ELSE	                                  'Unknown'

 END AS gender

FROM HR.Employees;

**-- 10. Write a query against the Sales.Customers table that returns for each customer the customer ID and region. Sort the rows in the output by region, having NULLs sort last (after non-NULL values).**

SELECT custid, region

FROM Sales.Customers

ORDER BY 

 	CASE WHEN REGION IS NULL THEN 1 ELSE 0 END, region;


-- ## **Chapter 3 Exercises: Joins**

**-- 1. Write a query that generates five copies of each employee row.**

SELECT E.empid, E.firstname, E.lastname, N.n

FROM HR.Employees AS E

 	CROSS JOIN dbo.nums AS N
	
 	WHERE N.n <= 5
	
 	ORDER BY N, empid;

**--  1-2. Write a query that returns a row for each employee and day in the range June 12, 2016 through June 16, 2016**

SELECT E.empid,

DATEADD(day, D.n - 1, CAST ('20160612' AS DATE)) AS DT

FROM HR.Employees AS E

 	CROSS JOIN dbo.nums AS D
	
 	WHERE D.n <= DATEDIFF(day, '20160612', '20160616') + 1
	
 	ORDER BY empid, dt;

**-- 2. Explain wha's wrong in the following query, and provide a correct alternative:**
	
 --SELECT Customers.custid, Customers.companyname, Orders.orderid, Orders.orderdate

 	--FROM Sales.Customers AS C
	
  		--INNER JOIN Sales.Orders AS O
		
   			--ON Customers.custid = Orders.custid;

	--Aliases for both tables have been has been declared, so with logical processing steps, it is not recognizing them in the query if they do not have the same naming scheme. Changing it to below, fixes this problem. 

	SELECT C.custid, C.companyname, O.orderid, O.orderdate
	
 	FROM Sales.Customers AS C
	
  		INNER JOIN Sales.Orders AS O
		
   			ON C.custid = O.custid;


**-- 3. Return US customers, and for each customer return the total number of orders and total quantities**

SELECT C.custid,COUNT(DISTINCT O.orderid) AS OrderTotal, SUM(OD.qty) AS TotalQuantity

FROM Sales.Customers AS C

INNER JOIN Sales.Orders AS O

ON O.custid = C.custid

INNER JOIN Sales.OrderDetails AS OD

ON OD.orderid = O.orderid

WHERE C.country = N'USA'

GROUP BY C.custid;

**-- 4. Return customers and their orders, including customers who placed no orders**

SELECT C.custid, C.companyname, O.orderid, O.orderdate

FROM Sales.Customers AS C

LEFT OUTER JOIN Sales.Orders AS O

ON O.custid = C.custid;

**-- 5. Return customers who placed no orders**

SELECT C.custid, C.companyname

FROM Sales.Customers AS C

LEFT OUTER JOIN Sales.Orders AS O

ON O.custid = C.custid

WHERE orderid IS NULL;

**-- 6. Return Customers with orders placed on Feburary 12, 2016 along with their orders:**

SELECT C.custid, C.companyname, O.orderid, O.orderdate

FROM Sales.Customers AS C

INNER JOIN Sales.Orders AS O

ON O.custid = C.custid

WHERE O.orderdate = '2016-02-12';

**--7. Write a query that returns all customers in the output, but matches them with their respective orders only if they were placed on February 12, 2016**

SELECT C.custid, C.companyname, O.orderid, O.orderdate

FROM Sales.Customers AS C

LEFT OUTER JOIN Sales.Orders AS O

ON O.custid = C.custid

AND O.orderdate = '20160212';

**-- 8. Explain why the following query isn't a correct query for exercise 7**

--SELECT C.custid, C.companyname, O.orderid, O.orderdate

--FROM Sales.Customers AS C

 	--LEFT OUTER JOIN Sales.Orders AS O
	
 	-- ON O.custid = C.custid

--WHERE O.orderdate = '20160212'
	
 	--OR O.orderid IS NULL

-- This query will only return those who have not placed an order, or those who did on the given date. All other customers will not be included.

**-- 9.  Return all customers, and for each return a Yes/No value depending on whether the customer placed orders on Feburary 12, 2016:**

 SELECT DISTINCT C.custid, C.companyname, 

 	CASE WHEN O.orderid IS NOT NULL THEN 'Yes' ELSE 'No' END AS OrderOn20160212

FROM Sales.Customers AS C

 	LEFT OUTER JOIN Sales.Orders AS O
	
  		ON O.custid = C.Custid
	
  		AND O.orderdate = '20160212';

