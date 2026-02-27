---
name: databases-architecture-skill
description: Master database design (SQL, NoSQL), system architecture, API design (REST, GraphQL), and building scalable systems. Learn PostgreSQL, MongoDB, system design patterns, and enterprise architectures.
---

# Databases & Architecture Skill

Complete guide to designing databases, systems, and APIs that scale.

## Quick Start

### Learning Path

```
Data → Schema → APIs → Architecture
  ↓        ↓        ↓        ↓
SQL     Normalize REST   Microservices
NoSQL  Indexes    GraphQL Patterns
```

### Get Started in 5 Steps

1. **SQL Fundamentals** (2-3 weeks)
   - SELECT, INSERT, UPDATE, DELETE
   - Joins and aggregations

2. **Database Design** (3-4 weeks)
   - Normalization
   - Entity-relationship modeling
   - Indexing

3. **NoSQL Databases** (2-3 weeks)
   - Document stores (MongoDB)
   - Key-value (Redis)
   - When to use each

4. **API Design** (3-4 weeks)
   - REST principles
   - GraphQL basics
   - Error handling

5. **System Architecture** (ongoing)
   - Scalability patterns
   - Caching strategies
   - Distributed systems

---

## SQL Databases

### **SQL Fundamentals**

```sql
-- CREATE TABLE
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE,
  age INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- INSERT
INSERT INTO users (name, email, age)
VALUES ('Alice', 'alice@example.com', 25);

-- SELECT (basic)
SELECT * FROM users;
SELECT name, email FROM users;

-- WHERE (filtering)
SELECT * FROM users WHERE age > 25;
SELECT * FROM users WHERE age >= 25 AND age <= 35;

-- LIKE (pattern matching)
SELECT * FROM users WHERE name LIKE 'A%';  -- Starts with A

-- ORDER BY (sorting)
SELECT * FROM users ORDER BY age DESC;  -- Highest first

-- LIMIT (pagination)
SELECT * FROM users LIMIT 10 OFFSET 20;  -- Skip 20, show 10
```

### **Advanced SQL**

```sql
-- JOINS
SELECT users.name, orders.amount
FROM users
INNER JOIN orders ON users.id = orders.user_id;

-- LEFT JOIN (include nulls)
SELECT users.name, COUNT(orders.id) as order_count
FROM users
LEFT JOIN orders ON users.id = orders.user_id
GROUP BY users.id, users.name;

-- GROUP BY & AGGREGATION
SELECT age, COUNT(*) as count, AVG(salary) as avg_salary
FROM users
GROUP BY age
HAVING COUNT(*) > 5;  -- Filter groups

-- Window functions
SELECT name, salary,
  AVG(salary) OVER (PARTITION BY department) as dept_avg,
  RANK() OVER (ORDER BY salary DESC) as salary_rank
FROM employees;

-- CTEs (Common Table Expressions)
WITH high_earners AS (
  SELECT * FROM employees WHERE salary > 100000
)
SELECT department, COUNT(*) as count
FROM high_earners
GROUP BY department;

-- UPDATE
UPDATE users SET age = 26 WHERE name = 'Alice';

-- DELETE
DELETE FROM users WHERE age < 18;
```

### **Database Design**

**Normalization (Reduce data redundancy):**

```
1NF: Each column has atomic value
2NF: Remove partial dependencies
3NF: Remove transitive dependencies
BCNF: Every determinant is a candidate key
```

**Example - Poor vs Good Design:**

```sql
-- POOR (denormalized)
CREATE TABLE students (
  id INT PRIMARY KEY,
  name VARCHAR(255),
  course1 VARCHAR(255),
  course2 VARCHAR(255),
  course3 VARCHAR(255),
  teacher1 VARCHAR(255),
  teacher2 VARCHAR(255)
);

-- GOOD (normalized)
CREATE TABLE students (
  id INT PRIMARY KEY,
  name VARCHAR(255)
);

CREATE TABLE courses (
  id INT PRIMARY KEY,
  name VARCHAR(255),
  teacher_id INT FOREIGN KEY
);

CREATE TABLE enrollments (
  student_id INT FOREIGN KEY,
  course_id INT FOREIGN KEY,
  PRIMARY KEY (student_id, course_id)
);
```

