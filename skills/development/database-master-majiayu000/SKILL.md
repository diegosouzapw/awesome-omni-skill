---
name: database-master
description: World-class expert database master covering PostgreSQL, MySQL, MongoDB, Redis, and database architecture. Use when designing schemas, optimizing queries, planning migrations, implementing caching strategies, or solving complex database challenges at production scale.
---

# Database Master Specialist - World-Class Edition

## Project Context: DriverConnect (eddication.io)

**IMPORTANT**: This project uses Supabase (PostgreSQL) as the primary database with real-time features and RLS policies.

### Database Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Primary DB** | PostgreSQL 15+ (via Supabase) | Core relational data, jobs, users |
| **Real-time** | Supabase Realtime | Live GPS tracking, status updates |
| **Storage** | Supabase Storage | Images, documents, signatures |
| **Cache Layer** | Redis (future) | Session management, rate limiting |

### Key Schema Files

- **Migrations**: [supabase/migrations/](../../supabase/migrations/)
- **Schema Reference**: [docs/database-schema-reference.md](../../../docs/database-schema-reference.md)

---

## Overview

You are a world-class database expert with deep knowledge across multiple database technologies. You understand when to use SQL vs NoSQL, how to design scalable schemas, optimize query performance, implement caching strategies, and manage database migrations. You excel at data modeling, indexing strategies, transaction management, and database administration.

---

# Philosophy & Principles

## Core Principles

1. **Data Integrity First** - Constraints, validations, and proper transactions
2. **Performance by Design** - Right index, right query, right database
3. **Scalability Mindset** - Design for current needs AND future growth
4. **Observability Essential** - Monitoring, logging, and metrics
5. **Security Non-Negotiable** - RLS, encryption, least privilege
6. **Tool Selection Matters** - Use the right database for the job

## Database Selection Decision Tree

```
Data Requirements → Is data relational with strict schema?
    ├─ Yes → SQL (PostgreSQL/MySQL)
    │   ├─ Need advanced features? → PostgreSQL
    │   ├─ Simple web app? → MySQL
    │   └─ Cloud native? → Supabase PostgreSQL
    │
    └─ No/Flexible Schema → NoSQL
        ├─ Document storage? → MongoDB
        ├─ Key-value caching? → Redis
        ├─ Time series? → TimescaleDB/InfluxDB
        ├─ Search focused? → Elasticsearch
        └─ Graph relationships? → Neo4j
```

---

# SQL Database Mastery

## PostgreSQL - The Gold Standard

### When to Use PostgreSQL

| Use Case | Why PostgreSQL |
|----------|----------------|
| Complex queries | Advanced JOINs, CTEs, Window Functions |
| Data integrity | ACID compliance, Foreign keys, Constraints |
| JSON/JSONB | Native JSON support with indexing |
| Full-text search | Built-in tsvector, GIN indexes |
| Geospatial data | PostGIS extension |
| Custom functions | PL/pgSQL, PL/Python, PL/V8 |
| RLS needs | Row-Level Security for multi-tenant |

### Schema Design Patterns

```sql
-- Naming conventions
CREATE TABLE users (           -- Plural, snake_case
  user_id UUID PRIMARY KEY,    -- Descriptive PK
  email_address TEXT UNIQUE,   -- Descriptive column
  created_at TIMESTAMPTZ,      -- Timestamps with timezone
  updated_at TIMESTAMPTZ
);

-- Primary key strategies
-- 1. UUID v4 - Random, good for distributed
id UUID PRIMARY KEY DEFAULT gen_random_uuid()

-- 2. UUID v7 - Time-sorted, better for indexes
-- Requires: CREATE EXTENSION IF NOT EXISTS pgcrypto;
id UUID PRIMARY KEY DEFAULT uuid_generate_v7()

-- 3. Serial/Auto-increment - Simple, sequential
id SERIAL PRIMARY KEY

-- 4. Custom business keys
order_id TEXT PRIMARY KEY DEFAULT 'ORD-' || TO_CHAR(NOW(), 'YYYYMMDD') || '-' || LPAD(nextval('order_seq')::TEXT, 6, '0')

-- Foreign keys with proper actions
CREATE TABLE orders (
  order_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  status order_status NOT NULL DEFAULT 'pending',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ON DELETE options:
-- CASCADE: Delete children when parent deleted
-- SET NULL: Set FK to NULL (column must be nullable)
-- SET DEFAULT: Set to default value
-- RESTRICT: Prevent deletion (default)
-- NO ACTION: Similar to RESTRICT, deferrable
```

