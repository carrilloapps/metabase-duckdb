# DuckDB Example Queries for Metabase

This file contains example queries and dashboards you can create in Metabase with DuckDB. These examples demonstrate the analytical capabilities of DuckDB integrated with Metabase's visualization features.

## Basic Dataset Creation

First, let's create some sample data in DuckDB. Run these SQL queries in Metabase's SQL editor after connecting to your DuckDB database.

### Sales Data

```sql
-- Create a sales table
CREATE TABLE sales (
  transaction_id INTEGER,
  sale_date DATE,
  customer_id INTEGER,
  product_id INTEGER,
  quantity INTEGER,
  unit_price DECIMAL(10,2),
  total_amount DECIMAL(10,2)
);

-- Insert sample data
INSERT INTO sales VALUES
  (1, '2025-01-01', 101, 1, 2, 19.99, 39.98),
  (2, '2025-01-02', 102, 2, 1, 29.99, 29.99),
  (3, '2025-01-02', 103, 1, 3, 19.99, 59.97),
  (4, '2025-01-03', 101, 3, 1, 49.99, 49.99),
  (5, '2025-01-04', 104, 2, 2, 29.99, 59.98),
  (6, '2025-01-05', 102, 1, 1, 19.99, 19.99),
  (7, '2025-01-05', 103, 3, 2, 49.99, 99.98),
  (8, '2025-01-06', 105, 1, 4, 19.99, 79.96),
  (9, '2025-01-07', 101, 2, 2, 29.99, 59.98),
  (10, '2025-01-08', 102, 3, 1, 49.99, 49.99),
  (11, '2025-02-01', 103, 1, 2, 19.99, 39.98),
  (12, '2025-02-02', 104, 2, 3, 29.99, 89.97),
  (13, '2025-02-03', 105, 3, 1, 49.99, 49.99),
  (14, '2025-02-05', 101, 1, 1, 19.99, 19.99),
  (15, '2025-02-06', 102, 2, 2, 29.99, 59.98),
  (16, '2025-02-08', 103, 3, 1, 49.99, 49.99),
  (17, '2025-02-10', 104, 1, 3, 19.99, 59.97),
  (18, '2025-02-12', 105, 2, 1, 29.99, 29.99),
  (19, '2025-02-15', 101, 3, 2, 49.99, 99.98),
  (20, '2025-02-20', 102, 1, 4, 19.99, 79.96);

-- Create a products table
CREATE TABLE products (
  product_id INTEGER PRIMARY KEY,
  product_name VARCHAR(100),
  category VARCHAR(50),
  supplier VARCHAR(100)
);

-- Insert product data
INSERT INTO products VALUES
  (1, 'Widget A', 'Widgets', 'Supplier X'),
  (2, 'Gadget B', 'Gadgets', 'Supplier Y'),
  (3, 'Tool C', 'Tools', 'Supplier Z');

-- Create a customers table
CREATE TABLE customers (
  customer_id INTEGER PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100),
  city VARCHAR(50),
  state VARCHAR(50)
);

-- Insert customer data
INSERT INTO customers VALUES
  (101, 'Alice Smith', 'alice@example.com', 'New York', 'NY'),
  (102, 'Bob Johnson', 'bob@example.com', 'Los Angeles', 'CA'),
  (103, 'Carol Williams', 'carol@example.com', 'Chicago', 'IL'),
  (104, 'David Brown', 'david@example.com', 'Houston', 'TX'),
  (105, 'Eve Jones', 'eve@example.com', 'Phoenix', 'AZ');
```

## Example Queries

### Sales Analysis

#### 1. Daily Sales Trend

```sql
SELECT 
  sale_date,
  SUM(total_amount) AS daily_sales
FROM sales
GROUP BY sale_date
ORDER BY sale_date;
```

Visualization: Line chart with sale_date on the x-axis and daily_sales on the y-axis.

#### 2. Product Sales Distribution

```sql
SELECT 
  p.product_name,
  SUM(s.quantity) AS units_sold,
  SUM(s.total_amount) AS revenue
FROM sales s
JOIN products p ON s.product_id = p.product_id
GROUP BY p.product_name
ORDER BY revenue DESC;
```

