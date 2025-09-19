# Retail Sales Analysis SQL Project
Project Overview

## Project Title: Retail Sales Analysis
Level: Beginner
Database: super_store

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a sales database, performing exploratory data analysis (EDA), and deriving insights from the data using SQL queries. This project is ideal for beginners in data analysis who want to build a solid foundation in SQL and business analytics.

### Objectives

- Set up a sales database: Create and populate a sales database with the provided Superstore dataset.

- Data Cleaning: Identify and handle missing or null values to ensure reliable analysis.

- Exploratory Data Analysis (EDA): Understand the dataset through basic descriptive statistics and aggregations.

- Business Analysis: Use SQL to analyze key metrics, trends, customer behavior, and product performance.

Project Structure
1. Database Setup

- Database Creation: Create a database to store Superstore sales data.

- Table Creation: A table named super_store stores columns such as order ID, order date, ship date, customer ID, product category, sub-category, sales, profit, discount, quantity, ship mode, segment, city, and state.
```sql
    CREATE TABLE super_store (
        order_id VARCHAR(50) PRIMARY KEY,
        order_date VARCHAR(20),
        ship_date VARCHAR(20),
        customer_id VARCHAR(50),
        customer_name VARCHAR(100),
        segment VARCHAR(50),
        country VARCHAR(50),
        city VARCHAR(50),
        state VARCHAR(50),
        postal_code VARCHAR(20),
        region VARCHAR(50),
        product_id VARCHAR(50),
        category VARCHAR(50),
        sub_category VARCHAR(50),
        product_name VARCHAR(100),
        sales FLOAT,
        quantity INT,
        discount FLOAT,
        profit FLOAT,
        ship_mode VARCHAR(50)
    );
```

2. Data Exploration & Cleaning

- Record Count: Determine total records.

- Unique Customers: Count unique customer IDs.

- Unique Products & Categories: Identify all products and categories.

- Null Check: Detect and handle missing data.
```sql
    SELECT COUNT(*) FROM super_store;
    SELECT COUNT(DISTINCT customer_id) FROM super_store;
    SELECT DISTINCT category FROM super_store;

    SELECT * FROM super_store
    WHERE order_date IS NULL OR customer_id IS NULL OR category IS NULL;
```

3. Data Analysis & Findings
The project contains multiple SQL analyses to explore business insights, including:
  1.Calculate the overall business metrics: total number of orders, unique customers, total sales, total profit, and average profit margin.
```sql
SELECT 
    COUNT(DISTINCT order_id) AS total_orders,
    COUNT(DISTINCT customer_id) AS unique_customers,
    SUM(sales) AS total_sales,
    SUM(profit) AS total_profit,
    SUM(profit) / NULLIF(SUM(sales), 0) AS avg_profit_margin
FROM super_store;
```

  2.Find the top 10 products (by product_name) with the highest sales, including quantity sold and profit margin for each.
```sqlsql
SELECT 
    product_name,
    SUM(sales) AS total_sales,
    SUM(quantity) AS total_quantity,
    SUM(profit) / NULLIF(SUM(sales), 0) AS profit_margin
FROM super_store
GROUP BY product_name
ORDER BY total_sales DESC
LIMIT 10;
```

3.Analyze sales, profit, and average order value by customer segment (Consumer, Corporate, Home Office).
```sql
SELECT 
    segment,
    SUM(sales) AS total_sales,
    SUM(profit) AS total_profit,
    SUM(sales) / COUNT(order_id) AS average_order_value
FROM super_store
GROUP BY segment;
```

4.Calculate monthly sales and profit trends over time to identify seasonal patterns and growth trends.
```sqlsql
WITH monthly AS (
    SELECT
        DATE_FORMAT(STR_TO_DATE(order_date, '%m/%d/%Y'), '%Y-%m') AS month,
        SUM(sales) AS total_sales,
        SUM(profit) AS total_profit
    FROM super_store
    GROUP BY DATE_FORMAT(STR_TO_DATE(order_date, '%m/%d/%Y'), '%Y-%m')
)
SELECT
    month,
    total_sales,
    total_profit,
    LAG(total_sales, 1) OVER (ORDER BY month) AS prev_sales,
    ROUND(
        (total_sales - LAG(total_sales, 1) OVER (ORDER BY month)) 
        / NULLIF(LAG(total_sales, 1) OVER (ORDER BY month), 0) * 100, 
        2
    ) AS sales_growth_pct,
    LAG(total_profit, 1) OVER (ORDER BY month) AS prev_profit,
    ROUND(
        (total_profit - LAG(total_profit, 1) OVER (ORDER BY month)) 
        / NULLIF(LAG(total_profit, 1) OVER (ORDER BY month), 0) * 100, 
        2
    ) AS profit_growth_pct
FROM monthly
ORDER BY month;
```