### **Indexing & Performance**

```sql
-- Create index
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_age_salary ON users(age, salary);  -- Composite

-- Analyze query performance
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';

-- Index types
-- B-tree: General purpose (default)
-- Hash: Exact matches only
-- GiST: Geospatial, full-text search
-- BRIN: Large datasets, sequential data

-- When to index
-- ✓ Columns in WHERE clause
-- ✓ Columns in JOIN ON clause
-- ✗ Low cardinality (yes/no, status)
-- ✗ Small tables
```

### **PostgreSQL Advanced**

```sql
-- JSONB (JSON with indexing)
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  metadata JSONB
);

INSERT INTO products VALUES (1, 'Laptop', '{"color": "silver", "specs": {"cpu": "M1"}}');

-- Query JSONB
SELECT * FROM products WHERE metadata->>'color' = 'silver';
SELECT * FROM products WHERE metadata->'specs'->>'cpu' = 'M1';

-- Array columns
CREATE TABLE tags (
  id SERIAL PRIMARY KEY,
  article_id INT,
  tags TEXT[]
);

SELECT * FROM tags WHERE 'database' = ANY(tags);

-- Full-text search
CREATE TABLE articles (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255),
  content TEXT,
  search_vector tsvector
);

UPDATE articles SET search_vector = to_tsvector('english', title || ' ' || content);
SELECT * FROM articles WHERE search_vector @@ to_tsquery('database');
```

---

## NoSQL Databases

### **MongoDB Document Storage**

```javascript
// Insert documents
db.users.insertOne({
  _id: ObjectId(),
  name: "Alice",
  email: "alice@example.com",
  age: 25,
  tags: ["developer", "python"],
  address: {
    street: "123 Main St",
    city: "New York",
    zip: "10001"
  }
});

// Query documents
db.users.find({ name: "Alice" });
db.users.find({ age: { $gt: 25 } });  // Greater than
db.users.find({ tags: "python" });  // Array contains

// Update
db.users.updateOne(
  { name: "Alice" },
  { $set: { age: 26 } }
);

db.users.updateOne(
  { _id: ObjectId(...) },
  { $push: { tags: "javascript" } }  // Add to array
);

// Aggregation pipeline
db.users.aggregate([
  { $match: { age: { $gt: 20 } } },
  { $group: { _id: null, avg_age: { $avg: "$age" } } },
  { $sort: { avg_age: -1 } }
]);

// Indexes
db.users.createIndex({ email: 1 });
db.users.createIndex({ name: 1, age: 1 });
db.users.createIndex({ search: "text" });  // Full-text search
```

### **Redis Caching**

```python
import redis

r = redis.Redis(host='localhost', port=6379)

# Strings
r.set('user:1:name', 'Alice')
r.get('user:1:name')  # b'Alice'
r.incr('page:views')  # Increment counter

# TTL (Time to live)
r.setex('token:xyz', 3600, 'valid')  # Expires in 1 hour

# Lists
r.lpush('queue:jobs', 'job1', 'job2')
r.rpop('queue:jobs')  # Dequeue
r.llen('queue:jobs')  # Length

# Sets
r.sadd('tags:post:1', 'python', 'database', 'backend')
r.smembers('tags:post:1')
r.sismember('tags:post:1', 'python')  # Is member?

# Hashes
r.hset('user:1', mapping={'name': 'Alice', 'email': 'alice@example.com'})
r.hgetall('user:1')

# Pub/Sub
r.publish('channel:notifications', 'New message')

# Transactions
pipe = r.pipeline()
pipe.set('key1', 'value1')
pipe.set('key2', 'value2')
pipe.execute()
```

---

## API Design

### **REST API Best Practices**

