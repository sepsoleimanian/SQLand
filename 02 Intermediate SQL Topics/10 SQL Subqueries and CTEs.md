# SQL Subqueries and Common Table Expressions (CTEs) for Data Analysts

Subqueries and Common Table Expressions (CTEs) are powerful SQL tools that allow data analysts to break down complex queries, improve readability, and perform advanced data manipulations. As a data analyst, mastering these techniques will enable you to handle sophisticated reporting, data cleaning, and analysis tasks efficiently. This guide provides an in-depth exploration of subqueries and CTEs, with practical examples, use cases, tips, and exercises tailored for real-world data analysis.

---

## 1. Understanding Subqueries

### Concept
A subquery is a query nested inside another query, typically enclosed in parentheses. It runs first, and its result is used by the outer query. Subqueries can appear in `SELECT`, `FROM`, `WHERE`, or `HAVING` clauses.

### Syntax
```sql
SELECT column1
FROM table1
WHERE column2 = (SELECT column2 FROM table2 WHERE condition);
```

### Why It’s Used in Data Analysis
- **Filtering data**: Subqueries help filter rows based on dynamic conditions (e.g., customers with above-average purchases).
- **Calculating metrics**: Compute values like maximums or averages for comparisons.
- **Breaking down complexity**: Solve problems that require intermediate results without creating temporary tables.
- **Data exploration**: Compare datasets or identify outliers.

### Real-Life Example
You’re analyzing customer data and need to find customers whose total purchases exceed the average purchase amount across all customers.

### Code Sample
```sql
SELECT first_name, total_purchases
FROM customers
WHERE total_purchases > (SELECT AVG(total_purchases) FROM customers);
```

**Explanation**:
- `(SELECT AVG(total_purchases) FROM customers)`: Subquery calculates the average `total_purchases`.
- `WHERE total_purchases > ...`: Outer query filters customers with above-average purchases.

**Result** (example):
```
first_name | total_purchases
-----------|----------------
Alice      | 1500.00
Carol      | 2500.00
```

### Practical Tips
- Keep subqueries simple to maintain readability.
- Use meaningful aliases for clarity:
  ```sql
  SELECT first_name
  FROM customers
  WHERE total_purchases > (SELECT AVG(total_purchases) AS avg_purchases FROM customers);
  ```
- Test subqueries independently to ensure they return expected results.

### Common Pitfalls
- **Performance**: Subqueries can be slow on large datasets; consider CTEs or joins for optimization.
- **Single-row expectation**: Subqueries in `WHERE` (e.g., with `=`) must return one value, or they’ll error:
  ```sql
  -- This fails if subquery returns multiple rows
  WHERE customer_id = (SELECT customer_id FROM orders);
  ```
- **NULL handling**: Subqueries returning NULL can lead to unexpected results.

### Advanced Trick
Use `IN` or `EXISTS` for multi-row subqueries:
```sql
SELECT first_name
FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders WHERE total_amount > 1000);
```
**Why**: Filters customers with high-value orders.

---

## 2. Understanding Common Table Expressions (CTEs)

### Concept
A CTE is a temporary result set defined within a query using the `WITH` clause. It can be referenced multiple times in the main query, improving readability and maintainability compared to subqueries.

### Syntax
```sql
WITH cte_name AS (
  SELECT column1, column2
  FROM table_name
  WHERE condition
)
SELECT column1
FROM cte_name
WHERE condition;
```

### Why It’s Used in Data Analysis
- **Readability**: CTEs make complex queries easier to understand and maintain.
- **Reusability**: Reference the same intermediate result multiple times in a query.
- **Data transformation**: Simplify steps like aggregations, joins, or filtering.
- **Reporting**: Structure queries for dashboards or recurring reports.

### Real-Life Example
You need a report showing the top 10% of customers by purchase amount, including their order details.

### Code Sample
```sql
WITH high_value_customers AS (
  SELECT customer_id, first_name, total_purchases,
         PERCENT_RANK() OVER (ORDER BY total_purchases DESC) AS purchase_rank
  FROM customers
)
SELECT h.first_name, o.order_id, o.total_amount
FROM high_value_customers h
JOIN orders o ON h.customer_id = o.customer_id
WHERE h.purchase_rank <= 0.1;
```

**Explanation**:
- `WITH high_value_customers AS ...`: CTE calculates purchase ranks using `PERCENT_RANK`.
- `WHERE h.purchase_rank <= 0.1`: Filters top 10% of customers.
- `JOIN orders o`: Combines with order details.

**Result** (example):
```
first_name | order_id | total_amount
-----------|----------|-------------
Carol      | 3        | 1200.00
Alice      | 1        | 1000.00
```

### Practical Tips
- Name CTEs descriptively (e.g., `high_value_customers` instead of `temp`).
- Use CTEs for queries you’ll reuse or share with colleagues.
- Break complex queries into multiple CTEs for clarity:
  ```sql
  WITH step1 AS (...),
       step2 AS (SELECT ... FROM step1 WHERE ...)
  SELECT ... FROM step2;
  ```

