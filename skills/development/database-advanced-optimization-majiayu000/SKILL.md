---
name: database-advanced-optimization
category: database
difficulty: advanced
estimated_time: 60 minutes
prerequisites:
- Basic SQL knowledge
- Database design fundamentals
- Understanding of indexes and query execution
tags:
- postgresql
- mongodb
- redis
- performance
- optimization
- scaling
- caching
- sharding
learning_objectives:
- Master database-specific optimization techniques
- Implement effective caching strategies
- Design scalable database architectures
- Optimize query performance across multiple database systems
- Configure connection pooling and monitoring
version: 1.0.0
last_updated: 2025-01-17
description: 'Use SQL (PostgreSQL) when:'
---


# Database Advanced Optimization

## Level 1: Quick Reference (5 minutes)

### Database Selection Guide

**Use SQL (PostgreSQL) when:**

- ACID compliance is critical
- Complex joins and transactions required
- Data has clear relational structure
- Strong consistency needed
- Rich query capabilities required

**Use NoSQL (MongoDB) when:**

- Flexible schema needed
- Horizontal scaling is priority
- Document-oriented data model fits
- High write throughput required
- Eventual consistency acceptable

**Use In-Memory (Redis) when:**

- Sub-millisecond latency required
- Caching layer needed
- Real-time features (pub/sub, streams)
- Session management
- Rate limiting or counters

### Common Optimization Patterns

```sql
-- PostgreSQL: Create covering index
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at)
INCLUDE (status, total_amount);

-- PostgreSQL: Analyze query plan
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders WHERE user_id = 123;
```

```javascript
// MongoDB: Create compound index
db.orders.createIndex({ userId: 1, createdAt: -1 }, { background: true });

// MongoDB: Use aggregation pipeline efficiently
db.orders.aggregate([
  { $match: { status: "pending" } },
  { $sort: { createdAt: -1 } },
  { $limit: 100 }
]);
```

```python
# Redis: Implement cache-aside pattern
def get_user(user_id):
    cache_key = f"user:{user_id}"
    user = redis.get(cache_key)

    if user is None:
        user = db.query("SELECT * FROM users WHERE id = %s", user_id)
        redis.setex(cache_key, 3600, json.dumps(user))

    return json.loads(user)
```

### Essential Optimization Checklist

#### PostgreSQL

- [ ] Create appropriate indexes (B-tree, GIN, GiST)
- [ ] Analyze query plans with EXPLAIN ANALYZE
- [ ] Configure autovacuum appropriately
- [ ] Set up connection pooling (PgBouncer)
- [ ] Monitor with pg_stat_statements
- [ ] Optimize shared_buffers and work_mem
- [ ] Configure appropriate WAL settings

#### MongoDB

- [ ] Design effective shard keys
- [ ] Create compound indexes for common queries
- [ ] Enable profiler for slow queries
- [ ] Configure replica sets for read scaling
- [ ] Optimize aggregation pipelines
- [ ] Set appropriate write concerns
- [ ] Monitor with MongoDB Compass/Atlas

#### Redis

- [ ] Implement appropriate caching strategy
- [ ] Configure maxmemory and eviction policies
- [ ] Use pipelining for bulk operations
- [ ] Set appropriate TTLs
- [ ] Monitor memory usage and hit rates
- [ ] Configure persistence (RDB/AOF) appropriately
- [ ] Use Redis Cluster for scaling

### Quick Wins

**Immediate Impact:**

1. Add indexes for frequently queried columns
2. Enable query result caching
3. Implement connection pooling
4. Add Redis cache layer for hot data
5. Optimize N+1 queries with batch loading

**Performance Monitoring:**

```bash
# PostgreSQL query stats
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY mean_time DESC LIMIT 10;

# MongoDB profiler
db.setProfilingLevel(1, { slowms: 100 });
db.system.profile.find().sort({ ts: -1 }).limit(10);

# Redis stats
redis-cli INFO stats | grep -E 'keyspace_hits|keyspace_misses'
```

---

## Level 2:
>
> **📚 Full Examples**: See [REFERENCE.md](./REFERENCE.md) for complete code samples, detailed configurations, and production-ready implementations.

 Implementation Guide (45 minutes)

### PostgreSQL Advanced Optimization

#### Index Strategies

PostgreSQL supports multiple index types, each optimized for specific use cases:

**B-tree Indexes (Default)**