```
HTTP Methods:
GET    - Retrieve resource (safe, idempotent)
POST   - Create resource
PUT    - Replace entire resource (idempotent)
PATCH  - Partial update
DELETE - Remove resource (idempotent)

Status Codes:
200 OK - Success
201 Created - Resource created
204 No Content - Success, no body
400 Bad Request - Client error
401 Unauthorized - Auth required
403 Forbidden - Not allowed
404 Not Found - Resource missing
500 Internal Server Error
```

**Resource URLs:**

```
GET    /api/users              # List all
GET    /api/users/:id          # Get one
POST   /api/users              # Create
PUT    /api/users/:id          # Update (full)
PATCH  /api/users/:id          # Update (partial)
DELETE /api/users/:id          # Delete

// Nested resources
GET    /api/users/:id/posts    # User's posts
POST   /api/users/:id/posts    # Create post for user
```

**Request/Response Example:**

```json
POST /api/users
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com",
  "age": 25
}

Response (201 Created):
{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com",
  "age": 25,
  "created_at": "2024-01-15T10:30:00Z"
}
```

### **GraphQL**

```graphql
# Schema
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}

type Query {
  user(id: ID!): User
  users(limit: Int): [User!]!
  post(id: ID!): Post
}

type Mutation {
  createUser(name: String!, email: String!): User!
  updateUser(id: ID!, name: String): User
  deleteUser(id: ID!): Boolean!
}
```

```graphql
# Query
query GetUserWithPosts {
  user(id: "123") {
    name
    email
    posts {
      title
      id
    }
  }
}

# Mutation
mutation CreateUser {
  createUser(name: "Alice", email: "alice@example.com") {
    id
    name
    email
  }
}
```

**GraphQL vs REST:**

| Aspect | REST | GraphQL |
|--------|------|---------|
| Over-fetching | Common | None |
| Under-fetching | Need multiple requests | Single query |
| Caching | Easy (HTTP caching) | More complex |
| Learning curve | Low | High |
| Use case | Simple CRUD | Complex, nested data |

---

## System Design & Architecture

### **Scalability Patterns**

**Vertical Scaling (Scale Up):**
- Add more CPU, RAM, storage
- Simple but has limits
- Single point of failure

**Horizontal Scaling (Scale Out):**
- Add more servers
- Load balancing needed
- Better resilience

### **Caching Strategy**

```
Cache Levels:
1. Client-side (browser cache)
2. CDN (edge caching)
3. Application cache (Redis, Memcached)
4. Database (query caching)
```

**Cache Invalidation Strategies:**

```
1. TTL (Time to Live) - Automatic expiration
2. Event-based - Invalidate on change
3. Purge - Manual invalidation
```

### **Microservices Architecture**

```
Advantages:
✓ Independent scaling
✓ Technology diversity
✓ Faster deployment

Challenges:
✗ Network latency
✗ Distributed transactions
✗ Operational complexity

Pattern:
API Gateway → Services → Databases
     ↓
  Service Discovery
  Message Queue
  Logging/Monitoring
```

### **Database Sharding**

```
Split data across multiple databases
- Range-based: User ID 1-1000 → DB1, 1001-2000 → DB2
- Hash-based: hash(user_id) % num_shards
- Directory-based: Lookup table maps to shard

Tradeoffs:
✓ Horizontal scaling
✗ Complex queries
✗ Operational overhead
```

---

## Learning Checklist

- [ ] Understand SQL SELECT with WHERE, JOIN
- [ ] Can design normalized schema
- [ ] Know when to use indexes
- [ ] Understand NoSQL document stores
- [ ] Built API with proper status codes
- [ ] Know REST vs GraphQL trade-offs
- [ ] Understand caching strategies
- [ ] Know sharding and replication
- [ ] Understand microservices patterns
- [ ] Ready for architect role!

---

**Source**: https://roadmap.sh/sql, https://roadmap.sh/system-design, https://roadmap.sh/api-design
