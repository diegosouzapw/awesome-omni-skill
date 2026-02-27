---
name: NoSQL Patterns
description: "Các mẫu tối ưu cho cơ sở dữ liệu NoSQL (MongoDB, Redis và hệ phân tán đa)."
category: Database
difficulty: Intermediate
last_updated: 2026-01-16
---

# NoSQL Patterns

Expert patterns for working with NoSQL databases (MongoDB, Redis, DynamoDB, etc.)

---

## When to Use This Skill

- MongoDB document database design
- Redis caching and data structures
- DynamoDB single-table design
- NoSQL performance optimization
- Choosing SQL vs NoSQL

---

## Content Map

### MongoDB
- **mongodb-patterns.md** - Document modeling
- Schema design patterns
- Indexing strategies
- Aggregation pipelines
- Transactions

### Redis
- **redis-patterns.md** - Caching strategies
- Data structures (String, Hash, List, Set, Sorted Set)
- Pub/Sub patterns
- Session storage
- Rate limiting

### DynamoDB
- **dynamodb-patterns.md** - Single-table design
- Partition & sort keys
- GSI/LSI strategies
- Query optimization

### Performance
- **performance.md** - Indexing best practices
- Query optimization
- Sharding strategies
- Connection pooling

---

## Quick Reference

### MongoDB Document Design
```javascript
// ✅ Good - Embedded for 1-to-few
{
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  addresses: [
    { street: "123 Main", city: "NYC", zip: "10001" },
    { street: "456 Oak", city: "LA", zip: "90001" }
  ]
}

// ✅ Good - Referenced for 1-to-many
{
  _id: ObjectId("..."),
  title: "Post Title",
  authorId: ObjectId("..."), // Reference to User
  comments: [ObjectId("..."), ObjectId("...")] // References
}
```

### MongoDB Aggregation
```javascript
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: {
    _id: "$userId",
    totalSpent: { $sum: "$amount" },
    orderCount: { $sum: 1 }
  }},
  { $sort: { totalSpent: -1 } },
  { $limit: 10 }
]);
```

### Redis Caching Pattern
```javascript
async function getUser(userId) {
  // Try cache first
  const cached = await redis.get(`user:${userId}`);
  if (cached) return JSON.parse(cached);
  
  // Cache miss - fetch from DB
  const user = await db.users.findById(userId);
  
  // Store in cache (TTL: 1 hour)
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
  
  return user;
}
```

### Redis Data Structures
```javascript
// Counter
await redis.incr('page:views');

// Set (unique items)
await redis.sadd('users:online', userId);
await redis.sismember('users:online', userId);

// Sorted Set (leaderboard)
await redis.zadd('leaderboard', score, userId);
await redis.zrevrange('leaderboard', 0, 9); // Top 10
```

---

## Anti-Patterns

❌ **Treating NoSQL like SQL** → Embrace denormalization  
❌ **No indexing** → Critical for performance  
❌ **Deep nesting** → Limit to 2-3 levels  
❌ **Large arrays** → Use references for > 100 items  
❌ **No TTL on cache** → Memory leaks

---

## Best Practices

✅ **Embed for 1-to-few, reference for 1-to-many**  
✅ **Index frequently queried fields**  
✅ **Use aggregation pipelines for complex queries**  
✅ **Implement caching layer (Redis)**  
✅ **Set TTL on cached data**  
✅ **Monitor query performance**

---

## When to Use

**Use NoSQL when:**
- Flexible/evolving schema
- Horizontal scaling needed
- High write throughput
- Document/graph data model fits

**Use SQL when:**
- Complex joins required
- ACID transactions critical
- Structured, relational data
- Strong consistency needed

---

## Official Resources

- [MongoDB Manual](https://docs.mongodb.com/)
- [Redis Documentation](https://redis.io/documentation)
- [DynamoDB Guide](https://docs.aws.amazon.com/dynamodb/)