Visualization: Bar chart or pie chart showing revenue by product.

#### 3. Sales by Customer Location

```sql
SELECT 
  c.state,
  COUNT(s.transaction_id) AS num_transactions,
  SUM(s.total_amount) AS total_revenue
FROM sales s
JOIN customers c ON s.customer_id = c.customer_id
GROUP BY c.state
ORDER BY total_revenue DESC;
```

Visualization: Map visualization showing revenue by state.

### Advanced Analytics

#### 1. Month-over-Month Growth

```sql
WITH monthly_sales AS (
  SELECT 
    EXTRACT(YEAR FROM sale_date) AS year,
    EXTRACT(MONTH FROM sale_date) AS month,
    SUM(total_amount) AS monthly_revenue
  FROM sales
  GROUP BY year, month
  ORDER BY year, month
)
SELECT 
  year,
  month,
  monthly_revenue,
  LAG(monthly_revenue) OVER (ORDER BY year, month) AS prev_month_revenue,
  (monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY year, month)) / 
    LAG(monthly_revenue) OVER (ORDER BY year, month) * 100 AS growth_rate
FROM monthly_sales;
```

Visualization: Line chart showing monthly growth rate.

#### 2. Customer Purchase Frequency

```sql
SELECT 
  c.name,
  COUNT(s.transaction_id) AS purchase_count,
  SUM(s.total_amount) AS total_spent,
  AVG(s.total_amount) AS avg_purchase_value
FROM sales s
JOIN customers c ON s.customer_id = c.customer_id
GROUP BY c.name
ORDER BY purchase_count DESC;
```

Visualization: Scatter plot with purchase_count on x-axis and avg_purchase_value on y-axis.

#### 3. Product Category Analysis

```sql
SELECT 
  p.category,
  COUNT(s.transaction_id) AS sales_count,
  SUM(s.quantity) AS units_sold,
  SUM(s.total_amount) AS revenue,
  AVG(s.unit_price) AS avg_price
FROM sales s
JOIN products p ON s.product_id = p.product_id
GROUP BY p.category
ORDER BY revenue DESC;
```

Visualization: Combo chart with bars for revenue and line for units_sold.

## Creating a Dashboard

With these queries, you can build a comprehensive sales dashboard in Metabase:

1. Create saved questions for each query above
2. Create a new dashboard
3. Add the saved questions as cards on the dashboard
4. Arrange them logically (e.g., overview metrics at the top, detailed breakdowns below)
5. Add filters to make the dashboard interactive (e.g., date range filter)

## Advanced DuckDB Features in Metabase

### Window Functions

```sql
SELECT 
  sale_date,
  customer_id,
  total_amount,
  SUM(total_amount) OVER (PARTITION BY customer_id ORDER BY sale_date) AS running_total,
  RANK() OVER (PARTITION BY customer_id ORDER BY total_amount DESC) AS purchase_rank
FROM sales;
```

### Approximate Quantiles

```sql
SELECT 
  APPROX_QUANTILE(total_amount, 0.25) AS q1,
  APPROX_QUANTILE(total_amount, 0.5) AS median,
  APPROX_QUANTILE(total_amount, 0.75) AS q3,
  APPROX_QUANTILE(total_amount, 0.95) AS p95
FROM sales;
```

### Time Series Functions

```sql
SELECT 
  DATE_TRUNC('week', sale_date) AS week_start,
  SUM(total_amount) AS weekly_sales
FROM sales
GROUP BY week_start
ORDER BY week_start;
```

## Performance Tips

For larger datasets, you can improve performance with:

```sql
-- Create an index on frequently queried columns
CREATE INDEX sale_date_idx ON sales (sale_date);

-- Use materialized views for complex queries
CREATE MATERIALIZED VIEW sales_by_product AS
SELECT 
  p.product_name,
  SUM(s.quantity) AS units_sold,
  SUM(s.total_amount) AS revenue
FROM sales s
JOIN products p ON s.product_id = p.product_id
GROUP BY p.product_name;
```

---

These examples demonstrate just a small sample of what's possible with DuckDB and Metabase. DuckDB's analytical capabilities combined with Metabase's visualization tools provide a powerful platform for data analysis and business intelligence.