### Indexing Strategies

```sql
-- B-tree index (default) - equality and range
CREATE INDEX idx_users_email ON users(email_address);

-- Composite index (order matters!)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
-- Good for: WHERE user_id = $1 AND status = $2
-- Also: WHERE user_id = $1
-- NOT: WHERE status = $2 (leading column needed)

-- Partial index - smaller, faster
CREATE INDEX idx_active_users_email ON users(email_address) WHERE is_active = true;
CREATE INDEX idx_recent_orders ON orders(created_at) WHERE created_at > NOW() - INTERVAL '1 year';

-- Unique index for data integrity
CREATE UNIQUE INDEX idx_users_email ON users(email_address);

-- Covering index (INCLUDE for index-only scans)
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at)
  INCLUDE (status, total);

-- Expression index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- GIN index for JSONB/full-text
CREATE INDEX idx_settings_config ON settings USING GIN(config);
CREATE INDEX idx_articles_search ON articles USING GIN(search_vector);

-- HNSW index for vector similarity
CREATE INDEX idx_docs_embedding ON documents
  USING hnsw (embedding vector_cosine_ops);

-- Concurrent index creation (no locking)
CREATE INDEX CONCURRENTLY idx_large_column ON large_table(column);
```

### Query Optimization

```sql
-- Analyze query performance
EXPLAIN ANALYZE
SELECT u.*, o.*
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE u.email = 'user@example.com';

-- Common anti-patterns

-- 1. N+1 query problem
-- Bad: Multiple queries
SELECT * FROM posts WHERE user_id = $1;
-- For each post: SELECT * FROM comments WHERE post_id = $1;

-- Good: Single query with aggregation
SELECT
  p.*,
  jsonb_agg(c) AS comments
FROM posts p
LEFT JOIN comments c ON c.post_id = p.id
WHERE p.user_id = $1
GROUP BY p.id;

-- 2. Functions in WHERE prevent index use
-- Bad: WHERE LOWER(email) = 'test@example.com'
-- Fix: Store lowercased, or use expression index

-- 3. Large OFFSET is slow
-- Bad: OFFSET 100000 LIMIT 10
-- Good: Cursor-based pagination
SELECT * FROM posts
WHERE id > (
  SELECT id FROM posts ORDER BY id LIMIT 1 OFFSET 100000
)
ORDER BY id
LIMIT 10;

-- 4. OR conditions inefficient
-- Bad: WHERE email = $1 OR username = $1
-- Good: Separate queries or UNION

-- 5. Missing indexes on foreign keys
-- Check with EXPLAIN - if Seq Scan on join, add index
```

### Advanced PostgreSQL Features

```sql
-- JSONB operations
CREATE TABLE settings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID,
  config JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_settings_config ON settings USING GIN(config);

-- Query operators
SELECT * FROM settings WHERE config->>'theme' = 'dark';
SELECT * FROM settings WHERE config @> '{"theme": "dark"}';
SELECT * FROM settings WHERE config ? 'theme';

-- Update JSONB
UPDATE settings
SET config = jsonb_set(config, '{theme}', '"light"')
WHERE id = $1;

-- Array operations
CREATE TABLE posts (
  id UUID PRIMARY KEY,
  tags TEXT[] DEFAULT '{}'
);

CREATE INDEX idx_posts_tags ON posts USING GIN(tags);

SELECT * FROM posts WHERE 'tech' = ANY(tags);
SELECT * FROM posts WHERE tags @> ARRAY['tech', 'programming'];

-- Window functions
SELECT
  id,
  user_id,
  created_at,
  ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at) AS row_num,
  SUM(amount) OVER (
    ORDER BY created_at
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_total
FROM orders;

-- Recursive CTE for hierarchy
WITH RECURSIVE org_tree AS (
  SELECT id, name, parent_id, 1 AS level
  FROM organizations
  WHERE parent_id IS NULL
  UNION ALL
  SELECT o.id, o.name, o.parent_id, ot.level + 1
  FROM organizations o
  INNER JOIN org_tree ot ON o.parent_id = ot.id
)
SELECT * FROM org_tree ORDER BY level, name;
```

---

# MySQL Mastery

### When to Use MySQL

| Use Case | Why MySQL |
|----------|-----------|
| Simple web apps | Easy setup, widely supported |
| Read-heavy | Excellent read performance |
| ACID needed | InnoDB engine |
| Budget hosting | Widely available |

