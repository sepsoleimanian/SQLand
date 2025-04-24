# SQL Window Functions for Data Analysts

Window functions in SQL are powerful tools that allow data analysts to perform calculations across a set of rows (a "window") related to the current row, without collapsing the result set like aggregations do. As a data analyst, mastering window functions will enable you to compute running totals, rankings, and moving averages, making them invaluable for reporting, trend analysis, and data cleaning. This guide provides an in-depth exploration of window functions, with practical examples, use cases, tips, and exercises tailored for real-world data analysis.

---

## 1. Understanding Window Functions

### Concept
A window function performs a calculation across a defined set of rows (the "window") for each row in the result set, without grouping the rows into a single output like `GROUP BY`. The window is defined using the `OVER` clause, which specifies how rows are partitioned and ordered.

### Syntax
```sql
SELECT column1, 
       WINDOW_FUNCTION(column2) OVER (
         [PARTITION BY column3]
         [ORDER BY column4]
         [ROWS BETWEEN start AND end]
       ) AS result_column
FROM table_name;
```

- **WINDOW_FUNCTION**: Functions like `ROW_NUMBER`, `RANK`, `SUM`, `AVG`, `LAG`, etc.
- **PARTITION BY**: Divides the data into groups (like `GROUP BY`, but keeps all rows).
- **ORDER BY**: Defines the order of rows within the window.
- **ROWS BETWEEN**: Specifies the range of rows in the window (e.g., for running totals).

### Why It’s Used in Data Analysis
- **Ranking and ordering**: Assign ranks or row numbers to data (e.g., top customers by sales).
- **Trend analysis**: Calculate running totals, moving averages, or differences over time.
- **Data comparison**: Access previous or next rows (e.g., compare sales to the prior day).
- **Data cleaning**: Identify duplicates or gaps in sequences.
- **Reporting**: Enhance reports with per-group metrics without losing row-level detail.

### Real-Life Example
You’re a data analyst at an e-commerce company tasked with ranking customers by their total purchases and calculating each customer’s contribution to their region’s total sales.

---

## 2. Core Window Functions with Examples

### 2.1 Ranking Functions
**Purpose**: Assign positions or ranks to rows.

- **`ROW_NUMBER()`**: Assigns a unique number to each row.
- **`RANK()`**: Assigns ranks, with ties getting the same rank and gaps in subsequent ranks.
- **`DENSE_RANK()`**: Like `RANK`, but no gaps in ranks after ties.

**Code Sample**:
```sql
SELECT first_name, total_purchases,
       ROW_NUMBER() OVER (ORDER BY total_purchases DESC) AS purchase_rank,
       RANK() OVER (ORDER BY total_purchases DESC) AS rank_with_gaps,
       DENSE_RANK() OVER (ORDER BY total_purchases DESC) AS dense_rank
FROM customers;
```

**Explanation**:
- `OVER (ORDER BY total_purchases DESC)`: Defines the window, ordering by `total_purchases` descending.
- `ROW_NUMBER()`: Assigns unique numbers (1, 2, 3, ...).
- `RANK()`: Assigns ranks, with ties sharing ranks (e.g., 1, 1, 3).
- `DENSE_RANK()`: Assigns ranks without gaps (e.g., 1, 1, 2).

**Result** (example):
```
first_name | total_purchases | purchase_rank | rank_with_gaps | dense_rank
-----------|----------------|---------------|----------------|-----------
Carol      | 2500.00        | 1             | 1              | 1
Alice      | 2500.00        | 2             | 1              | 1
Bob        | 1500.00        | 3             | 3              | 2
```

**Use Case**: Rank customers by purchase amount for a loyalty program.
**Tip**: Use `PARTITION BY` to rank within groups:
```sql
SELECT first_name, region, total_purchases,
       ROW_NUMBER() OVER (PARTITION BY region ORDER BY total_purchases DESC) AS regional_rank
FROM customers;
```
**Pitfall**: Ensure `ORDER BY` is specified, or results may be unpredictable.

---

### 2.2 Aggregate Window Functions
**Purpose**: Apply aggregate functions (`SUM`, `AVG`, `COUNT`, etc.) over a window, keeping all rows.

