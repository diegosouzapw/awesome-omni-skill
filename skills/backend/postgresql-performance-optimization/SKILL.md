---
name: PostgreSQL Performance Optimization
skill_id: postgresql-optimization
version: 1.0.0
description: Production-grade PostgreSQL query optimization, indexing strategies, performance tuning, and modern features including pgvector for AI/ML workloads. Master EXPLAIN plans, query analysis, and database design for high-performance applications
category: Database & Data
tags:
  - postgresql
  - database
  - performance
  - optimization
  - sql
  - indexing
  - query-tuning
  - pgvector
  - ai-database
  - explain-plans
author: mcp-skillset
license: MIT
created: 2025-11-25
last_updated: 2025-11-25
toolchain:
  - PostgreSQL 15+
  - pgvector
  - pg_stat_statements
  - EXPLAIN ANALYZE
frameworks:
  - PostgreSQL
  - SQLAlchemy
  - asyncpg
related_skills:
  - fastapi-web-development
  - systematic-debugging
  - observability-monitoring
---

# PostgreSQL Performance Optimization

## Overview

Master PostgreSQL performance optimization with modern techniques for query tuning, indexing, and AI/ML workloads. With 55% of Postgres developers adopting AI tools in 2024, understanding pgvector and performance optimization is critical for building scalable data-intensive applications.

## When to Use This Skill

- Slow queries requiring optimization (>100ms response time)
- Designing database schemas for high-performance applications
- Implementing vector similarity search for AI/ML features
- Scaling PostgreSQL for high-concurrency workloads
- Migrating from NoSQL to PostgreSQL for better consistency
- Optimizing ORMs (SQLAlchemy, Django ORM) for production

## Core Principles

### 1. EXPLAIN ANALYZE is Your Best Friend

```sql
-- ALWAYS use EXPLAIN ANALYZE for slow queries
EXPLAIN (ANALYZE, BUFFERS, VERBOSE, COSTS, TIMING)
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > NOW() - INTERVAL '30 days'
GROUP BY u.id, u.name
ORDER BY order_count DESC
LIMIT 100;

-- Read output top-to-bottom, focus on:
-- 1. Seq Scan (bad for large tables) vs Index Scan (good)
-- 2. Actual time vs Planning time
-- 3. Rows estimates vs actual rows
-- 4. Buffers (disk I/O indicators)
```

**Key Metrics**:
- **Planning Time**: How long query planner took (<10ms ideal)
- **Execution Time**: Actual query runtime (<100ms for OLTP)
- **Rows**: Estimated vs actual (mismatches indicate stale statistics)
- **Buffers**: Shared hits (good), reads (disk I/O, slow)

### 2. Indexing Strategy

```sql
-- B-Tree Index (default, most common)
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at);

-- Partial Index (smaller, faster for filtered queries)
CREATE INDEX idx_active_users ON users(email)
WHERE is_active = true AND deleted_at IS NULL;

-- Covering Index (includes extra columns to avoid table lookups)
CREATE INDEX idx_orders_covering ON orders(user_id, status)
INCLUDE (created_at, total_amount);

-- GIN Index (for full-text search, JSONB, arrays)
CREATE INDEX idx_products_search ON products
USING GIN (to_tsvector('english', name || ' ' || description));

CREATE INDEX idx_tags_gin ON posts USING GIN(tags);

-- GiST Index (for geometric data, ranges, full-text)
CREATE INDEX idx_locations_gist ON stores
USING GIST (location);

-- BRIN Index (block range, time-series data)
CREATE INDEX idx_logs_created ON logs USING BRIN(created_at);
```

**Index Selection Rules**:
- Equality filters (`WHERE id = 123`) → B-Tree
- Range queries (`WHERE created_at > NOW() - '7 days'`) → B-Tree
- Full-text search → GIN with `tsvector`
- JSONB queries → GIN
- Time-series/append-only → BRIN (90% smaller than B-Tree)
- Vector similarity (`ORDER BY embedding <=> query_vector`) → HNSW (pgvector)

### 3. Query Optimization Patterns

```sql
-- BAD: SELECT *
SELECT * FROM users WHERE email = 'user@example.com';

-- GOOD: Select only needed columns
SELECT id, name, email FROM users WHERE email = 'user@example.com';

-- BAD: N+1 Query Problem
-- Application code:
FOR user IN (SELECT id FROM users):
    SELECT * FROM orders WHERE user_id = user.id;  -- N queries!

-- GOOD: JOIN or use IN clause
SELECT u.name, o.id, o.total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- BAD: Function in WHERE clause prevents index use
SELECT * FROM users WHERE LOWER(email) = 'user@example.com';

-- GOOD: Functional index or case-insensitive comparison
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = 'user@example.com';

-- Or use CITEXT type
ALTER TABLE users ALTER COLUMN email TYPE CITEXT;
SELECT * FROM users WHERE email = 'USER@EXAMPLE.COM';  -- Works!

-- BAD: OR conditions often don't use indexes
SELECT * FROM products WHERE category = 'electronics' OR category = 'books';

-- GOOD: Use IN or UNION
SELECT * FROM products WHERE category IN ('electronics', 'books');

-- GOOD: UNION for complex OR (sometimes faster)
SELECT * FROM products WHERE category = 'electronics'
UNION ALL
SELECT * FROM products WHERE category = 'books';
```

### 4. Connection Pooling

```python
# BAD: Opening connection per request
import psycopg2

def get_user(user_id):
    conn = psycopg2.connect("postgresql://localhost/db")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    result = cursor.fetchone()
    conn.close()  # Creates new connection each time!
    return result

# GOOD: Use connection pooling (asyncpg with FastAPI)
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

engine = create_async_engine(
    "postgresql+asyncpg://localhost/db",
    pool_size=20,          # Max persistent connections
    max_overflow=0,        # Don't allow exceeding pool_size
    pool_pre_ping=True,    # Verify connection before using
    echo_pool=True,        # Log pool activity (disable in production)
)

AsyncSessionLocal = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)
```