*See [REFERENCE.md](./REFERENCE.md#example-0) for complete implementation.*


**GIN Indexes (Full-text and Array Search)**


*See [REFERENCE.md](./REFERENCE.md#example-1) for complete implementation.*


**GiST Indexes (Geometric and Range Data)**


*See [REFERENCE.md](./REFERENCE.md#example-2) for complete implementation.*


#### Query Optimization with EXPLAIN ANALYZE


*See [REFERENCE.md](./REFERENCE.md#example-3) for complete implementation.*


**Understanding EXPLAIN Output:**


*See [REFERENCE.md](./REFERENCE.md#example-4) for complete implementation.*


**Optimization Decisions:**

- Index Scan = Good (using indexes)
- Buffers shared hit = Good (data in cache)
- Nested Loop = Appropriate for small result sets
- Hash Join = Better for large joins

#### VACUUM and Maintenance


*See [REFERENCE.md](./REFERENCE.md#example-5) for complete implementation.*


#### Connection Pooling with PgBouncer


*See [REFERENCE.md](./REFERENCE.md#example-6) for complete implementation.*


**Connection Pooling Best Practices:**

- Use `transaction` mode for most applications
- Set `default_pool_size` = (CPU cores × 2) + effective_spindle_count
- Monitor pool usage: `SHOW POOLS;` in PgBouncer console
- Application connection string: `postgresql://user:pass@pgbouncer:6432/myapp`

#### PostgreSQL Performance Tuning


*See [REFERENCE.md](./REFERENCE.md#example-7) for complete implementation.*


### MongoDB Advanced Optimization

#### Sharding Strategies

**Shard Key Selection:**


*See [REFERENCE.md](./REFERENCE.md#example-8) for complete implementation.*


**Shard Key Patterns:**

1. **Range-based Sharding** (ordered data):

```javascript
// Good for time-series data
sh.shardCollection("analytics.events", { "date": 1, "userId": 1 })

// Queries benefit from targeted routing
db.events.find({ date: ISODate("2025-01-17"), userId: 123 })
```

2. **Hash-based Sharding** (random distribution):

```javascript
// Good for even distribution, poor for range queries
sh.shardCollection("users.profiles", { "_id": "hashed" })

// All shards queried for range
db.profiles.find({ _id: { $gt: 1000, $lt: 2000 } })
```

3. **Zone Sharding** (geographic distribution):


*See [REFERENCE.md](./REFERENCE.md#example-11) for complete implementation.*


#### Index Optimization


*See [REFERENCE.md](./REFERENCE.md#example-12) for complete implementation.*


**Index Analysis:**


*See [REFERENCE.md](./REFERENCE.md#example-13) for complete implementation.*


#### Aggregation Pipeline Optimization


*See [REFERENCE.md](./REFERENCE.md#example-14) for complete implementation.*


#### Replica Set Configuration


*See [REFERENCE.md](./REFERENCE.md#example-15) for complete implementation.*


### Redis Advanced Patterns

#### Caching Strategies

**1. Cache-Aside (Lazy Loading)**


*See [REFERENCE.md](./REFERENCE.md#example-16) for complete implementation.*


**2. Write-Through Caching**


*See [REFERENCE.md](./REFERENCE.md#example-17) for complete implementation.*


**3. Write-Behind (Write-Back) Caching**


*See [REFERENCE.md](./REFERENCE.md#example-18) for complete implementation.*


**4. Read-Through Caching**


*See [REFERENCE.md](./REFERENCE.md#example-19) for complete implementation.*


#### Redis Data Structures and Use Cases


*See [REFERENCE.md](./REFERENCE.md#example-20) for complete implementation.*


#### Pub/Sub and Streams

**Pub/Sub Pattern:**


*See [REFERENCE.md](./REFERENCE.md#example-21) for complete implementation.*


**Redis Streams (Preferred for Reliable Messaging):**


*See [REFERENCE.md](./REFERENCE.md#example-22) for complete implementation.*


#### Redis Cluster Configuration


*See [REFERENCE.md](./REFERENCE.md#example-23) for complete implementation.*


**Redis Configuration (redis.conf):**


*See [REFERENCE.md](./REFERENCE.md#example-24) for complete implementation.*


### Query Optimization Techniques

#### Solving the N+1 Problem

**Bad: N+1 Queries**


*See [REFERENCE.md](./REFERENCE.md#example-25) for complete implementation.*


**Good: Join or Batch Loading**


*See [REFERENCE.md](./REFERENCE.md#example-26) for complete implementation.*


#### Database Scaling Strategies

**1. Vertical Scaling (Scale Up)**

- Increase CPU, RAM, storage
- Simple but has limits
- Good for initial growth

**2. Horizontal Scaling (Scale Out)**

**Read Replicas:**


*See [REFERENCE.md](./REFERENCE.md#example-27) for complete implementation.*


**Sharding:**


*See [REFERENCE.md](./REFERENCE.md#example-28) for complete implementation.*


#### Connection Pooling Best Practices

**HikariCP (Java):**


*See [REFERENCE.md](./REFERENCE.md#example-29) for complete implementation.*


**Python (SQLAlchemy):**


*See [REFERENCE.md](./REFERENCE.md#example-30) for complete implementation.*


**Node.js (pg-pool):**


*See [REFERENCE.md](./REFERENCE.md#example-31) for complete implementation.*


### Database Monitoring and Profiling

#### PostgreSQL Monitoring

**pg_stat_statements Extension:**


*See [REFERENCE.md](./REFERENCE.md#example-32) for complete implementation.*


**Key Metrics to Monitor:**


*See [REFERENCE.md](./REFERENCE.md#example-33) for complete implementation.*


#### MongoDB Profiling

**Enable Profiler:**


*See [REFERENCE.md](./REFERENCE.md#example-34) for complete implementation.*


**Analyze Slow Queries:**


*See [REFERENCE.md](./REFERENCE.md#example-35) for complete implementation.*


**Server Status Metrics:**


*See [REFERENCE.md](./REFERENCE.md#example-36) for complete implementation.*


#### Redis Monitoring

**INFO Command:**


*See [REFERENCE.md](./REFERENCE.md#example-37) for complete implementation.*


**Key Metrics:**


*See [REFERENCE.md](./REFERENCE.md#example-38) for complete implementation.*


**Slow Log:**


*See [REFERENCE.md](./REFERENCE.md#example-39) for complete implementation.*


### Backup and Recovery Strategies

#### PostgreSQL Backup

**Physical Backup (pg_basebackup):**

```bash
# Full backup
pg_basebackup -h localhost -U postgres -D /backup/pg_data -Fp -Xs -P

# Compressed backup
pg_basebackup -h localhost -U postgres -D /backup/pg_data -Ft -z -P
```

**Logical Backup (pg_dump):**


*See [REFERENCE.md](./REFERENCE.md#example-41) for complete implementation.*


**Restore:**


*See [REFERENCE.md](./REFERENCE.md#example-42) for complete implementation.*


**Point-in-Time Recovery (PITR):**


*See [REFERENCE.md](./REFERENCE.md#example-43) for complete implementation.*


#### MongoDB Backup

**mongodump:**


*See [REFERENCE.md](./REFERENCE.md#example-44) for complete implementation.*


**mongorestore:**


*See [REFERENCE.md](./REFERENCE.md#example-45) for complete implementation.*


**Filesystem Snapshots (Replica Set):**


*See [REFERENCE.md](./REFERENCE.md#example-46) for complete implementation.*


#### Redis Backup

**RDB (Snapshot):**


*See [REFERENCE.md](./REFERENCE.md#example-47) for complete implementation.*


**AOF (Append-Only File):**

```bash
# Enable AOF (redis.conf)
appendonly yes
appendfsync everysec

# Rewrite AOF
redis-cli BGREWRITEAOF

# Backup AOF file
cp /var/lib/redis/appendonly.aof /backup/appendonly-$(date +%Y%m%d).aof
```

**Restore:**

```bash
# Stop Redis
systemctl stop redis

# Restore RDB or AOF
cp /backup/dump.rdb /var/lib/redis/
# or
cp /backup/appendonly.aof /var/lib/redis/

# Start Redis
systemctl start redis
```

---

## Level 3: Deep Dive Resources (10 minutes)

### Official Documentation

**PostgreSQL:**

- [PostgreSQL Performance Tips](https://www.postgresql.org/docs/current/performance-tips.html)
- [pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html)
- [EXPLAIN Documentation](https://www.postgresql.org/docs/current/sql-explain.html)
- [PgBouncer Documentation](https://www.pgbouncer.org/usage.html)

**MongoDB:**

- [MongoDB Performance Best Practices](https://www.mongodb.com/docs/manual/administration/analyzing-mongodb-performance/)
- [Sharding Guide](https://www.mongodb.com/docs/manual/sharding/)
- [Index Strategies](https://www.mongodb.com/docs/manual/applications/indexes/)
- [Aggregation Pipeline Optimization](https://www.mongodb.com/docs/manual/core/aggregation-pipeline-optimization/)

**Redis:**

- [Redis Documentation](https://redis.io/documentation)
- [Redis Cluster Tutorial](https://redis.io/docs/manual/scaling/)
- [Redis Persistence](https://redis.io/docs/manual/persistence/)
- [Redis Best Practices](https://redis.io/docs/manual/patterns/)

### Books and Courses

- "High Performance PostgreSQL" by Gregory Smith
- "PostgreSQL Query Optimization" by Henrietta Dombrovskaya
- "MongoDB: The Definitive Guide" by Shannon Bradshaw
- "Redis in Action" by Josiah Carlson
- "Designing Data-Intensive Applications" by Martin Kleppmann

### Tools and Utilities

**PostgreSQL:**

- pgAdmin 4 - Database management
- pg_top - Real-time monitoring
- pgBadger - Log analyzer
- pgtune - Configuration generator

**MongoDB:**

- MongoDB Compass - GUI client
- MongoDB Atlas - Managed service
- mongostat/mongotop - Command-line monitoring
- Studio 3T - Advanced GUI

**Redis:**

- RedisInsight - GUI client
- redis-cli - Command-line interface
- redis-stat - Real-time monitoring
- redis-rdb-tools - RDB file analysis

### Practice Exercises

1. **Index Optimization Challenge**: Given a slow query, design optimal indexes
2. **Sharding Strategy**: Design shard key for multi-tenant application
3. **Cache Implementation**: Build cache-aside pattern with TTL and invalidation
4. **Query Analysis**: Use EXPLAIN to optimize complex join queries
5. **Backup Strategy**: Design comprehensive backup/recovery plan

### Related Skills

- [Database Design Fundamentals](../design-fundamentals/SKILL.md)
- [API Performance Optimization](../../backend/api-performance/SKILL.md)
- [System Architecture Patterns](../../architecture/system-patterns/SKILL.md)
- [Monitoring and Observability](../../devops/monitoring/SKILL.md)

## Examples

### Basic Usage

```python
// TODO: Add basic example for advanced-optimization
// This example demonstrates core functionality
```

### Advanced Usage

```python
// TODO: Add advanced example for advanced-optimization
// This example shows production-ready patterns
```

### Integration Example

```python
// TODO: Add integration example showing how advanced-optimization
// works with other systems and services
```

See `examples/advanced-optimization/` for complete working examples.

## Integration Points

This skill integrates with:

### Upstream Dependencies

- **Tools**: Common development tools and frameworks
- **Prerequisites**: Basic understanding of general concepts

### Downstream Consumers

- **Applications**: Production systems requiring advanced-optimization functionality
- **CI/CD Pipelines**: Automated testing and deployment workflows
- **Monitoring Systems**: Observability and logging platforms

### Related Skills

- See other skills in this category

### Common Integration Patterns

1. **Development Workflow**: How this skill fits into daily development
2. **Production Deployment**: Integration with production systems
3. **Monitoring & Alerting**: Observability integration points

## Common Pitfalls

### Pitfall 1: Insufficient Testing

**Problem:** Not testing edge cases and error conditions leads to production bugs

**Solution:** Implement comprehensive test coverage including:

- Happy path scenarios
- Error handling and edge cases
- Integration points with external systems

**Prevention:** Enforce minimum code coverage (80%+) in CI/CD pipeline

### Pitfall 2: Hardcoded Configuration

**Problem:** Hardcoding values makes applications inflexible and environment-dependent

**Solution:** Use environment variables and configuration management:

- Separate config from code
- Use environment-specific configuration files
- Never commit secrets to version control

**Prevention:** Use tools like dotenv, config validators, and secret scanners

### Pitfall 3: Ignoring Security Best Practices

**Problem:** Security vulnerabilities from not following established security patterns

**Solution:** Follow security guidelines:

- Input validation and sanitization
- Proper authentication and authorization
- Encrypted data transmission (TLS/SSL)
- Regular security audits and updates

**Prevention:** Use security linters, SAST tools, and regular dependency updates

**Best Practices:**

- Follow established patterns and conventions for advanced-optimization
- Keep dependencies up to date and scan for vulnerabilities
- Write comprehensive documentation and inline comments
- Use linting and formatting tools consistently
- Implement proper error handling and logging
- Regular code reviews and pair programming
- Monitor production metrics and set up alerts

---

## Bundled Resources

This skill includes 6 production-ready templates and scripts:

1. **templates/postgres-optimization.sql** - PostgreSQL optimization queries and index strategies
2. **templates/mongodb-sharding.yaml** - MongoDB sharding configuration
3. **templates/redis-caching.py** - Redis caching pattern implementations
4. **scripts/query-profiling.sh** - Automated database query profiling
5. **config/pgbouncer.ini** - PgBouncer connection pooling configuration
6. **templates/monitoring-dashboard.json** - Grafana dashboard for database metrics

All resources are located in the `skills/database/advanced-optimization/` directory.
