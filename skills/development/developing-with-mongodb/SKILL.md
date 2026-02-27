---
name: Developing with MongoDB
description: The agent implements MongoDB NoSQL database solutions with document modeling, aggregation pipelines, and Mongoose ODM. Use when building document-based applications, designing schemas, writing aggregations, or implementing NoSQL patterns.
---

# Developing with MongoDB

## Quick Start

```typescript
// Schema with Mongoose
import mongoose, { Schema, Document } from 'mongoose';

interface IUser extends Document {
  email: string;
  profile: { firstName: string; lastName: string };
  createdAt: Date;
}

const userSchema = new Schema<IUser>({
  email: { type: String, required: true, unique: true, lowercase: true },
  profile: {
    firstName: { type: String, required: true },
    lastName: { type: String, required: true },
  },
}, { timestamps: true });

userSchema.index({ email: 1 });
export const User = mongoose.model<IUser>('User', userSchema);
```

## Features

| Feature | Description | Guide |
|---------|-------------|-------|
| Document Schema | Type-safe schema design with Mongoose | Embed related data, use references for large collections |
| CRUD Operations | Create, read, update, delete with type safety | Use `findById`, `findOne`, `updateOne`, `deleteOne` |
| Aggregation Pipelines | Complex data transformations and analytics | Chain `$match`, `$group`, `$lookup`, `$project` stages |
| Indexing | Query optimization with proper indexes | Create compound indexes matching query patterns |
| Transactions | Multi-document ACID operations | Use sessions for operations requiring atomicity |
| Change Streams | Real-time data change notifications | Watch collections for inserts, updates, deletes |

## Common Patterns

### Repository Pattern with Pagination

```typescript
async findPaginated(filter: FilterQuery<IUser>, page = 1, limit = 20) {
  const [data, total] = await Promise.all([
    User.find(filter).sort({ createdAt: -1 }).skip((page - 1) * limit).limit(limit),
    User.countDocuments(filter),
  ]);
  return { data, pagination: { page, limit, total, totalPages: Math.ceil(total / limit) } };
}
```

### Aggregation Pipeline

```typescript
const stats = await Order.aggregate([
  { $match: { status: 'completed', createdAt: { $gte: startDate } } },
  { $group: { _id: '$userId', total: { $sum: '$amount' }, count: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 10 },
]);
```

### Transaction for Order Creation

```typescript
const session = await mongoose.startSession();
try {
  session.startTransaction();
  await Product.updateOne({ _id: productId }, { $inc: { stock: -quantity } }, { session });
  const order = await Order.create([{ userId, items, total }], { session });
  await session.commitTransaction();
  return order[0];
} catch (error) {
  await session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```

## Best Practices

| Do | Avoid |
|----|-------|
| Design schemas based on query patterns | Embedding large arrays in documents |
| Create indexes for frequently queried fields | Using `$where` or mapReduce in production |
| Use `lean()` for read-only queries | Skipping validation on writes |
| Implement pagination for large datasets | Storing large files directly (use GridFS) |
| Set connection pool size appropriately | Hardcoding connection strings |
| Use transactions for multi-document ops | Ignoring index usage in explain plans |
| Add TTL indexes for expiring data | Creating too many indexes (write overhead) |