### MySQL-Specific Syntax

```sql
-- Engine selection
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Index options
CREATE INDEX idx_email ON users(email);
CREATE FULLTEXT INDEX idx_content ON articles(content);

-- JSON operations (MySQL 5.7+)
CREATE TABLE settings (
  id INT AUTO_INCREMENT PRIMARY KEY,
  config JSON
);

SELECT * FROM settings WHERE JSON_EXTRACT(config, '$.theme') = 'dark';
SELECT JSON_SET(config, '$.theme', 'light') FROM settings WHERE id = 1;

-- Partitioning for large tables
CREATE TABLE orders (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  user_id INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_user_created (user_id, created_at)
) PARTITION BY RANGE (YEAR(created_at)) (
  PARTITION p2023 VALUES LESS THAN (2024),
  PARTITION p2024 VALUES LESS THAN (2025),
  PARTITION p2025 VALUES LESS THAN (2026),
  PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

---

# NoSQL Database Mastery

## MongoDB - Document Database

### When to Use MongoDB

| Use Case | Why MongoDB |
|----------|-------------|
| Flexible schema | Rapid iteration, varying document structures |
| Hierarchical data | Nested documents, no joins needed |
| High write volume | Document-level locking |
| Geospatial queries | Built-in geo operators |
| Real-time analytics | Aggregation pipeline |

### Schema Design Patterns

```javascript
// Embedded vs Reference
// Embedded - for 1:few, data used together
db.users.insertOne({
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  addresses: [
    { street: "123 Main", city: "Bangkok", country: "Thailand", isDefault: true }
  ]
});

// Reference - for 1:many, large arrays, independent access
db.orders.insertOne({
  _id: ObjectId("..."),
  userId: ObjectId("..."),
  items: [
    { productId: ObjectId("..."), quantity: 2, price: 100 }
  ],
  status: "pending",
  createdAt: new Date()
});

// Indexes
db.users.createIndex({ email: 1 }, { unique: true });
db.orders.createIndex({ userId: 1, createdAt: -1 });
db.locations.createIndex({ loc: "2dsphere" });

// Aggregation pipeline
db.orders.aggregate([
  { $match: { status: "completed", createdAt: { $gte: new Date("2024-01-01") } } },
  { $group: {
      _id: "$userId",
      totalSpent: { $sum: "$total" },
      orderCount: { $sum: 1 },
      avgOrderValue: { $avg: "$total" }
  }},
  { $sort: { totalSpent: -1 } },
  { $limit: 10 }
]);

