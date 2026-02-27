---
name: rust-backend-advance
description: Production-ready Rust backend development with Axum framework and PostgreSQL. Master async patterns, tower middleware, SQLx database operations, authentication (JWT/OAuth), testing strategies, and deployment. Use when building REST APIs, microservices, or any Rust web backend with Axum.
license: MIT
---

# Rust + Axum Backend Mastery

Production-ready patterns for building scalable Rust backends with Axum and PostgreSQL.

## When to Use This Skill

- Building REST APIs or GraphQL with Axum
- Designing database schemas with SQLx + PostgreSQL
- Implementing authentication (JWT, OAuth 2.1)
- Writing async code with Tokio
- Creating middleware and extractors
- Testing Axum applications
- Deploying to production (Docker, Kubernetes)
- Performance optimization and monitoring

---

## Quick Start

### Minimal Axum Server

```rust
use axum::{routing::get, Router};

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(|| async { "Hello, Axum!" }));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

### Essential Dependencies (Cargo.toml)

```toml
[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
tower = "0.4"
tower-http = { version = "0.5", features = ["cors", "trace"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
sqlx = { version = "0.7", features = ["runtime-tokio", "postgres", "uuid", "time"] }
uuid = { version = "1", features = ["v4", "serde"] }
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
thiserror = "1"
anyhow = "1"
```

---

## Reference Navigation

### Core Rust Patterns
| Topic | File | Description |
|-------|------|-------------|
| Rust Idioms | [rust-patterns.md](references/rust-patterns.md) | Enums, iterators, error handling |
| Async/Tokio | [async-tokio.md](references/async-tokio.md) | Async patterns, spawn, channels |

### Axum Framework (70%)
| Topic | File | Description |
|-------|------|-------------|
| **Axum Guide** | [axum-complete-guide.md](references/axum-complete-guide.md) | Routing, handlers, state |
| Extractors | [axum-extractors.md](references/axum-extractors.md) | Path, Query, Json, State |
| Middleware | [middleware-patterns.md](references/middleware-patterns.md) | Tower layers, auth middleware |

### Database (PostgreSQL)
| Topic | File | Description |
|-------|------|-------------|
| SQLx | [sqlx-postgresql.md](references/sqlx-postgresql.md) | Queries, transactions, migrations |
| Patterns | [database-patterns.md](references/database-patterns.md) | Connection pools, optimization |

### Architecture
| Topic | File | Description |
|-------|------|-------------|
| Project Structure | [project-structure.md](references/project-structure.md) | Folder organization |
| Patterns | [architecture-patterns.md](references/architecture-patterns.md) | Microservices, modulith |

### Security & Auth
| Topic | File | Description |
|-------|------|-------------|
| Authentication | [authentication.md](references/authentication.md) | JWT, OAuth 2.1, sessions |
| Security | [security-owasp.md](references/security-owasp.md) | OWASP Top 10 for Rust |

### Testing & Quality
| Topic | File | Description |
|-------|------|-------------|
| Testing | [testing-guide.md](references/testing-guide.md) | Unit, integration, E2E |
| Error Handling | [error-handling.md](references/error-handling.md) | HTTP errors, thiserror |

### DevOps & Production
| Topic | File | Description |
|-------|------|-------------|
| Deployment | [deployment.md](references/deployment.md) | Docker, Kubernetes |
| Monitoring | [monitoring.md](references/monitoring.md) | Tracing, Prometheus |

---

## Decision Guide

### When to Choose Axum (70% - Primary)

```
✅ Choose Axum when:
- Building new Rust web projects
- Need tower ecosystem compatibility
- Want ergonomic, type-safe extractors
- Prefer modular, composable design
- Need excellent async performance
```

### When to Consider Alternatives (30%)

```
Actix-web - When you need:
- Maximum raw performance (benchmarks leader)
- Actor model for complex state
- Established ecosystem with more examples

Rocket - When you need:
- Simplest learning curve
- Most "magical" developer experience
- Rapid prototyping
```

---

## Core Patterns Summary

### Error Handling
```rust
use axum::{http::StatusCode, response::IntoResponse, Json};
use serde_json::json;

pub enum AppError {
    NotFound(String),
    Database(sqlx::Error),
    Unauthorized,
}

impl IntoResponse for AppError {
    fn into_response(self) -> axum::response::Response {
        let (status, message) = match self {
            Self::NotFound(msg) => (StatusCode::NOT_FOUND, msg),
            Self::Database(e) => (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()),
            Self::Unauthorized => (StatusCode::UNAUTHORIZED, "Unauthorized".into()),
        };
        (status, Json(json!({ "error": message }))).into_response()
    }
}
```

### Handler Pattern
```rust
use axum::{extract::{Path, State}, Json};
use uuid::Uuid;

async fn get_user(
    State(pool): State<PgPool>,
    Path(id): Path<Uuid>,
) -> Result<Json<User>, AppError> {
    let user = sqlx::query_as!(User, "SELECT * FROM users WHERE id = $1", id)
        .fetch_optional(&pool)
        .await?
        .ok_or_else(|| AppError::NotFound("User not found".into()))?;
    
    Ok(Json(user))
}
```

### State Management
```rust
use std::sync::Arc;
use sqlx::PgPool;

#[derive(Clone)]
pub struct AppState {
    pub db: PgPool,
    pub config: Arc<Config>,
}

let app = Router::new()
    .route("/users/:id", get(get_user))
    .with_state(AppState { db: pool, config: Arc::new(config) });
```

---

## Examples

- **[axum-starter](examples/axum-starter/)** - Minimal project template
- **[axum-rest-api](examples/axum-rest-api/)** - Complete REST API with auth

---

## Best Practices Checklist

### API Design
- [ ] Use proper HTTP methods (GET, POST, PUT, DELETE)
- [ ] Return appropriate status codes
- [ ] Validate input with extractors
- [ ] Document with OpenAPI/Swagger

### Database
- [ ] Use connection pooling (SQLx built-in)
- [ ] Always use parameterized queries
- [ ] List columns explicitly (no SELECT *)
- [ ] Use transactions for multi-step operations

### Security
- [ ] Validate all input
- [ ] Use Argon2id for passwords
- [ ] Implement rate limiting
- [ ] Set security headers (tower-http)

### Testing
- [ ] Unit tests for business logic
- [ ] Integration tests for handlers
- [ ] Use testcontainers for database tests

### Production
- [ ] Structured logging (tracing)
- [ ] Health check endpoints
- [ ] Graceful shutdown
- [ ] Docker multi-stage builds

---

## Resources

- **Axum**: https://docs.rs/axum
- **Tower**: https://docs.rs/tower
- **SQLx**: https://docs.rs/sqlx
- **Tokio**: https://tokio.rs
