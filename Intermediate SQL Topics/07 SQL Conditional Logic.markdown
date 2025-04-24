# SQL Conditional Logic for Data Analysts

Conditional logic in SQL, primarily through `CASE` statements, allows data analysts to apply dynamic transformations, categorize data, and pivot datasets within queries. Mastering `CASE` statements, handling multiple conditions, and using `CASE` for manual pivoting is essential for reporting, data cleaning, and advanced analysis. This guide provides an in-depth exploration of these techniques, with practical examples, use cases, tips, and exercises tailored for real-world data analysis scenarios.

---

## 1. Overview of Conditional Logic

### Concept
Conditional logic in SQL uses the `CASE` statement to evaluate conditions and return values based on those conditions, similar to `if-then-else` logic in programming. It can be used in `SELECT`, `WHERE`, `ORDER BY`, and aggregations, and is particularly useful for transforming data or creating custom categories.

### Basic Syntax
```sql
SELECT column1,
       CASE
         WHEN condition1 THEN result1
         WHEN condition2 THEN result2
         ELSE result_default
       END AS new_column
FROM table_name;
```

### Why It’s Used in Data Analysis
- **Data transformation**: Categorize or reformat data (e.g., group customers by purchase tiers).
- **Reporting**: Create custom metrics or labels for dashboards (e.g., order status flags).
- **Data cleaning**: Handle inconsistent or missing data (e.g., replace NULLs with defaults).
- **Pivoting**: Reshape data for analysis without dedicated pivot functions.
- **Conditional filtering**: Apply logic within queries for dynamic results.

### Real-Life Example
You’re a data analyst at an e-commerce company tasked with categorizing orders by value (e.g., low, medium, high), flagging late shipments, and pivoting sales data by region for a quarterly report.

---

## 2. Using CASE Statements

### 2.1 Basic CASE Statement
**Concept**: Evaluates conditions and returns a value for the first true condition.

**Code Sample**:
```sql
SELECT order_id, total_amount,
       CASE
         WHEN total_amount < 100 THEN 'Low'
         WHEN total_amount BETWEEN 100 AND 500 THEN 'Medium'
         ELSE 'High'
       END AS order_category
FROM orders;
```

**Explanation**:
- `CASE`: Evaluates `total_amount` for each row.
- `WHEN total_amount < 100`: Assigns 'Low' if true.
- `WHEN total_amount BETWEEN 100 AND 500`: Assigns 'Medium' if true.
- `ELSE`: Assigns 'High' for all other cases.
- `AS order_category`: Names the output column.

**Result** (example):
```
order_id | total_amount | order_category
---------|--------------|---------------
1        | 50.00        | Low
2        | 250.00       | Medium
3        | 600.00       | High
```

**When to Use**:
- Categorize data (e.g., customer segments, order priorities).
- Create flags for reporting (e.g., active vs. inactive users).
- Simplify complex logic into readable categories.

**Practical Tips**:
- Keep conditions mutually exclusive to avoid ambiguity.
- Use `ELSE` to handle unexpected cases:
  ```sql
  ELSE 'Unknown'
  ```
- Test conditions on sample data to ensure coverage.

**Pitfalls**:
- **Missing ELSE**: Without `ELSE`, unmatched rows return `NULL`.
- **Overlapping conditions**: Ensure conditions don’t overlap (e.g., `total_amount < 100` and `total_amount <= 100`).
- **Performance**: Complex `CASE` statements can slow queries; optimize conditions.

---

### 2.2 Searched vs. Simple CASE
**Concept**:
- **Searched CASE**: Evaluates arbitrary conditions (shown above).
- **Simple CASE**: Compares a single expression to multiple values.

**Code Sample (Simple CASE)**:
```sql
SELECT customer_id, region,
       CASE region
         WHEN 'North' THEN 'Region 1'
         WHEN 'South' THEN 'Region 2'
         ELSE 'Other'
       END AS region_group
FROM customers;
```

**Explanation**:
- `CASE region`: Compares `region` to each `WHEN` value.
- Assigns custom region groups.

**Result** (example):
```
customer_id | region | region_group
------------|--------|-------------
101         | North  | Region 1
102         | South  | Region 2
103         | West   | Other
```

**When to Use**:
- Simple `CASE` for single-column comparisons with fixed values.
- Searched `CASE` for complex or multi-column conditions.

**Practical Tips**:
- Use simple `CASE` for readability when comparing one column.
- Combine with `IN` for multiple values:
  ```sql
  CASE region
    WHEN 'North', 'South' THEN 'Primary'
    ELSE 'Secondary'
  END
  ```

**Pitfalls**:
- **Limited scope**: Simple `CASE` can’t handle complex logic; use searched `CASE` instead.
- **NULL handling**: `CASE column WHEN NULL` doesn’t work; use `IS NULL` in searched `CASE`.

