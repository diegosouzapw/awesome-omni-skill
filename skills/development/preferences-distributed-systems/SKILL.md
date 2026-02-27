---
name: preferences-distributed-systems
description: Distributed systems patterns including consistency models, consensus, and fault tolerance. Load when designing or debugging distributed architectures.
---

# Distributed systems

## Purpose

Distributed systems require explicit tradeoff decisions at architectural boundaries.
This document provides a language-agnostic decision framework mapping theoretical foundations (CAP theorem, PACELC, linearizability) to practical architectural choices.

Topics covered:
- Consistency models and their tradeoffs
- Algebraic foundations of CRDTs
- Reactive streams for distributed messaging
- The authority question (who owns truth?)
- Pattern tensions and what you sacrifice
- Dual-write avoidance strategies
- Idempotency as architectural primitive
- Saga patterns for distributed transactions
- Deterministic replay for durability

For local concurrency primitives, see rust-development/11-concurrency.md.
For Rust-specific distributed systems implementations, see rust-development/12-distributed-systems.md.
For aggregates as local consistency boundaries, see domain-modeling.md.
For effect isolation patterns, see architectural-patterns.md.

## Theoretical foundations

### CAP theorem

In the presence of network partitions (P), you must choose between consistency (C) and availability (A).

**Consistency**: All nodes see the same data at the same time.
**Availability**: Every request receives a response (success or failure).
**Partition tolerance**: System continues operating despite network failures.

Since network partitions are inevitable in distributed systems, the real choice is CA during normal operation, then CP or AP during partitions.

**CP systems** (Consistency over Availability):
- Return errors or timeouts during partitions
- Guarantee linearizability
- Examples: Zookeeper, etcd, traditional RDBMS with distributed transactions

**AP systems** (Availability over Consistency):
- Return potentially stale data during partitions
- Eventually consistent
- Examples: Cassandra, DynamoDB (default), DNS

### PACELC extension

PACELC extends CAP by considering tradeoffs during normal operation (no partition):

**If Partition**: Choose Availability or Consistency (CAP)
**Else**: Choose Latency or Consistency

During normal operation, you still face consistency vs. latency tradeoffs:

**PC/EL** (Partition-Consistency, Else-Latency):
- Consistent during partitions, but sacrifice latency during normal operation for stronger consistency
- Example: Traditional databases with strict serializable isolation

**PC/EC** (Partition-Consistency, Else-Consistency):
- Consistent always, even during normal operation
- Example: Spanner (uses synchronized clocks), VoltDB

**PA/EL** (Partition-Availability, Else-Latency):
- Available during partitions, low latency during normal operation, eventual consistency
- Example: Cassandra, DynamoDB, most NoSQL systems

**PA/EC** (Partition-Availability, Else-Consistency):
- Available during partitions, but stronger consistency during normal operation
- Example: MongoDB with majority reads/writes

### Linearizability

Linearizability is the strongest single-object consistency model.

**Definition**: Operations appear to occur instantaneously at some point between their invocation and completion, and all operations observe a single, total order.

**Characteristics**:
- Reads return the most recent write
- Operations have a total order that matches real-time ordering
- No stale reads
- Composable across objects

**Cost**: Requires coordination, reduces availability, increases latency.

**When required**:
- Financial transactions (balances, transfers)
- Distributed locking
- Leader election
- Configuration updates requiring instant visibility

**When not required**:
- Analytics dashboards (eventual consistency acceptable)
- Social media feeds (stale data tolerable)
- Content delivery (eventual propagation fine)

## Consistency models

### Strong consistency (linearizability)

**Guarantees**:
- Every read returns the most recent write
- Operations have a total order matching wall-clock time
- No stale reads ever

**Sacrifices**:
- Availability during partitions (CP)
- Latency (requires coordination, quorum reads/writes)
- Throughput (coordination overhead)

**When appropriate**:
- Financial systems requiring exact balances
- Inventory management preventing double-booking
- Distributed locks and leader election
- Any system where stale data causes correctness violations

**Implementation patterns**:
- Consensus algorithms (Raft, Paxos, Multi-Paxos)
- Distributed transactions with two-phase commit
- Synchronous replication with quorum reads/writes
- Single writer per partition (sacrifices write availability)

### Eventual consistency

**Guarantees**:
- All replicas converge to the same state eventually
- No guarantee when convergence occurs
- Reads may return arbitrarily stale data

**Sacrifices**:
- Consistency (temporarily divergent states visible)
- Requires conflict resolution mechanisms
- Application complexity (must handle stale data)

**When appropriate**:
- Social media feeds, activity streams
- Metrics and analytics dashboards
- Content delivery networks
- Systems where availability matters more than freshness

**Implementation patterns**:
- Asynchronous replication
- Gossip protocols
- Conflict-free replicated data types (CRDTs)
- Last-write-wins with vector clocks
- Application-level conflict resolution

