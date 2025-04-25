# balanced_tree_sql

## Overview 
Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!

Danny, the CEO of this trendy fashion company has asked you to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.

### Objectives 
- High Level Sales Analysis
- Transaction Analysis
- Product Analysis

## Dataset
The dataset for this project is sourced from [Danny Ma](https://www.linkedin.com/in/datawithdanny)

## Tools Used
- MSSQL for querying the database.
- The sales table and product details were clean,  ichanged the column type of the price column from INT to BIGINT to accumulate calculations.
```
ALTER TABLE sales
ALTER COLUMN price BIGINT;
```

## About Dataset 
- balanced_tree.product_details includes all information about the entire range that Balanced Clothing sells in their store.
- balanced_tree.sales contains product level information for all the transactions made for Balanced Tree including quantity, price, percentage discount, member status, a transaction ID and also the transaction timestamp.
  
## Business Questions and solutions 
# High level Sales analysis
- What was the total quantity sold for all products?
```
SELECT DISTINCT s.prod_id, SUM(qty) AS total_qty,pd.product_name
FROM sales s
INNER JOIN product_details pd ON s.prod_id = pd.product_id
GROUP BY s.prod_id,pd.product_name
ORDER BY 2 DESC;
-- the womens grey fashion jacket is the most sold item, 3876 quantities of the jacket was sold.
```
- What is the total generated revenue for all products before discounts?
```
SELECT prod_id,SUM(qty) AS total_qty,SUM(price) AS total_price,SUM(qty) *SUM(price) AS revenue_before_discount
FROM sales
GROUP BY prod_id
ORDER BY 4 DESC;
-- product with ID 2a2353 has the highest generated revenue before discount
```
- What was the total discount amount for all products?
```
SELECT prod_id,SUM(qty*price*discount) AS total_discount
FROM sales
GROUP BY prod_id
ORDER BY 2 DESC;
-- product with ID 2a2353 has the highest total discount
-- the product that would bring the highest revenue was given the highest discount.
```
# Transaction Analysis
- How many unique transactions were there?
```
SELECT COUNT(DISTINCT txn_id) AS unique_txn
FROM sales;
-- 2500 unique transactions were made.
```
- What is the average unique products purchased in each transaction?
```
SELECT  AVG(product_count) AS average_unique_product_count
FROM 
	(SELECT  txn_id,COUNT (DISTINCT prod_id) AS product_count
	FROM sales
	WHERE qty > 0
	GROUP BY txn_id) AS purchases_table;
-- the number of average unique product purchased in each transaction is 6
```
- What are the 25th, 50th and 75th percentile values for the revenue per transaction?
```
SELECT TOP 1PERCENTILE_CONT(0.25) WITHIN GROUP(ORDER BY revenue) OVER() AS p25_revenue,
PERCENTILE_CONT(0.50) WITHIN GROUP(ORDER BY revenue) OVER() AS median_revenue,
PERCENTILE_CONT(0.75) WITHIN GROUP(ORDER BY revenue) OVER() AS p75_revenue
FROM
	(SELECT txn_id,SUM(qty) AS total_qty,SUM(price) AS total_price,
	SUM(qty*price*discount) AS total_discount,SUM(qty) *SUM(price)* SUM(100-discount) AS revenue
	FROM sales
	GROUP BY txn_id) AS revenue_table;
-- now we can see the distribution of customer spending
-- p25 revenue shows low-spending customers or transactions (812565)
-- median revenue shows the middle value (1597566)
-- p75 shows the high-spending transactions or customers (2796651)
```
- what is the average discount value per transaction ?
```
SELECT AVG(total_discount) AS average_discount_value_per_txn
	FROM 
		(SELECT txn_id,SUM(qty*price*discount) AS total_discount
		FROM sales
		GROUP BY txn_id) AS discount_per_txn;
-- the average discount value per transaction is 6249.
```
- What is the percentage split of all transactions for members vs non-members?
```
SELECT member,txn_count,ROUND(txn_count*100.0/SUM(txn_count) OVER(), 2) AS percentage_split
FROM
(SELECT member,COUNT(DISTINCT txn_id) AS txn_count
FROM sales
GROUP BY member) AS member_table
GROUP BY member,txn_count
;
-- note that t stands for true meaning its a member(60.2%) while f stands for false (not a member)39.8%
```
- What is the average revenue for member transactions and non-member transactions?
```
SELECT member,AVG(revenue) AS average_revenue
FROM
	(SELECT member,(qty*price*(100-discount)) AS revenue
	FROM sales) AS revenue_table
GROUP BY member;
-- members bring a higher average revenue (7543) than non-members(7453).
```

# Product Analysis
- What are the top 3 products by total revenue before discount?
```
SELECT TOP 3 prod_id,SUM(qty) AS total_qty,
SUM(s.price) AS total_price,SUM(qty) *SUM(s.price) AS revenue_before_discount,pd.product_name
FROM sales s
INNER JOIN product_details pd ON s.prod_id = pd.product_id
GROUP BY prod_id,pd.product_name
ORDER BY 4 DESC;
-- Mens blue polo shirt,womens grey fashion jacket and mens white tee shirt
```
- What is the total quantity, revenue and discount for each segment?
```
SELECT pd.segment_name,SUM(qty) AS total_qty,SUM(qty*s.price*discount) AS total_discount,
SUM(qty) *SUM(s.price)*SUM(100-discount) AS revenue
FROM sales s
INNER JOIN product_details pd ON s.prod_id = pd.product_id
GROUP BY pd.segment_name
ORDER BY 4 DESC;
-- Shirts segment has the highest revenue
-- Jacket has the highest quantity (11385) sold
-- Shirt has the highest discount
```
- What is the top selling product for each segment?
```
SELECT TOP 5 pd.segment_name,pd.product_name,SUM(qty)*SUM(s.price)*SUM(100-discount) AS revenue
FROM sales s
INNER JOIN product_details pd ON s.prod_id = pd.product_id
GROUP BY pd.segment_name,pd.product_name
ORDER BY 3 DESC;
-- for the shirt segment we have blue polo and white tee shirt for Mens
-- for the Jacket segment we have grey fashion jacket for women
-- for the socks segment we have the Mens navy solid socks
-- for the Jeans segment we have the womens black straight jeans
```
- What is the total quantity, revenue and discount for each category?
```
SELECT pd.category_name,SUM(qty) AS total_quantity,SUM(qty)*SUM(s.price)*SUM(100-discount) AS revenue,
SUM(qty*s.price*discount) AS total_discount
FROM sales s
INNER JOIN product_details pd ON s.prod_id = pd.product_id
GROUP BY pd.category_name;
-- womens category sold more quantities but Mens category has a higher revenue even with the high discount.
-- This tells us that Mens category cost more .
```
- What is the top selling product for each category?
```
SELECT TOP 2 pd.category_name,pd.product_name,SUM(qty)*SUM(s.price)*SUM(100-discount) AS revenue
FROM sales s
INNER JOIN product_details pd ON s.prod_id = pd.product_id
GROUP BY pd.category_name,pd.product_name
ORDER BY 3 DESC;
-- for mens its the blue polo shirt and women its the grey fashion jacket
```
- What is the percentage split of revenue by product for each segment?
```
SELECT segment_name,product_name,revenue,ROUND(revenue*100.00/SUM(revenue) OVER(PARTITION BY segment_name),3) AS percentage_revenue
FROM
	(SELECT  pd.segment_name,product_name,SUM(qty)*SUM(s.price)*SUM(100-discount) AS revenue
	FROM sales s
	INNER JOIN product_details pd ON s.prod_id = pd.product_id
	GROUP BY pd.segment_name,product_name) AS revenue_segment
GROUP BY segment_name,product_name,revenue;
-- in jacket segment, womens grey fashion has the highest revenue
-- in jeans segment, womens black straight jeans has the highets revenue
-- in shirt segment, mens blue polo shirt has the highest revenue
-- in socks segment, mens navy solid socks has the highest revenue.
```
- What is the percentage split of revenue by segment for each category?
```
SELECT category_name,segment_name,revenue,ROUND(revenue*100.00/SUM(revenue)
OVER(PARTITION BY category_name),2) AS percentage_revenue
FROM
	(SELECT  pd.category_name,pd.segment_name,SUM(qty)*SUM(s.price)*SUM(100-discount) AS revenue
	FROM sales s
	INNER JOIN product_details pd ON s.prod_id = pd.product_id
	GROUP BY pd.segment_name,pd.category_name) AS revenue_category
GROUP BY category_name,revenue,segment_name;
-- in the mens category, shirt has a higher percentage than socks.
-- in womens category, jacket has a higher percentage than jeans
```
- What is the percentage split of total revenue by category?
```
SELECT category_name,revenue,ROUND(revenue*100.00/SUM(revenue) OVER(),2) AS percentage_revenue
FROM
	(SELECT pd.category_name,SUM(qty)*SUM(s.price)*SUM(100-discount) AS revenue
	FROM sales s
	INNER JOIN product_details pd ON s.prod_id = pd.product_id
	GROUP BY pd.category_name) AS category_revenue
GROUP BY category_name,revenue;
-- its obvious the Mens category are winning
```
we want to look at a metric called penetration
- this is the number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions
- what is the total transaction penetration for each product ?
```
SELECT prod_id,product_name,COUNT(DISTINCT txn_id) AS txn_count,
ROUND(COUNT(DISTINCT txn_id)*100.0/SUM(COUNT(DISTINCT txn_id))OVER(),2) AS penetration_percentage
FROM sales s
INNER JOIN product_details pd ON s.prod_id = pd.product_id
WHERE qty > 0
GROUP BY prod_id,product_name
ORDER BY 3 DESC;
-- product with ID f084eb (mens navy solid socks) has the highest transaction penetration.
-- with a transaction count of 1281
```
- What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
```
WITH filtered_sales AS (SELECT txn_id,prod_id
FROM sales
WHERE qty > 0
),
three_combinations AS (
SELECT f1.txn_id,f1.prod_id AS product1,
f2.prod_id AS product2, f3.prod_id AS product3
FROM filtered_sales f1
JOIN filtered_sales f2
ON f1.txn_id = f2.txn_id AND f1.prod_id < f2.prod_id
JOIN filtered_sales f3
ON f1.txn_id = f3.txn_id AND f2.prod_id < f3.prod_id
),
combo_counts AS (
SELECT product1,product2,product3,
COUNT(DISTINCT txn_id) AS combo_txn_count
FROM three_combinations
GROUP BY product1,product2,product3
)
SELECT TOP 1*
FROM combo_counts
ORDER BY combo_txn_count DESC;
-- products with ID 5d267b,9ec847,c8d436 has the most common combination(352) in a txn
```
## KEY INSIGHTS
# FINANCIAL REPORT
We want to combine all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).
So, after test running these, i was able to create a report for January, 2021.