5.Analyze sales and profit margin by city and state, identifying the top 5 best-performing states.
```sql
SELECT
    state,
    SUM(sales) AS total_sales,
    SUM(profit) AS total_profit,
    ROUND(SUM(profit) / NULLIF(SUM(sales), 0) * 100, 2) AS profit_margin_pct
FROM super_store
GROUP BY state
ORDER BY profit_margin_pct DESC
LIMIT 5;
```

6.Calculate average shipping time (ship_date - order_date) by ship_mode and city to identify efficiency patterns.
```sql
SELECT 
    ship_mode,
    city,
    AVG(DATEDIFF(
        STR_TO_DATE(ship_date, '%m/%d/%Y'),
        STR_TO_DATE(order_date, '%m/%d/%Y')
    )) AS avg_shipping_days
FROM super_store
GROUP BY ship_mode, city 
ORDER BY avg_shipping_days;
```

7.Analyze the relationship between discounts and profit: compare average profit of discounted vs non-discounted orders by category.
```sql
SELECT 
    category,
    CASE WHEN discount > 0 THEN 'Discounted' ELSE 'Non-Discounted' END AS discount_flag,
    ROUND(AVG(profit), 2) AS avg_profit
FROM super_store
GROUP BY category, discount_flag
ORDER BY category, discount_flag;
```

8.Rank customers by total sales within each region, showing top 3 customers per state.
```sql
WITH customer_sales AS (
    SELECT 
        state,
        customer_name,
        SUM(sales) AS total_sales
    FROM super_store
    GROUP BY state, customer_name
),
ranked AS (
    SELECT 
        state,
        customer_name,
        total_sales,
        RANK() OVER (PARTITION BY state ORDER BY total_sales DESC) AS rnk
    FROM customer_sales
)
SELECT *
FROM ranked
WHERE rnk <= 3
ORDER BY state, rnk, total_sales DESC;
```

9.Calculate running total of sales over time (by month) to show cumulative business growth.
```sql
WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(STR_TO_DATE(order_date, '%m/%d/%Y'), '%Y-%m') AS month,
        SUM(sales) AS total_sales
    FROM super_store
    GROUP BY DATE_FORMAT(STR_TO_DATE(order_date, '%m/%d/%Y'), '%Y-%m')
)
SELECT 
    month,
    total_sales,
    SUM(total_sales) OVER (ORDER BY month) AS running_total_sales
FROM monthly_sales
ORDER BY month;

10.Compare monthly sales with the same month from the previous year and calculate growth percentage.
WITH monthly_sales AS (
    SELECT 
        YEAR(STR_TO_DATE(order_date, '%m/%d/%Y')) AS year,
        MONTH(STR_TO_DATE(order_date, '%m/%d/%Y')) AS month,
        SUM(sales) AS total_sales
    FROM super_store
    GROUP BY YEAR(STR_TO_DATE(order_date, '%m/%d/%Y')), 
             MONTH(STR_TO_DATE(order_date, '%m/%d/%Y'))
)
SELECT 
    year,
    month,
    total_sales,
    LAG(total_sales, 1) OVER (PARTITION BY month ORDER BY year) AS prev_year_sales,
    ROUND(
        (total_sales - LAG(total_sales, 1) OVER (PARTITION BY month ORDER BY year)) 
        / NULLIF(LAG(total_sales, 1) OVER (PARTITION BY month ORDER BY year), 0) * 100, 
        2
    ) AS yoy_growth_pct
FROM monthly_sales
ORDER BY year, month;
```

11.Calculate Customer Lifetime Value: total sales, number of orders, and time span from first to last order for each customer.
```sql
SELECT 
    customer_id,
    SUM(sales) AS total_sales,
    COUNT(DISTINCT order_id) AS num_orders,
    MIN(STR_TO_DATE(order_date, '%m/%d/%Y')) AS first_order_date,
    MAX(STR_TO_DATE(order_date, '%m/%d/%Y')) AS last_order_date,
    DATEDIFF(    
		MAX(STR_TO_DATE(order_date, '%m/%d/%Y')), 
        MIN(STR_TO_DATE(order_date, '%m/%d/%Y'))
    ) AS time_span_days
FROM super_store
GROUP BY customer_id
ORDER BY total_sales DESC;
```