**Conflict resolution strategies**:
- Last-write-wins (requires wall-clock sync or vector clocks)
- Application-specific merge functions
- CRDTs (automatically convergent without coordination)
- Manual resolution (show conflicts to users)

### Causal consistency

**Guarantees**:
- Operations that are causally related are seen in the same order by all nodes
- Concurrent operations (not causally related) may be seen in different orders
- Reads return all causally-prior writes

**Sacrifices**:
- Not as strong as linearizability (concurrent operations unordered)
- Requires tracking causality (vector clocks, version vectors)
- More complex than eventual consistency

**When appropriate**:
- Collaborative editing (maintain causality, allow concurrent edits)
- Distributed social graphs (preserve happened-before relationships)
- Systems requiring stronger consistency than eventual but not linearizability

**Implementation patterns**:
- Vector clocks or version vectors
- Lamport timestamps for partial ordering
- Causal broadcast protocols
- Session guarantees (read-your-writes, monotonic reads)

**Causality examples**:

```
Causally related:
  - User posts message (A)
  - User edits message (B)
  - B causally depends on A (must see edit after original)

Not causally related (concurrent):
  - User Alice posts message (A)
  - User Bob posts different message (B)
  - A and B are concurrent (order doesn't matter)
```

### Decision matrix

| Requirement | Strong Consistency | Causal Consistency | Eventual Consistency |
|-------------|-------------------|-------------------|---------------------|
| Correctness requires exact ordering | Required | Insufficient | Insufficient |
| Causality must be preserved | Sufficient | Required | Insufficient |
| High availability during partitions | Impossible | Possible | Guaranteed |
| Low latency reads/writes | Impossible | Possible | Guaranteed |
| Simple programming model | Yes (always fresh) | Moderate | No (handle staleness) |
| Financial transactions | Required | Insufficient | Insufficient |
| Collaborative editing | Overkill | Ideal | Possible (CRDTs) |
| Content delivery | Overkill | Overkill | Ideal |

## Algebraic foundations of CRDTs

Conflict-free Replicated Data Types (CRDTs) achieve eventual consistency through algebraic properties.
Understanding the underlying algebraic structures clarifies when and how to use each CRDT type.

### CRDTs as semilattices

State-based CRDTs (CvRDTs) form join-semilattices: partially ordered sets where any two elements have a least upper bound (join).

Properties required:
- Associativity: `join(join(a, b), c) = join(a, join(b, c))`
- Commutativity: `join(a, b) = join(b, a)`
- Idempotence: `join(a, a) = a`

These properties ensure:
- Message reordering does not affect final state
- Duplicate messages are harmless
- Convergence is guaranteed

### Common CRDT algebraic structures

| CRDT | Algebraic Structure | Operation |
|------|---------------------|-----------|
| G-Counter | Commutative monoid | Pointwise max of version vectors |
| PN-Counter | Abelian group (over integers) | Increment/decrement pairs |
| G-Set | Join-semilattice | Set union |
| 2P-Set | Pair of G-Sets | Add-set ∪, Remove-set ∪ |
| LWW-Register | Bounded join-semilattice | Max by timestamp |
| OR-Set | Complex semilattice | Tagged elements with tombstones |

### G-Counter as commutative monoid

The G-Counter (grow-only counter) demonstrates the monoid pattern:

```rust
use std::collections::HashMap;

#[derive(Debug, Clone)]
struct GCounter {
    counts: HashMap<NodeId, u64>,
}

type NodeId = String;

impl GCounter {
    // Monoid identity
    fn zero() -> Self {
        GCounter {
            counts: HashMap::new()
        }
    }

    // Monoid operation (commutative, associative, idempotent)
    fn merge(&self, other: &GCounter) -> GCounter {
        let mut result = self.counts.clone();
        for (node, count) in &other.counts {
            let entry = result.entry(node.clone()).or_insert(0);
            *entry = (*entry).max(*count);
        }
        GCounter { counts: result }
    }

    fn value(&self) -> u64 {
        self.counts.values().sum()
    }
}
```

### Operation-based CRDTs

Op-based CRDTs (CmRDTs) require operations to be commutative.
The algebraic requirement shifts from state merge to operation application.

See `theoretical-foundations.md` for the foundational laws these structures must satisfy (monoid axioms, semilattice properties, and free algebraic constructions).

## Reactive streams for distributed messaging

Reactive streams provide backpressure-aware communication between distributed components.
The stream abstraction composes algebraically while handling practical concerns like flow control.

### Stream algebra

Streams form a compositional algebra with three core components:
- **Source**: Produces elements (no input)
- **Flow/Pipe**: Transforms elements (input → output)
- **Sink**: Consumes elements (no output)

