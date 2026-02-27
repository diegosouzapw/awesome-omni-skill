---
name: backend-architecture
description: Design and implement scalable backend infrastructure, microservices, and system architecture patterns.
---

# Backend Architecture Methodology

When designing backend systems:

1. **Analyze Requirements**
   - Identify scalability needs (concurrent users, data volume)
   - Determine consistency vs. availability trade-offs (CAP theorem)
   - Assess latency and throughput requirements

2. **Choose Architecture Pattern**
   - **Monolith**: Simple deployments, tight coupling acceptable
   - **Microservices**: Independent scaling, team autonomy
   - **Serverless**: Event-driven, variable load, minimal ops
   - **Event-Driven**: Async processing, decoupled systems

3. **Design Data Layer**
   - Select appropriate databases (SQL vs. NoSQL vs. hybrid)
   - Design schema with normalization/denormalization trade-offs
   - Plan caching strategy (Redis, Memcached)
   - Consider data partitioning and sharding

4. **Infrastructure Decisions**
   - Container orchestration (Kubernetes, Cloud Run)
   - Message queues (Pub/Sub, RabbitMQ, Kafka)
   - Load balancing and auto-scaling
   - Observability (logging, metrics, tracing)

5. **Security & Reliability**
   - Authentication/authorization (OAuth2, JWT, API keys)
   - Rate limiting and DDoS protection
   - Disaster recovery and backup strategies
   - Circuit breakers and retry policies

## Best Practices

- **Start simple, scale incrementally**: Don't over-engineer early
- **Design for failure**: Assume components will fail
- **Automate everything**: CI/CD, testing, deployments
- **Monitor proactively**: Set up alerts before issues occur
- **Document architecture decisions**: Use ADRs (Architecture Decision Records)