**Connection Pool Settings**:
- `pool_size`: Core connections (20-50 for web apps)
- `max_overflow`: Extra connections (0 to prevent overload)
- `pool_recycle`: Close connections after N seconds (3600 = 1 hour)

### 5. pgvector for AI/ML Workloads

```sql
-- Install pgvector extension
CREATE EXTENSION vector;

-- Create table with vector column
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding vector(1536)  -- OpenAI ada-002 dimensions
);

-- Create HNSW index for fast similarity search
CREATE INDEX ON documents
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);

-- Similarity search (finds nearest neighbors)
SELECT id, content, 1 - (embedding <=> '[0.1, 0.2, ...]'::vector) AS similarity
FROM documents
ORDER BY embedding <=> '[0.1, 0.2, ...]'::vector
LIMIT 10;

-- Operators:
-- <->  L2 distance (Euclidean)
-- <#>  Inner product
-- <=>  Cosine distance (most common for LLM embeddings)
```

**pgvector Best Practices**:
- Use HNSW index for production (10-100x faster than IVFFlat)
- Normalize embeddings before storage for cosine similarity
- Tune `m` (16-48) and `ef_construction` (64-200) for accuracy/speed tradeoff
- Use `SET ivfflat.probes = 10` for IVFFlat search quality

## Performance Tuning Checklist

### PostgreSQL Configuration (postgresql.conf)

```ini
# Memory Settings (for 16GB RAM server)
shared_buffers = 4GB              # 25% of RAM
effective_cache_size = 12GB       # 75% of RAM
work_mem = 64MB                   # Per operation memory
maintenance_work_mem = 1GB        # For VACUUM, CREATE INDEX

# Query Planner
random_page_cost = 1.1            # SSD (4.0 for HDD)
effective_io_concurrency = 200    # SSD parallelism

# Write-Ahead Log (WAL)
wal_buffers = 16MB
max_wal_size = 4GB
min_wal_size = 1GB
checkpoint_completion_target = 0.9

# Connections
max_connections = 200

# Logging
log_min_duration_statement = 1000  # Log queries >1s
log_line_prefix = '%m [%p] %u@%d '
log_statement = 'none'             # 'all' for debugging
```

### Essential Extensions

```sql
-- Query statistics (find slow queries)
CREATE EXTENSION pg_stat_statements;

-- View top 10 slowest queries
SELECT query, calls, total_exec_time, mean_exec_time, max_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Auto-explain for slow queries (in postgresql.conf)
-- shared_preload_libraries = 'auto_explain'
-- auto_explain.log_min_duration = '1s'

-- UUID generation
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Full-text search (built-in, no extension needed)
SELECT * FROM products
WHERE to_tsvector('english', name) @@ to_tsquery('laptop & portable');
```

## Common Anti-Patterns

### ❌ DON'T: Use OFFSET for pagination

```sql
-- BAD: OFFSET becomes slow with large offsets
SELECT * FROM posts
ORDER BY created_at DESC
LIMIT 50 OFFSET 100000;  -- Scans 100,050 rows!

-- GOOD: Keyset pagination (cursor-based)
SELECT * FROM posts
WHERE created_at < '2024-01-01 12:00:00'
ORDER BY created_at DESC
LIMIT 50;
```

### ❌ DON'T: Run VACUUM FULL in production

```sql
-- BAD: Locks entire table
VACUUM FULL users;  -- Hours of downtime!

-- GOOD: Regular VACUUM (non-blocking)
VACUUM ANALYZE users;

-- Or configure autovacuum (postgresql.conf)
-- autovacuum = on
-- autovacuum_naptime = 1min
```

### ❌ DON'T: Ignore table statistics

```sql
-- Run ANALYZE regularly (auto after bulk inserts)
ANALYZE users;

-- Check last analyze time
SELECT schemaname, tablename, last_analyze, last_autoanalyze
FROM pg_stat_user_tables
WHERE schemaname = 'public';
```

## Testing & Monitoring

```sql
-- Check bloat in tables and indexes
SELECT
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) -
                   pg_relation_size(schemaname||'.'||tablename)) AS bloat
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;

-- Check index usage
SELECT
    schemaname || '.' || tablename AS table,
    indexname AS index,
    idx_scan as scans,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan ASC, pg_relation_size(indexrelid) DESC;

-- Missing indexes (sequential scans on large tables)
SELECT
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    idx_scan,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_stat_user_tables
WHERE seq_scan > 0 AND idx_scan = 0
    AND pg_total_relation_size(schemaname||'.'||tablename) > 1000000
ORDER BY seq_tup_read DESC;
```

## Related Skills

- **fastapi-web-development**: Optimize FastAPI + PostgreSQL integration
- **observability-monitoring**: Monitor PostgreSQL with Prometheus
- **systematic-debugging**: Debug query performance issues

## Additional Resources

- PostgreSQL Performance: https://wiki.postgresql.org/wiki/Performance_Optimization
- pgvector Documentation: https://github.com/pgvector/pgvector
- Use The Index, Luke: https://use-the-index-luke.com
- pg_stat_statements: https://www.postgresql.org/docs/current/pgstatstatements.html

## Example Questions

- "Why is this query slow? How do I read the EXPLAIN plan?"
- "What type of index should I use for full-text search?"
- "How do I implement vector similarity search with pgvector?"
- "Show me how to find unused indexes in my database"
- "What's the best way to paginate large result sets?"
- "How do I configure connection pooling for a high-traffic API?"
