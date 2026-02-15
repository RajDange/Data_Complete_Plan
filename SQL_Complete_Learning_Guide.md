# 🎯 SQL MASTERY - Complete Learning Guide

## 📋 Table of Contents
1. [SQL Fundamentals](#fundamentals)
2. [Intermediate SQL](#intermediate)
3. [Advanced SQL](#advanced)
4. [Performance & Optimization](#performance)
5. [Practice Resources](#practice)
6. [Real-World Projects](#projects)
7. [Interview Preparation](#interview)

---

## 🔰 LEVEL 1: FUNDAMENTALS (Weeks 1-2)

### 1.1 Basic SELECT Queries
**What to Learn:**
- SELECT statement syntax
- WHERE clause (filtering)
- ORDER BY (sorting)
- LIMIT/TOP (row limiting)
- DISTINCT (unique values)
- Column aliases (AS)

**Practice Exercises:**
```sql
-- Exercise 1: Basic retrieval
SELECT first_name, last_name, salary
FROM employees
WHERE salary > 50000
ORDER BY salary DESC
LIMIT 10;

-- Exercise 2: DISTINCT values
SELECT DISTINCT department
FROM employees
ORDER BY department;

-- Exercise 3: Multiple conditions
SELECT product_name, price, category
FROM products
WHERE category = 'Electronics' 
  AND price BETWEEN 100 AND 500
ORDER BY price;
```

**What You Should Be Able to Do:**
✓ Retrieve specific columns from a table
✓ Filter data using WHERE with AND/OR/NOT
✓ Sort results in ascending/descending order
✓ Get unique values
✓ Use comparison operators (=, !=, <, >, <=, >=)
✓ Use BETWEEN, IN, LIKE, IS NULL

---

### 1.2 Data Filtering & Operators
**What to Learn:**
- Comparison operators (=, <>, <, >, <=, >=)
- Logical operators (AND, OR, NOT)
- BETWEEN operator
- IN operator
- LIKE operator (pattern matching with % and _)
- IS NULL / IS NOT NULL
- CASE statements (basic)

**Practice Exercises:**
```sql
-- Exercise 1: LIKE pattern matching
SELECT customer_name, email
FROM customers
WHERE email LIKE '%@gmail.com';

-- Exercise 2: IN operator
SELECT * FROM orders
WHERE status IN ('Pending', 'Processing', 'Shipped');

-- Exercise 3: CASE statement
SELECT 
    product_name,
    price,
    CASE 
        WHEN price < 20 THEN 'Budget'
        WHEN price BETWEEN 20 AND 100 THEN 'Mid-Range'
        ELSE 'Premium'
    END AS price_category
FROM products;

-- Exercise 4: NULL handling
SELECT customer_name, phone
FROM customers
WHERE phone IS NOT NULL
  AND email IS NULL;
```

**What You Should Be Able to Do:**
✓ Filter using wildcards (%, _)
✓ Handle NULL values properly
✓ Use CASE for conditional logic
✓ Combine multiple conditions effectively

---

### 1.3 Aggregate Functions
**What to Learn:**
- COUNT(), SUM(), AVG(), MIN(), MAX()
- GROUP BY clause
- HAVING clause (filtering groups)
- COUNT(*) vs COUNT(column) vs COUNT(DISTINCT column)

**Practice Exercises:**
```sql
-- Exercise 1: Basic aggregations
SELECT 
    COUNT(*) AS total_employees,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    SUM(salary) AS total_payroll
FROM employees;

-- Exercise 2: GROUP BY
SELECT 
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_dept_salary,
    MAX(salary) AS highest_salary
FROM employees
GROUP BY department
ORDER BY avg_dept_salary DESC;

-- Exercise 3: HAVING clause
SELECT 
    category,
    COUNT(*) AS product_count,
    AVG(price) AS avg_price
FROM products
GROUP BY category
HAVING COUNT(*) > 10 AND AVG(price) > 50;

-- Exercise 4: Multiple grouping
SELECT 
    department,
    job_title,
    COUNT(*) AS count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department, job_title
ORDER BY department, avg_salary DESC;
```

**What You Should Be Able to Do:**
✓ Calculate summary statistics
✓ Group data and aggregate
✓ Filter grouped results with HAVING
✓ Understand the difference between WHERE and HAVING

---

### 1.4 Basic Joins
**What to Learn:**
- INNER JOIN (matching rows)
- LEFT JOIN (all from left + matches)
- RIGHT JOIN (all from right + matches)
- FULL OUTER JOIN (all rows)
- Self joins
- Join conditions (ON vs USING)

**Practice Exercises:**
```sql
-- Exercise 1: INNER JOIN
SELECT 
    e.first_name,
    e.last_name,
    d.department_name,
    e.salary
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- Exercise 2: LEFT JOIN (include employees without departments)
SELECT 
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- Exercise 3: Multiple joins
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name,
    oi.quantity,
    oi.price
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id;

-- Exercise 4: Self join (employees and their managers)
SELECT 
    e.first_name AS employee,
    m.first_name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

**What You Should Be Able to Do:**
✓ Join 2+ tables correctly
✓ Choose the right join type
✓ Understand NULL handling in joins
✓ Use table aliases effectively

---

## 🚀 LEVEL 2: INTERMEDIATE (Weeks 3-5)

### 2.1 Subqueries
**What to Learn:**
- Subqueries in SELECT
- Subqueries in FROM (derived tables)
- Subqueries in WHERE (IN, EXISTS)
- Correlated vs Non-correlated subqueries
- Scalar subqueries

**Practice Exercises:**
```sql
-- Exercise 1: Subquery in WHERE
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Exercise 2: Subquery with IN
SELECT product_name, price
FROM products
WHERE category_id IN (
    SELECT category_id 
    FROM categories 
    WHERE category_name IN ('Electronics', 'Computers')
);

-- Exercise 3: Correlated subquery
SELECT 
    e.first_name,
    e.last_name,
    e.salary,
    e.department_id
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);

-- Exercise 4: EXISTS
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.customer_id 
      AND o.order_date >= '2024-01-01'
);

-- Exercise 5: Derived table
SELECT 
    dept_summary.department_name,
    dept_summary.avg_salary,
    dept_summary.employee_count
FROM (
    SELECT 
        d.department_name,
        AVG(e.salary) AS avg_salary,
        COUNT(*) AS employee_count
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
    GROUP BY d.department_name
) AS dept_summary
WHERE dept_summary.avg_salary > 60000;
```

**What You Should Be Able to Do:**
✓ Write nested queries
✓ Use EXISTS vs IN appropriately
✓ Create derived tables
✓ Understand correlated subqueries

---

### 2.2 Window Functions (CRITICAL FOR DATA ROLES!)
**What to Learn:**
- ROW_NUMBER(), RANK(), DENSE_RANK()
- LAG(), LEAD()
- FIRST_VALUE(), LAST_VALUE()
- SUM(), AVG(), COUNT() as window functions
- PARTITION BY clause
- ORDER BY in window functions
- Frame specifications (ROWS BETWEEN)

**Practice Exercises:**
```sql
-- Exercise 1: ROW_NUMBER for ranking
SELECT 
    first_name,
    last_name,
    department_id,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank_in_dept
FROM employees;

-- Exercise 2: RANK vs DENSE_RANK
SELECT 
    product_name,
    category,
    price,
    RANK() OVER (PARTITION BY category ORDER BY price DESC) AS price_rank,
    DENSE_RANK() OVER (PARTITION BY category ORDER BY price DESC) AS dense_rank
FROM products;

-- Exercise 3: LAG and LEAD (compare with previous/next)
SELECT 
    order_date,
    total_sales,
    LAG(total_sales, 1) OVER (ORDER BY order_date) AS previous_day_sales,
    LEAD(total_sales, 1) OVER (ORDER BY order_date) AS next_day_sales,
    total_sales - LAG(total_sales, 1) OVER (ORDER BY order_date) AS daily_change
FROM daily_sales
ORDER BY order_date;

-- Exercise 4: Running totals
SELECT 
    order_date,
    daily_revenue,
    SUM(daily_revenue) OVER (ORDER BY order_date) AS running_total,
    AVG(daily_revenue) OVER (ORDER BY order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_7days
FROM daily_sales;

-- Exercise 5: FIRST_VALUE and LAST_VALUE
SELECT 
    employee_id,
    salary,
    department_id,
    FIRST_VALUE(salary) OVER (PARTITION BY department_id ORDER BY salary DESC) AS highest_salary_in_dept,
    LAST_VALUE(salary) OVER (PARTITION BY department_id ORDER BY salary DESC 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS lowest_salary_in_dept
FROM employees;
```

**What You Should Be Able to Do:**
✓ Calculate rankings without GROUP BY
✓ Compare rows with LAG/LEAD
✓ Calculate running totals and moving averages
✓ Understand PARTITION BY vs GROUP BY

---

### 2.3 Common Table Expressions (CTEs)
**What to Learn:**
- WITH clause syntax
- Single vs Multiple CTEs
- Recursive CTEs
- When to use CTE vs subquery

**Practice Exercises:**
```sql
-- Exercise 1: Basic CTE
WITH high_earners AS (
    SELECT *
    FROM employees
    WHERE salary > 80000
)
SELECT 
    department_id,
    COUNT(*) AS high_earner_count,
    AVG(salary) AS avg_high_earner_salary
FROM high_earners
GROUP BY department_id;

-- Exercise 2: Multiple CTEs
WITH 
    sales_2023 AS (
        SELECT customer_id, SUM(amount) AS total_2023
        FROM orders
        WHERE YEAR(order_date) = 2023
        GROUP BY customer_id
    ),
    sales_2024 AS (
        SELECT customer_id, SUM(amount) AS total_2024
        FROM orders
        WHERE YEAR(order_date) = 2024
        GROUP BY customer_id
    )
SELECT 
    c.customer_name,
    COALESCE(s23.total_2023, 0) AS sales_2023,
    COALESCE(s24.total_2024, 0) AS sales_2024,
    COALESCE(s24.total_2024, 0) - COALESCE(s23.total_2023, 0) AS yoy_growth
FROM customers c
LEFT JOIN sales_2023 s23 ON c.customer_id = s23.customer_id
LEFT JOIN sales_2024 s24 ON c.customer_id = s24.customer_id;

-- Exercise 3: Recursive CTE (organizational hierarchy)
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor member: top-level employees
    SELECT 
        employee_id,
        first_name,
        manager_id,
        1 AS level,
        CAST(first_name AS VARCHAR(1000)) AS hierarchy_path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive member
    SELECT 
        e.employee_id,
        e.first_name,
        e.manager_id,
        eh.level + 1,
        CAST(eh.hierarchy_path || ' -> ' || e.first_name AS VARCHAR(1000))
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy
ORDER BY level, first_name;
```

**What You Should Be Able to Do:**
✓ Write readable, modular queries with CTEs
✓ Chain multiple CTEs
✓ Use recursive CTEs for hierarchical data

---

### 2.4 String & Date Functions
**What to Learn:**

**String Functions:**
- CONCAT(), CONCAT_WS()
- SUBSTRING(), LEFT(), RIGHT()
- UPPER(), LOWER(), INITCAP()
- TRIM(), LTRIM(), RTRIM()
- REPLACE()
- LENGTH(), CHAR_LENGTH()
- POSITION(), CHARINDEX()

**Date Functions:**
- CURRENT_DATE, CURRENT_TIMESTAMP
- DATE_ADD(), DATE_SUB()
- DATEDIFF(), TIMESTAMPDIFF()
- EXTRACT(), DATE_PART()
- DATE_TRUNC()
- TO_DATE(), TO_TIMESTAMP()

**Practice Exercises:**
```sql
-- String manipulation
SELECT 
    CONCAT(first_name, ' ', last_name) AS full_name,
    UPPER(email) AS email_upper,
    SUBSTRING(phone, 1, 3) AS area_code,
    LENGTH(address) AS address_length,
    REPLACE(product_code, '-', '') AS clean_code
FROM customers;

-- Date operations
SELECT 
    order_date,
    EXTRACT(YEAR FROM order_date) AS order_year,
    EXTRACT(MONTH FROM order_date) AS order_month,
    DATE_TRUNC('month', order_date) AS month_start,
    DATEDIFF(CURRENT_DATE, order_date) AS days_since_order,
    order_date + INTERVAL '7 days' AS expected_delivery
FROM orders;

-- Complex date analysis
SELECT 
    customer_id,
    COUNT(*) AS order_count,
    MIN(order_date) AS first_order,
    MAX(order_date) AS last_order,
    DATEDIFF(MAX(order_date), MIN(order_date)) AS customer_lifetime_days,
    DATEDIFF(CURRENT_DATE, MAX(order_date)) AS days_since_last_order
FROM orders
GROUP BY customer_id;
```

**What You Should Be Able to Do:**
✓ Manipulate strings (concatenate, extract, clean)
✓ Work with dates (add/subtract, extract parts, calculate differences)
✓ Format dates and strings

---

### 2.5 Data Modification (DML)
**What to Learn:**
- INSERT (single row, multiple rows, from SELECT)
- UPDATE (with WHERE, with JOIN)
- DELETE (with WHERE)
- MERGE/UPSERT
- TRUNCATE vs DELETE
- Transaction safety

**Practice Exercises:**
```sql
-- INSERT examples
INSERT INTO customers (first_name, last_name, email, phone)
VALUES ('John', 'Doe', 'john@example.com', '555-1234');

INSERT INTO customers (first_name, last_name, email)
VALUES 
    ('Jane', 'Smith', 'jane@example.com'),
    ('Bob', 'Jones', 'bob@example.com');

INSERT INTO archived_orders
SELECT * FROM orders
WHERE order_date < '2020-01-01';

-- UPDATE examples
UPDATE employees
SET salary = salary * 1.10
WHERE department_id = 5;

UPDATE employees e
SET salary = salary * 1.05
FROM departments d
WHERE e.department_id = d.department_id
  AND d.department_name = 'Sales';

-- DELETE examples
DELETE FROM orders
WHERE order_date < '2019-01-01'
  AND status = 'Cancelled';

-- MERGE (UPSERT) example
MERGE INTO customer_summary cs
USING (
    SELECT 
        customer_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent
    FROM orders
    GROUP BY customer_id
) o
ON cs.customer_id = o.customer_id
WHEN MATCHED THEN
    UPDATE SET 
        order_count = o.order_count,
        total_spent = o.total_spent,
        last_updated = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN
    INSERT (customer_id, order_count, total_spent, last_updated)
    VALUES (o.customer_id, o.order_count, o.total_spent, CURRENT_TIMESTAMP);
```

**What You Should Be Able to Do:**
✓ Insert data safely
✓ Update records with conditions
✓ Delete data carefully
✓ Understand UPSERT logic

---

## 💪 LEVEL 3: ADVANCED (Weeks 6-8)

### 3.1 Advanced Joins & Set Operations
**What to Learn:**
- CROSS JOIN
- UNION, UNION ALL
- INTERSECT, EXCEPT/MINUS
- Complex multi-table joins
- Join optimization

**Practice Exercises:**
```sql
-- CROSS JOIN (Cartesian product)
SELECT 
    s.size,
    c.color,
    s.size || '-' || c.color AS variant
FROM sizes s
CROSS JOIN colors c;

-- UNION (combine results)
SELECT customer_id, 'VIP' AS segment
FROM vip_customers
UNION
SELECT customer_id, 'Regular' AS segment
FROM regular_customers;

-- INTERSECT (common records)
SELECT customer_id
FROM customers_2023
INTERSECT
SELECT customer_id
FROM customers_2024;

-- EXCEPT (difference)
SELECT customer_id
FROM customers_2023
EXCEPT
SELECT customer_id
FROM customers_2024;  -- Customers from 2023 who didn't order in 2024
```

**What You Should Be Able to Do:**
✓ Use set operations correctly
✓ Combine multiple queries
✓ Find overlaps and differences

---

### 3.2 Advanced Window Functions
**What to Learn:**
- NTILE() for percentiles
- CUME_DIST(), PERCENT_RANK()
- Frame clauses (ROWS vs RANGE)
- Complex partitioning

**Practice Exercises:**
```sql
-- Exercise 1: Percentiles with NTILE
SELECT 
    customer_id,
    total_purchases,
    NTILE(4) OVER (ORDER BY total_purchases) AS quartile,
    NTILE(10) OVER (ORDER BY total_purchases) AS decile
FROM customer_totals;

-- Exercise 2: Cumulative distribution
SELECT 
    salary,
    CUME_DIST() OVER (ORDER BY salary) AS cumulative_dist,
    PERCENT_RANK() OVER (ORDER BY salary) AS percent_rank,
    CASE 
        WHEN PERCENT_RANK() OVER (ORDER BY salary) >= 0.9 THEN 'Top 10%'
        WHEN PERCENT_RANK() OVER (ORDER BY salary) >= 0.75 THEN 'Top 25%'
        ELSE 'Below 75%'
    END AS salary_tier
FROM employees;

-- Exercise 3: Complex frames
SELECT 
    date,
    revenue,
    -- 7-day moving average (current + 6 preceding)
    AVG(revenue) OVER (
        ORDER BY date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day,
    -- Month-to-date total
    SUM(revenue) OVER (
        PARTITION BY DATE_TRUNC('month', date)
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS mtd_total
FROM daily_sales;
```

**What You Should Be Able to Do:**
✓ Calculate percentiles and distributions
✓ Use complex window frames
✓ Solve sliding window problems

---

### 3.3 Pivot & Unpivot
**What to Learn:**
- PIVOT for rows to columns
- UNPIVOT for columns to rows
- Dynamic pivot
- CASE-based pivoting

**Practice Exercises:**
```sql
-- Manual PIVOT with CASE
SELECT 
    product_category,
    SUM(CASE WHEN EXTRACT(MONTH FROM sale_date) = 1 THEN sales_amount ELSE 0 END) AS Jan,
    SUM(CASE WHEN EXTRACT(MONTH FROM sale_date) = 2 THEN sales_amount ELSE 0 END) AS Feb,
    SUM(CASE WHEN EXTRACT(MONTH FROM sale_date) = 3 THEN sales_amount ELSE 0 END) AS Mar
FROM sales
GROUP BY product_category;

-- PIVOT syntax (SQL Server, Oracle)
SELECT *
FROM (
    SELECT product_category, month, sales_amount
    FROM monthly_sales
) AS source
PIVOT (
    SUM(sales_amount)
    FOR month IN ([Jan], [Feb], [Mar], [Apr])
) AS pivoted;

-- UNPIVOT
SELECT product_id, month, sales
FROM product_sales
UNPIVOT (
    sales FOR month IN (Jan, Feb, Mar, Apr)
) AS unpivoted;
```

**What You Should Be Able to Do:**
✓ Transform rows to columns and vice versa
✓ Create cross-tabulation reports
✓ Handle dynamic pivoting

---

### 3.4 Query Optimization Fundamentals
**What to Learn:**
- Reading execution plans
- Index usage (B-tree, hash, bitmap)
- Query hints
- WHERE vs HAVING optimization
- JOIN order matters
- EXISTS vs IN vs JOIN

**Concepts to Master:**
```sql
-- Bad: Function on indexed column
SELECT * FROM orders
WHERE YEAR(order_date) = 2024;  -- Can't use index!

-- Good: SARGable query
SELECT * FROM orders
WHERE order_date >= '2024-01-01' 
  AND order_date < '2025-01-01';  -- Can use index

-- Avoid SELECT *
-- Bad
SELECT * FROM large_table;

-- Good
SELECT order_id, customer_id, order_date, total
FROM large_table;

-- Use EXISTS instead of COUNT
-- Bad
SELECT customer_id FROM customers c
WHERE (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.customer_id) > 0;

-- Good
SELECT customer_id FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);
```

**What You Should Be Able to Do:**
✓ Write SARGable queries
✓ Avoid common performance pitfalls
✓ Read basic execution plans
✓ Know when to use indexes

---

### 3.5 Stored Procedures & Functions
**What to Learn:**
- Creating stored procedures
- Input/Output parameters
- User-defined functions (UDF)
- Scalar vs Table-valued functions
- Error handling (TRY-CATCH)
- Transactions

**Practice Exercises:**
```sql
-- Stored Procedure
CREATE PROCEDURE GetEmployeesByDepartment
    @DepartmentID INT,
    @MinSalary DECIMAL(10,2) = 0
AS
BEGIN
    SELECT 
        employee_id,
        first_name,
        last_name,
        salary
    FROM employees
    WHERE department_id = @DepartmentID
      AND salary >= @MinSalary
    ORDER BY salary DESC;
END;

-- Execute
EXEC GetEmployeesByDepartment @DepartmentID = 5, @MinSalary = 50000;

-- User-Defined Function (Scalar)
CREATE FUNCTION CalculateBonus(@Salary DECIMAL(10,2), @PerformanceRating INT)
RETURNS DECIMAL(10,2)
AS
BEGIN
    DECLARE @Bonus DECIMAL(10,2);
    
    SET @Bonus = CASE 
        WHEN @PerformanceRating = 5 THEN @Salary * 0.15
        WHEN @PerformanceRating = 4 THEN @Salary * 0.10
        WHEN @PerformanceRating = 3 THEN @Salary * 0.05
        ELSE 0
    END;
    
    RETURN @Bonus;
END;

-- Use function
SELECT 
    employee_id,
    salary,
    performance_rating,
    dbo.CalculateBonus(salary, performance_rating) AS bonus
FROM employees;

-- Table-Valued Function
CREATE FUNCTION GetTopCustomers(@TopN INT)
RETURNS TABLE
AS
RETURN (
    SELECT TOP (@TopN)
        customer_id,
        customer_name,
        total_purchases
    FROM customer_summary
    ORDER BY total_purchases DESC
);

-- Transaction example
BEGIN TRANSACTION;

BEGIN TRY
    UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1;
    UPDATE accounts SET balance = balance + 1000 WHERE account_id = 2;
    
    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    -- Log error
    SELECT ERROR_MESSAGE() AS ErrorMessage;
END CATCH;
```

**What You Should Be Able to Do:**
✓ Create and use stored procedures
✓ Write reusable functions
✓ Handle transactions safely
✓ Implement error handling

---

## 🎯 LEVEL 4: PERFORMANCE & OPTIMIZATION (Weeks 9-10)

### 4.1 Indexing Strategies
**What to Learn:**
- Clustered vs Non-clustered indexes
- Composite indexes
- Covering indexes
- Index selectivity
- When NOT to index
- Index maintenance

**Key Concepts:**
```sql
-- Create indexes
CREATE INDEX idx_customer_email ON customers(email);
CREATE INDEX idx_order_date ON orders(order_date);

-- Composite index (order matters!)
CREATE INDEX idx_customer_date ON orders(customer_id, order_date);

-- Covering index
CREATE INDEX idx_order_covering 
ON orders(customer_id, order_date)
INCLUDE (total_amount, status);

-- View index usage
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;

-- Drop unused indexes
DROP INDEX idx_old_index ON orders;
```

**What You Should Be Able to Do:**
✓ Design appropriate indexes
✓ Understand index overhead
✓ Use execution plans
✓ Identify missing indexes

---

### 4.2 Partitioning
**What to Learn:**
- Range partitioning
- List partitioning
- Hash partitioning
- Partition pruning
- Maintenance operations

**Concepts:**
```sql
-- Range partitioning (conceptual)
CREATE TABLE sales_2024 (
    sale_id INT,
    sale_date DATE,
    amount DECIMAL(10,2)
)
PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025)
);

-- Query with partition pruning
SELECT * FROM sales_2024
WHERE sale_date >= '2024-01-01' 
  AND sale_date < '2024-02-01';  -- Only scans p2024
```

---

### 4.3 Query Tuning Techniques
**Checklist:**
- [ ] Avoid SELECT *
- [ ] Use WHERE instead of HAVING when possible
- [ ] Write SARGable queries
- [ ] Use appropriate JOIN types
- [ ] Limit result sets (TOP/LIMIT)
- [ ] Use EXISTS instead of IN for large subqueries
- [ ] Avoid functions on indexed columns
- [ ] Use UNION ALL instead of UNION when possible
- [ ] Analyze execution plans
- [ ] Update statistics regularly

---

## 📚 PRACTICE RESOURCES

### Free Online Platforms:
1. **LeetCode SQL** - https://leetcode.com/problemset/database/
   - 200+ SQL problems
   - Easy to Hard difficulty
   - Great for interviews

2. **HackerRank SQL** - https://www.hackerrank.com/domains/sql
   - Structured learning path
   - Certificates

3. **SQLZoo** - https://sqlzoo.net/
   - Interactive tutorials
   - Great for beginners

4. **Mode Analytics SQL Tutorial** - https://mode.com/sql-tutorial/
   - Real-world datasets
   - Analytics focused

5. **W3Schools SQL** - https://www.w3schools.com/sql/
   - Quick reference
   - Try it yourself editor

6. **StrataScratch** - https://www.stratascratch.com/
   - Real interview questions
   - Company-specific problems

### Practice Databases:
1. **PostgreSQL Sample Databases:**
   - DVD Rental Database
   - Pagila (film rental)
   - Chinook (music store)

2. **MySQL Sample Databases:**
   - Sakila (DVD rental)
   - World Database
   - Employee Database

3. **Microsoft AdventureWorks:**
   - Comprehensive business database
   - Sales, HR, Production data

### Setup Your Own Practice Environment:

**Option 1: PostgreSQL (Recommended)**
```bash
# Install PostgreSQL
# Download from: https://www.postgresql.org/download/

# Load sample database
pg_restore -U postgres -d dvdrental dvdrental.tar
```

**Option 2: MySQL**
```bash
# Install MySQL
# Download from: https://dev.mysql.com/downloads/

# Load Sakila database
mysql -u root -p < sakila-schema.sql
mysql -u root -p < sakila-data.sql
```

**Option 3: SQLite (Lightweight)**
```bash
# No installation needed
# Download DB Browser for SQLite
# Create and practice immediately
```

---

## 🏗️ REAL-WORLD PROJECTS

### Project 1: E-Commerce Analytics
**Build a complete analytics solution:**
- Customer segmentation (RFM analysis)
- Product performance analysis
- Sales trends and forecasting
- Cohort analysis

```sql
-- RFM Analysis Example
WITH rfm_calc AS (
    SELECT 
        customer_id,
        DATEDIFF(CURRENT_DATE, MAX(order_date)) AS recency,
        COUNT(DISTINCT order_id) AS frequency,
        SUM(total_amount) AS monetary
    FROM orders
    WHERE order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR)
    GROUP BY customer_id
),
rfm_score AS (
    SELECT 
        customer_id,
        recency,
        frequency,
        monetary,
        NTILE(5) OVER (ORDER BY recency DESC) AS r_score,
        NTILE(5) OVER (ORDER BY frequency) AS f_score,
        NTILE(5) OVER (ORDER BY monetary) AS m_score
    FROM rfm_calc
)
SELECT 
    customer_id,
    r_score || f_score || m_score AS rfm_score,
    CASE 
        WHEN r_score >= 4 AND f_score >= 4 THEN 'Champions'
        WHEN r_score >= 3 AND f_score >= 3 THEN 'Loyal Customers'
        WHEN r_score >= 4 AND f_score <= 2 THEN 'Promising'
        WHEN r_score <= 2 AND f_score <= 2 THEN 'At Risk'
        ELSE 'Others'
    END AS customer_segment
FROM rfm_score;
```

### Project 2: HR Analytics Dashboard
**Queries to build:**
- Employee turnover analysis
- Department performance metrics
- Salary benchmarking
- Hiring trends

### Project 3: Financial Reporting
**Build reports for:**
- Monthly/Quarterly revenue
- Profit margins by product
- Customer lifetime value
- Revenue forecasting

---

## 🎤 INTERVIEW PREPARATION

### Top 20 Interview Questions:

1. **Find the 2nd highest salary**
```sql
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Or using window function
SELECT DISTINCT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk = 2;
```

2. **Find duplicate records**
```sql
SELECT email, COUNT(*) AS count
FROM customers
GROUP BY email
HAVING COUNT(*) > 1;
```

3. **Delete duplicate records (keep one)**
```sql
DELETE FROM customers
WHERE customer_id NOT IN (
    SELECT MIN(customer_id)
    FROM customers
    GROUP BY email
);

-- Or using window function
DELETE FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM (
        SELECT customer_id,
               ROW_NUMBER() OVER (PARTITION BY email ORDER BY customer_id) AS rn
        FROM customers
    ) t
    WHERE rn > 1
);
```

4. **Find employees earning more than their managers**
```sql
SELECT e.first_name AS employee, e.salary AS emp_salary,
       m.first_name AS manager, m.salary AS mgr_salary
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;
```

5. **Cumulative sum**
```sql
SELECT 
    order_date,
    daily_revenue,
    SUM(daily_revenue) OVER (ORDER BY order_date) AS cumulative_revenue
FROM daily_sales;
```

6. **Find customers with no orders**
```sql
SELECT c.customer_id, c.customer_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;

-- Or using NOT EXISTS
SELECT customer_id, customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

7. **Top N per group**
```sql
SELECT *
FROM (
    SELECT 
        product_name,
        category,
        price,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) AS rn
    FROM products
) ranked
WHERE rn <= 3;
```

8. **Self-join for date comparison**
```sql
SELECT 
    t1.date,
    t1.temperature,
    t2.temperature AS prev_day_temp
FROM weather t1
LEFT JOIN weather t2 ON t1.date = DATE_ADD(t2.date, INTERVAL 1 DAY)
WHERE t1.temperature > t2.temperature;
```

9. **Percentage calculations**
```sql
SELECT 
    department,
    COUNT(*) AS dept_count,
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER(), 2) AS percentage
FROM employees
GROUP BY department;
```

10. **Moving average**
```sql
SELECT 
    date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day
FROM daily_sales;
```

### SQL Interview Tips:
✓ Always clarify requirements before writing
✓ Think about edge cases (NULLs, duplicates, empty results)
✓ Optimize for readability first, then performance
✓ Explain your thought process out loud
✓ Test your queries mentally
✓ Know multiple approaches (JOIN vs subquery vs window function)

---

## 📅 10-WEEK LEARNING SCHEDULE

**Week 1: Fundamentals**
- Day 1-2: SELECT, WHERE, ORDER BY
- Day 3-4: Aggregations (COUNT, SUM, AVG)
- Day 5: GROUP BY, HAVING
- Day 6-7: Practice on SQLZoo + LeetCode Easy

**Week 2: Joins**
- Day 1-2: INNER JOIN, LEFT JOIN
- Day 3: RIGHT JOIN, FULL OUTER JOIN
- Day 4-5: Multiple joins, Self joins
- Day 6-7: Practice joins on HackerRank

**Week 3: Intermediate Concepts**
- Day 1-3: Subqueries (WHERE, FROM, SELECT)
- Day 4-5: CTEs (WITH clause)
- Day 6-7: Practice on LeetCode Medium

**Week 4: Window Functions**
- Day 1-2: ROW_NUMBER, RANK, DENSE_RANK
- Day 3-4: LAG, LEAD, FIRST_VALUE
- Day 5: Aggregate window functions
- Day 6-7: Window function practice

**Week 5: Data Manipulation**
- Day 1-2: INSERT, UPDATE, DELETE
- Day 3-4: MERGE/UPSERT
- Day 5: String functions
- Day 6-7: Date functions

**Week 6: Advanced Queries**
- Day 1-2: Complex subqueries
- Day 3-4: PIVOT/UNPIVOT
- Day 5: Set operations (UNION, INTERSECT)
- Day 6-7: Practice advanced problems

**Week 7: Performance**
- Day 1-3: Indexing strategies
- Day 4-5: Query optimization
- Day 6-7: Execution plan analysis

**Week 8: Stored Procedures**
- Day 1-3: Creating procedures
- Day 4-5: Functions (UDF)
- Day 6-7: Transactions, error handling

**Week 9: Real Projects**
- Day 1-3: E-commerce analytics project
- Day 4-5: HR analytics project
- Day 6-7: Financial reporting project

**Week 10: Interview Prep**
- Day 1-3: Practice top interview questions
- Day 4-5: Mock interviews
- Day 6-7: Review weak areas

---

## 🎯 MASTERY CHECKLIST

### Beginner ✓
- [ ] Write SELECT queries with WHERE, ORDER BY
- [ ] Use aggregate functions (COUNT, SUM, AVG)
- [ ] GROUP BY with HAVING
- [ ] Basic JOINs (INNER, LEFT)
- [ ] Filter with IN, LIKE, BETWEEN

### Intermediate ✓
- [ ] Write subqueries in WHERE and FROM
- [ ] Use CTEs for complex queries
- [ ] Master window functions (ROW_NUMBER, LAG, LEAD)
- [ ] Multiple table joins (3+ tables)
- [ ] String and date manipulation

### Advanced ✓
- [ ] Recursive CTEs
- [ ] Complex window functions with frames
- [ ] PIVOT/UNPIVOT operations
- [ ] Query optimization techniques
- [ ] Read execution plans
- [ ] Write stored procedures and functions

### Expert ✓
- [ ] Design indexing strategies
- [ ] Partition large tables
- [ ] Optimize complex queries
- [ ] Handle millions of rows efficiently
- [ ] Debug production SQL issues

---

## 🚀 DAILY PRACTICE ROUTINE

**30 minutes/day minimum:**
- 10 min: Solve 1-2 LeetCode/HackerRank problems
- 10 min: Read/understand a new concept
- 10 min: Write queries on sample database

**1 hour/day (recommended):**
- 20 min: LeetCode problems (2-3)
- 20 min: Work on a real project
- 20 min: Review solutions, learn new techniques

---

## 📖 RECOMMENDED READING

**Books:**
1. "SQL Queries for Mere Mortals" - Michael J. Hernandez
2. "SQL Performance Explained" - Markus Winand
3. "Learning SQL" - Alan Beaulieu

**Online Resources:**
- PostgreSQL Documentation (excellent learning resource)
- Mode Analytics SQL School
- SQL Server documentation

---

## ✅ YOU'VE MASTERED SQL WHEN:

✓ You can solve any LeetCode SQL problem (Easy/Medium)
✓ You can write complex queries without looking up syntax
✓ You understand when to use window functions vs GROUP BY
✓ You can optimize slow queries
✓ You can design database schemas
✓ You can explain SQL concepts to others
✓ You dream in SELECT statements 😄

---

**Remember:** 
- SQL is a skill you build through **practice**, not just reading
- Start with easy problems and gradually increase difficulty
- Don't rush - master each concept before moving on
- Real projects teach you more than tutorials
- Code every single day, even if just for 20 minutes

Good luck! 🚀
