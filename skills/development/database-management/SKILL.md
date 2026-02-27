---
name: database-management
description: Database schema design, migrations, query optimization, and ORM best practices. Use for database setup, performance tuning, and data modeling.
---

# 🗄️ Database Management Skill

## Schema Design Patterns

### Normalization Levels
| Level | Description | When to Use |
|-------|-------------|-------------|
| 1NF | No repeating groups | Always |
| 2NF | No partial dependencies | Transactional data |
| 3NF | No transitive dependencies | Most applications |
| Denormalized | Redundant data | Read-heavy workloads |

### Common Patterns
```sql
-- One-to-Many
CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(100));
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  content TEXT
);

-- Many-to-Many (Junction Table)
CREATE TABLE tags (id SERIAL PRIMARY KEY, name VARCHAR(50));
CREATE TABLE post_tags (
  post_id INTEGER REFERENCES posts(id),
  tag_id INTEGER REFERENCES tags(id),
  PRIMARY KEY (post_id, tag_id)
);
```

---

## Migration Strategies

### Prisma
```bash
# Create migration
npx prisma migrate dev --name add_users_table

# Apply to production
npx prisma migrate deploy

# Reset database
npx prisma migrate reset
```

### Drizzle
```bash
# Generate migration
npx drizzle-kit generate:pg

# Push to database
npx drizzle-kit push:pg
```

### Safe Migration Checklist
- [ ] Backup database first
- [ ] Test on staging environment
- [ ] Plan rollback strategy
- [ ] Run during low-traffic hours
- [ ] Monitor after deployment

---

## Query Optimization

### Index Strategies
```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (order matters!)
CREATE INDEX idx_posts_user_date ON posts(user_id, created_at);

-- Partial index
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
```

### Common N+1 Problem
```javascript
// Bad ❌ - N+1 queries
const users = await User.findAll();
for (const user of users) {
  user.posts = await Post.findAll({ where: { userId: user.id } });
}

// Good ✅ - Eager loading
const users = await User.findAll({
  include: [{ model: Post }]
});
```

---

## ORM Best Practices

| Practice | Description |
|----------|-------------|
| Use Transactions | Wrap related operations |
| Connection Pooling | Reuse connections |
| Soft Deletes | Use `deleted_at` instead of DELETE |
| Audit Fields | Always add `created_at`, `updated_at` |
| Use Migrations | Never modify schema manually |

---

## Backup & Recovery

```bash
# PostgreSQL backup
pg_dump -U user -d database > backup.sql

# PostgreSQL restore
psql -U user -d database < backup.sql

# MySQL backup
mysqldump -u user -p database > backup.sql
```
