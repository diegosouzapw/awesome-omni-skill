---
name: database-query
description: Natural language database queries with multi-database support, query optimization, and visual results
allowed-tools: ["Bash", "Read", "Write", "Task"]
version: 1.0.0
author: GLINCKER Team
license: Apache-2.0
keywords: [database, sql, query, natural-language, postgresql, mysql, mongodb, optimization]
---

# Database Query (Natural Language)

**⚡ UNIQUE FEATURE**: Query any database using natural language - automatically generates optimized SQL/NoSQL queries, explains query plans, suggests indexes, and visualizes results. Supports PostgreSQL, MySQL, MongoDB, SQLite, and more.

## What This Skill Does

Transform natural language into optimized database queries:

- **Natural language to SQL**: "Show me users who signed up last month" → `SELECT * FROM users WHERE created_at >= NOW() - INTERVAL '1 month'`
- **Multi-database support**: PostgreSQL, MySQL, MongoDB, SQLite, Redis
- **Query optimization**: Analyzes queries and suggests improvements
- **Index suggestions**: Recommends indexes for slow queries
- **Visual results**: Formats query results as tables, charts, JSON
- **Query explanation**: EXPLAIN ANALYZE with human-readable insights
- **Safe mode**: Read-only by default with confirmation for writes
- **Schema discovery**: Auto-learns database structure

## Why This Is Unique

First Claude Code skill that:
- **Understands intent**: Translates vague requests to precise queries
- **Cross-database compatible**: Same natural language works across SQL/NoSQL
- **Performance-aware**: Automatically optimizes and suggests indexes
- **Safety-first**: Prevents destructive operations without confirmation
- **Learning mode**: Improves by understanding your schema

## Instructions

### Phase 1: Database Connection & Discovery

1. **Identify Database**:
   ```
   Ask user:
   - Database type (PostgreSQL, MySQL, MongoDB, SQLite, etc.)
   - Connection method (local, remote, Docker, MCP server)
   - Connection string or credentials
   ```

2. **Test Connection**:
   ```bash
   # PostgreSQL
   psql -h localhost -U user -d database -c "SELECT version();"

   # MySQL
   mysql -h localhost -u user -p database -e "SELECT VERSION();"

   # MongoDB
   mongosh "mongodb://localhost:27017/database" --eval "db.version()"

   # SQLite
   sqlite3 database.db "SELECT sqlite_version();"
   ```

3. **Discover Schema**:
   ```bash
   # PostgreSQL: Get all tables and columns
   psql -d database -c "\dt"
   psql -d database -c "\d+ table_name"

   # MySQL: Show database structure
   mysql database -e "SHOW TABLES;"
   mysql database -e "DESCRIBE table_name;"

   # MongoDB: List collections and sample documents
   mongosh database --eval "db.getCollectionNames()"
   mongosh database --eval "db.collection.findOne()"
   ```

4. **Build Schema Cache**:
   - Store table/collection names
   - Store column names and types
   - Store relationships (foreign keys)
   - Cache common queries

### Phase 2: Natural Language to Query Translation

When user makes a request:

1. **Parse Intent**:
   ```
   Analyze the request:
   - Action: SELECT, INSERT, UPDATE, DELETE, aggregation
   - Entities: Which tables/collections
   - Conditions: WHERE clauses
   - Aggregations: COUNT, SUM, AVG, GROUP BY
   - Sorting: ORDER BY
   - Limits: TOP N, pagination
   ```

2. **Generate Query**:

   **Example 1**: "Show me all active users"
   ```sql
   -- PostgreSQL/MySQL
   SELECT * FROM users WHERE status = 'active';
   ```

   **Example 2**: "Count orders by status for last 7 days"
   ```sql
   SELECT status, COUNT(*) as count
   FROM orders
   WHERE created_at >= NOW() - INTERVAL '7 days'
   GROUP BY status
   ORDER BY count DESC;
   ```

   **Example 3**: "Find top 10 customers by revenue"
   ```sql
   SELECT
     c.name,
     c.email,
     SUM(o.total) as revenue
   FROM customers c
   JOIN orders o ON c.id = o.customer_id
   GROUP BY c.id, c.name, c.email
   ORDER BY revenue DESC
   LIMIT 10;
   ```

   **Example 4**: MongoDB aggregation
   ```javascript
   db.orders.aggregate([
     { $match: { status: "completed" } },
     { $group: {
         _id: "$customer_id",
         total: { $sum: "$amount" }
     }},
     { $sort: { total: -1 } },
     { $limit: 10 }
   ])
   ```

