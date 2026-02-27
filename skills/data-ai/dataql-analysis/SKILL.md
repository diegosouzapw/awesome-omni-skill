---
name: dataql-analysis
description: Analyze data files using SQL queries with DataQL. Use when working with CSV, JSON, Parquet, Excel files or when the user mentions data analysis, filtering, aggregation, or SQL queries on files.
tools:
  - Bash
---

# DataQL Data Analysis

You have access to DataQL, a powerful CLI tool for querying data files using SQL.

## Capabilities

- Query CSV, JSON, JSONL, XML, YAML, Parquet, Excel, Avro, ORC files
- Filter, aggregate, join data from multiple sources
- Export results to CSV or JSONL
- Connect to databases (PostgreSQL, MySQL, MongoDB)
- Query data from S3, GCS, Azure Blob Storage, and HTTP URLs

## Usage Patterns

### Single File Query
```bash
dataql run -f <file> -q "<SQL query>"
```

### Multiple Files (JOIN)
```bash
dataql run -f users.csv -f orders.json -q "SELECT u.name, o.total FROM users u JOIN orders o ON u.id = o.user_id"
```

### Export Results
```bash
dataql run -f data.csv -q "SELECT * FROM data WHERE amount > 100" -e output.jsonl -t jsonl
```

### Get Schema
```bash
dataql run -f data.csv -q ".schema data"
```

### Query from URL
```bash
dataql run -f "https://example.com/data.json" -q "SELECT * FROM data LIMIT 10"
```

### Query from S3
```bash
dataql run -f "s3://bucket/path/data.csv" -q "SELECT * FROM data"
```

### Query from Database
```bash
dataql run -f "postgres://user:pass@host/db?table=users" -q "SELECT * FROM users WHERE active = true"
```

## Best Practices for Token Efficiency

1. **Always use LIMIT**: Start with `LIMIT 10` to preview data before running full queries
2. **Select specific columns**: Avoid `SELECT *` when possible - specify only needed columns
3. **Use aggregations**: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()` instead of returning all rows
4. **Filter early**: Use WHERE clauses to reduce result size
5. **Check schema first**: Run `.schema` before complex queries to understand structure

## Common Workflow

### Step 1: Understand the data structure
```bash
dataql run -f data.csv -q ".schema data"
```

### Step 2: Preview a few rows
```bash
dataql run -f data.csv -q "SELECT * FROM data LIMIT 5"
```

### Step 3: Get summary statistics
```bash
dataql run -f data.csv -q "SELECT COUNT(*) as total, AVG(amount) as avg_amount FROM data"
```

### Step 4: Run targeted queries
```bash
dataql run -f data.csv -q "SELECT category, SUM(amount) as total FROM data GROUP BY category ORDER BY total DESC LIMIT 10"
```

## Examples

### Analyze Sales Data
```bash
# Check schema
dataql run -f sales.csv -q ".schema sales"

# Get summary
dataql run -f sales.csv -q "SELECT COUNT(*) as transactions, SUM(amount) as revenue, AVG(amount) as avg_sale FROM sales"

# Top products
dataql run -f sales.csv -q "SELECT product, SUM(amount) as total FROM sales GROUP BY product ORDER BY total DESC LIMIT 10"

# Sales by date
dataql run -f sales.csv -q "SELECT date, SUM(amount) as daily_total FROM sales GROUP BY date ORDER BY date"
```

### Join Multiple Sources
```bash
dataql run -f customers.csv -f orders.json -q "
SELECT c.name, c.email, COUNT(o.id) as order_count, SUM(o.total) as total_spent
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name, c.email
ORDER BY total_spent DESC
LIMIT 20"
```

### Filter and Export
```bash
# Find and export active users
dataql run -f users.json -q "SELECT id, name, email FROM users WHERE status = 'active'" -e active_users.csv -t csv

# Export as JSONL for further processing
dataql run -f data.parquet -q "SELECT * FROM data WHERE region = 'US'" -e us_data.jsonl -t jsonl
```

## Advanced Query Examples

### Window Functions

```bash
# Moving average (7-day)
dataql run -f sales.csv -q "
SELECT
  date,
  amount,
  AVG(amount) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as moving_avg_7d
FROM sales
ORDER BY date"

# Running total
dataql run -f transactions.csv -q "
SELECT
  date,
  amount,
  SUM(amount) OVER (ORDER BY date) as running_total
FROM transactions"

# Rank within groups
dataql run -f employees.csv -q "
SELECT
  department,
  name,
  salary,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
FROM employees"

# Percent of total
dataql run -f sales.csv -q "
SELECT
  product,
  amount,
  ROUND(100.0 * amount / SUM(amount) OVER (), 2) as pct_of_total
