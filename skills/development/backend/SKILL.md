---
name: backend
description: "Backend skills: data modeling, caching, WebSocket, reliability patterns, service architecture."
license: MIT
metadata:
  category: backend
  contains: data-modeling, caching-strategy, websocket-realtime, backend-reliability, backend-service-patterns
---

# Backend Engineering Skills

## 1. Data Modeling

Design database schemas with normalization, indexing strategy, and evolution planning.

### Schema design workflow
1. Identify entities, relationships, and cardinality.
2. Map primary access patterns and query shapes.
3. Normalize to 3NF, then selectively denormalize for read performance.
4. Define primary keys, foreign keys, and unique constraints.
5. Add indexes aligned with WHERE, JOIN, and ORDER BY patterns.

### Indexing strategy
- Composite indexes: column order must match query filter/sort order.
- Covering indexes: include all SELECT columns to avoid table lookups.
- Partial indexes: filter unused rows to reduce index size.
- Avoid over-indexing: each index adds write overhead.

### Schema evolution
- Prefer additive migrations (add column, add table).
- Use expand-contract pattern for breaking changes.
- Always plan rollback migration alongside forward migration.

---

## 2. Caching Strategy

Design caching layers with invalidation policies, TTL planning, and consistency trade-offs.

### Cache layers
- **Browser cache**: Static assets, API responses with ETag.
- **CDN cache**: Public content, immutable assets.
- **Application cache (Redis/Memcached)**: Session data, computed results, rate counters.
- **In-process cache**: Hot config, lookup tables, small reference data.

### Invalidation strategies
- **TTL-based**: Simple, eventual consistency.
- **Event-driven**: Invalidate on write/update events. Strongest consistency.
- **Write-through**: Update cache on every write.
- **Cache-aside (lazy load)**: Populate on cache miss.

### Anti-patterns
- Cache stampede: use locking or probabilistic early expiration.
- Unbounded growth: always set maxmemory and eviction policy.
- Caching errors: never cache 500 responses.

---

## 3. WebSocket & Real-time

Design real-time communication with connection lifecycle, reconnection, and backpressure.

### Connection lifecycle
1. **Handshake**: Authenticate during upgrade.
2. **Heartbeat**: Ping/pong to detect dead connections.
3. **Message framing**: JSON or binary with type + sequence number.
4. **Graceful close**: Close frame with reason code.

### Client patterns
- Auto-reconnect with exponential backoff + jitter.
- State sync on reconnect (request missed messages or full snapshot).
- Offline queue for outgoing messages during disconnect.

### Server patterns
- Room/channel abstraction for targeted broadcast.
- Horizontal scaling via Redis pub/sub or NATS.
- Backpressure: throttle slow consumers.
- Rate limiting per connection.

---

## 4. Backend Reliability

Design resilient services with circuit breakers, retries, graceful degradation, and health checks.

### Patterns
- Circuit breaker with configurable thresholds and recovery windows.
- Retry with exponential backoff and jitter for transient failures.
- Bulkhead isolation to prevent cascade failures.
- Graceful degradation: serve partial/cached responses when dependencies fail.
- Health check endpoints: liveness and readiness probes.

---

## 5. Service Architecture Patterns

Choose and implement backend service patterns: monolith, microservices, CQRS, event sourcing.

### Patterns
- Modular monolith: domain-separated modules with clear boundaries.
- Microservices: independently deployable, single-responsibility services.
- CQRS: separate read and write models for complex domains.
- Event sourcing: persist state as sequence of events for audit and replay.
- Saga pattern: distributed transactions across services.