**Code Sample**:
```sql
SELECT order_date, total_amount,
       SUM(total_amount) OVER (PARTITION BY DATE_TRUNC('month', order_date)) AS monthly_total
FROM orders;
```

**Explanation**:
- `PARTITION BY DATE_TRUNC('month', order_date)`: Groups orders by month.
- `SUM(total_amount)`: Calculates the total sales for each month’s window.

**Result** (example):
```
order_date  | total_amount | monthly_total
------------|--------------|--------------
2024-01-01  | 100.50       | 350.75
2024-01-02  | 250.25       | 350.75
2024-02-01  | 75.00        | 75.00
```

**Use Case**: Calculate each order’s contribution to monthly sales.
**Tip**: Add `ORDER BY` for running totals:
```sql
SELECT order_date, total_amount,
       SUM(total_amount) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM orders;
```
**Pitfall**: Without `PARTITION BY`, the function applies to all rows, which may not be intended.

---

### 2.3 Value Functions
**Purpose**: Access data from other rows in the window (e.g., previous or next row).

- **`LAG()`**: Returns the value from the previous row.
- **`LEAD()`**: Returns the value from the next row.
- **`FIRST_VALUE()`**: Returns the first value in the window.

**Code Sample**:
```sql
SELECT order_date, total_amount,
       LAG(total_amount) OVER (ORDER BY order_date) AS previous_amount,
       LEAD(total_amount) OVER (ORDER BY order_date) AS next_amount
FROM orders;
```

**Explanation**:
- `LAG(total_amount)`: Retrieves the `total_amount` from the prior row.
- `LEAD(total_amount)`: Retrieves the `total_amount` from the next row.
- `OVER (ORDER BY order_date)`: Orders the window by `order_date`.

**Result** (example):
```
order_date  | total_amount | previous_amount | next_amount
------------|--------------|-----------------|------------
2024-01-01  | 100.50       | NULL            | 250.25
2024-01-02  | 250.25       | 100.50          | 75.00
2024-01-03  | 75.00        | 250.25          | NULL
```

**Use Case**: Compare daily sales to the previous day’s sales.
**Tip**: Use `PARTITION BY` for group-specific comparisons:
```sql
SELECT customer_id, order_date, total_amount,
       LAG(total_amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_customer_order
FROM orders;
```
**Pitfall**: NULLs appear for the first/last rows unless handled (e.g., with `COALESCE`).

---

## 3. Defining the Window

### Key Components
- **PARTITION BY**: Divides the data into groups (e.g., by region or customer).
- **ORDER BY**: Specifies the order within the window (e.g., by date or amount).
- **Frame Specification** (`ROWS BETWEEN`):
  - `UNBOUNDED PRECEDING`: From the start of the window.
  - `CURRENT ROW`: Up to the current row.
  - `UNBOUNDED FOLLOWING`: To the end of the window.
  - Example: `ROWS BETWEEN 1 PRECEDING AND CURRENT ROW` for a moving average over the current and previous row.

**Code Sample** (Running Total):
```sql
SELECT order_date, total_amount,
       SUM(total_amount) OVER (
         ORDER BY order_date
         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS cumulative_sales
FROM orders;
```

**Result** (example):
```
order_date  | total_amount | cumulative_sales
------------|--------------|-----------------
2024-01-01  | 100.50       | 100.50
2024-01-02  | 250.25       | 350.75
2024-01-03  | 75.00        | 425.75
```

---

## 4. Practical Tips
- **Reuse windows**: Define a window once and reuse it:
  ```sql
  SELECT first_name, total_purchases,
         ROW_NUMBER() OVER w AS row_num,
         RANK() OVER w AS rank
  FROM customers
  WINDOW w AS (ORDER BY total_purchases DESC);
  ```
- **Test incrementally**: Run the window function alone to verify results before adding to a larger query.
- **Optimize performance**: Use indexes on `PARTITION BY` and `ORDER BY` columns.
- **Combine with CTEs**: Use CTEs to organize complex window function queries:
  ```sql
  WITH ranked AS (
    SELECT customer_id, total_purchases,
           DENSE_RANK() OVER (ORDER BY total_purchases DESC) AS rank
    FROM customers
  )
  SELECT * FROM ranked WHERE rank <= 5;
  ```