3. **Validate Query**:
   - Check table/column names exist
   - Verify data types match
   - Ensure joins are valid
   - Detect potentially dangerous operations

### Phase 3: Query Optimization

Before execution:

1. **Analyze Query Plan**:
   ```sql
   -- PostgreSQL
   EXPLAIN ANALYZE
   SELECT * FROM users WHERE email LIKE '%@example.com';
   ```

2. **Suggest Optimizations**:
   ```
   If sequential scan detected:
   - "This query is scanning all rows. Consider adding an index:"
   - CREATE INDEX idx_users_email ON users(email);

   If N+1 query pattern:
   - "Use JOIN instead of multiple queries"
   - Show optimized version

   If missing WHERE clause:
   - "This will return all rows. Add filters or LIMIT?"
   ```

3. **Rewrite for Performance**:
   ```sql
   -- Before (slow)
   SELECT * FROM users WHERE LOWER(email) = 'user@example.com';

   -- After (fast - uses index)
   SELECT * FROM users WHERE email = 'user@example.com';
   ```

### Phase 4: Safe Execution

1. **Determine Query Type**:
   - **Read-only** (SELECT): Execute immediately
   - **Write** (INSERT, UPDATE, DELETE): Ask confirmation
   - **DDL** (CREATE, DROP, ALTER): Require explicit confirmation

2. **Confirmation for Writes**:
   ```
   ⚠️ This query will modify data:

   UPDATE users SET status = 'inactive'
   WHERE last_login < '2024-01-01'

   Estimated affected rows: 1,247

   Proceed? [yes/no]
   ```

3. **Transaction Support**:
   ```sql
   BEGIN;
   -- Execute query
   -- Show results
   -- Ask: COMMIT or ROLLBACK?
   ```

### Phase 5: Results Formatting

1. **Table Format** (default):
   ```
   ┌────┬─────────────┬──────────────────────┬──────────┐
   │ id │ name        │ email                │ status   │
   ├────┼─────────────┼──────────────────────┼──────────┤
   │ 1  │ John Doe    │ john@example.com     │ active   │
   │ 2  │ Jane Smith  │ jane@example.com     │ active   │
   └────┴─────────────┴──────────────────────┴──────────┘

   2 rows returned in 0.023s
   ```

2. **Chart Format** (for aggregations):
   ```
   Orders by Status:

   pending   ████████████░░░░░░░░ 62
   completed ████████████████████ 128
   cancelled ████░░░░░░░░░░░░░░░░ 15
   ```

3. **JSON Format** (for APIs):
   ```json
   {
     "query": "SELECT * FROM users LIMIT 2",
     "execution_time": "0.023s",
     "row_count": 2,
     "results": [
       {"id": 1, "name": "John Doe", ...},
       {"id": 2, "name": "Jane Smith", ...}
     ]
   }
   ```

4. **Export Options**:
   - CSV file
   - JSON file
   - Markdown table
   - Copy to clipboard

## Examples

### Example 1: Simple Query

**User**: "Show me recent users"

**Skill**:
1. Interprets "recent" as last 7 days
2. Generates query:
   ```sql
   SELECT * FROM users
   WHERE created_at >= NOW() - INTERVAL '7 days'
   ORDER BY created_at DESC;
   ```
3. Executes and displays results
4. Suggests: "Want to filter by status or role?"

### Example 2: Complex Aggregation

**User**: "Which products had the most revenue last quarter?"

**Skill**:
1. Determines tables: products, orders, order_items
2. Calculates "last quarter" date range
3. Generates optimized query:
   ```sql
   SELECT
     p.id,
     p.name,
     SUM(oi.quantity * oi.price) as revenue,
     COUNT(DISTINCT o.id) as order_count
   FROM products p
   JOIN order_items oi ON p.id = oi.product_id
   JOIN orders o ON oi.order_id = o.id
   WHERE o.created_at >= DATE_TRUNC('quarter', NOW() - INTERVAL '3 months')
     AND o.created_at < DATE_TRUNC('quarter', NOW())
     AND o.status = 'completed'
   GROUP BY p.id, p.name
   ORDER BY revenue DESC
   LIMIT 10;
   ```
