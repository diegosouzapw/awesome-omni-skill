---
name: Database
description: ทำงานกับ PostgreSQL และ MongoDB อย่างมีประสิทธิภาพ
---

# Database Skill

## Overview

Skill สำหรับออกแบบ จัดการ และ optimize databases ทั้ง SQL และ NoSQL

---

## PostgreSQL

### Setup with Docker

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

volumes:
  postgres_data:
```

### Schema Design Best Practices

#### Naming Conventions

```sql
-- Tables: plural, snake_case
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Foreign keys: singular_table_id
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    total_amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(50) DEFAULT 'pending'
);

-- Junction tables: table1_table2
CREATE TABLE user_roles (
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    role_id BIGINT REFERENCES roles(id) ON DELETE CASCADE,
    PRIMARY KEY (user_id, role_id)
);
```

#### Indexes

```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Partial index
CREATE INDEX idx_orders_pending ON orders(created_at)
    WHERE status = 'pending';

-- GIN index for JSONB
CREATE INDEX idx_products_metadata ON products USING GIN(metadata);
```

### Common Queries

#### Pagination

```sql
-- Offset pagination (simple but slow for large offsets)
SELECT * FROM users
ORDER BY id
LIMIT 20 OFFSET 40;

-- Cursor/Keyset pagination (better performance)
SELECT * FROM users
WHERE id > :last_id
ORDER BY id
LIMIT 20;
```

#### Full-text Search

```sql
-- Create search vector
ALTER TABLE products ADD COLUMN search_vector tsvector;

UPDATE products SET search_vector =
    to_tsvector('english', name || ' ' || description);

CREATE INDEX idx_products_search ON products USING GIN(search_vector);

-- Search query
SELECT * FROM products
WHERE search_vector @@ to_tsquery('english', 'laptop & gaming');
```

### Performance Optimization

#### EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT u.*, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id;
```

#### Tips

1. **ใช้ Index อย่างเหมาะสม** - ไม่มากไม่น้อย
2. **Avoid SELECT \*** - เลือกเฉพาะ columns ที่ต้องการ
3. **Use Connection Pooling** - PgBouncer หรือ built-in pools
4. **Partition large tables** - โดยเฉพาะ time-series data
5. **Vacuum regularly** - ป้องกัน bloat

---

## MongoDB

### Setup with Docker

```yaml
# docker-compose.yml
services:
  mongodb:
    image: mongo:7
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

### Schema Design Best Practices

#### Embedding vs Referencing

```javascript
// Embedding - When data is accessed together
{
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  addresses: [
    { street: "123 Main St", city: "Bangkok", country: "Thailand" },
    { street: "456 Oak Ave", city: "Chiang Mai", country: "Thailand" }
  ]
}

// Referencing - When data is accessed separately or grows unbounded
// Users collection
{
  _id: ObjectId("user123"),
  name: "John Doe",
  email: "john@example.com"
}

// Orders collection
{
  _id: ObjectId("order456"),
  userId: ObjectId("user123"),
  items: [...],
  total: 1500
}
```

#### Indexes

```javascript
// Single field index
db.users.createIndex({ email: 1 }, { unique: true });

// Compound index
db.orders.createIndex({ userId: 1, createdAt: -1 });

// Text index
db.products.createIndex({ name: "text", description: "text" });

// TTL index (auto-delete after time)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });
```

### Common Operations

#### Aggregation Pipeline

```javascript
// Sales summary by category
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $unwind: "$items" },
  {
    $group: {
      _id: "$items.category",
      totalSales: { $sum: "$items.price" },
      count: { $sum: 1 },
    },
  },
  { $sort: { totalSales: -1 } },
  { $limit: 10 },
]);

// Join collections
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user",
    },
  },
  { $unwind: "$user" },
  {
    $project: {
      _id: 1,
      total: 1,
      "user.name": 1,
      "user.email": 1,
    },
  },
]);
```

#### Transactions

```javascript
const session = client.startSession();

try {
  session.startTransaction();

  await users.updateOne(
    { _id: userId },
    { $inc: { balance: -amount } },
    { session },
  );

  await transactions.insertOne(
    { userId, amount, type: "debit", createdAt: new Date() },
    { session },
  );

  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```

### Performance Optimization

1. **Use projections** - Return only needed fields
2. **Create appropriate indexes** - Check with `explain()`
3. **Use aggregation** - Let DB do the work
4. **Shard for scale** - Distribute data across servers
5. **Use connection pooling** - Reuse connections

---

## Migrations

### PostgreSQL (with Prisma)

```bash
# Create migration
npx prisma migrate dev --name add_users_table

# Apply to production
npx prisma migrate deploy
```

### PostgreSQL (with Flyway)

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- V2__add_role_to_users.sql
ALTER TABLE users ADD COLUMN role VARCHAR(50) DEFAULT 'user';
```

---

## Database Checklist

- [ ] ออกแบบ schema ที่เหมาะสม
- [ ] สร้าง indexes ที่จำเป็น
- [ ] Setup connection pooling
- [ ] Configure backups
- [ ] Setup migrations
- [ ] Monitor performance
- [ ] Plan for scaling
- [ ] Implement soft deletes (optional)
- [ ] Add audit columns (created_at, updated_at)
