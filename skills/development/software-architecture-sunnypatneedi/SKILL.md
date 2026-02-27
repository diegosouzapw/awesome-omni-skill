---
name: software-architecture
description: Design scalable software systems with proven architectural patterns (MVC, microservices, event-driven), SOLID principles, system design trade-offs, and architectural decision records (ADRs).
---

# Software Architecture

Complete framework for designing software systems that are scalable, maintainable, and aligned with business requirements.

## When to Use

- Starting a new project or greenfield development
- Refactoring a monolith
- System is growing beyond current architecture
- Making technology stack decisions
- Designing for scale (10x users expected)
- Multiple teams working on same codebase
- Performance or reliability issues
- Planning microservices migration

## Core Principles

**Architecture Serves Business:**
- Technology choices follow business needs
- Trade-offs are intentional
- Over-engineering is waste
- Simplest solution that works

**SOLID Principles:**
```
S - Single Responsibility Principle
O - Open/Closed Principle
L - Liskov Substitution Principle
I - Interface Segregation Principle
D - Dependency Inversion Principle
```

**Other Key Principles:**
- **DRY** (Don't Repeat Yourself)
- **KISS** (Keep It Simple, Stupid)
- **YAGNI** (You Aren't Gonna Need It)
- **Separation of Concerns**
- **Principle of Least Surprise**

---

## Workflow

### Step 1: Understand Requirements

**Functional Requirements:**
```markdown
## What the System Must Do

**User Stories:**
- As a [user], I want to [action] so that [benefit]

**Features:**
- User authentication
- Product catalog
- Shopping cart
- Payment processing
- Order tracking

**Business Rules:**
- Discount codes can only be used once per user
- Orders over $50 get free shipping
- Inventory decrements on successful payment
```

**Non-Functional Requirements (The "ilities"):**
```markdown
## How the System Must Perform

**Scalability:**
- Support 10K concurrent users
- Handle 100K products in catalog
- Process 1K orders per hour

**Performance:**
- Page load <2 seconds
- API response <100ms (p95)
- Search results <500ms

**Reliability:**
- 99.9% uptime (8.7 hours downtime/year)
- Zero data loss
- Graceful degradation under load

**Security:**
- PCI DSS compliant for payments
- GDPR compliant for EU users
- Data encrypted at rest and in transit

**Maintainability:**
- New developers productive in 1 week
- Deploy multiple times per day
- Rollback within 5 minutes

**Observability:**
- Full request tracing
- Error rate monitoring
- Performance metrics
```

### Step 2: Choose Architectural Pattern

**Monolith:**
```
Best for:
- Small teams (<10 people)
- Simple domains
- Early-stage startups
- Rapid iteration

Architecture:
┌─────────────────────────┐
│   Web Application       │
│  ┌──────┬──────┬──────┐ │
│  │ UI   │Logic │ Data │ │
│  └──────┴──────┴──────┘ │
└─────────────────────────┘
         ↓
    Single Database

Pros:
✅ Simple to develop
✅ Simple to deploy
✅ Simple to test
✅ Low latency between components

Cons:
❌ Scaling requires scaling everything
❌ Tight coupling
❌ One failure affects all
❌ Hard to work on independently
```

**Microservices:**
```
Best for:
- Large teams (multiple squads)
- Complex domains
- Independent scaling needs
- Polyglot requirements

Architecture:
┌──────────┐  ┌──────────┐  ┌──────────┐
│  User    │  │  Order   │  │  Payment │
│ Service  │  │ Service  │  │  Service │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │              │
     ↓             ↓              ↓
  User DB      Order DB      Payment DB

Pros:
✅ Independent deployment
✅ Technology flexibility
✅ Team autonomy
✅ Fault isolation

Cons:
❌ Network complexity
❌ Distributed transactions hard
❌ More operational overhead
❌ Debugging across services
```

**Event-Driven:**
```
Best for:
- Async workflows
- Real-time data processing
- Audit trails
- Decoupled systems

Architecture:
┌─────────┐       ┌────────────┐
│Producer │──────>│Event Queue │
└─────────┘       └─────┬──────┘
                        │
         ┌──────────────┼──────────────┐
         ↓              ↓              ↓
    Consumer 1     Consumer 2     Consumer 3

Pros:
✅ Loose coupling
✅ Easy to add consumers
✅ Natural audit log
✅ Handles spikes well

Cons:
❌ Eventual consistency
❌ Harder to debug
❌ Message ordering challenges
❌ More moving parts
```

**Layered Architecture (N-Tier):**
```
Best for:
- Traditional enterprise apps
- Clear separation of concerns
- Team specialization (frontend/backend/data)

Architecture:
┌─────────────────────────┐
│  Presentation Layer     │ (UI, API)
├─────────────────────────┤
│  Business Logic Layer   │ (Domain, Services)
├─────────────────────────┤
│  Data Access Layer      │ (Repositories, ORM)
├─────────────────────────┤
│  Database Layer         │ (PostgreSQL, etc.)
└─────────────────────────┘

Rules:
- Upper layers can call lower layers
- Lower layers cannot call upper layers
- Each layer has clear responsibility

Pros:
✅ Clear separation
✅ Testable layers
✅ Familiar pattern

Cons:
❌ Can become rigid
❌ Changes ripple across layers
❌ Performance overhead
```

**Hexagonal Architecture (Ports & Adapters):**
```
Best for:
- Domain-driven design
- Testing-heavy environments
- Swappable infrastructure

Architecture:
        ┌─────────────┐
        │   Domain    │
        │   (Core)    │
        └──────┬──────┘
               │
     ┌─────────┼─────────┐
     ↓         ↓         ↓
  HTTP API  Database  Queue
  (Adapter) (Adapter) (Adapter)

Core never depends on adapters
Adapters depend on core

Pros:
✅ Highly testable
✅ Infrastructure-agnostic
✅ DDD-friendly

Cons:
❌ More abstraction
❌ Steeper learning curve
❌ Can be over-engineered
```

### Step 3: Design System Components

**Component Design Template:**
```markdown
## [Component Name]

**Purpose:**
What does this component do?

**Responsibilities:**
- Responsibility 1
- Responsibility 2

**Dependencies:**
- Component A (for X)
- Component B (for Y)

**Interfaces:**
```typescript
interface ComponentAPI {
  operation1(input: Type): Promise<Result>;
  operation2(input: Type): Result;
}
```

**Data:**
What data does it own/manage?

**Events:**
What events does it emit/consume?

**Error Handling:**
How does it handle failures?
```

**Example - Order Service:**
```markdown
## Order Service

**Purpose:**
Manage order lifecycle from creation to fulfillment

**Responsibilities:**
- Create orders
- Update order status
- Calculate totals with discounts
- Validate inventory availability

**Dependencies:**
- User Service (get user details)
- Inventory Service (check/reserve stock)
- Payment Service (process payment)

**Interfaces:**
```typescript
interface OrderService {
  createOrder(cart: Cart, userId: string): Promise<Order>;
  getOrder(orderId: string): Promise<Order>;
  updateStatus(orderId: string, status: OrderStatus): Promise<void>;
}
```

**Events Emitted:**
- OrderCreated
- OrderPaid
- OrderShipped
- OrderCancelled

**Events Consumed:**
- PaymentSucceeded
- PaymentFailed

**Error Handling:**
- Invalid cart → 400 Bad Request
- Out of stock → 409 Conflict
- Payment fails → Reverse inventory reservation
```

### Step 4: Make Technology Choices

**Decision Framework:**
```markdown
## Technology Decision: [Name]

**Problem:**
What are we trying to solve?

**Options:**
1. Option A
2. Option B
3. Option C

**Criteria:**
- Performance requirements
- Team expertise
- Community support
- Cost
- Scalability
- Security

**Evaluation:**
| Criteria | Option A | Option B | Option C |
|----------|----------|----------|----------|
| Performance | 8/10 | 9/10 | 7/10 |
| Expertise | 9/10 | 5/10 | 8/10 |
| Community | 10/10 | 7/10 | 9/10 |
| Cost | Free | $X/mo | Free |
| Scalability | 7/10 | 10/10 | 8/10 |

**Decision:** Option A

**Rationale:**
Why we chose this option.

**Trade-offs:**
What we're giving up.

**Review Date:**
When we'll reconsider this decision.
```

**Example - Database Choice:**
```markdown
## Database for Order Service

**Problem:**
Need persistent storage for orders with ACID guarantees

**Options:**
1. PostgreSQL (Relational)
2. MongoDB (Document)
3. DynamoDB (NoSQL)

**Criteria:**
- ACID compliance (critical)
- Complex queries (important)
- Scalability (important)
- Team expertise (important)

**Evaluation:**
| Criteria | PostgreSQL | MongoDB | DynamoDB |
|----------|------------|---------|----------|
| ACID | ✅ Full | ⚠️ Limited | ⚠️ Eventual |
| Queries | ✅ Excellent | ⚠️ Good | ❌ Limited |
| Scale | ✅ Vertical+ | ✅ Horizontal | ✅ Managed |
| Expertise | ✅ High | ⚠️ Medium | ❌ Low |

**Decision:** PostgreSQL

**Rationale:**
- ACID compliance is non-negotiable for financial transactions
- Team has 5 years PostgreSQL experience
- Can scale vertically to meet current needs
- Complex reporting queries needed

**Trade-offs:**
- Harder to horizontally scale than MongoDB
- More expensive at large scale than DynamoDB
- Self-managed vs fully managed

**Review Date:** When we hit 100K orders/day
```

### Step 5: Plan for Scale

**Scaling Strategies:**

```markdown
## Vertical Scaling (Scale Up)
Add more resources to single machine

**When:**
- Quick fix needed
- Simple deployment
- Under 10K users

**How:**
- Bigger CPU
- More RAM
- Faster disk

**Limits:**
- Hardware ceiling
- Single point of failure
- Expensive at scale

---

## Horizontal Scaling (Scale Out)
Add more machines

**When:**
- Growth expected
- High availability needed
- Cost-effective at scale

**How:**
- Load balancer
- Stateless services
- Shared database or sharding

**Challenges:**
- Session management
- Distributed state
- Data consistency

---

## Caching Strategy
Reduce load on database/services

**Layers:**
```
Browser Cache → CDN → App Cache → Database Cache

**Patterns:**
- Cache-Aside (lazy loading)
- Write-Through (sync write)
- Write-Behind (async write)
- Refresh-Ahead (proactive)

**Example:**
```typescript
async function getUser(id: string): Promise<User> {
  // 1. Check cache
  const cached = await cache.get(`user:${id}`);
  if (cached) return cached;

  // 2. Cache miss: fetch from DB
  const user = await db.users.findById(id);

  // 3. Store in cache (TTL: 1 hour)
  await cache.set(`user:${id}`, user, 3600);

  return user;
}
```

---

## Database Scaling

**Read Replicas:**
```
┌────────┐
│Primary │ (writes)
└───┬────┘
    │
    ├──────────┬──────────┐
    ↓          ↓          ↓
 Replica    Replica    Replica
 (reads)    (reads)    (reads)
```

**Sharding:**
```
User IDs 0-999      → Shard 1
User IDs 1000-1999  → Shard 2
User IDs 2000-2999  → Shard 3

Challenges:
- Rebalancing
- Cross-shard queries
- Transactions across shards
```

**Partitioning:**
```
Orders by date:
├── 2024-Q1 → Partition 1
├── 2024-Q2 → Partition 2
├── 2024-Q3 → Partition 3
└── 2024-Q4 → Partition 4

Benefits:
- Query performance
- Easier archival
- Smaller indexes
```

### Step 6: Document Decisions (ADRs)

**Architecture Decision Record Template:**

```markdown
# ADR [Number]: [Title]

**Status:** [Proposed | Accepted | Deprecated | Superseded]

**Date:** YYYY-MM-DD

**Deciders:** [Names]

---

## Context

What is the issue we're trying to solve?

**Current Situation:**
[Describe current state]

**Problem:**
[What needs to change and why]

**Constraints:**
- Technical constraints
- Business constraints
- Time constraints

---

## Decision

We will [decision].

**Details:**
[Explain the decision in detail]

---

## Options Considered

### Option 1: [Name]

**Pros:**
- Pro 1
- Pro 2

**Cons:**
- Con 1
- Con 2

### Option 2: [Name]

**Pros:**
- Pro 1
- Pro 2

**Cons:**
- Con 1
- Con 2

---

## Consequences

**Positive:**
- What improves
- What becomes easier

**Negative:**
- What becomes harder
- What we give up

**Risks:**
- What could go wrong
- Mitigation strategies

**Technical Debt:**
- What shortcuts are we taking
- When will we revisit

---

## Follow-up Actions

- [ ] Action 1 (Owner, Due Date)
- [ ] Action 2 (Owner, Due Date)

---

## References

- Link to design doc
- Link to RFC
- Related ADRs
```

**Example ADR:**

```markdown
# ADR 001: Migrate from Monolith to Microservices

**Status:** Accepted

**Date:** 2026-01-15

**Deciders:** Architecture Team, Engineering Leads

---

## Context

**Current Situation:**
Single Rails monolith serving all traffic. 50K daily active users.

**Problem:**
- Deployment takes 30 minutes, blocks all teams
- Database at 80% capacity
- Cannot scale teams independently
- Different services have different scaling needs (API vs background jobs)

**Constraints:**
- Must maintain 99.9% uptime during migration
- Complete within 6 months
- Team of 15 engineers

---

## Decision

We will migrate to microservices using the Strangler Fig pattern.

**Approach:**
1. Start with highest-value, lowest-risk services (User Service, Notifications)
2. Extract one service per month
3. API Gateway routes to new services
4. Monolith remains for remaining functionality
5. Gradual data migration

**Tech Stack:**
- Services: Node.js/TypeScript
- Communication: REST + Message Queue (RabbitMQ)
- Deployment: Kubernetes
- Data: PostgreSQL per service

---

## Options Considered

### Option 1: Continue Scaling Monolith

**Pros:**
- Simplest
- Team already knows it
- No migration risk

**Cons:**
- Doesn't solve team scaling
- Database still bottleneck
- Deployment still blocking

### Option 2: Big Bang Rewrite

**Pros:**
- Fresh start
- Modern architecture

**Cons:**
- High risk
- 6+ months no features
- Likely to fail

### Option 3: Strangler Fig Migration (CHOSEN)

**Pros:**
- Low risk (gradual)
- Continuous value delivery
- Reversible
- Learn as we go

**Cons:**
- Longer timeline
- Temporary complexity
- Some duplication

---

## Consequences

**Positive:**
- Teams can deploy independently
- Services scale independently
- Technology flexibility
- Fault isolation

**Negative:**
- Operational complexity (15+ services)
- Distributed debugging harder
- Network latency between services
- More infrastructure cost

**Risks:**
- Data consistency across services
- Authentication/authorization complexity
- Monitoring/observability gaps

**Mitigation:**
- Event sourcing for data sync
- Shared auth service
- OpenTelemetry from day 1

**Technical Debt:**
- Monolith will coexist for 12-18 months
- Some duplication during migration
- Revisit architecture Q3 2026

---

## Follow-up Actions

- [x] Create migration roadmap (Sarah, 2026-01-20)
- [x] Set up Kubernetes cluster (DevOps, 2026-01-25)
- [ ] Extract User Service (Team A, 2026-02-15)
- [ ] Implement API Gateway (Team B, 2026-02-01)
- [ ] Set up observability (DevOps, 2026-01-30)

---

## References

- [Migration Roadmap](link)
- [Microservices RFC](link)
- Related: ADR 002 (Service Communication Pattern)
```

---

## Common Patterns & Practices

**API Gateway Pattern:**
```
Client
  ↓
API Gateway (routes, auth, rate limiting)
  ├──→ User Service
  ├──→ Order Service
  └──→ Payment Service

Benefits:
- Single entry point
- Handles cross-cutting concerns
- Backend for frontend
```

**Circuit Breaker Pattern:**
```typescript
class CircuitBreaker {
  state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
  failures = 0;
  threshold = 5;

  async call(fn: Function) {
    if (this.state === 'OPEN') {
      throw new Error('Circuit breaker OPEN');
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onFailure() {
    this.failures++;
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
      setTimeout(() => this.state = 'HALF_OPEN', 60000);
    }
  }

  onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }
}
```

**Saga Pattern (Distributed Transactions):**
```
Order Saga:
1. Create Order     → Success
2. Reserve Inventory → Success
3. Charge Payment   → FAILS

Compensation (rollback):
3. Refund Payment     ← (skipped, never charged)
2. Release Inventory  ← Execute
1. Cancel Order       ← Execute

Result: Consistent state, no partial orders
```

**CQRS (Command Query Responsibility Segregation):**
```
Commands (Writes):      Queries (Reads):
Create Order            Get Order
Update User             List Orders
Delete Product          Search Products
    ↓                       ↑
  Write DB  ──────→    Read DB
(normalized)         (denormalized)

Benefits:
- Optimize read/write separately
- Scale independently
- Complex queries without impacting writes
```

---

## Architecture Checklist

```markdown
## Pre-Development

- [ ] Functional requirements documented
- [ ] Non-functional requirements defined
- [ ] Architecture pattern chosen
- [ ] Technology stack decided
- [ ] Data model designed
- [ ] API contracts defined
- [ ] Security reviewed
- [ ] Scalability plan created

## During Development

- [ ] Code organized by domain/feature
- [ ] Dependencies point inward (clean architecture)
- [ ] Interfaces define contracts
- [ ] Error handling consistent
- [ ] Logging and monitoring instrumented
- [ ] Tests cover critical paths
- [ ] Documentation up to date

## Pre-Production

- [ ] Load testing completed
- [ ] Security audit passed
- [ ] Monitoring dashboards ready
- [ ] Alerts configured
- [ ] Runbooks written
- [ ] Rollback plan tested
- [ ] DR plan documented
- [ ] Team trained
```

---

## Common Mistakes

| Don't | Do |
|-------|-----|
| Microservices for everything | Start monolith, extract when needed |
| Premature optimization | Optimize when you have data |
| Architecture astronaut | Solve today's problems, not future maybes |
| Copy Big Tech architecture | Your scale != their scale |
| Ignore non-functional requirements | Performance/security/reliability matter |
| Big Bang rewrites | Incremental refactoring |
| One size fits all | Different components, different patterns |
| Skip documentation | ADRs, diagrams, runbooks |

---

## Tools & Resources

**Diagramming:**
- draw.io (free, versatile)
- Lucidchart (collaborative)
- Mermaid (code-based)
- C4 Model (structured approach)

**Books:**
- "Clean Architecture" by Robert Martin
- "Designing Data-Intensive Applications" by Martin Kleppmann
- "Building Microservices" by Sam Newman
- "Domain-Driven Design" by Eric Evans

**Patterns:**
- microservices.io (pattern catalog)
- martinfowler.com (architecture articles)

---

## Related Skills

- `/systems-decompose` - Break down features
- `/database-schema` - Design data models
- `/api-design` - Design API contracts
- `/code-review` - Review architectural decisions

---

**Last Updated**: 2026-01-22