```
Source[A] → Flow[A, B] → Flow[B, C] → Sink[C]
```

These compose via connection operators, building complex pipelines from simple stages.

### Backpressure as first-class concern

Backpressure prevents fast producers from overwhelming slow consumers.
The algebraic model includes demand signaling flowing opposite to data.

```
Data:    Source ──────────────────→ Sink
Demand:  Source ←────────────────── Sink
```

This bidirectional flow ensures:
- Consumers process at their own pace
- Producers throttle to match consumer capacity
- No unbounded buffering or dropped messages

### Domain event streams

Event-driven bounded context communication maps naturally to streams:

```rust
use futures::StreamExt;

// Domain events flow between bounded contexts
async fn process_events(event_store: EventStore, context_id: ContextId) {
    let events: impl Stream<Item = DomainEvent> =
        event_store.subscribe(context_id);

    let processed = events
        .filter(|e| matches!(e, DomainEvent::OrderPlaced { .. }))
        .map(|e| transform_to_downstream_context(e))
        .ready_chunks(100); // Buffer with backpressure

    downstream_context.consume(processed).await;
}
```

### Stream composition preserves algebraic properties

If individual stream stages preserve certain properties (e.g., ordering, exactly-once), composition preserves them.
This enables reasoning about end-to-end guarantees from stage-level properties.

See `event-sourcing.md` for how event streams integrate with event-sourced aggregates.

## The authority question

Every distributed system must answer: **Who owns the current state?**

Three architectural positions with different tradeoffs.

### Authority and the Decider pattern

The Decider pattern separates write authority from read authority through its algebraic structure.

The `decide` function is **write authority**: it validates commands against current state and produces events representing state changes.
This function embodies the business rules that determine what transitions are valid.

The `evolve` function is **read authority**: it reconstructs state from the event log by applying events sequentially.
This function represents the definitive interpretation of what each event means for the current state.

The **event log is the system of record** in Decider-based architectures.
State is always derivable by folding `evolve` over the event sequence: `fold(evolve, initialState, events)`.
This separation enables distributed replay: any node can reconstruct state by consuming the event log, making the system resilient to failures and enabling temporal queries.

The functional purity of `decide` and `evolve` enables better testing (pure functions with no side effects) and deployment flexibility (deterministic replay on any node).
See rust-development/12-distributed-systems.md for Rust implementation patterns combining Decider with distributed event logs.

### Position 1: Event log as authority

**Pattern**: Event sourcing with derived state.

State is computed by replaying events from authoritative log.
Services derive views by consuming events.
The log is the system of record.

**Guarantees**:
- Complete audit trail (every state change recorded)
- Time travel (reconstruct state at any point)
- Event replay for recovery and debugging

**Sacrifices**:
- Query latency (state derived from events)
- Schema evolution complexity (old events must remain processable)
- Storage costs (events retained forever)

**When appropriate**:
- Audit requirements (financial, compliance)
- Complex domain logic requiring complete history
- Systems needing deterministic replay
- CQRS architectures (command/query separation)

**Implementation**:
- Append-only event log (Kafka, EventStore, PostgreSQL append table)
- Projection services consume events and build queryable views
- Commands validated against current state, emit events if valid
- State rebuilt by replaying events from log

**Example**: Banking system where account balance is derived from transaction events.

**See also**: `event-sourcing.md` for comprehensive event sourcing patterns, `theoretical-foundations.md` for the algebraic foundation (free monoid of events, state as catamorphism).

### Position 2: Service as authority

**Pattern**: Service owns state, events are notifications.

Service maintains canonical state (database, memory).
Events published after state changes to notify other services.
The service's database is the system of record.

**Guarantees**:
- Low query latency (state immediately queryable)
- Simpler schema evolution (only current state matters)
- Lower storage costs (history optional)

**Sacrifices**:
- No inherent audit trail (must be added separately)
- Dual-write problem (update state AND publish event)
- Cannot reconstruct past states without additional logging

**When appropriate**:
- Low-latency query requirements
- Simpler domain logic not requiring complete history
- Microservices where service owns its data
- Systems without strict audit requirements

**Implementation**:
- Service updates its database
- After commit, publish event (use transactional outbox to avoid dual-write)
- Other services consume events for their own purposes

**Example**: User profile service maintains current user data, publishes UserUpdated events.

### Position 3: Hybrid (event log + service state)

**Pattern**: Event log authoritative for recovery, service authoritative for operations.

Used by systems like Golem for durable execution.

Service maintains current state for fast queries.
Event log used for recovery after failures.
Both are maintained, serving different purposes.

**Guarantees**:
- Low query latency (service state)
- Deterministic recovery (event log)
- Durability without sacrificing performance

**Sacrifices**:
- Dual-write complexity (must maintain both)
- Storage costs (both state and log)
- Implementation complexity

