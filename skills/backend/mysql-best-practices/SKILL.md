---
name: mysql-best-practices
description: MySQL performance optimization and best practices. Use this skill when writing, reviewing, or optimizing MySQL queries, schema designs, or database configurations.
license: MIT
metadata:
  author: mysql-community
  version: "1.0.0"
  organization: MySQL Community
  date: January 2026
  abstract: Comprehensive MySQL performance optimization guide for developers. Contains performance rules across 8 categories, prioritized by impact from critical (query performance, connection management) to incremental (advanced features). Each rule includes detailed explanations, incorrect vs. correct SQL examples, query plan analysis, and specific performance metrics to guide automated optimization and code generation.
---

# MySQL Best Practices

Comprehensive performance optimization guide for MySQL, adapted from PostgreSQL best practices. Contains rules across 8 categories, prioritized by impact to guide automated query optimization and schema design.

## When to Apply

Reference these guidelines when:
- Writing SQL queries or designing schemas
- Implementing indexes or query optimization
- Reviewing database performance issues
- Configuring connection pooling or scaling
- Optimizing for MySQL-specific features
- Working with MySQL security features

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Query Performance | CRITICAL | `query-` |
| 2 | Connection Management | CRITICAL | `conn-` |
| 3 | Security | CRITICAL | `security-` |
| 4 | Schema Design | HIGH | `schema-` |
| 5 | Concurrency & Locking | MEDIUM-HIGH | `lock-` |
| 6 | Data Access Patterns | MEDIUM | `data-` |
| 7 | Monitoring & Diagnostics | LOW-MEDIUM | `monitor-` |
| 8 | Advanced Features | LOW | `advanced-` |

## How to Use

Read individual rule files for detailed explanations and SQL examples:

```
references/query-missing-indexes.md
references/schema-partial-indexes.md
references/_sections.md
```

Each rule file contains:
- Brief explanation of why it matters
- Incorrect SQL example with explanation
- Correct SQL example with explanation
- Optional EXPLAIN output or metrics
- Additional context and references
- MySQL-specific notes

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`

## MySQL vs PostgreSQL Key Differences

This skill adapts PostgreSQL best practices to MySQL. Key differences:

- **JSON**: MySQL uses `JSON` type (not `JSONB`), with `JSON_EXTRACT()`, `JSON_CONTAINS()` functions
- **Arrays**: MySQL doesn't have native arrays; use JSON arrays or normalized tables
- **Full-Text Search**: MySQL uses `FULLTEXT` indexes with `MATCH() AGAINST()` syntax
- **UUIDs**: Use `CHAR(36)` or `BINARY(16)` instead of native UUID type
- **String Matching**: Use `LIKE` (case-sensitive) or `LIKE BINARY` instead of `ILIKE`
- **Type Casting**: Use `CAST()` function instead of `::` operator
- **Window Functions**: MySQL 8.0+ supports window functions (similar to PostgreSQL)

## References

- https://dev.mysql.com/doc/
- https://dev.mysql.com/doc/refman/8.0/en/optimization.html
- https://dev.mysql.com/doc/refman/8.0/en/explain.html
- https://dev.mysql.com/doc/refman/8.0/en/json-functions.html
- https://dev.mysql.com/doc/refman/8.0/en/fulltext-search.html