```
-- firtsly set the reporting month and year
-- set reporting period
DECLARE @report_year INT = 2021;-- u can change the year
DECLARE @report_month INT = 1;-- 1- JANUARY, 2-FEBRUARY
-- this gives Danny a detailed report for the month of January 2021

-- common date boundaries
-- SET START AND END DATES FOR FILTERING
DECLARE @start_date DATE = DATEFROMPARTS(@report_year, @report_month, 1);
DECLARE @end_date DATE = EOMONTH(@start_date);

-- filter your sales data using the sales table timestamp column
-- create temporary filtered dataset
IF OBJECT_ID('tempdb..#monthly_sales') IS NOT NULL DROP TABLE #monthly_sales;

SELECT *
INTO #monthly_sales
FROM sales
WHERE start_txn_time BETWEEN  @start_date AND  @end_date;

-- average unique products per transaction
SELECT 'Q1_Avg_Unique_products' AS section, AVG(product_count) AS average_unique_product_count
FROM 
	(SELECT  txn_id,COUNT (DISTINCT prod_id) AS product_count
	FROM #monthly_sales
	WHERE qty >0
	GROUP BY txn_id) AS purchases_table
;

-- what is total quantity sold for all products ?
SELECT 'Q2 - Total Qty Sold' AS section, s.prod_id, SUM(s.qty) AS total_qty,pd.product_name
FROM #monthly_sales s
INNER JOIN product_details pd ON s.prod_id = pd.product_id
GROUP BY s.prod_id,pd.product_name
ORDER BY 3 DESC;

-- what are the 25th , 50th and 75th percentile values for the revenue per transaction ?
SELECT TOP 1'Q3 - Revenue_percentiles' AS section, PERCENTILE_CONT(0.25) WITHIN GROUP(ORDER BY revenue) OVER() AS p25_revenue,
PERCENTILE_CONT(0.50) WITHIN GROUP(ORDER BY revenue) OVER() AS median_revenue,
PERCENTILE_CONT(0.75) WITHIN GROUP(ORDER BY revenue) OVER() AS p75_revenue
FROM
	(SELECT txn_id,SUM(qty) AS total_qty,SUM(price) AS total_price,
	SUM(qty*price*discount) AS total_discount,SUM(qty) *SUM(price)* SUM(100-discount) AS revenue
	FROM #monthly_sales
	GROUP BY txn_id) AS revenue_table
;
-- what is the average discount value per transaction ?
SELECT 'Q4_Avg_Discount' AS metric,AVG(total_discount) AS average_discount_per_txn
	FROM 
		(SELECT txn_id,SUM(qty*price*discount) AS total_discount
		FROM #monthly_sales
		GROUP BY txn_id) AS discount_per_txn;

-- what is the percentage split of all transactions for members vs non members?
SELECT 'Q5- percentage split by member_type' AS section,member,txn_count,ROUND(txn_count*100.0/SUM(txn_count) OVER(), 2) AS percentage_split
FROM
(SELECT member,COUNT(DISTINCT txn_id) AS txn_count
FROM #monthly_sales
GROUP BY member) AS member_table
GROUP BY member,txn_count
;

-- percentage split of total revenue by category
SELECT 'Q6- Revenue % split by category' AS section, category_name,revenue,ROUND(revenue*100.00/SUM(revenue) OVER(),2) AS percentage_revenue
FROM
	(SELECT pd.category_name,SUM(qty)*SUM(m.price)*SUM(100-discount) AS revenue
	FROM #monthly_sales m
	INNER JOIN product_details pd ON m.prod_id = pd.product_id
	GROUP BY pd.category_name) AS category_revenue
GROUP BY category_name,revenue;

-- percentage split of revenue by product for each segment
SELECT 'Q7- Revenue % split by product by segment' AS section,segment_name,product_name,revenue,ROUND(revenue*100.00/SUM(revenue) OVER(PARTITION BY segment_name),3) AS percentage_revenue
FROM
	(SELECT  pd.segment_name,product_name,SUM(qty)*SUM(s.price)*SUM(100-discount) AS revenue
	FROM #monthly_sales s
	INNER JOIN product_details pd ON s.prod_id = pd.product_id
	GROUP BY pd.segment_name,product_name) AS revenue_segment
GROUP BY segment_name,product_name,revenue;

-- what is the total transaction penetration for each product ?
SELECT 'Q8- Txn_penetration by product' AS section,prod_id,product_name,COUNT(DISTINCT txn_id) AS txn_count,
ROUND(COUNT(DISTINCT txn_id)*100.0/SUM(COUNT(DISTINCT txn_id))OVER(),2) AS penetration_percentage
FROM #monthly_sales s
INNER JOIN product_details pd ON s.prod_id = pd.product_id
WHERE qty > 0
GROUP BY prod_id,product_name
ORDER BY 5 DESC;

--- what is the most combination of at least 1 quantity of any 3 products in a 1 single transaction
WITH filtered_sales AS (SELECT txn_id,prod_id
FROM #monthly_sales
WHERE qty > 0
),
three_combinations AS (
SELECT f1.txn_id,f1.prod_id AS product1,
f2.prod_id AS product2, f3.prod_id AS product3
FROM filtered_sales f1
JOIN filtered_sales f2
ON f1.txn_id = f2.txn_id AND f1.prod_id < f2.prod_id
JOIN filtered_sales f3
ON f1.txn_id = f3.txn_id AND f2.prod_id < f3.prod_id
),
combo_counts AS (
SELECT product1,product2,product3,
COUNT(DISTINCT txn_id) AS combo_txn_count
FROM three_combinations
GROUP BY product1,product2,product3
)
SELECT TOP 1 'Q9 - Top 3 products combo' AS section,*
FROM combo_counts
ORDER BY combo_txn_count DESC;

-- how many unique transactions were made ?
SELECT	'Q10 - Unique Txn made' AS section,COUNT (DISTINCT txn_id) AS unique_txn
FROM #monthly_sales
;

-- this gives important insights in the month of January 2021.
```