12.Find products that have sales higher than the average sales of their respective category, showing the percentage above category average.
```sql
WITH category_avg AS (
    SELECT 
        category,
        AVG(sales) AS avg_cat_sales
    FROM super_store
    GROUP BY category
)
SELECT 
    s.product_id,
    s.product_name,
    s.category,
    SUM(s.sales) AS product_sales,
    c.avg_cat_sales,
    ROUND(
        ((SUM(s.sales) - c.avg_cat_sales) / c.avg_cat_sales) * 100, 2
    ) AS pct_above_avg
FROM super_store s
JOIN category_avg c 
    ON s.category = c.category
GROUP BY s.product_id, s.product_name, s.category, c.avg_cat_sales
HAVING SUM(s.sales) > c.avg_cat_sales
ORDER BY pct_above_avg DESC;
```

13.Identify and analyze loss-making orders (negative profit): count by category and identify patterns and potential root causes.
```sql
SELECT 
    category,
    COUNT(order_id) AS loss_orders,
    SUM(profit) AS total_loss,
    ROUND(AVG(discount), 2) AS avg_discount
FROM super_store
WHERE profit < 0
GROUP BY category
ORDER BY total_loss ASC;
```

14.Find pairs of sub_categories that are frequently purchased together in the same order_id (simple co-occurrence analysis).
```sql
SELECT 
    a.sub_category AS subcat_1,
    b.sub_category AS subcat_2,
    COUNT(DISTINCT a.order_id) AS order_count
FROM super_store a
JOIN super_store b 
    ON a.order_id = b.order_id 
   AND a.sub_category < b.sub_category   
GROUP BY a.sub_category, b.sub_category
ORDER BY order_count DESC
LIMIT 10;
```

15.Create a comprehensive monthly report using CTEs: include sales, profit, top category, top customer segment, and growth rate compared to previous month.
```sql
WITH monthly AS (
    SELECT 
        DATE_FORMAT(STR_TO_DATE(order_date, '%m/%d/%Y'), '%Y-%m') AS month,
        SUM(sales) AS total_sales,
        SUM(profit) AS total_profit
    FROM super_store
    GROUP BY DATE_FORMAT(STR_TO_DATE(order_date, '%m/%d/%Y'), '%Y-%m')
),
top_category AS (
    SELECT 
        DATE_FORMAT(STR_TO_DATE(order_date, '%m/%d/%Y'), '%Y-%m') AS month,
        category,
        SUM(sales) AS cat_sales,
        RANK() OVER (PARTITION BY DATE_FORMAT(STR_TO_DATE(order_date, '%m/%d/%Y'), '%Y-%m') 
                     ORDER BY SUM(sales) DESC) AS rnk
    FROM super_store
    GROUP BY month, category
),
top_segment AS (
    SELECT 
        DATE_FORMAT(STR_TO_DATE(order_date, '%m/%d/%Y'), '%Y-%m') AS month,
        segment,
        SUM(sales) AS seg_sales,
        RANK() OVER (PARTITION BY DATE_FORMAT(STR_TO_DATE(order_date, '%m/%d/%Y'), '%Y-%m') 
                     ORDER BY SUM(sales) DESC) AS rnk
    FROM super_store
    GROUP BY month, segment
)
SELECT 
    m.month,
    m.total_sales,
    m.total_profit,
    tc.category AS top_category,
    ts.segment AS top_segment,
    ROUND(
        (m.total_sales - LAG(m.total_sales) OVER (ORDER BY m.month)) 
        / NULLIF(LAG(m.total_sales) OVER (ORDER BY m.month), 0) * 100, 
        2
    ) AS growth_rate_pct
FROM monthly m
LEFT JOIN top_category tc ON m.month = tc.month AND tc.rnk = 1
LEFT JOIN top_segment ts ON m.month = ts.month AND ts.rnk = 1
ORDER BY m.month;
```

4.Findings
- Customer Behavior: Segment contributions and top-spending customers were identified.
- Product Performance: Top products and categories were highlighted.
- Sales Trends: Seasonal fluctuations and monthly growth rates were analyzed.
- Profitability Insights: Loss-making orders and discount impact were examined.
- Operational Insights: Shipping efficiency patterns were uncovered.
5. Reports & Insights
- Overall Performance: Revenue, customer segmentation, and category-level insights.
- Trend Monitoring: Monthly and cumulative growth tracking.
- Customer Reports: Lifetime value and top customers.
- Product Reports: Top-selling products and best-performing categories.
6. Conclusion
This project demonstrates a complete SQL workflow for retail sales analysis—from database setup and cleaning to advanced business insights. The results provide actionable guidance for business decisions and show the power of SQL in data-driven analytics.
7. How to Use
- Clone the Repository: Download the project from GitHub.
- Set Up Database: Run the SQL scripts to create and populate the super_store table.
- Run Analyses: Execute the provided queries to generate insights.
- Explore & Extend: Modify or create queries to answer additional business questions.

Author: Van Huu Hien
This project is part of my SQL portfolio and reflects my learning journey as a Business Analytics student, showcasing skills in data cleaning, analysis, and reporting.