**When appropriate**:
- Durable execution systems (workflow engines, actor systems)
- Systems requiring both fast queries and guaranteed recovery
- Long-running processes that must survive failures

**Implementation**:
- Operations update service state and append to event log atomically
- Queries served from service state
- Recovery replays events from log to rebuild state
- Snapshots + event deltas for efficiency

**Example**: Golem's durable execution combines in-memory state for performance with event log for recovery.

### Decision framework

| Question | Event Log Authority | Service Authority | Hybrid |
|----------|-------------------|------------------|--------|
| Need complete audit trail? | Required | Add separately | Available |
| Need time travel / debugging? | Built-in | Not available | Replay from log |
| Low-latency queries? | Derived views | Direct | Direct |
| Durability guarantees? | Inherent | Must implement | Inherent |
| Schema evolution complexity? | High | Low | Medium |
| Implementation complexity? | Medium | Low | High |

## Pattern tensions matrix

Choosing architectural patterns involves explicit tradeoffs.
This matrix shows what you sacrifice and what you gain.

| If you choose... | You sacrifice... | You gain... |
|-----------------|-----------------|------------|
| **Event sourcing** | Query latency, schema simplicity | Complete audit trail, time travel, deterministic replay |
| **CQRS** | Consistency between read/write models | Independent scaling, optimized read models, simpler queries |
| **Actor model** | Direct function calls, debuggability | Location transparency, fault isolation, natural concurrency |
| **Strong consistency** | Availability during partitions, latency | Correctness guarantees, simple programming model |
| **Eventual consistency** | Consistency, simple programming model | High availability, low latency, partition tolerance |
| **Synchronous calls** | Availability (cascading failures), scalability | Immediate feedback, simpler error handling |
| **Asynchronous messaging** | Immediate feedback, request-response simplicity | Decoupling, failure isolation, temporal independence |
| **Microservices** | Latency (network hops), transactions | Independent deployment, technology diversity, team autonomy |
| **Monolith** | Independent deployment, team autonomy | Low latency, ACID transactions, simpler deployment |
| **Saga pattern** | ACID transactions, rollback simplicity | Distributed transactions, service autonomy |
| **Two-phase commit** | Availability, performance | ACID across services (at high cost) |

**Note**: The CQRS pattern separates read and write models using profunctor-like structure, where commands flow one direction (write) and queries another (read).
See theoretical-foundations.md for the formal profunctor interpretation.

### Anti-patterns to avoid

**Never sacrifice correctness for convenience**:
- Do not use eventual consistency where strong consistency is required for correctness
- Do not ignore dual-write problems (they will cause divergence)
- Do not assume clocks are synchronized across distributed nodes

**Never assume the network is reliable**:
- Do not use synchronous calls without timeouts and retries
- Do not fail to handle partial failures
- Do not ignore idempotency (messages will be delivered multiple times)

**Never hide complexity in the wrong place**:
- Do not make consistency models implicit in implementation
- Do not bury distributed transaction logic in application code
- Do not use framework magic that obscures failure modes

## Dual-write avoidance

**The problem**: Updating state AND emitting event creates divergence risk.

Dual-write occurs when two state changes must happen together but use separate mechanisms:
- Update database, then publish event to message queue
- Update cache, then update database
- Update primary database, then update secondary index

**Why it's almost always wrong**:
- Either operation can fail independently
- No atomic commitment across both
- Leads to inconsistent state (database updated but event not sent, or vice versa)

**Example failure scenarios**:

```
Scenario 1: Database succeeds, event publish fails
- State updated in database
- No event published
- Other services never learn about change
- System diverges

Scenario 2: Event published, database update fails
- Event published to queue
- Database update fails/rolls back
- Other services process event for non-existent state
- System diverges
```

### Pattern 1: Single writer (event log only)

**Approach**: Event log is the only write target. State is derived.

Write events to append-only log.
Derive all state by consuming events.
No dual-write because there's only one write.

**Guarantees**:
- Events and state cannot diverge (state derived from events)
- Complete audit trail
- Deterministic replay

**Sacrifices**:
- Query latency (state must be derived)
- Schema evolution complexity

**Implementation**:
```
1. Validate command against current state (derived view)
2. If valid, append event to log
3. Projection services consume events and update queryable views
4. No direct state writes
```

**When appropriate**:
- Event-sourced systems
- CQRS architectures
- Systems requiring complete audit trail

### Pattern 2: Transactional outbox

**Approach**: Write state and event in the same database transaction.

Store event in outbox table in same transaction as state update.
Separate process polls outbox and publishes events.
Exactly-once event publishing via transactional writes.

**Guarantees**:
- State and event writes are atomic (single transaction)
- Events eventually published (outbox processor retries)
- No lost events

