# 🛒 Retail Sales Analysis Using SQL

**Author:** Paul Muriithi  
**Database:** PostgreSQL  
**Goal:** Perform end-to-end SQL-based retail data analysis — from cleaning, filtering, summarizing, to extracting business insights from transaction-level data.

---

## 📦 1. Database & Table Setup

- Create a clean SQL workspace.
- Ensure the main transactional table is structured for querying.
- Confirm the setup with a sample preview.

```sql
-- Create database
CREATE DATABASE Sales_Database_SQL_Project;

-- Drop existing table to avoid conflicts
DROP TABLE IF EXISTS retail_sales;

-- Create a structured retail_sales table to store customer transactions
CREATE TABLE retail_sales (
    transactions_id INT PRIMARY KEY,
    sale_date DATE,	
    sale_time TIME,
    customer_id INT,
    gender VARCHAR(10),
    age INT,
    category VARCHAR(15),
    quantity INT,
    price_per_unit FLOAT,	
    cogs FLOAT,
    total_sale FLOAT
);

-- Preview the table to check column structure and sample data
SELECT * FROM retail_sales
LIMIT 10;
```

---

## 🧹 2. Data Cleaning & Validation

- Count total records.
- Check number of unique customers and categories.
- Identify and remove rows with null/missing values.

```sql
-- Count total rows
SELECT COUNT(*) FROM retail_sales;

-- Count of unique customers
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;

-- Distinct product categories in dataset
SELECT DISTINCT category FROM retail_sales;

-- Check for nulls in critical fields
SELECT * FROM retail_sales
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;

-- Remove rows with nulls to ensure data integrity
DELETE FROM retail_sales
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;
```

---

## 📊 3. Exploratory Queries

- Examine specific transactions by date or category.
- Identify high-volume orders in a month.
- Aggregate sales by category and customer segment.
- Spot high-value orders.

```sql
-- View all transactions on a specific date
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';

-- Clothing orders with quantity ≥ 4 in November 2022
SELECT transactions_id, sale_date, category, quantity, total_sale
FROM retail_sales
WHERE 
    category = 'Clothing'
    AND TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
    AND quantity >= 4;

-- Total orders and net sales per category
SELECT 
    category,
    COUNT(*) AS total_orders,
    SUM(total_sale) AS net_sale
FROM retail_sales
GROUP BY category;

-- Average age of customers buying 'Beauty' products
SELECT
    ROUND(AVG(age), 2) AS beauty_customers_avg_age
FROM retail_sales
WHERE category = 'Beauty';

-- Identify high-value transactions
SELECT * FROM retail_sales
WHERE total_sale > 1000;

-- Number of transactions by gender per product category
SELECT 
    category,
    gender,
    COUNT(*) AS total_trans
FROM retail_sales
GROUP BY category, gender
ORDER BY category;
```

---

## 📅 4. Monthly & Customer Trends

- Determine best-performing months by average sale.
- Identify top 5 spenders.
- Count unique buyers per category.

```sql
-- Best-selling month each year based on average sales
SELECT 
    year, month, avg_sale
FROM 
(    
    SELECT 
        EXTRACT(YEAR FROM sale_date) AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        AVG(total_sale) AS avg_sale,
        RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY AVG(total_sale) DESC) AS rank
    FROM retail_sales
    GROUP BY 1, 2
) AS t1
WHERE rank = 1;

-- Top 5 customers by total spending
SELECT 
    customer_id,
    SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC
LIMIT 5;

-- Number of unique buyers per category
SELECT 
    category,    
    COUNT(DISTINCT customer_id) AS cnt_unique_cs
FROM retail_sales
GROUP BY category;
```

---

## ⏰ 5. Time-of-Day Analysis

- Create sales shifts based on time.
- Evaluate order counts across Morning, Afternoon, and Evening.

```sql
-- Sales shift: classify orders by time of day
WITH hourly_sale AS (
    SELECT *,
        CASE
            WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
            WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
            ELSE 'Evening'
        END AS shift
    FROM retail_sales
)
SELECT 
    shift,
    COUNT(*) AS total_orders    
FROM hourly_sale
GROUP BY shift;
```

---

## 📌 6. Key Questions Answered

| Question                                                      | SQL Logic / Strategy                                                     |
|---------------------------------------------------------------|--------------------------------------------------------------------------|
| What’s the best month by average sales in each year?          | EXTRACT, GROUP BY, RANK() over year partitions                           |
| Which product category generates the most sales?              | GROUP BY category with SUM(total_sale)                                   |
| Who are the highest-spending customers?                       | GROUP BY customer_id with SUM and ORDER BY                               |
| When do customers buy the most?                               | Use EXTRACT(HOUR) and classify shifts via CASE                           |
| How many unique customers per category?                       | COUNT(DISTINCT customer_id) with GROUP BY category                       |
| Which age group buys 'Beauty' items?                          | AVG(age) filter WHERE category = 'Beauty'                                |
| How frequent are high-ticket orders?                          | Filter WHERE total_sale > 1000                                           |
| Does gender affect category-wise shopping patterns?           | GROUP BY gender, category with COUNT(*)                                  |

---

---

## ✅ Optional Extensions

- Add gross margin (`total_sale - cogs`) for profit analysis.
- Group customers by frequency or recency.
- Visualize results in Tableau or Power BI.
- Connect to Python for automated ETL or dashboards.

---