---

## 3. Handling Multiple Conditions in a Single Query

### Concept
Multiple `CASE` statements or nested conditions can be used in a single query to apply complex logic across columns or calculations.

**Code Sample**:
```sql
SELECT order_id, total_amount, order_date, shipped_date,
       CASE
         WHEN shipped_date IS NULL THEN 'Pending'
         WHEN shipped_date <= order_date + INTERVAL '3 days' THEN 'On Time'
         ELSE 'Late'
       END AS shipping_status,
       CASE
         WHEN total_amount > 500 THEN 'High Value'
         ELSE 'Standard'
       END AS order_value
FROM orders;
```

**Explanation**:
- First `CASE`: Categorizes orders as 'Pending', 'On Time', or 'Late' based on `shipped_date`.
- Second `CASE`: Labels orders as 'High Value' or 'Standard' based on `total_amount`.
- Multiple `CASE` statements create two new columns.

**Result** (example):
```
order_id | total_amount | order_date  | shipped_date | shipping_status | order_value
---------|--------------|-------------|--------------|----------------|------------
1        | 600.00       | 2024-01-01  | 2024-01-02   | On Time        | High Value
2        | 200.00       | 2024-01-02  | NULL         | Pending        | Standard
3        | 300.00       | 2024-01-03  | 2024-01-07   | Late           | Standard
```

**When to Use**:
- Apply multiple transformations (e.g., status flags and value categories).
- Combine logic for comprehensive reporting.
- Clean or reformat data across several columns.

**Practical Tips**:
- Break complex queries into CTEs for readability:
  ```sql
  WITH order_status AS (
    SELECT order_id, total_amount,
           CASE WHEN total_amount > 500 THEN 'High Value' ELSE 'Standard' END AS order_value
    FROM orders
  )
  SELECT * FROM order_status;
  ```
- Use consistent `CASE` output types (e.g., all strings or all numbers) to avoid errors.

**Pitfalls**:
- **Readability**: Too many `CASE` statements can make queries hard to follow; use comments or CTEs.
- **NULL handling**: Ensure conditions account for `NULL` values explicitly.
- **Performance**: Multiple `CASE` statements can slow large queries; optimize conditions.

---

## 4. Pivoting Data with CASE (Manual Pivoting)

### Concept
Manual pivoting uses `CASE` with aggregations (e.g., `SUM`, `COUNT`) to transform rows into columns, creating a wide-format table. This is useful when databases lack a `PIVOT` operator (e.g., MySQL).

**Code Sample**:
```sql
SELECT EXTRACT(year FROM order_date) AS order_year,
       SUM(CASE WHEN region = 'North' THEN total_amount ELSE 0 END) AS north_sales,
       SUM(CASE WHEN region = 'South' THEN total_amount ELSE 0 END) AS south_sales,
       SUM(CASE WHEN region = 'West' THEN total_amount ELSE 0 END) AS west_sales
FROM orders
GROUP BY EXTRACT(year FROM order_date);
```

**Explanation**:
- `CASE WHEN region = 'North' THEN total_amount ELSE 0`: Assigns `total_amount` for North, 0 otherwise.
- `SUM(...)`: Aggregates amounts per region, creating separate columns.
- `GROUP BY EXTRACT(year FROM order_date)`: Groups results by year.

**Result** (example):
```
order_year | north_sales | south_sales | west_sales
-----------|-------------|-------------|------------
2023       | 5000.00     | 3000.00     | 2000.00
2024       | 6000.00     | 4000.00     | 2500.00
```

**When to Use**:
- Reshape data for reporting (e.g., sales by region or category).
- Create cross-tab reports or dashboards.
- Analyze metrics across categories without a `PIVOT` operator.

**Practical Tips**:
- Use `COUNT` for frequency pivots:
  ```sql
  SUM(CASE WHEN region = 'North' THEN 1 ELSE 0 END) AS north_orders
  ```
- Combine with `ORDER BY` for sorted results.
- Use CTEs to prepare data before pivoting:
  ```sql
  WITH clean_orders AS (
    SELECT EXTRACT(year FROM order_date) AS year, region, total_amount
    FROM orders
    WHERE total_amount IS NOT NULL
  )
  SELECT year,
         SUM(CASE WHEN region = 'North' THEN total_amount ELSE 0 END) AS north_sales
  FROM clean_orders
  GROUP BY year;
  ```

**Pitfalls**:
- **Scalability**: Manual pivoting is cumbersome for many categories; consider `PIVOT` if available.
- **NULL handling**: Ensure `ELSE 0` or equivalent to avoid NULLs in aggregations.
- **Hardcoding**: Fixed categories (e.g., 'North') require query updates if categories change.

---

