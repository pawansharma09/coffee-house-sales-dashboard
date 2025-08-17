## SQL Queries

### Data Cleaning

-- Convert transaction_date column to proper date format
UPDATE coffee_shop_sales
SET transaction_date = STR_TO_DATE(transaction_date, '%d-%m-%Y');

-- Alter transaction_date column to DATE data type
ALTER TABLE coffee_shop_sales
MODIFY COLUMN transaction_date DATE;

-- Convert transaction_time column to proper time format
UPDATE coffee_shop_sales
SET transaction_time = STR_TO_DATE(transaction_time, '%H:%i:%s');

-- Alter transaction_time column to TIME data type
ALTER TABLE coffee_shop_sales
MODIFY COLUMN transaction_time TIME;

-- Fix transaction_id column name
ALTER TABLE coffee_shop_sales
CHANGE COLUMN `ï»¿transaction_id` transaction_id INT;

-- Check data types
DESCRIBE coffee_shop_sales;

### Total Sales

SELECT ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales 
FROM coffee_shop_sales 
WHERE MONTH(transaction_date) = 5;

### Total Sales KPI - MoM Growth

SELECT 
    MONTH(transaction_date) AS month,
    ROUND(SUM(unit_price * transaction_qty)) AS total_sales,
    (SUM(unit_price * transaction_qty) - LAG(SUM(unit_price * transaction_qty), 1)
    OVER (ORDER BY MONTH(transaction_date))) / LAG(SUM(unit_price * transaction_qty), 1) 
    OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM coffee_shop_sales
WHERE MONTH(transaction_date) IN (4, 5)
GROUP BY MONTH(transaction_date)
ORDER BY MONTH(transaction_date);

### Total Orders

SELECT COUNT(transaction_id) AS Total_Orders
FROM coffee_shop_sales 
WHERE MONTH(transaction_date) = 5;

### Total Orders KPI - MoM Growth

SELECT 
    MONTH(transaction_date) AS month,
    COUNT(transaction_id) AS total_orders,
    (COUNT(transaction_id) - LAG(COUNT(transaction_id), 1) 
    OVER (ORDER BY MONTH(transaction_date))) / LAG(COUNT(transaction_id), 1) 
    OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM coffee_shop_sales
WHERE MONTH(transaction_date) IN (4, 5)
GROUP BY MONTH(transaction_date)
ORDER BY MONTH(transaction_date);

### Total Quantity Sold

SELECT SUM(transaction_qty) AS Total_Quantity_Sold
FROM coffee_shop_sales 
WHERE MONTH(transaction_date) = 5;

### Total Quantity Sold KPI - MoM Growth

SELECT 
    MONTH(transaction_date) AS month,
    SUM(transaction_qty) AS total_quantity_sold,
    (SUM(transaction_qty) - LAG(SUM(transaction_qty), 1) 
    OVER (ORDER BY MONTH(transaction_date))) / LAG(SUM(transaction_qty), 1) 
    OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM coffee_shop_sales
WHERE MONTH(transaction_date) IN (4, 5)
GROUP BY MONTH(transaction_date)
ORDER BY MONTH(transaction_date);

### Calendar Table – Daily Sales, Quantity, Orders

SELECT
    SUM(unit_price * transaction_qty) AS total_sales,
    SUM(transaction_qty) AS total_quantity_sold,
    COUNT(transaction_id) AS total_orders
FROM coffee_shop_sales
WHERE transaction_date = '2023-05-18';

### Daily Sales Trend

SELECT 
    DAY(transaction_date) AS day_of_month,
    SUM(unit_price * transaction_qty) AS total_sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5
GROUP BY DAY(transaction_date)
ORDER BY DAY(transaction_date);

### Compare Daily Sales with Average

SELECT 
    day_of_month,
    CASE 
        WHEN total_sales > avg_sales THEN 'Above Average'
        WHEN total_sales < avg_sales THEN 'Below Average'
        ELSE 'Average'
    END AS sales_status,
    total_sales
FROM (
    SELECT 
        DAY(transaction_date) AS day_of_month,
        SUM(unit_price * transaction_qty) AS total_sales,
        AVG(SUM(unit_price * transaction_qty)) OVER () AS avg_sales
    FROM coffee_shop_sales
    WHERE MONTH(transaction_date) = 5
    GROUP BY DAY(transaction_date)
) AS sales_data
ORDER BY day_of_month;

### Sales by Weekday vs Weekend

SELECT 
    CASE WHEN DAYOFWEEK(transaction_date) IN (1, 7) THEN 'Weekends' ELSE 'Weekdays' END AS day_type,
    SUM(unit_price * transaction_qty) AS total_sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5
GROUP BY CASE WHEN DAYOFWEEK(transaction_date) IN (1, 7) THEN 'Weekends' ELSE 'Weekdays' END;

### Sales by Store Location

SELECT store_location, SUM(unit_price * transaction_qty) AS Total_Sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5
GROUP BY store_location
ORDER BY Total_Sales DESC;

### Sales by Product Category

SELECT product_category, SUM(unit_price * transaction_qty) AS Total_Sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5
GROUP BY product_category
ORDER BY Total_Sales DESC;

### Top 10 Products by Sales

SELECT product_type, SUM(unit_price * transaction_qty) AS Total_Sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5
GROUP BY product_type
ORDER BY Total_Sales DESC
LIMIT 10;

### Sales by Day and Hour

SELECT 
    SUM(unit_price * transaction_qty) AS Total_Sales,
    SUM(transaction_qty) AS Total_Quantity,
    COUNT(*) AS Total_Orders
FROM coffee_shop_sales
WHERE DAYOFWEEK(transaction_date) = 3 
  AND HOUR(transaction_time) = 8 
  AND MONTH(transaction_date) = 5;

### Sales by Day of Week (Full Week)

SELECT 
    CASE 
        WHEN DAYOFWEEK(transaction_date) = 2 THEN 'Monday'
        WHEN DAYOFWEEK(transaction_date) = 3 THEN 'Tuesday'
        WHEN DAYOFWEEK(transaction_date) = 4 THEN 'Wednesday'
        WHEN DAYOFWEEK(transaction_date) = 5 THEN 'Thursday'
        WHEN DAYOFWEEK(transaction_date) = 6 THEN 'Friday'
        WHEN DAYOFWEEK(transaction_date) = 7 THEN 'Saturday'
        ELSE 'Sunday'
    END AS Day_of_Week,
    SUM(unit_price * transaction_qty) AS Total_Sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5
GROUP BY Day_of_Week;

### Sales by Hour (Full Day)

SELECT 
    HOUR(transaction_time) AS Hour_of_Day,
    SUM(unit_price * transaction_qty) AS Total_Sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5
GROUP BY Hour_of_Day
ORDER BY Hour_of_Day;