**Sacrifices**:
- Slight delay between state update and event publication
- Requires outbox table and processor
- Must handle duplicate events (at-least-once delivery)

**Implementation**:
```
BEGIN TRANSACTION
  UPDATE application_state SET ...;
  INSERT INTO outbox (event_type, payload, published) VALUES (..., false);
COMMIT

-- Separate process:
1. SELECT * FROM outbox WHERE published = false
2. Publish event to message queue
3. UPDATE outbox SET published = true WHERE id = ...
4. If publish fails, retry later (at-least-once delivery)
```

**When appropriate**:
- Microservices publishing events
- Systems using relational databases
- Need exactly-once state update with at-least-once event delivery

### Pattern 3: Change data capture (CDC)

**Approach**: Capture database changes as events automatically.

Database transaction log is the source of truth.
CDC tool (Debezium, Maxwell) streams changes as events.
No application-level dual-write.

**Guarantees**:
- State changes automatically become events
- No application code for event publishing
- Exactly-once capture (database transaction log)

**Sacrifices**:
- Events are low-level (row changes, not domain events)
- Requires transforming database changes into domain events
- Dependency on CDC infrastructure

**Implementation**:
```
1. Application updates database normally (single write)
2. CDC tool captures transaction log
3. Transform row changes into domain events
4. Publish transformed events to message queue
```

**When appropriate**:
- Existing systems with no event infrastructure
- Databases supporting CDC (PostgreSQL, MySQL, MongoDB)
- Want to avoid modifying application code

### Decision criteria

| Pattern | Atomic Writes | Event Granularity | Infrastructure | Schema Evolution |
|---------|--------------|------------------|----------------|------------------|
| Event log only | Inherent | Domain events | Event store | Complex |
| Transactional outbox | Yes | Domain events | Outbox processor | Medium |
| Change data capture | Yes | Low-level (rows) | CDC tool + transforms | Simple |

**Recommendation**: Use transactional outbox for most microservices architectures.
Use event sourcing (event log only) when complete history is required.
Avoid CDC unless retrofitting existing systems.

## Idempotency as architectural primitive

Distributed systems deliver messages at-least-once.
Operations must be safe to retry.
Idempotency must be designed in, not added later.

### Definition

An operation is idempotent if executing it multiple times has the same effect as executing it once.

**Mathematical definition**: `f(f(x)) = f(x)`

This corresponds to monoid identity laws: applying an operation multiple times has the same effect as applying it once.
See theoretical-foundations.md for the algebraic foundation of idempotency in distributed systems.

**Examples**:
- Idempotent: SET balance = 100 (same result regardless of repetition)
- Not idempotent: INCREMENT balance (result changes each time)

### Why required in distributed systems

**At-least-once delivery**: Message queues guarantee delivery but may deliver duplicates.
**Retry mechanisms**: Failed operations are retried, potentially succeeding multiple times.
**Network failures**: Unclear if operation succeeded when network fails mid-request.

**Without idempotency**:
- Duplicate messages cause duplicate effects (charge customer twice)
- Retries cause unintended state changes
- No safe way to handle uncertain outcomes

### Pattern 1: Idempotency keys (request-level deduplication)

**Approach**: Client provides unique key per request. Server tracks processed keys.

Client generates idempotency key (UUID, request ID).
Server checks if key already processed.
If yes, return cached result.
If no, process request, store key with result, return result.

**Guarantees**:
- Same request (same key) processed exactly once
- Retries safe (return cached result)

**Implementation**:
```
1. Client: POST /payment {"amount": 100, "idempotency_key": "uuid-1234"}
2. Server:
   - Check if uuid-1234 in processed_requests table
   - If found: return cached result
   - If not found:
     BEGIN TRANSACTION
       INSERT INTO processed_requests (key, result) VALUES ("uuid-1234", NULL)
       result = process_payment(100)
       UPDATE processed_requests SET result = result WHERE key = "uuid-1234"
     COMMIT
     return result
```

**When appropriate**:
- HTTP APIs (client retries)
- Payment processing
- Any operation with side effects

