# SQL Basic Filtering and Conditions for Data Analysts

Filtering and conditions in SQL are fundamental for selecting specific data from tables using the `WHERE` clause. As a data analyst, mastering logical operators (`AND`, `OR`, `NOT`), handling `NULL` values (`IS NULL`, `IS NOT NULL`), and pattern matching with `LIKE` is critical for querying, reporting, and cleaning data. This guide provides an in-depth exploration of these techniques, with practical examples, use cases, tips, and exercises tailored for real-world data analysis scenarios.

---

## 1. Overview of Filtering and Conditions

### Concept
Filtering in SQL uses the `WHERE` clause to select rows that meet specific conditions. Conditions are built with logical operators, NULL checks, and pattern matching to refine datasets for analysis. These tools allow you to isolate relevant data, exclude unwanted records, and handle special cases like missing values.

### Basic Syntax
```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

### Why It’s Used in Data Analysis
- **Data selection**: Extract specific subsets (e.g., high-value customers or recent orders).
- **Data cleaning**: Identify and address missing or invalid data (e.g., NULL emails).
- **Reporting**: Create targeted reports (e.g., sales in specific regions).
- **Exploratory analysis**: Filter data to uncover patterns or anomalies.

---

## 2. Logical Operators

Logical operators (`AND`, `OR`, `NOT`) combine conditions to create complex filtering logic.

### 2.1 `AND`
**Concept**: Returns rows where all conditions are true.

**Code Sample**:
```sql
SELECT first_name, total_purchases
FROM customers
WHERE total_purchases > 1000 AND region = 'North';
```

**Explanation**:
- `total_purchases > 1000 AND region = 'North'`: Filters customers with purchases over $1000 who are in the North region.

**Result** (example):
```
first_name | total_purchases
-----------|----------------
Alice      | 1500.00
```

**When to Use**:
- Narrow down data with multiple criteria (e.g., high spenders in a specific area).
- Ensure all conditions are met for precise filtering.

**Practical Tips**:
- Use parentheses to clarify complex logic:
  ```sql
  WHERE (total_purchases > 1000 AND region = 'North') OR region = 'West';
  ```
- Keep conditions simple to avoid performance issues.

**Pitfalls**:
- **Over-filtering**: Too many `AND` conditions may exclude valid data.
- **Operator precedence**: `AND` has higher precedence than `OR`; use parentheses to avoid errors.

---

### 2.2 `OR`
**Concept**: Returns rows where at least one condition is true.

**Code Sample**:
```sql
SELECT first_name, region
FROM customers
WHERE region = 'North' OR region = 'West';
```

**Explanation**:
- `region = 'North' OR region = 'West'`: Filters customers in either the North or West region.

**Result** (example):
```
first_name | region
-----------|-------
Alice      | North
Carol      | West
```

**When to Use**:
- Include multiple possibilities (e.g., customers from different regions).
- Broaden the scope of a query.

**Practical Tips**:
- Combine with `IN` for cleaner syntax:
  ```sql
  WHERE region IN ('North', 'West');
  ```
- Avoid combining too many `OR` conditions, as it can slow queries.

**Pitfalls**:
- **Ambiguity**: Without parentheses, `OR` can lead to unintended results:
  ```sql
  -- Ambiguous without parentheses
  WHERE total_purchases > 1000 AND region = 'North' OR region = 'West';
  ```

---

### 2.3 `NOT`
**Concept**: Excludes rows where the condition is true.

**Code Sample**:
```sql
SELECT first_name, region
FROM customers
WHERE NOT region = 'South';
```

**Explanation**:
- `NOT region = 'South'`: Filters customers not in the South region.

**Result** (example):
```
first_name | region
-----------|-------
Alice      | North
Carol      | West
```

**When to Use**:
- Exclude specific data (e.g., remove low-value orders).
- Combine with other operators for complex logic.

**Practical Tips**:
- Use `NOT IN` for multiple exclusions:
  ```sql
  WHERE region NOT IN ('South', 'East');
  ```
- Rewrite `NOT` conditions for clarity when possible (e.g., `region <> 'South'`).

**Pitfalls**:
- **Performance**: `NOT` can be slower than positive conditions; use sparingly on large datasets.
- **Double negatives**: Avoid confusing logic like `NOT NOT condition`.

---

## 3. Handling NULL Values

### Concept
`NULL` represents missing or unknown data. Use `IS NULL` or `IS NOT NULL` to filter rows with or without NULL values, as `NULL` doesn’t work with standard operators like `=` or `<>`.

### 3.1 `IS NULL`
**Code Sample**:
```sql
SELECT customer_id, first_name, email
FROM customers
WHERE email IS NULL;
```

**Explanation**:
- `email IS NULL`: Filters customers with missing email addresses.

**Result** (example):
```
customer_id | first_name | email
------------|------------|------
103         | Carol      | NULL
```

**When to Use**:
- Identify missing data for cleaning (e.g., incomplete customer records).
- Audit data quality before analysis.

**Practical Tips**:
- Combine with other conditions:
  ```sql
  WHERE email IS NULL AND region = 'North';
  ```
- Use `COALESCE` to handle NULLs in results:
  ```sql
  SELECT first_name, COALESCE(email, 'No Email') AS email
  FROM customers;
  ```

**Pitfalls**:
- **Using `= NULL`**: `WHERE email = NULL` doesn’t work; always use `IS NULL`.
- **Ignoring NULLs**: Unhandled NULLs can skew analysis (e.g., in averages).

---

### 3.2 `IS NOT NULL`
**Code Sample**:
```sql
SELECT customer_id, first_name, email
FROM customers
WHERE email IS NOT NULL;
```

**Explanation**:
- `email IS NOT NULL`: Filters customers with valid email addresses.

**Result** (example):
```
customer_id | first_name | email
------------|------------|------------------
101         | Alice      | alice@gmail.com
102         | Bob        | bob@yahoo.com
```

**When to Use**:
- Ensure data completeness for analysis (e.g., customers with contact info).
- Prepare data for marketing or reporting.

**Practical Tips**:
- Use before aggregations to avoid NULL-related errors:
  ```sql
  SELECT AVG(total_purchases)
  FROM customers
  WHERE total_purchases IS NOT NULL;
  ```
- Check for non-NULL values in joins to avoid missing matches.

**Pitfalls**:
- **Overlooking NULLs**: Failing to check for NULLs can lead to incomplete results.
- **Assuming non-NULL**: Verify data assumptions, as NULLs may exist unexpectedly.

---

## 4. Pattern Matching with LIKE

### Concept
`LIKE` performs pattern matching on text data using wildcards:
- `%`: Matches any sequence of characters.
- `_`: Matches a single character.

### 4.1 Using `%` Wildcard
**Code Sample**:
```sql
SELECT first_name, email
FROM customers
WHERE email LIKE '%@gmail.com';
```

**Explanation**:
- `email LIKE '%@gmail.com'`: Matches emails ending with `@gmail.com`.

**Result** (example):
```
first_name | email
-----------|------------------
Alice      | alice@gmail.com
```

**When to Use**:
- Segment data by patterns (e.g., email domains, product names).
- Search for partial matches in text fields.

**Practical Tips**:
- Use `ILIKE` (PostgreSQL) for case-insensitive matching:
  ```sql
  WHERE email ILIKE '%@gmail.com';
  ```
- Combine with `NOT LIKE` to exclude patterns:
  ```sql
  WHERE email NOT LIKE '%@yahoo.com';
  ```

**Pitfalls**:
- **Performance**: `LIKE` with leading wildcards (`%text%`) can be slow, as it prevents index usage.
- **Overly broad patterns**: `%a%` may return too many results; refine patterns.

---

### 4.2 Using `_` Wildcard
**Code Sample**:
```sql
SELECT product_name, product_code
FROM products
WHERE product_code LIKE 'A_1';
```

**Explanation**:
- `product_code LIKE 'A_1'`: Matches codes starting with 'A', followed by any single character, then '1' (e.g., 'AB1', 'AC1').

**Result** (example):
```
product_name | product_code
-------------|-------------
Laptop       | AB1
Mouse        | AC1
```

**When to Use**:
- Match fixed-length patterns (e.g., codes or IDs with specific formats).
- Validate data formats.

**Practical Tips**:
- Use multiple `_` for longer patterns:
  ```sql
  WHERE product_code LIKE 'A__1'; -- Matches AXX1
  ```
- Test patterns on sample data to ensure accuracy.

**Pitfalls**:
- **Misjudging length**: `_` requires exactly one character; use `%` for variable lengths.
- **Complex patterns**: `LIKE` is limited; consider regular expressions (`~` in PostgreSQL) for advanced matching.

---

## 5. Real-World Use Cases

1. **Reporting**:
   - **Scenario**: Generate a report of high-value customers in specific regions with valid emails.
   - **Query**:
     ```sql
     SELECT first_name, total_purchases, email
     FROM customers
     WHERE total_purchases > 1000 
       AND region IN ('North', 'West')
       AND email IS NOT NULL;
     ```
   - **Why**: Targets customers for a premium marketing campaign.

2. **Data Cleaning**:
   - **Scenario**: Identify records with missing or invalid contact information.
   - **Query**:
     ```sql
     SELECT customer_id, first_name, email, phone
     FROM customers
     WHERE email IS NULL OR phone IS NULL OR email NOT LIKE '%@%.%';
     ```
   - **Why**: Flags incomplete records for data quality improvement.
   - 
---

## 6. Advanced Tricks and Edge Cases

### 6.1 Combining Logical Operators
Use complex logic for precise filtering:
```sql
SELECT first_name, total_purchases, region
FROM customers
WHERE (total_purchases > 1000 OR region = 'West')
  AND email IS NOT NULL
  AND NOT region = 'South';
```
**Why**: Filters high-value or Western customers with valid emails, excluding South.

---

### 6.2 Case-Insensitive Pattern Matching
Handle case variations:
```sql
SELECT first_name, email
FROM customers
WHERE LOWER(email) LIKE '%@gmail.com';
```
**Why**: Matches emails regardless of case (e.g., `@Gmail.com`).

---

### 6.3 Escaping Wildcards
Match literal `%` or `_` in `LIKE`:
```sql
SELECT product_name
FROM products
WHERE product_name LIKE 'Sale\_%'; -- Matches "Sale_abc"
```
**Why**: Handles special characters in data.

---

## Conclusion

Mastering basic filtering and conditions in SQL empowers you to extract precise datasets for analysis, reporting, and data cleaning. Logical operators (`AND`, `OR`, `NOT`), NULL handling (`IS NULL`, `IS NOT NULL`), and pattern matching (`LIKE`) provide the flexibility to handle diverse real-world scenarios. Practice combining these techniques with joins, aggregations, and subqueries to tackle complex tasks. With these skills, you’ll be well-equipped to deliver actionable insights and maintain high-quality data as a data analyst.