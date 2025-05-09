# SQL-Project-Sales

-- selecting the Database


use bike_sales


-- Customers: Remove duplicates based on customer_id


SELECT customer_id, COUNT(*) AS count
FROM customers
GROUP BY customer_id
HAVING COUNT(*) > 1;


-- Checking for NULL values


SELECT * 
FROM customers 
WHERE customer_id IS NULL;


-- Drop rows with missing values


DELETE FROM customers
WHERE customer_id IS NULL;


-- Remove leading/trailing spaces


UPDATE customers 
SET customer_id = TRIM(customer_id);


-- Concatenate first name and last name


SELECT CONCAT(first_name, ' ', last_name) AS full_name 
FROM customers;


-- Check invalid emails id's


SELECT * 
FROM customers 
WHERE email NOT LIKE '%@%.%';


-- Counting how many customers present


SELECT distinct(count(*)) AS total_customers 
FROM customers;


-- Counting how many products present


SELECT distinct(count(*)) AS total_products 
FROM products;


-- Counting total orders placed


SELECT distinct(count(*)) AS total_orders 
FROM orders;


-- Total Revenue calculation


SELECT ROUND(SUM(oi.quantity * oi.list_price), 2) AS total_revenue
FROM order_items oi;


-- Total number of orders by store


SELECT st.store_name, COUNT(o.order_id) AS order_count
FROM stores st
JOIN orders o ON st.store_id = o.store_id
GROUP BY st.store_name;


-- Show average product price in each category


SELECT cat.category_name, ROUND(AVG(p.list_price), 2) AS avg_price
FROM products p
JOIN categories cat ON p.category_id = cat.category_id
GROUP BY cat.category_name;


-- Which brands have more than 5 products?


SELECT b.brand_name, COUNT(p.product_id) AS product_count
FROM brands b
JOIN products p ON b.brand_id = p.brand_id
GROUP BY b.brand_name
HAVING product_count > 5;


-- What is the geographic distribution of customers?


SELECT city, state, COUNT(*) AS customer_count
FROM customers
GROUP BY city, state
ORDER BY customer_count DESC;


-- Which products are consistently out of stock?


SELECT 
  p.product_name,
  COUNT(*) AS stockouts
FROM stocks s
JOIN products p ON s.product_id = p.product_id
WHERE s.quantity = 0
GROUP BY p.product_name
ORDER BY stockouts DESC;


--  Monthly Revenue Trend


SELECT 
    DATE_FORMAT(o.order_date, '%Y-%m') AS month,
    ROUND(SUM(oi.quantity * oi.list_price), 2) AS revenue
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY month
ORDER BY month;


-- Top 10 Customers by Spend


SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    ROUND(SUM(oi.quantity * oi.list_price), 2) AS total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, customer_name
ORDER BY total_spent DESC
LIMIT 10;


--  Top Selling Products


SELECT 
    p.product_name,
    SUM(oi.quantity) AS total_units_sold,
    ROUND(SUM(oi.quantity * oi.list_price), 2) AS revenue
FROM products p
JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY p.product_id, p.product_name
ORDER BY revenue DESC
LIMIT 10;


--  Revenue by Store


SELECT 
    s.store_name,
    ROUND(SUM(oi.quantity * oi.list_price), 2) AS store_revenue
FROM stores s
JOIN staffs st ON s.store_id = st.store_id
JOIN orders o ON st.staff_id = o.staff_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY s.store_id
ORDER BY store_revenue DESC;


--  Staff Performance by Sales


SELECT 
    st.staff_id,
    CONCAT(st.first_name, ' ', st.last_name) AS staff_name,
    ROUND(SUM(oi.quantity * oi.list_price), 2) AS total_sales
FROM staffs st
JOIN orders o ON st.staff_id = o.staff_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY st.staff_id, staff_name
ORDER BY total_sales DESC;

-- Average Order Value (AOV)


SELECT 
    ROUND(SUM(oi.quantity * oi.list_price) / COUNT(DISTINCT oi.order_id), 2) AS avg_order_value
FROM order_items oi;

-- Product Categories Revenue


SELECT 
    cat.category_name,
    ROUND(SUM(oi.quantity * oi.list_price), 2) AS category_revenue
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN categories cat ON p.category_id = cat.category_id
GROUP BY cat.category_id,cat.category_name
ORDER BY category_revenue DESC;

-- Category-Wise Product Ranking


SELECT 
  c.category_name,
  p.product_name,
  SUM(oi.quantity) AS total_units_sold,
  RANK() OVER (PARTITION BY c.category_name ORDER BY SUM(oi.quantity) DESC) AS category_rank
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_name, p.product_name;

