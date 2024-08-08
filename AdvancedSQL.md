# Advanced SQL Techniques
---
**Introduction:**
This file covers some SQL techniques I use specifically in different inventory management systems. These examples will help you optimize queries, improve database performance, and access cool analytical capabilities.

**Contents:**
1. Window Functions
2. Common Table Expressions (CTEs)
3. Recursive Queries
4. Indexing Strategies
5. Partitioning Data
6. JSON Handling in SQL
7. Temporal Tables
8. Advanced Joins

---

## 1. Window Functions

Perform calculations across a set of rows related to the current row, useful for analysis.

```sql
-- Calculate the average sales over the last 7 days for a product
SELECT 
    product_id,
    sale_date,
    sales_amount,
    AVG(sales_amount) OVER (PARTITION BY product_id ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS avg_sales
FROM 
    sales;
```

## 2. Common Table Expressions (CTEs)

Improve the readability of complex queries.

```sql
-- Calculate total stock and total sales for each product
WITH Stock AS (
    SELECT 
        product_id, 
        SUM(quantity) AS total_stock
    FROM 
        inventory
    GROUP BY 
        product_id
),
Sales AS (
    SELECT 
        product_id, 
        SUM(quantity) AS total_sales
    FROM 
        sales
    GROUP BY 
        product_id
)
SELECT 
    s.product_id,
    s.total_stock,
    sa.total_sales
FROM 
    Stock s
JOIN 
    Sales sa ON s.product_id = sa.product_id;
```

## 3. Recursive Queries

Hierarchical data querying, useful for modeling supply chains.

```sql
-- List all components and sub-components for a given product
WITH RECURSIVE ProductComponents AS (
    SELECT 
        product_id,
        component_id,
        1 AS level
    FROM 
        product_components
    WHERE 
        product_id = 'P001'
    UNION ALL
    SELECT 
        pc.product_id,
        pc.component_id,
        pc.level + 1
    FROM 
        product_components pc
    JOIN 
        ProductComponents p ON pc.product_id = p.component_id
)
SELECT 
    product_id,
    component_id,
    level
FROM 
    ProductComponents;
```

## 4. Indexing Strategies

improves query performance, useful for frequently queried cols

```sql
CREATE INDEX idx_product_name ON products (name);
CREATE INDEX idx_sales_date ON sales (sale_date);
```

## 5. Partitioning table

Partitioning tables improves performance and manageability, useful for large-scale tables/Dbs.

```sql
-- Partition the sales table by year
CREATE TABLE sales (
    sale_id INT,
    sale_date DATE,
    product_id INT,
    quantity INT
) PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2019 VALUES LESS THAN (2020),
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022)
);
```

## 6. JSON Handling in SQL

Handling JSON data 🤷‍♂️

```sql
-- Extract details from JSON
SELECT 
    product_data->>'name' AS name,
    product_data->'dimensions'->>'width' AS width,
    product_data->'dimensions'->>'height' AS height
FROM 
    products
WHERE 
    product_data->>'category' = 'Electronics';
```

## 7. Temporal Tables

Track historical changes, useful for audits and versioning.

```sql
-- track inventory changes
CREATE TABLE inventory_history (
    product_id INT,
    quantity INT,
    period FOR SYSTEM_TIME
) WITH SYSTEM VERSIONING;
```

## 8. Joins

combining and integrating information.

```sql
-- combine inventory and sales data
SELECT 
    i.product_id, 
    i.quantity AS inventory_quantity,
    s.quantity AS sales_quantity
FROM 
    inventory i
FULL OUTER JOIN 
    sales s ON i.product_id = s.product_id;
```

**Final thoughts:**

This cheat sheet is your go-to guide for essential SQL tricks. I created it as a handy reference for myself, but feel free to bookmark, star, and share it with your crew. Let's help each other level up!

---
> **Warning:** This document is subject to periodic updates!