FROM sales"
```

### Common Table Expressions (CTEs)

```bash
# Daily totals with filtering
dataql run -f transactions.csv -q "
WITH daily_totals AS (
  SELECT date, SUM(amount) as total
  FROM transactions
  GROUP BY date
)
SELECT * FROM daily_totals WHERE total > 1000 ORDER BY date"

# Recursive CTE for hierarchical data
dataql run -f categories.csv -q "
WITH RECURSIVE category_tree AS (
  SELECT id, name, parent_id, 1 as level
  FROM categories WHERE parent_id IS NULL
  UNION ALL
  SELECT c.id, c.name, c.parent_id, ct.level + 1
  FROM categories c
  JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree ORDER BY level, name"

# Multiple CTEs
dataql run -f orders.csv -q "
WITH
  monthly_sales AS (
    SELECT strftime('%Y-%m', date) as month, SUM(amount) as total
    FROM orders GROUP BY month
  ),
  monthly_avg AS (
    SELECT AVG(total) as avg_monthly FROM monthly_sales
  )
SELECT m.month, m.total,
  CASE WHEN m.total > a.avg_monthly THEN 'Above Avg' ELSE 'Below Avg' END as performance
FROM monthly_sales m, monthly_avg a
ORDER BY m.month"
```

### Pivoting Data

```bash
# Monthly sales by category
dataql run -f sales.csv -q "
SELECT
  category,
  SUM(CASE WHEN strftime('%m', date) = '01' THEN amount ELSE 0 END) as jan,
  SUM(CASE WHEN strftime('%m', date) = '02' THEN amount ELSE 0 END) as feb,
  SUM(CASE WHEN strftime('%m', date) = '03' THEN amount ELSE 0 END) as mar,
  SUM(CASE WHEN strftime('%m', date) = '04' THEN amount ELSE 0 END) as apr
FROM sales
GROUP BY category"

# Status counts by department
dataql run -f employees.csv -q "
SELECT
  department,
  COUNT(CASE WHEN status = 'active' THEN 1 END) as active,
  COUNT(CASE WHEN status = 'inactive' THEN 1 END) as inactive,
  COUNT(CASE WHEN status = 'pending' THEN 1 END) as pending
FROM employees
GROUP BY department"
```

### Time Series Analysis

```bash
# Year-over-year comparison
dataql run -f sales.csv -q "
SELECT
  strftime('%m', date) as month,
  SUM(CASE WHEN strftime('%Y', date) = '2024' THEN amount ELSE 0 END) as y2024,
  SUM(CASE WHEN strftime('%Y', date) = '2023' THEN amount ELSE 0 END) as y2023,
  ROUND(100.0 * (SUM(CASE WHEN strftime('%Y', date) = '2024' THEN amount ELSE 0 END) /
    NULLIF(SUM(CASE WHEN strftime('%Y', date) = '2023' THEN amount ELSE 0 END), 0) - 1), 2) as yoy_growth
FROM sales
GROUP BY month
ORDER BY month"

# Gap detection in time series
dataql run -f events.csv -q "
SELECT
  date,
  LAG(date) OVER (ORDER BY date) as prev_date,
  date - LAG(date) OVER (ORDER BY date) as days_gap
FROM events
HAVING days_gap > 1
ORDER BY date"
```

## Common Use Cases

### Analyzing Log Files

```bash
# Parse and analyze access logs
dataql run -f access.jsonl -q "
SELECT
  strftime('%Y-%m-%d %H', timestamp) as hour,
  COUNT(*) as requests,
  COUNT(CASE WHEN status >= 400 THEN 1 END) as errors
FROM access
GROUP BY hour
ORDER BY hour"

# Find top error paths
dataql run -f access.jsonl -q "
SELECT path, status, COUNT(*) as count
FROM access
WHERE status >= 400
GROUP BY path, status
ORDER BY count DESC
LIMIT 20"
```

### Processing CSV Exports from Spreadsheets

```bash
# Clean and transform data
dataql run -f export.csv -q "
SELECT
  TRIM(name) as name,
  LOWER(email) as email,
  COALESCE(phone, 'N/A') as phone,
  CAST(REPLACE(REPLACE(amount, '$', ''), ',', '') AS DECIMAL) as amount
FROM export
WHERE email IS NOT NULL AND email != ''"

# Deduplicate records
dataql run -f contacts.csv -q "
SELECT DISTINCT ON (email) *
FROM contacts
ORDER BY email, updated_at DESC"
```

### Joining Multiple Data Sources

```bash
# Enrich customer data with orders and payments
dataql run -f customers.csv -f orders.json -f payments.parquet -q "
SELECT
  c.id,
  c.name,
  c.email,
  COUNT(DISTINCT o.id) as total_orders,
  COALESCE(SUM(o.amount), 0) as order_total,
  COALESCE(SUM(p.amount), 0) as paid_total
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
LEFT JOIN payments p ON o.id = p.order_id
GROUP BY c.id, c.name, c.email"
```

### Data Transformation and Cleaning

```bash
# Normalize inconsistent data
dataql run -f products.csv -q "
SELECT
  UPPER(TRIM(sku)) as sku,
  INITCAP(TRIM(name)) as name,
  CASE
    WHEN LOWER(category) IN ('electronics', 'tech', 'gadgets') THEN 'Electronics'
    WHEN LOWER(category) IN ('clothes', 'apparel', 'fashion') THEN 'Apparel'
    ELSE 'Other'
  END as category_normalized,
  ABS(price) as price
FROM products
WHERE sku IS NOT NULL"

# Split and extract data
dataql run -f users.csv -q "
SELECT
  id,
  SPLIT_PART(full_name, ' ', 1) as first_name,
  SPLIT_PART(full_name, ' ', -1) as last_name,
  SPLIT_PART(email, '@', 2) as email_domain
FROM users"
```

### Data Validation

```bash
# Find data quality issues
dataql run -f orders.csv -q "
SELECT
  'Missing customer_id' as issue, COUNT(*) as count
FROM orders WHERE customer_id IS NULL
UNION ALL
SELECT
  'Negative amount' as issue, COUNT(*) as count
FROM orders WHERE amount < 0
UNION ALL
SELECT
  'Future date' as issue, COUNT(*) as count
FROM orders WHERE date > CURRENT_DATE"

# Find duplicates
dataql run -f users.csv -q "
SELECT email, COUNT(*) as occurrences
FROM users
GROUP BY email
HAVING COUNT(*) > 1
ORDER BY occurrences DESC"
```

## Troubleshooting

### Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `column "xyz" not found` | Typo in column name or wrong case | Run `.schema table` to see exact column names |
| `table "xyz" not found` | Wrong table name | Table name = filename without extension (users.csv -> users) |
| `mixed file formats` | Files have different extensions | Use same format or use `-c` to set collection name |
| `memory allocation failed` | File too large for memory | Use `LIMIT`, add `WHERE` filters, or enable caching with `--cache` |
| `failed to parse row` | Malformed data in file | Check file for encoding issues, use `-v` for details |
| `ambiguous column reference` | Same column in multiple tables | Prefix with table alias: `t.column` instead of `column` |
| `division by zero` | Dividing by zero in query | Use `NULLIF(divisor, 0)` to handle zero values |
| `type mismatch` | Comparing incompatible types | Cast values: `CAST(column AS INTEGER)` or `CAST(column AS VARCHAR)` |

### Debugging Tips

```bash
# Check what tables are available
dataql run -f data.csv -q ".tables"

# See table schema
dataql run -f data.csv -q ".schema data"

# Get descriptive statistics
dataql describe -f data.csv

# Verbose mode for detailed logging
dataql run -f data.csv -q "SELECT * FROM data LIMIT 5" -v

# Preview raw data structure
dataql run -f data.json -q "SELECT * FROM data LIMIT 1" --vertical
```

### Performance Tips

```bash
# Use caching for repeated queries on large files
dataql run -f large_file.csv -q "SELECT * FROM large_file" --cache

# Filter early to reduce memory usage
dataql run -f huge.csv -q "SELECT col1, col2 FROM huge WHERE date > '2024-01-01' LIMIT 1000"

# Use parameterized queries for repeated similar queries
dataql run -f data.csv -q "SELECT * FROM data WHERE status = :status" -p status=active
```

### Working with Different File Types

```bash
# CSV with custom delimiter (semicolon)
dataql run -f data.csv -d ";" -q "SELECT * FROM data"

# Compressed files (gzip)
dataql run -f data.csv.gz -q "SELECT * FROM data"

# Excel files (specific sheet becomes table)
dataql run -f report.xlsx -q "SELECT * FROM Sheet1"

# JSONL (each line is a record)
dataql run -f events.jsonl -q "SELECT * FROM events LIMIT 10"
```

## Notes

- Table name defaults to filename without extension (e.g., `users.csv` -> `users`)
- Use `-c` flag to specify custom table name: `dataql run -f data.csv -c mytable -q "SELECT * FROM mytable"`
- For stdin: use `-f -` and table name is `stdin`
- Use `-v` for verbose output with detailed logging
- Use `--cache` to cache imported data for faster subsequent queries
- Use `-Q` (quiet mode) to suppress progress bar for scripting