**Caching strategy**:
- Store keys with TTL (expire after 24 hours)
- Store result for successful operations
- Store error for failed operations (don't retry validation failures)

### Pattern 2: Natural idempotency

**Approach**: Design operations to be naturally idempotent.

Operations are inherently safe to retry without explicit deduplication.

**Examples**:

```
Idempotent by nature:
- SET status = "COMPLETED"
- DELETE record WHERE id = 123
- PUT /resource/123 {"name": "Alice"}
- Add item to set (sets ignore duplicates)

Not naturally idempotent:
- INCREMENT counter
- APPEND to list
- POST /resource (creates new resource each time)
```

**When appropriate**:
- State-based operations (set to specific value)
- HTTP PUT (replace entire resource)
- Database upserts (INSERT ... ON CONFLICT UPDATE)

### Pattern 3: Idempotent operations via preconditions

**Approach**: Operations check preconditions and succeed only once.

Use version numbers, timestamps, or state checks to ensure single application.

**Examples**:

```
Version-based:
  UPDATE account SET balance = 100 WHERE id = 123 AND version = 5

State-based:
  UPDATE order SET status = "SHIPPED" WHERE id = 123 AND status = "PENDING"
  (only succeeds if status is PENDING, retries are no-ops)

Timestamp-based:
  INSERT INTO events (id, timestamp) VALUES (123, "2025-01-01")
  (unique constraint on id prevents duplicates)
```

**Guarantees**:
- Operation succeeds once, fails or no-ops thereafter
- No separate deduplication infrastructure

**When appropriate**:
- Operations with clear preconditions
- Database-backed systems with version columns
- State machines (transitions only valid from specific states)

### Idempotency across service boundaries

When composing multiple services, idempotency must compose:

**Anti-pattern**:
```
Service A (idempotent) calls Service B (not idempotent)
- Retry in A causes duplicate call to B
- B processes duplicate
- System diverges
```

**Pattern**:
```
Service A generates idempotency key, passes to Service B
- Service A: POST /process {"key": "uuid-1234", ...}
- Service B: checks key, processes once
- Retry in A sends same key to B
- Service B returns cached result
- End-to-end idempotency
```

**Recommendation**: Require idempotency keys in all inter-service APIs with side effects.

## Saga patterns

Sagas coordinate distributed transactions across multiple services without requiring distributed locks or two-phase commit.

A saga is a sequence of local transactions where each service updates its own data and publishes an event or message.
If any step fails, compensating transactions undo completed steps.

**Key insight**: Sagas are isomorphic to actors (state + messages).
Each step is a message to a stateful service.
Compensation is reverse message flow.

### Pattern 1: Orchestration (central coordinator)

**Approach**: Central orchestrator service directs the saga.

Orchestrator knows all steps and their order.
Orchestrator sends commands to services.
Services respond with success or failure.
On failure, orchestrator triggers compensating transactions.

**Structure**:
```
Orchestrator
   ├─> Service A: do X
   ├─> Service B: do Y
   ├─> Service C: do Z
   └─> If failure: Service A: undo X, Service B: undo Y
```

**Guarantees**:
- Centralized control flow (easy to understand)
- Explicit compensation logic
- State machine in orchestrator

**Sacrifices**:
- Single point of failure (orchestrator)
- Tight coupling (orchestrator knows all services)
- Orchestrator becomes complex with many sagas

**Implementation**:
```
Orchestrator state machine:
  1. Send command to Service A
  2. Await response
  3. If success: Send command to Service B
  4. If failure: Send compensation to Service A, exit
  5. Continue for all steps
  6. If any step fails: walk backward sending compensations
```

**When appropriate**:
- Complex sagas with conditional logic
- Need centralized monitoring/observability
- Few sagas with well-defined steps

**Example**: Order processing saga:
```
1. Reserve inventory (Inventory Service)
2. Charge payment (Payment Service)
3. Ship order (Shipping Service)
4. If any step fails: reverse prior steps
```

### Pattern 2: Choreography (event-driven, no coordinator)

**Approach**: Services listen for events and decide their own actions.

No central orchestrator.
Each service knows what to do when certain events occur.
Services publish events, other services react.

**Structure**:
```
Service A: does X, publishes EventX
Service B: listens for EventX, does Y, publishes EventY
Service C: listens for EventY, does Z
Service B: listens for FailureZ, undoes Y, publishes CompensationY
Service A: listens for CompensationY, undoes X
```

**Guarantees**:
- Decoupled services (no central knowledge)
- No single point of failure
- Services evolve independently

**Sacrifices**:
- Difficult to understand global flow (implicit in event handlers)
- Debugging complexity (trace across multiple services)
- Risk of circular dependencies (event cycles)

**Implementation**:
```
Service A:
  on PlaceOrderCommand:
    reserve_inventory()
    publish InventoryReserved

Service B:
  on InventoryReserved:
    charge_payment()
    publish PaymentCharged

Service C:
  on PaymentCharged:
    ship_order()
    publish OrderShipped

Service B:
  on ShippingFailed:
    refund_payment()
    publish PaymentRefunded

Service A:
  on PaymentRefunded:
    release_inventory()
    publish InventoryReleased
```

**When appropriate**:
- Simple sagas (few steps)
- Services already event-driven
- High autonomy between services

**Example**: User signup saga:
```
UserService: creates user, publishes UserCreated
EmailService: sends welcome email
NotificationService: sends push notification
(No compensation needed, all steps eventually succeed)
```

### Compensating actions for rollback

Compensating transactions undo the effects of completed transactions.

**Not true rollback**: Original transaction is committed. Compensation creates new transaction to reverse effects.

**Semantic compensation**: Compensation must make semantic sense (not just database rollback).

**Examples**:

```
Transaction: Reserve inventory (decrement stock count)
Compensation: Release inventory (increment stock count)

Transaction: Charge credit card
Compensation: Refund credit card

Transaction: Send email
Compensation: Send cancellation email (cannot unsend original)
```

**Design guidelines**:

1. **Make compensations explicit**: Define compensation for every transaction at design time
2. **Compensations must be idempotent**: May be retried
3. **Compensations may fail**: Need retry logic and dead-letter queues
4. **Not all transactions are compensatable**: Sending email cannot be undone, only mitigated

**Partial compensation**:

Some transactions cannot be fully compensated.
Use best-effort compensation and notify users.

Example: Cannot cancel shipped order.
Compensation: Issue refund, arrange return pickup, notify user.

### Decision criteria

| Criteria | Orchestration | Choreography |
|----------|--------------|-------------|
| Complex logic | Better | Worse |
| Service coupling | High | Low |
| Observability | Easy (central view) | Hard (distributed trace) |
| Single point of failure | Yes (orchestrator) | No |
| Evolution | Hard (orchestrator changes) | Easy (services independent) |
| Debugging | Easy (state machine) | Hard (event flow) |

**Recommendation**: Use orchestration for complex sagas requiring control flow.
Use choreography for simple event-driven workflows.

## Deterministic replay

Deterministic replay rebuilds system state by re-executing operations from a log.
Critical for durability, debugging, and compliance.

**Theoretical foundation**: State reconstruction from event log is a catamorphism (fold) over the event sequence.
See theoretical-foundations.md for the category-theoretic interpretation of event replay as structural recursion.

### When to design for it

**Durability guarantees**:
- System must survive crashes and restore exact state
- Long-running workflows must resume after failures
- Actor systems with persistent state

**Debugging and auditing**:
- Reproduce bugs by replaying production events
- Time-travel debugging (step through history)
- Compliance auditing (prove exact sequence of operations)

**Compliance requirements**:
- Financial regulations requiring complete audit trail
- Medical systems requiring procedure history
- Legal discovery requiring exact event reconstruction

### Constraints deterministic replay imposes

**No non-deterministic operations in replay path**:

Operations must produce the same result when replayed.

**Forbidden**:
- Random number generation (different each replay)
- Wall-clock time (different each replay)
- Network calls (different responses)
- Reading from external systems (may change)

**Allowed**:
- Pure functions (deterministic by definition)
- Recorded external inputs (captured during original execution)
- Explicitly recorded timestamps (from original execution)

**Example**:

```
Bad (non-deterministic):
  let order_id = generate_random_uuid()  // Different each replay
  let timestamp = now()                   // Different each replay

Good (deterministic):
  let order_id = event.order_id          // Captured in event
  let timestamp = event.timestamp         // Captured in event
```

**Strict operation ordering**:

Operations must be totally ordered.
Concurrent operations are not deterministic during replay.

**Solution**: Assign sequence numbers or timestamps to establish total order.

**Snapshot + event replay hybrid**:

Full replay from beginning is expensive.
Use snapshots to reduce replay time.

**Pattern**:
```
1. Take snapshot of state at time T
2. Store snapshot
3. Replay events from time T to present
4. Reconstruct current state
```

**Guarantees**:
- Fast recovery (replay only recent events)
- Exact state reconstruction (snapshot + events)

**Tradeoffs**:
- Snapshot storage costs
- Snapshot consistency (must be transactionally consistent with event position)

### Implementation patterns

**Event sourcing**: Events are the source of truth, state derived.

```
1. Append event to log
2. Replay events from log to rebuild state
3. Snapshot state periodically
4. Replay only events since last snapshot
```

**Actor persistence (Akka, Orleans)**:

```
1. Actor processes command
2. Persist event to journal
3. Update in-memory state
4. On recovery: replay events from journal
```

**Database write-ahead log (WAL)**:

```
1. Write operation to WAL before applying to database
2. On crash: replay WAL to restore state
3. Checkpoint periodically to truncate WAL
```

### Determinism in practice

**Capture external inputs**:

Store responses from external systems in event log.
Replay uses recorded responses, not live calls.

**Example**:
```
Original execution:
  1. Call external service: response = fetch_price(product_id)
  2. Store event: PriceFetched(product_id, response)

Replay:
  1. Read event: PriceFetched(product_id, response)
  2. Use recorded response (no external call)
```

**Handle time carefully**:

Do not use `now()` during replay.
Store timestamps in events.

**Example**:
```
Command: PlaceOrder(items, timestamp)
  timestamp: client-provided or captured at ingestion

Event: OrderPlaced(order_id, items, timestamp)
  timestamp: from command (deterministic)
```

**Random number generation**:

Record random values in events.

**Example**:
```
Original execution:
  1. Generate: request_id = uuid4()
  2. Store event: RequestCreated(request_id)

Replay:
  1. Read event: RequestCreated(request_id)
  2. Use recorded request_id (no random generation)
```

## Architectural decision record template

Use this checklist before implementing distributed system features.

### 1. Authority model choice

**Question**: Who owns the current state?

- [ ] Event log as authority (event sourcing)
- [ ] Service as authority (events are notifications)
- [ ] Hybrid (event log for recovery, service for operations)

**Rationale**: [Explain why this authority model fits your requirements]

**Tradeoffs accepted**: [What are you sacrificing with this choice?]

### 2. Consistency model choice

**Question**: What consistency guarantees are required?

- [ ] Strong consistency (linearizability)
- [ ] Causal consistency
- [ ] Eventual consistency

**Rationale**: [Why this consistency model?]

**CAP choice**: During partitions, favor:
- [ ] Consistency (CP)
- [ ] Availability (AP)

**PACELC choice**: During normal operation, favor:
- [ ] Consistency (EC)
- [ ] Latency (EL)

### 3. Dual-write avoidance strategy

**Question**: How do you avoid dual-write problems?

- [ ] Event log only (single writer)
- [ ] Transactional outbox
- [ ] Change data capture
- [ ] Not applicable (single write target)

**Implementation**: [Describe how state and events remain consistent]

### 4. Idempotency approach

**Question**: How are operations made idempotent?

- [ ] Idempotency keys (request-level deduplication)
- [ ] Natural idempotency (state-based operations)
- [ ] Preconditions (version numbers, state checks)
- [ ] Combination: [specify]

**Cross-service propagation**: [How do idempotency keys flow across services?]

### 5. Failure handling strategy

**Question**: How are failures and partial failures handled?

**Distributed transactions**:
- [ ] Sagas (orchestration)
- [ ] Sagas (choreography)
- [ ] Not needed (single service transactions)

**Compensation**: [Describe compensating transactions for each step]

**Retry logic**:
- [ ] At-least-once delivery with idempotency
- [ ] At-most-once delivery (no retries)
- [ ] Exactly-once delivery (if achievable)

**Timeouts**: [Specify timeout values and backoff strategies]

### 6. Deterministic replay requirements

**Question**: Must the system support deterministic replay?

- [ ] Yes (for durability, debugging, compliance)
- [ ] No (not required)

**If yes**:
- [ ] No non-deterministic operations in replay path
- [ ] External inputs recorded in events
- [ ] Timestamps captured in events (not from `now()`)
- [ ] Total ordering of operations enforced
- [ ] Snapshot + event replay for efficiency

### 7. Observability and debugging

**Question**: How will you debug distributed operations?

- [ ] Distributed tracing (trace IDs across services)
- [ ] Structured logging with correlation IDs
- [ ] Metrics for latency, errors, saturation
- [ ] Event log for audit trail

**Correlation strategy**: [How are requests correlated across services?]

### Example completed template

**Feature**: Order processing system

**1. Authority model**: Service as authority (transactional outbox)
- Order service owns order state in PostgreSQL
- Events published via outbox pattern for notifications

**2. Consistency model**: Strong consistency within order service, eventual across services
- CAP: CP (consistency during partitions)
- PACELC: PC/EL (low latency during normal operation)

**3. Dual-write avoidance**: Transactional outbox
- Order updates and event writes in single transaction
- Outbox processor publishes events to Kafka

**4. Idempotency**: Idempotency keys on all APIs
- Client provides `idempotency_key` header
- Server stores keys in `processed_requests` table (24h TTL)

**5. Failure handling**: Orchestrated saga pattern
- Order orchestrator coordinates: reserve inventory, charge payment, ship
- Compensations: release inventory, refund payment
- At-least-once delivery with idempotency

**6. Deterministic replay**: Not required (no compliance needs)

**7. Observability**: OpenTelemetry with trace IDs
- Every request has trace_id propagated to all services
- Structured logs include trace_id

## See also

- `event-sourcing.md` (comprehensive event sourcing patterns, process managers, CQRS)
- `rust-development/11-concurrency.md` (local concurrency primitives)
- `rust-development/12-distributed-systems.md` (Rust-specific distributed implementations)
- `domain-modeling.md` (aggregates as consistency boundaries)
- `architectural-patterns.md` (effect isolation, onion architecture)
- `railway-oriented-programming.md` (Result types for distributed error handling)
- `theoretical-foundations.md` (algebraic foundations: event sourcing as free monoid, catamorphisms, profunctors)
- `hypermedia-development/07-event-architecture.md` (SSE streaming as event projection, CQRS in hypermedia context)
