# Data-Analysis-for-Zomato---A-Food-Delivery-Company
## Overview

This project demonstrates my SQL problem-solving skills through the analysis of data for Zomato, a popular food delivery company in India. The project involves setting up the database, importing data, handling null values, and solving a variety of business problems using complex SQL queries.

CREATE DATABASE zomato_db;

-- connect to zomato_db;

-- Import this script click the run/play button it will create all the tables!



-- Drop existing tables if they exist
DROP TABLE IF EXISTS deliveries;
DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS restaurants;
DROP TABLE IF EXISTS riders;

-- Create restaurants table
CREATE TABLE restaurants (
    restaurant_id SERIAL PRIMARY KEY,
    restaurant_name VARCHAR(100) NOT NULL,
    city VARCHAR(50),
    opening_hours VARCHAR(50)
);

-- Create customers table
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    reg_date DATE
);

-- Create riders table
CREATE TABLE riders (
    rider_id SERIAL PRIMARY KEY,
    rider_name VARCHAR(100) NOT NULL,
    sign_up DATE
);

-- Create Orders table
CREATE TABLE Orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    restaurant_id INT,
    order_item VARCHAR(255),
    order_date DATE NOT NULL,
    order_time TIME NOT NULL,
    order_status VARCHAR(20) DEFAULT 'Pending',
    total_amount DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
);

-- Create deliveries table
CREATE TABLE deliveries (
    delivery_id SERIAL PRIMARY KEY,
    order_id INT,
    delivery_status VARCHAR(20) DEFAULT 'Pending',
    delivery_time TIME,
    rider_id INT,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (rider_id) REFERENCES riders(rider_id)
);

-- Schemas END

SELECT * FROM riders;
SELECT * FROM restaurants;
SELECT * FROM customers;
SELECT * FROM orders;
SELECT * FROM deliveries;

-- Import data hierarcy

-- First import to customers;
-- 2nd Import to restaurants;
-- 3rd Import to orders;
-- 4th Import to riders;
-- 5th Import to deliveries;

-- IF WE IMPORT DELIVERIES DATA FIRST IT WILL GIVE AN ERROR BECAUS OF FOREIGN KEY --



-- handling NULL values ---

SELECT * FROM orders
WHERE 
     customer_id IS NULL
	 OR
	 order_id is NULL
     OR 
     order_date is null
	 or
	 order_item is null
	 or
	 order_status is null
	 or
	 total_amount is null
     
----------------------------------------------
---- Analysis & Reports ----------------------
----------------------------------------------

-- Q1

SELECT CUSTOMER_NAME,
       ORDER_ITEM,
	   TOTAL_ORDERS
FROM
    (  SELECT c.customer_id,
              c.customer_name,
	          o.order_item,
	   count(*) as total_orders,
	   DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) AS RANK
       FROM orders as O
       join customers as c 
       ON c.customer_id = o.customer_id  
       WHERE
           o.order_date >= '2023-09-02'
       AND
           c.customer_name = 'Arjun Mehta' 
       GROUP BY 1,2,3
       ORDER BY 1,4 DESC
) AS t1
     WHERE RANK <= 5
 
 
--Q2 Identify the popular time slots during which the most orders are placed. based on '2 hour slot'

SELECT 
    FLOOR(EXTRACT(HOUR FROM ORDER_TIME)/2)*2 AS STERT_TIME,
	FLOOR(EXTRACT(HOUR FROM ORDER_TIME)/2)*2 + 2 AS END_TIME,
	COUNT(*) AS TOTAL_ORDERS,
	SUM(TOTAL_AMOUNT) AS REVENEW
FROM ORDERS
GROUP BY 1,2
ORDER BY 4 DESC


--Q3 Order value Analysis
--Q . find the average order value per customer who has placed more than 750 orders.
-- return customer_name, and AOV(Average Order Value) 


SELECT 
      C.CUSTOMER_NAME,
	  O.CUSTOMER_ID,
	  COUNT(*) AS ORDER_COUNT,
	  AVG(O.TOTAL_AMOUNT)
FROM CUSTOMERS AS C
JOIN ORDERS AS O
ON C.CUSTOMER_ID = O.CUSTOMER_ID
GROUP BY 1,2
HAVING COUNT(*) >750
ORDER BY 3 DESC



--Q4 HIGH VALUE CUSTOMERS
--QUESTION : List the customers who soend more than 100K IN TOTAL on food orders.
--return Customer_name and customer_id


SELECT 
      C.CUSTOMER_NAME,
	  O.CUSTOMER_ID,
	  SUM(O.TOTAL_AMOUNT)
FROM CUSTOMERS AS C
JOIN ORDERS AS O
ON C.CUSTOMER_ID = O.CUSTOMER_ID
GROUP BY 1,2
HAVING SUM(O.TOTAL_AMOUNT) > 100000
ORDER BY 3 DESC

-- Q5 orders without delivery
-- Question : write A Query to find orders were placed but ot delivered
-- returns each resturant_name, city, no of not delivered orders


SELECT * FROM oRDERS
SELECT * FROM deliveries;


SELECT  
      R.restaurant_NAMe,
	  COUNT (O.ORDER_ID)
FROM ORDERS AS O
LEFT JOIN restaurants AS R
ON R.RESTAURANT_ID = O.RESTAURANT_ID
LEFT JOIN DELIVERIES AS D
ON D.ORDER_ID = O.ORDER_ID
WHERE D.DELIVERY_ID IS NULL
GROUP BY 1
ORDER BY 2 DESC

--2ND QUERY 
select *
from orders as o
Join restaurants as r
on r.restaurant_id = o.restaurant_id
where 
     o.order_id not in (SELECT ORDER_ID FROM DELIVERIES)
	 
-- Q6 reasturnt revenew ranking
-- rank restanurant ranking by their total revenue from last year,including their name,
-- total revenue, and rank within their city

SELECT * FROM riders;
SELECT * FROM restaurants;
SELECT * FROM customers;
SELECT * FROM orders;
SELECT * FROM deliveries;

WITH RANKING_TABLE AS
     (   SELECT r.restaurant_id, 
                r.restaurant_name,
	            R.CITY,
	     RANK() OVER(PARTITION BY R.CITY ORDER BY sum(o.total_amount)DESC) AS RANK,
	     sum(o.total_amount) AS REVENUE
         FROM restaurants r
         JOIN ORDERS O
         ON r.RESTAURANT_ID = O.RESTAURANT_ID  
         GROUP BY 1,2,3
      )
SELECT * 
FROM RANKING_TABLE
WHERE RANK = 1

--Q7 



