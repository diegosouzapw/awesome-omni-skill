---
name: backend-guide
description: Complete backend development guide covering Node.js, Python, Go, Java, PHP, databases, APIs, authentication, and server architecture. Use when building server applications, APIs, or backend systems.
sasmp_version: "1.3.0"
bonded_agent: 02-backend-specialist
bond_type: PRIMARY_BOND
eqhm_enabled: true
---

# Backend Development Guide

Build scalable, secure, and maintainable backend systems with comprehensive learning resources.

## Quick Start

Choose your backend language journey:

### Node.js (JavaScript)
```javascript
// Express.js simple API
const express = require('express');
const app = express();

app.get('/api/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json(user);
});

app.listen(3000, () => console.log('Server running'));
```

### Python
```python
# FastAPI example
from fastapi import FastAPI
from sqlalchemy.orm import Session

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.id == user_id).first()
    return user
```

### Go
```go
// Gin framework example
package main

import "github.com/gin-gonic/gin"

func main() {
    r := gin.Default()

    r.GET("/api/users/:id", func(c *gin.Context) {
        user := getUser(c.Param("id"))
        c.JSON(200, user)
    })

    r.Run()
}
```

## Backend Technology Paths

### Node.js/JavaScript
- **Frameworks**: Express, Fastify, NestJS, Koa
- **Package Manager**: npm, yarn, pnpm
- **Database Drivers**: Mongoose, Sequelize, TypeORM
- **Async**: Promises, async/await, event loop

### Python
- **Frameworks**: Django, Flask, FastAPI, Pyramid
- **ORM**: SQLAlchemy, Django ORM
- **Async**: asyncio, FastAPI async support
- **Data Processing**: NumPy, Pandas

### Go
- **Frameworks**: Gin, Echo, Beego
- **Concurrency**: Goroutines, channels
- **Standard Library**: Excellent built-in packages
- **Performance**: Compiled, extremely fast

### Java
- **Frameworks**: Spring Boot, Quarkus
- **Build Tools**: Maven, Gradle
- **JVM Ecosystem**: Comprehensive libraries
- **Performance**: Mature optimization

### PHP
- **Frameworks**: Laravel, Symfony, Slim
- **Database**: Eloquent ORM, Doctrine
- **Modern PHP**: 8.0+, typed properties
- **Deployment**: Easy hosting, mature ecosystem

## Database Fundamentals

### Relational Databases
- **PostgreSQL**: Advanced features, JSON support, reliability
- **MySQL**: Wide adoption, good performance
- **SQL Server**: Enterprise choice

```sql
-- Design example
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  title VARCHAR(255),
  content TEXT
);
```

### NoSQL Databases
- **MongoDB**: Document-oriented, flexible schema
- **Redis**: In-memory, caching and sessions
- **Cassandra**: Distributed, high availability

### Database Best Practices
- Normalization (1NF, 2NF, 3NF)
- Indexing strategy
- Query optimization
- Connection pooling
- Backup and recovery

## API Development

### REST APIs
```javascript
// RESTful endpoints
GET    /api/users              # List all users
GET    /api/users/:id          # Get single user
POST   /api/users              # Create user
PUT    /api/users/:id          # Update user
DELETE /api/users/:id          # Delete user
```

### GraphQL
- Schema design
- Resolvers and data fetching
- Query and mutation design
- Subscriptions for real-time data
- Performance optimization (batching, caching)

### API Security
- Authentication (JWT, OAuth2, sessions)
- Authorization (RBAC, permissions)
- Rate limiting
- Input validation
- CORS configuration

## Authentication & Authorization

### Authentication Methods
```javascript
// JWT Example
const token = jwt.sign({ userId: user.id }, SECRET, { expiresIn: '7d' });
const verified = jwt.verify(token, SECRET);
```

### Authorization
- Role-Based Access Control (RBAC)
- Attribute-Based Access Control (ABAC)
- Permission scoping
- Token expiration strategies

## Caching Strategies

### HTTP Caching
- Cache headers (Cache-Control, ETag)
- Conditional requests (If-Modified-Since)
- Browser vs server caching

### Application Caching
- Redis for session and data caching
- Cache invalidation strategies
- Cache warming
- Distributed caching

## Testing

### Unit Testing
- Test frameworks (Jest, pytest, unittest)
- Mocking dependencies
- Test coverage targets

### Integration Testing
- Database testing
- API endpoint testing
- Third-party service mocking

### Load Testing
- Tools (Apache JMeter, k6)
- Performance baselines
- Bottleneck identification

## Deployment & DevOps

### Containerization
- Docker for consistency
- Container orchestration (Kubernetes)
- CI/CD pipelines

### Scaling
- Horizontal scaling (load balancing)
- Database replication
- Caching layers
- Microservices patterns

## Learning Resources

### Official Docs
- [Node.js Docs](https://nodejs.org/docs/)
- [Python Docs](https://docs.python.org/)
- [Go Official](https://golang.org/doc/)
- [Spring Boot](https://spring.io/projects/spring-boot)

### Platforms
- FreeCodeCamp
- Udemy
- Frontend Masters
- Coursera

### Projects
1. **Blog API** - CRUD, authentication
2. **E-commerce Backend** - Products, orders, payments
3. **Real-time Chat** - WebSockets, persistence
4. **Job Board** - Complex queries, filtering
5. **Social Network** - Relationships, feeds

## Next Steps

1. Choose a language and framework
2. Master the language fundamentals
3. Learn database design
4. Build 5+ API projects
5. Study system design
6. Learn DevOps basics
7. Contribute to backend open-source

**Roadmap.sh Reference**: https://roadmap.sh/backend

---

**Status**: ✅ Production Ready | **SASMP**: v1.3.0 | **Bonded Agent**: 02-backend-specialist
