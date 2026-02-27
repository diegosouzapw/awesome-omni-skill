---
name: supabase-postgres-best-practices
description: "Postgres query performance optimization and runtime best practices for Supabase. Covers indexing strategies (B-tree, GIN, GiST, BRIN, composite, partial, covering), EXPLAIN ANALYZE diagnostics, connection pooling (Supavisor transaction/session modes, pool sizing), RLS performance patterns (auth.uid() subquery caching, SECURITY DEFINER bypass), concurrency control (deadlock prevention, SKIP LOCKED, advisory locks), data access optimization (N+1 elimination, keyset pagination, batch inserts, UPSERT), runtime monitoring (pg_stat_statements, VACUUM/ANALYZE), and advanced tuning (full-text search, JSONB GIN indexing). Use when writing, reviewing, or optimizing SQL queries, diagnosing slow queries with EXPLAIN, configuring connection pooling, tuning RLS performance, implementing concurrent processing, detecting unused indexes, or resolving Postgres bottlenecks. Does NOT cover schema design (ansem-db-patterns), auth/RLS policy design (supabase-auth-patterns), or TypeScript types (typescript-best-practices)."
user-invocable: false
---

# Supabase Postgres Best Practices

Postgres query performance and runtime optimization for Supabase projects. Focus: making queries fast, connections efficient, and RLS performant at runtime.

## Scope & Boundaries

| Topic | This Skill | Other Skill |
|-------|-----------|-------------|
| Query optimization, indexing, EXPLAIN | **Here** | - |
| Connection pooling (Supavisor), timeouts | **Here** | - |
| RLS **performance** (subquery caching, index) | **Here** | `supabase-auth-patterns` (RLS policy design, auth flows) |
| Concurrency, locking, queues | **Here** | - |
| Monitoring (pg_stat_*, VACUUM) | **Here** | - |
| Unused index detection, index maintenance | **Here** | - |
| Full-text search, JSONB query tuning | **Here** | - |
| Edge Function DB connections | **Here** | - |
| Schema design (naming, types, constraints, FK) | - | `ansem-db-patterns` |
| Auth flows, session management, RLS creation | - | `supabase-auth-patterns` |
| Query performance debugging process | - | `systematic-debugging` (Phase 1 investigation) |
| Load testing, regression test for perf fixes | - | `testing-strategy` (test protocol) |

## When to Apply

- Writing or reviewing SQL queries for performance
- Adding, optimizing, or removing unused indexes
- Diagnosing slow queries with EXPLAIN ANALYZE
- Configuring connection pooling (Supavisor)
- Optimizing RLS policies that cause slowdowns
- Implementing concurrent processing (queues, locks)
- Bulk data loading or cursor pagination
- Profiling with pg_stat_statements
- Setting statement_timeout / idle timeouts
- Connecting from Edge Functions or serverless environments

## When NOT to Apply

- Table design, naming conventions, data types, FK policies -> `ansem-db-patterns`
- Auth flows, RLS policy creation, JWT handling -> `supabase-auth-patterns`
- Root cause investigation process for query issues -> `systematic-debugging`
- Writing regression tests after performance fix -> `testing-strategy`
- Application-level error handling -> `error-handling-logging`

---

## Category 1: Query Performance [CRITICAL]

### Index Missing WHERE/JOIN Columns

Unindexed columns cause full table scans (100-1000x slower on large tables).

```sql
-- Always index columns used in WHERE and JOIN
CREATE INDEX CONCURRENTLY orders_customer_id_idx ON orders (customer_id);

-- Verify with EXPLAIN
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;
-- Should show: Index Scan (not Seq Scan)
```

Always use `CREATE INDEX CONCURRENTLY` in production to avoid blocking writes.

### Composite Index Column Order

Equality columns first, range columns last. Leftmost prefix rule applies.

```sql
-- Good: status (=) before created_at (>)
CREATE INDEX idx ON orders (status, created_at);
-- Works for: WHERE status = 'pending'
-- Works for: WHERE status = 'pending' AND created_at > '2024-01-01'
-- Does NOT work for: WHERE created_at > '2024-01-01' alone
```