## 5. Real-World Use Cases

1. **Reporting**:
   - **Scenario**: Create a report categorizing customers by purchase tier and activity status.
   - **Query**:
     ```sql
     SELECT first_name, total_purchases,
            CASE
              WHEN total_purchases > 1000 THEN 'Premium'
              WHEN total_purchases > 500 THEN 'Standard'
              ELSE 'Basic'
            END AS purchase_tier,
            CASE
              WHEN last_order_date >= CURRENT_DATE - INTERVAL '30 days' THEN 'Active'
              ELSE 'Inactive'
            END AS activity_status
     FROM customers;
     ```
   - **Why**: Segments customers for targeted marketing.

2. **Data Cleaning**:
   - **Scenario**: Standardize inconsistent region codes.
   - **Query**:
     ```sql
     SELECT customer_id, region,
            CASE
              WHEN UPPER(region) IN ('N', 'NORTH', 'NRTH') THEN 'North'
              WHEN UPPER(region) IN ('S', 'SOUTH', 'STH') THEN 'South'
              ELSE region
            END AS clean_region
     FROM customers;
     ```
   - **Why**: Ensures consistent region names for analysis.

3. **Pivoting with Joins**:
   - **Scenario**: Pivot order counts by product category and year.
   - **Query**:
     ```sql
     SELECT EXTRACT(year FROM o.order_date) AS order_year,
            SUM(CASE WHEN p.category = 'Electronics' THEN 1 ELSE 0 END) AS electronics_orders,
            SUM(CASE WHEN p.category = 'Clothing' THEN 1 ELSE 0 END) AS clothing_orders
     FROM orders o
     INNER JOIN products p ON o.product_id = p.product_id
     GROUP BY EXTRACT(year FROM o.order_date);
     ```
   - **Why**: Summarizes sales trends by category.

---

## 6. Advanced Tricks and Edge Cases

### 6.1 Nested CASE Statements
Handle complex logic with nested `CASE`:
```sql
SELECT order_id, total_amount, region,
       CASE
         WHEN region = 'North' THEN
           CASE
             WHEN total_amount > 1000 THEN 'High Priority'
             ELSE 'Standard'
           END
         ELSE 'Regular'
       END AS order_priority
FROM orders;
```
**Why**: Applies region-specific logic to prioritize orders.

---

### 6.2 Dynamic CASE in ORDER BY
Sort results conditionally:
```sql
SELECT first_name, total_purchases
FROM customers
ORDER BY CASE
           WHEN total_purchases > 1000 THEN 1
           WHEN total_purchases > 500 THEN 2
           ELSE 3
         END;
```
**Why**: Prioritizes high-value customers in sorting.

---

### 6.3 Conditional Aggregations
Use `CASE` within aggregates for filtered metrics:
```sql
SELECT region,
       SUM(CASE WHEN total_amount > 500 THEN total_amount ELSE 0 END) AS high_value_sales,
       COUNT(CASE WHEN total_amount > 500 THEN 1 END) AS high_value_orders
FROM orders
GROUP BY region;
```
**Why**: Calculates metrics only for high-value orders.

---

### 6.4 Handling NULLs in CASE
Explicitly manage NULLs:
```sql
SELECT customer_id, email,
       CASE
         WHEN email IS NULL THEN 'Missing'
         WHEN email ~ '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$' THEN 'Valid'
         ELSE 'Invalid'
       END AS email_status
FROM customers;
```
**Why**: Validates email addresses and flags missing data.

---

## 7. Practice Exercises

1. **Basic CASE**:
   - Write a query to categorize products in the `products` table as 'Cheap' (< $50), 'Moderate' ($50-$200), or 'Expensive' (>$200) based on `price`.

2. **Multiple Conditions**:
   - Query the `orders` table to create two columns: one flagging orders as 'Recent' (within 30 days) or 'Old', and another labeling `total_amount` as 'High' (>$500) or 'Low'.

3. **Manual Pivoting**:
   - Pivot the `orders` table to show total `total_amount` by `region` (North, South, West) for each year, using `CASE`.

4. **Complex Logic**:
   - Join the `customers` and `orders` tables to categorize orders as 'VIP' (customers with >$2000 total purchases and order >$500) or 'Regular', including a status for 'Shipped' (shipped within 3 days) or 'Delayed'.

---

## Conclusion

Conditional logic with `CASE` statements is a powerful tool for data analysts, enabling dynamic transformations, categorization, and manual pivoting. By mastering `CASE`, handling multiple conditions, and applying pivoting techniques, you can create insightful reports, clean data, and reshape datasets for analysis. Practice combining `CASE` with joins, aggregations, and other SQL features to tackle complex real-world scenarios. With these skills, you’ll be well-equipped to deliver actionable insights and handle diverse analytical challenges.