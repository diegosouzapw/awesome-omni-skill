---
name: nosql-databases
description: Apply NoSQL best practices for MongoDB, Convex, and document databases. Use when designing schemas, writing queries, optimizing performance, or building applications with non-relational databases. Use with database-expert for query optimization and DBA-level tuning (20+ years experience).
---

# NoSQL Databases (MongoDB, Convex, Document Stores)

**Expertise**: Senior database administrator with 20+ years of experience in document stores, key-value systems, and non-relational data modeling. Focus on query optimization, indexing strategy, and data access best practices.

---

## General NoSQL Principles

### Document Design
- **Embed vs Reference**: Embed when data is always read together and rarely grows unbounded; reference when data is shared, large, or updated independently
- **Avoid unbounded arrays**: Documents with arrays that grow without limit cause performance degradation; use separate collections with references
- **Denormalize for read patterns**: Optimize for how data is read; duplicate when it improves query performance and consistency is acceptable

### Query Patterns
- **Index every query path**: Queries without indexes cause full collection scans; at scale, indexed queries are orders of magnitude faster
- **Project only needed fields**: Reduce network and memory by projecting only required fields (`projection` in MongoDB, selective fields in Convex)
- **Paginate large result sets**: Never `.collect()` or `.find()` without limits when result sets can be large (e.g. >1000 documents)

### Consistency
- **Understand read-your-writes**: Document stores often offer eventual consistency; use appropriate read concern when strong consistency is required
- **Design for idempotency**: Retries and eventual consistency make duplicate operations possible; design mutations to be idempotent

---

## MongoDB

### Index Types and When to Use

| Type | Use Case | Example |
|------|----------|---------|
| **Single-field** | Equality, sort on one field | `{ userId: 1 }` |
| **Compound** | Multi-field queries; order matters | `{ channel: 1, createdAt: -1 }` |
| **Multikey** | Arrays (one index entry per array element) | `{ tags: 1 }` |
| **Text** | Full-text search | `{ content: "text" }` |
| **Geospatial** | Location queries | `2dsphere`, `2d` |

### Index Rules

1. **Index fields in WHERE, sort, and projection**—avoid full collection scans
2. **Compound index order**: equality → sort → range; put most selective fields first
   ```javascript
   // Good for db.collection.find({ channel: "x" }).sort({ createdAt: -1 })
   db.collection.createIndex({ channel: 1, createdAt: -1 });
   ```
3. **Covered queries**: When query + projection use only indexed fields, MongoDB reads only the index (no document fetch)
4. **Avoid low-selectivity operators**: `$nin`, `$ne`, `$exists: false` often match large portions of the index
5. **Limit indexes per collection**: Max 64 indexes; each index adds write cost—measure before adding

### Aggregation Pipeline Optimization
- Use `$match` and `$project` early to reduce documents and fields early in the pipeline
- Use `$indexStats` and `$queryStats` to analyze query patterns and index usage
- Prefer `$lookup` with `pipeline` and `let` for complex joins; avoid unbounded `$lookup` on large collections

### Explain and Profiling
```javascript
db.collection.find({ userId: "x" }).explain("executionStats");
// Check: stage "IXSCAN" (index scan) vs "COLLSCAN" (full scan)
// Review: docsExamined, nReturned, executionTimeMillis
```

### Security
- Use parameterized queries; never concatenate user input into queries
- Apply principle of least privilege for database users
- Validate and sanitize `$where` and aggregation `$function` inputs

---

## Convex

### Schema and Indexes

Indexes are defined in the schema; every query should use an index via `.withIndex()`:

```typescript
// schema.ts
defineSchema({
  messages: defineTable({
    channel: v.string(),
    userId: v.id("users"),
    text: v.string(),
    createdAt: v.number(),
  })
    .index("by_channel", ["channel"])
    .index("by_channel_created", ["channel", "createdAt"])
    .index("by_user", ["userId"]),
});
```

### Query Best Practices

1. **Use `.withIndex()` instead of `.filter()`**: Index-based queries are efficient; `.filter()` scans the table
   ```typescript
   // Good: uses index
   const messages = await ctx.db.query("messages").withIndex("by_channel", q => q.eq("channel", channelId)).collect();
   // Avoid: full table scan
   const messages = await ctx.db.query("messages").filter(q => q.eq(q.field("channel"), channelId)).collect();
   ```

2. **Use `.withSearchIndex()` for full-text**: When you need search, define and use search indexes

3. **Paginate with `.paginate()`**: For large result sets, use pagination; avoid `.collect()` on unbounded queries

4. **Staged indexes for large tables**: When adding indexes to tables with substantial data, use staged indexes to avoid slow backfill during deployment

### Index Removal
- Ensure an index is completely unused before removing it; deployments will delete unused indexes

---

## Other Document Stores (Firestore, DynamoDB, etc.)

### Firestore
- Composite indexes for multi-field queries; define in Firebase Console or `firestore.indexes.json`
- Batch reads with `getAll()` to reduce round trips
- Use `limit()` and `startAfter()` for pagination

### DynamoDB
- Design for single-table access patterns; partition key + sort key define access
- Use GSIs (Global Secondary Indexes) for alternate query patterns
- Avoid scans; use `Query` with key conditions

---

## Performance Checklist

- [ ] Every query path has a supporting index
- [ ] No full collection/table scans in hot paths (verify with explain/profiler)
- [ ] Projections limit returned fields
- [ ] Large result sets use pagination
- [ ] Unbounded arrays avoided in document design
- [ ] Read/write patterns inform embedding vs referencing
- [ ] Mutations are idempotent where retries are possible