### Choose Index Type by Query Pattern

```sql
-- B-tree (default): =, <, >, BETWEEN, IN, IS NULL
CREATE INDEX idx ON users (created_at);

-- GIN: arrays, JSONB (@>, ?, ?&), full-text search (@@)
CREATE INDEX idx ON products USING gin (attributes);

-- GiST: geometric, range types, KNN nearest-neighbor (<->)
CREATE INDEX idx ON locations USING gist (coordinates);

-- BRIN: large time-series tables (10-100x smaller than B-tree)
-- Requires physically correlated data (e.g., append-only timestamp)
CREATE INDEX idx ON events USING brin (created_at);
```

### Partial & Covering Indexes

```sql
-- Partial: index only rows you query (5-20x smaller)
CREATE INDEX idx ON users (email) WHERE deleted_at IS NULL;

-- Covering: avoid table lookups with INCLUDE (index-only scan)
CREATE INDEX idx ON users (email) INCLUDE (name, created_at);
```

### Detect & Remove Unused Indexes

Unused indexes waste disk and slow writes. Check periodically.

```sql
SELECT schemaname, relname, indexrelname, idx_scan, pg_size_pretty(pg_relation_size(i.indexrelid))
FROM pg_stat_user_indexes i
JOIN pg_index USING (indexrelid)
WHERE idx_scan = 0 AND NOT indisunique
ORDER BY pg_relation_size(i.indexrelid) DESC;
-- Drop indexes with idx_scan = 0 after confirming (wait 2+ weeks of data)
```

-> Full details with EXPLAIN output: [reference.md](reference.md)
-> Index naming conventions and FK indexing rules -> `ansem-db-patterns`

---

## Category 2: Connection Management [CRITICAL]

### Connection Pooling (Supavisor)

Each Postgres connection uses 1-3MB RAM. Always pool. Supabase uses Supavisor.

```
-- Pool size formula: (CPU cores * 2) + spindle_count
-- 4 cores -> pool_size = 10

-- Transaction mode (port 6543): connection returned after each transaction (default, best)
-- Session mode (port 5432): needed for prepared statements, LISTEN/NOTIFY, temp tables
```

### Connection Limits & Timeouts

```sql
-- Set limits based on RAM (max_connections * work_mem < 25% RAM)
ALTER SYSTEM SET max_connections = 100;
ALTER SYSTEM SET work_mem = '8MB';

-- Kill runaway queries after 30s
ALTER SYSTEM SET statement_timeout = '30s';

-- Terminate idle-in-transaction after 30s
ALTER SYSTEM SET idle_in_transaction_session_timeout = '30s';

-- Monitor usage
SELECT count(*), state FROM pg_stat_activity GROUP BY state;
```

### Prepared Statements with Pooling

Named prepared statements break in transaction-mode pooling. Use unnamed statements or deallocate after use.

### Edge Function Connections [CRITICAL]

Edge Functions are short-lived and create many connections. Always use Supavisor pooling.

```typescript
// Edge Function: always use pooled connection string (port 6543)
const supabase = createClient(
  Deno.env.get('SUPABASE_URL')!,
  Deno.env.get('SUPABASE_ANON_KEY')!
)
// supabase-js automatically uses the pooled REST endpoint

// Direct Postgres from Edge Function (use pooler URL, never direct)
import postgres from 'https://deno.land/x/postgresjs/mod.js'
const sql = postgres(Deno.env.get('SUPABASE_DB_URL')!) // Must be pooler URL
```

-> Connection patterns, driver config: [reference.md](reference.md)

---

## Category 3: RLS Performance [CRITICAL]

### Wrap Functions in Subquery

`auth.uid()` called per-row is 100x slower than cached in subquery.

```sql
-- BAD: USING (auth.uid() = user_id)
-- GOOD: USING ((SELECT auth.uid()) = user_id)
```

Full example: see [reference.md](reference.md)

### Security Definer for Complex Checks