4. Shows results with chart
5. Offers to export

### Example 3: Performance Investigation

**User**: "Why is this query slow?"
```sql
SELECT * FROM orders WHERE customer_name LIKE 'John%';
```

**Skill**:
1. Runs EXPLAIN ANALYZE
2. Detects: Sequential scan on 10M rows
3. Suggests:
   ```
   ⚠️ Performance Issue Detected:

   Problem: Full table scan (10,485,234 rows)
   Solution: Add an index on customer_name

   CREATE INDEX idx_orders_customer_name ON orders(customer_name);

   Expected improvement: 10,485,234 rows → ~42 rows
   Estimated speed-up: 10,000x faster

   Would you like me to create this index?
   ```

## Configuration

Create `.database-query-config.yml`:

```yaml
databases:
  - name: production
    type: postgresql
    host: localhost
    port: 5432
    database: myapp
    user: readonly_user
    ssl: true
    read_only: true

  - name: analytics
    type: mongodb
    uri: mongodb://localhost:27017/analytics

  - name: cache
    type: redis
    host: localhost
    port: 6379

defaults:
  max_rows: 1000
  timeout: 30s
  explain_threshold: 1s  # Auto-explain queries slower than 1s
  auto_optimize: true

safety:
  require_confirmation_for_writes: true
  prevent_drop_table: true
  max_affected_rows: 10000
```

## Tool Requirements

- **Bash**: Execute database CLI commands
- **Read**: Read config files and schema cache
- **Write**: Save query results and reports
- **Task**: Launch optimization analyzer agent

## Integration with MCP

Connect to MCP database servers:

```yaml
# Using PostgreSQL MCP server
mcp_servers:
  - name: postgres
    command: postgres-mcp
    args:
      - --connection-string
      - postgresql://user:pass@localhost/db
```

## Advanced Features

### 1. Query History & Favorites

```bash
# Save favorite queries
claude db save "monthly_revenue" "SELECT..."

# Run saved query
claude db run monthly_revenue
```

### 2. Query Templates

```sql
-- Template: user_search
SELECT * FROM users
WHERE {{field}} = {{value}}
AND status = 'active';
```

### 3. Data Migration Helper

```python
# Generate migration between databases
claude db migrate --from postgres://... --to mysql://...
```

### 4. Schema Diff

```bash
# Compare two databases
claude db diff production staging
```

## Best Practices

1. **Start with schema**: Let skill discover your database first
2. **Use read-only mode**: For production databases
3. **Review before writes**: Always check UPDATE/DELETE affects
4. **Monitor performance**: Pay attention to optimization suggestions
5. **Save common queries**: Build a library of frequently-used queries
6. **Use transactions**: For multi-step operations

## Limitations

- Maximum 10,000 rows displayed (configurable)
- Query timeout: 30 seconds (configurable)
- Write operations require confirmation
- Some database-specific features may not translate
- Complex stored procedures not supported

## Security

- Never stores credentials in plain text
- Read-only mode by default
- SQL injection prevention
- Confirms destructive operations
- Audit logging available

## Related Skills

- [api-connector](../api-connector/SKILL.md) - Query APIs with natural language
- [data-analyzer](../../data-science/data-analyzer/SKILL.md) - Analyze query results
- [schema-designer](../../development/schema-designer/SKILL.md) - Design database schemas

## Changelog

### Version 1.0.0 (2025-01-13)
- Initial release
- PostgreSQL, MySQL, MongoDB, SQLite support
- Natural language query translation
- Query optimization and EXPLAIN
- Multiple output formats
- Safe mode with confirmations

## Contributing

Help expand database support:
- Add new database types (CockroachDB, DynamoDB, Cassandra)
- Improve query optimization
- Add more visualization options
- Create query templates

## License

Apache License 2.0 - See [LICENSE](../../../LICENSE)

## Author

**GLINCKER Team**
- GitHub: [@GLINCKER](https://github.com/GLINCKER)
- Repository: [claude-code-marketplace](https://github.com/GLINCKER/claude-code-marketplace)

---

**🌟 The most advanced natural language database query skill available!**