---

## 5. Common Pitfalls
- **Missing ORDER BY**: Without `ORDER BY`, results may be non-deterministic for ranking or value functions.
- **Over-partitioning**: Too many partitions can slow queries; ensure partitions are necessary.
- **Frame specification errors**: Omitting `ROWS BETWEEN` for aggregates defaults to the entire window, which may not be intended.
- **NULL handling**: `LAG` or `LEAD` may return NULLs; use `COALESCE` to handle them:
  ```sql
  COALESCE(LAG(total_amount) OVER (ORDER BY order_date), 0) AS previous_amount
  ```

---

## 6. Real-World Use Cases

1. **Reporting**:
   - **Scenario**: Create a report showing each customer’s rank within their region by total purchases.
   - **Query**:
     ```sql
     SELECT first_name, region, total_purchases,
            DENSE_RANK() OVER (PARTITION BY region ORDER BY total_purchases DESC) AS regional_rank
     FROM customers;
     ```
   - **Why**: Identifies top spenders per region for targeted marketing.

2. **Data Cleaning**:
   - **Scenario**: Identify duplicate orders by assigning row numbers.
   - **Query**:
     ```sql
     WITH numbered_orders AS (
       SELECT order_id, customer_id, order_date,
              ROW_NUMBER() OVER (PARTITION BY customer_id, order_date ORDER BY order_id) AS rn
       FROM orders
     )
     SELECT * FROM numbered_orders WHERE rn > 1;
     ```
   - **Why**: Flags duplicate orders for removal.

3. **Trend Analysis**:
   - **Scenario**: Calculate a 3-day moving average of daily sales.
   - **Query**:
     ```sql
     SELECT order_date, total_amount,
            AVG(total_amount) OVER (
              ORDER BY order_date
              ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
            ) AS moving_avg
     FROM orders;
     ```
   - **Why**: Smooths daily sales data to identify trends.

---

## 7. Advanced Tricks and Edge Cases

### 7.1 Conditional Window Functions
Use `CASE` within window functions for conditional calculations:
```sql
SELECT order_date, total_amount,
       SUM(CASE WHEN total_amount > 100 THEN total_amount ELSE 0 END) OVER (
         ORDER BY order_date
         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS high_value_running_total
FROM orders;
```
**Why**: Tracks cumulative sales for high-value orders only.

---

### 7.2 Percentiles with `NTILE`
Divide rows into buckets (e.g., quartiles):
```sql
SELECT first_name, total_purchases,
       NTILE(4) OVER (ORDER BY total_purchases) AS purchase_quartile
FROM customers;
```
**Why**: Segments customers into four groups by purchase amount.

---

### 7.3 Comparing to Group Totals
Calculate each row’s contribution to a group total:
```sql
SELECT c.first_name, o.total_amount,
       o.total_amount / SUM(o.total_amount) OVER (PARTITION BY c.customer_id) AS contribution_ratio
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```
**Why**: Shows each order’s share of a customer’s total spending.

---

## 8. Practice Exercises

1. **Ranking**:
   - Write a query to rank products in the `products` table by price within each category using `DENSE_RANK`.

2. **Running Total**:
   - Calculate the cumulative sales for each order in the `orders` table, ordered by `order_date`.

3. **Value Comparison**:
   - Query the `orders` table to show each order’s `total_amount` and the difference from the previous order’s amount using `LAG`.

4. **Moving Average**:
   - Calculate a 7-day moving average of daily order amounts in the `orders` table.

---

## Conclusion

Window functions are a game-changer for data analysts, offering flexibility to compute rankings, running totals, and comparisons without losing row-level detail. By mastering functions like `ROW_NUMBER`, `SUM`, `LAG`, and window specifications like `PARTITION BY` and `ROWS BETWEEN`, you can tackle complex reporting, trend analysis, and data cleaning tasks. Practice combining window functions with joins, CTEs, and aggregations to handle real-world scenarios. With these skills, you’ll be well-equipped to deliver insightful, data-driven solutions.