### Common Pitfalls
- **Overusing CTEs**: Too many CTEs can make queries harder to debug; balance with subqueries or joins.
- **Performance**: CTEs may not be optimized as well as subqueries in some databases (e.g., PostgreSQL materializes CTEs).
- **Scope**: CTEs are only available within the query they’re defined in.

### Advanced Trick
Use recursive CTEs for hierarchical data (e.g., employee-manager relationships):
```sql
WITH RECURSIVE org_chart AS (
  SELECT employee_id, employee_name, manager_id, 0 AS level
  FROM employees
  WHERE manager_id IS NULL
  UNION ALL
  SELECT e.employee_id, e.employee_name, e.manager_id, o.level + 1
  FROM employees e
  JOIN org_chart o ON e.manager_id = o.employee_id
)
SELECT employee_name, level
FROM org_chart;
```
**Why**: Builds a hierarchy of employees and their management levels.

---

## 3. Subqueries vs. CTEs: When to Use Which
- **Subqueries**:
  - Best for simple, one-off calculations or filters.
  - Use in `WHERE` or `SELECT` for quick comparisons.
  - Example: Comparing a value to an aggregate.
- **CTEs**:
  - Ideal for complex, multi-step queries or reusable intermediate results.
  - Improve readability for team collaboration.
  - Example: Structuring a report with multiple transformations.

---

## 4. Real-World Use Cases

1. **Reporting**:
   - **Scenario**: Calculate the percentage of total sales contributed by each product category.
   - **Query (Subquery)**:
     ```sql
     SELECT category, 
            SUM(price) AS category_sales,
            (SUM(price) / (SELECT SUM(price) FROM products)) * 100 AS sales_percentage
     FROM products
     GROUP BY category;
     ```
   - **Query (CTE)**:
     ```sql
     WITH total_sales AS (
       SELECT SUM(price) AS total FROM products
     ),
     category_sales AS (
       SELECT category, SUM(price) AS category_total
       FROM products
       GROUP BY category
     )
     SELECT c.category, c.category_total, 
            (c.category_total / t.total) * 100 AS sales_percentage
     FROM category_sales c, total_sales t;
     ```
   - **Why**: Provides a clear breakdown of sales contributions.

2. **Data Cleaning**:
   - **Scenario**: Identify orders with unusually high amounts (above the 95th percentile).
   - **Query (Subquery)**:
     ```sql
     SELECT order_id, total_amount
     FROM orders
     WHERE total_amount > (SELECT PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY total_amount) FROM orders);
     ```
   - **Why**: Flags outliers for review.

3. **Joining Tables**:
   - **Scenario**: Find customers who placed orders in the last 30 days, with their total order count.
   - **Query (CTE)**:
     ```sql
     WITH recent_orders AS (
       SELECT customer_id, order_id
       FROM orders
       WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
     )
     SELECT c.first_name, COUNT(r.order_id) AS order_count
     FROM customers c
     JOIN recent_orders r ON c.customer_id = r.customer_id
     GROUP BY c.first_name;
     ```
   - **Why**: Combines datasets for targeted analysis.

---

## 5. Advanced Tricks and Edge Cases

### 5.1 Correlated Subqueries
A subquery that references the outer query’s columns, executed for each row:
```sql
SELECT first_name, total_purchases
FROM customers c
WHERE total_purchases > (
  SELECT AVG(total_purchases)
  FROM customers
  WHERE region = c.region
);
```
**Why**: Compares each customer’s purchases to their region’s average.
**Pitfall**: Can be slow; consider CTEs or joins for performance.

---

### 5.2 CTEs for Window Functions
CTEs are great for organizing window function calculations:
```sql
WITH ranked_orders AS (
  SELECT customer_id, order_id, total_amount,
         RANK() OVER (PARTITION BY customer_id ORDER BY total_amount DESC) AS order_rank
  FROM orders
)
SELECT customer_id, order_id, total_amount
FROM ranked_orders
WHERE order_rank = 1;
```
**Why**: Identifies each customer’s highest-value order.

---

### 5.3 Subqueries in `SELECT`
Compute values for each row:
```sql
SELECT first_name, total_purchases,
       (SELECT COUNT(*) FROM orders WHERE customer_id = c.customer_id) AS order_count
FROM customers c;
```
**Why**: Adds per-customer order counts without grouping.

---

## 6. Practice Exercises

1. **Basic Subquery**:
   - Write a query to find products priced above the average price in the `products` table.

2. **Subquery with IN**:
   - Query the `customers` table to find customers who placed orders with amounts greater than $500.

3. **Basic CTE**:
   - Use a CTE to calculate the total sales per region from the `orders` table, then select regions with sales above $10,000.

4. **Complex CTE with Joins**:
   - Create a CTE to identify the top 5 customers by total purchases, then join with the `orders` table to list their order details.

---

## Conclusion

Subqueries and CTEs are essential tools for data analysts, enabling you to handle complex queries, improve readability, and perform advanced analyses. Subqueries are great for quick, dynamic calculations, while CTEs excel in structuring reusable, multi-step queries. Practice combining these techniques with joins, aggregations, and window functions to tackle real-world challenges like reporting, data cleaning, and exploratory analysis. With these skills, you’ll be well-equipped to deliver actionable insights from complex datasets.