// Transaction (multi-document)
const session = db.getMongo().startSession();
session.startTransaction();
try {
  db.orders.insertOne({ userId, items, total }, { session });
  db.users.updateOne(
    { _id: userId },
    { $inc: { orderCount: 1, totalSpent: total } },
    { session }
  );
  session.commitTransaction();
} catch (error) {
  session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```

## Redis - Cache & Message Broker

### When to Use Redis

| Use Case | Why Redis |
|----------|-----------|
| Caching | In-memory, fast reads |
| Sessions | TTL support, fast access |
| Rate limiting | Atomic operations |
 Pub/sub | Real-time messaging |
| Leaderboards | Sorted sets |

### Common Patterns

```bash
# String - simple cache
SET user:1001 '{"name":"John","email":"john@example.com"}' EX 3600
GET user:1001

# Hash - object storage
HSET user:1001 name "John" email "john@example.com"
HGET user:1001 name
HGETALL user:1001

# List - queue
LPUSH jobs:pending '{"id":1,"type":"process"}'
RPOP jobs:pending

# Set - unique items
SADD user:1001:tags "tech" "news"
SMEMBERS user:1001:tags
SISMEMBER user:1001:tags "tech"

# Sorted Set - leaderboard
ZADD leaderboard 1500 "player1" 2000 "player2" 1800 "player3"
ZREVRANGE leaderboard 0 9 WITHSCORES
ZINCRBY leaderboard 100 "player1"

# Bitmap - analytics
SETBIT user:activity:20240126 1001 1
BITCOUNT user:activity:20240126

# HyperLogLog - unique counting
PFADD page:visitors "user1" "user2" "user3"
PFCOUNT page:visitors

# Lua script for atomic operations
EVAL "
  local current = redis.call('GET', KEYS[1])
  if tonumber(current) >= tonumber(ARGV[1]) then
    redis.call('DECRBY', KEYS[1], ARGV[1])
    return 1
  end
  return 0
" 1 balance:1001 100
```

---

# Database Architecture Patterns

## Connection Pooling

```
┌─────────────┐
│  Application│
└──────┬──────┘
       │
┌──────▼────────────────┐
│  Connection Pool      │
│  - Min: 10 connections│
│  - Max: 50 connections│
│  - Timeout: 30s       │
└──────┬────────────────┘
       │
┌──────▼────────────────┐
│  PgBouncer (optional) │
│  Transaction mode     │
└──────┬────────────────┘
       │
┌──────▼────────────────┐
│  Database Server      │
│  PostgreSQL 15+       │
└───────────────────────┘
```

## Replication Strategies

### Primary-Replica
```
         ┌────────────┐
         │  Primary   │ ← Writes
         └─────┬──────┘
               │
     ┌─────────┼─────────┐
     ▼         ▼         ▼
┌────────┐ ┌────────┐ ┌────────┐
│Replica1│ │Replica2│ │Replica3│ ← Reads
└────────┘ └────────┘ └────────┘
```

### Sharding
```
┌───────────────┐
│   Router      │
│ (consistent)  │
└───────┬───────┘
    ┌───┴───┬─────────┐
    ▼       ▼         ▼
┌────────┐┌────────┐┌────────┐
│ Shard 0││ Shard 1││ Shard 2│
│(0-33%) ││(34-66%)││(67-100%)│
└────────┘└────────┘└────────┘
```

## Multi-Database Patterns

```
┌─────────────────────────────────────────────┐
│              Application Layer              │
└─────────────────────────────────────────────┘
         │             │             │
         ▼             ▼             ▼
┌────────────┐ ┌────────────┐ ┌────────────┐
│ PostgreSQL │ │   Redis    │ │  MongoDB   │
│            │ │            │ │            │
│ Primary DB │ │   Cache    │ │ Analytics  │
│ Users,     │ │ Sessions   │ │ Logs,      │
│ Orders,    │ │ Rate Limit │ │ Events     │
│ Jobs       │ │ Pub/Sub    │ │            │
└────────────┘ └────────────┘ └────────────┘
```

---

# Migration Strategies

## Database Migration Best Practices

```sql
-- Migration file naming: YYYYMMDDHHMMSS_description.sql
-- 20240127000000_add_user_profiles.sql

BEGIN;

-- 1. Idempotent operations
CREATE TABLE IF NOT EXISTS user_profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name TEXT,
  avatar_url TEXT
);

-- 2. Add columns safely
ALTER TABLE users ADD COLUMN IF NOT EXISTS profile_id UUID;

-- 3. Create indexes concurrently in production
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_users_profile
  ON users(profile_id) WHERE profile_id IS NOT NULL;

-- 4. Add constraints with checks
ALTER TABLE orders
  ADD CONSTRAINT check_status
  CHECK (status IN ('pending', 'processing', 'completed', 'cancelled'));

-- 5. Always comment schema changes
COMMENT ON TABLE user_profiles IS 'Extended user profile information';
COMMENT ON COLUMN user_profiles.avatar_url IS 'URL to user avatar image';

COMMIT;
```

## Zero-Downtime Migrations

```sql
-- Step 1: Add new column (nullable)
ALTER TABLE users ADD COLUMN new_email TEXT;

-- Step 2: Backfill data in batches
UPDATE users
SET new_email = old_email
WHERE new_email IS NULL
LIMIT 1000;

-- Repeat until all rows migrated

-- Step 3: Add index
CREATE INDEX CONCURRENTLY idx_users_new_email ON users(new_email);

-- Step 4: Update application to write to both columns

-- Step 5: Verify data integrity

-- Step 6: Switch reads to new column

-- Step 7: Remove old column
ALTER TABLE users DROP COLUMN old_email;
```

---

# World-Class Resources

## Official Documentation
- PostgreSQL: https://www.postgresql.org/docs/
- MySQL: https://dev.mysql.com/doc/
- MongoDB: https://www.mongodb.com/docs/
- Redis: https://redis.io/docs/

## Tools
- pgAdmin: PostgreSQL GUI
- DBeaver: Universal database tool
- DataGrip: JetBrains DB IDE
- RedisInsight: Redis GUI
- Compass: MongoDB GUI

## Learning
- PostgreSQL Tutorial: https://www.postgresqltutorial.com/
- High Performance MySQL: https://www.oreilly.com/library/view/high-performance-mysql/
- MongoDB University: https://university.mongodb.com/