```sql
-- BAD: complex JOIN inside RLS policy directly
-- GOOD: SECURITY DEFINER function wrapping the check
CREATE POLICY p ON orders USING ((SELECT is_team_member(team_id)));
```

Full example: see [reference.md](reference.md). Always index columns used in RLS policies.

-> RLS policy design and auth patterns -> `supabase-auth-patterns`
-> Debugging slow RLS queries: use `systematic-debugging` Phase 1 with EXPLAIN ANALYZE

---

## Category 4: Concurrency & Locking [HIGH]

### Short Transactions

Move external calls (HTTP, payment APIs) outside transactions. Set `statement_timeout` as safety net.

```sql
-- Per-session safety net
SET statement_timeout = '10s';
```

### Deadlock Prevention

Acquire locks in consistent order (e.g., by ID ascending).

```sql
SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;
```

### SKIP LOCKED Queue Pattern

```sql
-- SELECT ... FOR UPDATE SKIP LOCKED in subquery, UPDATE in outer query
-- Prevents worker contention on job queues
```

Full example: see [reference.md](reference.md)

-> Advisory locks, deadlock detection: [reference.md](reference.md)

---

## Category 5: Data Access Patterns [HIGH]

### Eliminate N+1 Queries

```sql
-- BAD: loop per user
SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM orders WHERE user_id = 2; -- ... N times

-- GOOD: single batch query
SELECT * FROM orders WHERE user_id = ANY($1::bigint[]);
```

### Cursor Pagination (not OFFSET)

```sql
-- OFFSET scans all skipped rows. Cursor is O(1).
SELECT * FROM products WHERE id > $last_id ORDER BY id LIMIT 20;
```

### Batch Inserts & UPSERT

```sql
-- Batch: 1 round trip instead of N
INSERT INTO events (user_id, action) VALUES (1,'click'), (2,'view'), ...;

-- UPSERT: atomic insert-or-update, no race conditions
INSERT INTO settings (user_id, key, value) VALUES ($1, $2, $3)
ON CONFLICT (user_id, key) DO UPDATE SET value = EXCLUDED.value;
```

-> UPSERT schema design (conflict target, idempotent inserts) -> `ansem-db-patterns`
-> After fixing N+1, write regression test to prevent recurrence -> `testing-strategy`

---

## Category 6: Monitoring & Diagnostics [HIGH]

### EXPLAIN ANALYZE

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) SELECT ...;
-- Look for: Seq Scan (missing index), Rows Removed by Filter (poor selectivity)
-- Buffers read >> hit (cold cache), Sort external merge (work_mem too low)
```

### pg_stat_statements

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
SELECT calls, round(mean_exec_time::numeric, 2), query
FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 10;

-- Reset after optimization to measure fresh baselines
SELECT pg_stat_statements_reset();
```

### VACUUM & ANALYZE

Run `ANALYZE` after large data changes. Tune autovacuum for high-churn tables.

```sql
ALTER TABLE orders SET (
  autovacuum_vacuum_scale_factor = 0.05,
  autovacuum_analyze_scale_factor = 0.02
);
```

-> Detailed diagnostics queries, Supabase Dashboard metrics: [reference.md](reference.md)

---

## Quick Decision Guide

```
Query slow?
  +-- Run EXPLAIN (ANALYZE, BUFFERS)
       +-- Seq Scan? -> Add index (Category 1)
       +-- Nested Loop high loops? -> Check N+1 (Category 5)
       +-- Sort external merge? -> Increase work_mem
       +-- Rows Removed by Filter high? -> Partial/composite index
  +-- RLS enabled? -> Wrap auth.uid() in subquery (Category 3)
  +-- Connection errors? -> Check pooling config (Category 2)
  +-- Lock waits? -> Shorten transactions (Category 4)
  +-- Which queries are slowest? -> pg_stat_statements (Category 6)
  +-- Unused indexes bloating? -> pg_stat_user_indexes (Category 1)
  +-- Can't find root cause? -> systematic-debugging Phase 1
  +-- After fix, prevent regression? -> testing-strategy (load test)
```
