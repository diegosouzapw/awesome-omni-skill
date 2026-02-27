---
name: architecture-documentation
description: Master architecture documentation with C4 model, ADRs, diagrams, and technical documentation for clear architecture communication.
---

# Architecture Documentation

Create clear, comprehensive architecture documentation that communicates design decisions and system structure to stakeholders.

## When to Use This Skill

- New system design
- Architecture changes
- Team onboarding
- Stakeholder communication
- Technical reviews
- Knowledge transfer
- Compliance requirements
- Decision tracking

## Core Concepts

### 1. Architecture Decision Record (ADR)

```markdown
# ADR 001: Use PostgreSQL for Primary Database

**Date:** 2024-01-15
**Status:** Accepted
**Deciders:** Tech Lead, Solutions Architect, Backend Team

## Context
Need to select a database for new e-commerce platform. Requirements:
- ACID transactions for order processing
- Complex queries (joins, aggregations)
- Strong consistency
- Support for 100K users initially, 1M within 2 years

## Decision
Use PostgreSQL as primary database.

## Alternatives Considered

**MySQL:**
- ✅ Widely used, good documentation
- ✅ Strong community support
- ❌ Historically weaker for complex queries
- ❌ JSON support not as robust

**MongoDB:**
- ✅ Flexible schema
- ✅ Horizontal scaling
- ❌ No multi-document ACID transactions (at time of decision)
- ❌ Eventual consistency by default

**DynamoDB:**
- ✅ Fully managed, auto-scaling
- ❌ Complex query limitations
- ❌ Vendor lock-in
- ❌ Cost unpredictable at scale

## Rationale
- PostgreSQL offers ACID guarantees needed for financial transactions
- Advanced SQL features (window functions, CTEs) for analytics
- Excellent JSON support for flexible data
- Proven at scale (Instagram, Uber)
- Can add read replicas for scaling
- Strong typing prevents data errors

## Consequences

**Positive:**
- Strong data consistency
- Rich query capabilities
- Battle-tested reliability
- Excellent tooling ecosystem

**Negative:**
- Vertical scaling limits (mitigated with read replicas)
- More complex horizontal sharding (if needed in future)
- Requires DBA expertise for optimization

## Compliance
No specific compliance requirements impacted.

## Notes
Will reassess if we need to shard beyond 10M users.
```

### 2. System Architecture Document

```markdown
# System Architecture: E-Commerce Platform

## Overview
Microservices-based e-commerce platform supporting 1M users with high availability and scalability.

## Architecture Principles
1. Microservices for domain separation
2. API-first design
3. Event-driven communication
4. Cloud-native (AWS)
5. Infrastructure as Code

## System Context (C4 Level 1)
[Diagram showing users, system, external systems]

## Container Diagram (C4 Level 2)

**Frontend:**
- React SPA (S3 + CloudFront)
- Mobile Apps (iOS/Android)

**Backend Services:**
- API Gateway (AWS API Gateway)
- User Service (Node.js, PostgreSQL)
- Product Service (Node.js, PostgreSQL)
- Order Service (Node.js, PostgreSQL)
- Payment Service (Node.js)
- Notification Service (Node.js, SES)

**Data Stores:**
- PostgreSQL (RDS Multi-AZ)
- Redis (ElastiCache)
- S3 (product images)

**Messaging:**
- EventBridge (event bus)
- SQS (queues)

**External:**
- Stripe (payment processing)
- SendGrid (email)

## Technology Stack

**Frontend:**
- React 18
- TypeScript
- Tailwind CSS

**Backend:**
- Node.js 20
- Express.js
- TypeScript
- PostgreSQL 15
- Redis 7

**Infrastructure:**
- AWS (primary cloud)
- Terraform (IaC)
- GitHub Actions (CI/CD)
- DataDog (monitoring)

## Security
- HTTPS only (TLS 1.3)
- JWT authentication
- Role-based access control
- Encryption at rest (AES-256)
- Regular security scans (Snyk, OWASP ZAP)

## Scalability
- Horizontal scaling with Auto Scaling Groups
- Database read replicas
- CDN for static assets
- Redis caching
- Async processing with SQS

## Disaster Recovery
- RPO: 1 hour
- RTO: 4 hours
- Multi-AZ deployment
- Daily database backups
- Cross-region replication for critical data

## Monitoring & Observability
- DataDog APM
- CloudWatch logs
- Custom metrics dashboard
- PagerDuty alerts
```

## Best Practices

1. **Keep it current** - Update with changes
2. **Multiple views** - Different stakeholders need different levels
3. **Visual diagrams** - Architecture diagrams, sequence diagrams
4. **Decision rationale** - Document "why", not just "what"
5. **ADRs for major decisions** - Track decision history
6. **Living documentation** - Evolve with system
7. **Version control** - Track changes over time
8. **Review regularly** - Quarterly architecture reviews

## Resources

- **C4 Model**: https://c4model.com
- **ADR Templates**: https://adr.github.io
- **Structurizr**: Architecture diagram tool
