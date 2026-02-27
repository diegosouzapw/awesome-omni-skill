---
name: principal-engineer
description: Principal Engineer Skill
---

## The Four Phases

You MUST complete each phase before proceeding to the next.

### Phase 1: Constraints & Trade-offs (The "No")

**BEFORE designing the system:**

1.  **Identify Non-Functional Requirements (NFRs)**
    - Scalability: 1k users or 10M users?
    - Latency: Real-time (Gaming) or Batch (Reporting)?
    - Compliance: HIPAA? GDPR? PCI-DSS?
    - **Rule:** These constraints dictate the tech stack, not your personal preference.

2.  **The "Buy vs. Build" Decision**
    - Should we build a Chat system or use Stream.io?
    - Should we build Auth or use Auth0?
    - **Rule:** Only build your "Core Differentiator." Buy everything else.

3.  **Define the Boundaries**
    - Where does the Mobile App end and the Backend begin?
    - Monolith or Microservices? (Hint: Start with Monolith).
    - **Output:** A high-level context diagram showing system interactions.

### Phase 1.5: Modern Architecture Patterns (2026)

**Emerging patterns that change the game:**

1.  **Edge Compute Strategies**
    - **Cloudflare Workers / Vercel Edge:** Run code closest to users (sub-50ms latency)
    - **Use Cases:** A/B testing, personalization, auth checks, rate limiting
    - **Trade-off:** Limited runtime (no filesystem, max 50ms CPU time)
    - **When:** Global apps needing <100ms response times

2.  **WebAssembly (WASM) Backends**
    - **Compile Rust/Go to WASM:** Run in serverless with near-native performance
    - **Benefits:** 10x faster cold starts vs containers, tiny bundles
    - **Tools:** wasmtime, Spin (Fermyon), WASI
    - **When:** CPU-intensive serverless functions (image processing, crypto)

3.  **Local-First Architecture**
    - **CRDTs:** Conflict-free Replicated Data Types (eventual consistency)
    - **Offline-First:** App works without network, syncs when available
    - **Tools:** Replicache, Electric SQL, PowerSync
    - **When:** Collaborative apps (Figma, Notion patterns)

4.  **AI Integration Points**
    - **Where to embed LLMs:** Content generation, search, recommendations
    - **Vector Databases:** pgvector (Postgres), Pinecone, Weaviate
    - **Cost Management:** Cache embeddings, batch requests, use smaller models
    - **Latency:** Stream responses (SSE/WebSockets), don't block UI

### Phase 2: Technical Consensus

**Aligning the experts:**

1.  **Write the RFC (Request for Comments)**
    - Don't dictate. Write a design doc proposing the solution.
    - List "Alternatives Considered" and why they were rejected.
    - **Review:** Get sign-off from the Lead Backend, Lead Mobile, and DevOps.

2.  **Data Flow & Consistency**
    - **Source of Truth:** Which database owns the "User Profile"?
    - **Consistency Model:** Strong (Banking) or Eventual (Social Feed)?
    - **Data Gravity:** Don't move massive data to the compute; move compute to the data.

3.  **Failure Mode Analysis**
    - "What happens if the Payment Gateway is down?"
    - "What happens if the Redis Cache is flushed?"
    - Design for resilience, not perfection.

### Phase 2.5: Observability-Driven Development

**Build systems you can actually debug:**

1.  **Think in Traces, Not Logs**
    - **OpenTelemetry Standard:** Unified traces, metrics, logs
    - **Trace ID Propagation:** Follow a request across 10 microservices
    - **Span Attributes:** Enrich with context (user_id, feature_flag, A/B test)
    - **Tools:** Jaeger, Tempo, Honeycomb, Datadog APM

2.  **Structured Events Over Text Logs**
    ```json
    {
      "trace_id": "abc123",
      "span_id": "xyz789",
      "service": "payment-api",
      "event": "charge_created",
      "user_id": 42,
      "amount_cents": 1999,
      "duration_ms": 234
    }
    ```
    - **Benefits:** Query like a database, not grep
    - **Anti-Pattern:** `"User 42 charged $19.99 in 234ms"` (unparseable)

3.  **Define SLOs Before SLAs**
    - **SLO:** Internal target (99.9% = 43min downtime/month)
    - **SLA:** External commitment to customers
    - **Error Budgets:** If SLO is 99.9%, you have 0.1% to "spend" on deploys/experiments
    - **Rule:** If you burn 50% of error budget, freeze risky changes

### Phase 3: Governance & Standards

**Setting the rules of the road:**

1.  **Tech Radar**
    - **Adopt:** TypeScript, Terraform, Postgres. (Safe to use).
    - **Trial:** Go, Rust. (Use with caution/permission).
    - **Hold:** MongoDB, PHP. (Do not use for new services).

2.  **Standardize Interfaces**
    - "All APIs must return standard HTTP error codes."
    - "All logs must be JSON."
    - "All services must expose a /metrics endpoint."
    - "MANDATORY: All code changes must undergo a deep-dive 'fine-toothed comb' review before git deployment."

3.  **Cost Governance**
    - Review the architecture for hidden costs (e.g., Cross-AZ data transfer).
    - Set the budget ceiling per active user.

### Phase 4: Evolution & Modernization

**Managing the rot:**

1.  **Tech Debt Management**
    - Treat debt like a financial loan. Pay interest (refactoring) regularly.
    - Identify "Load Bearing" legacy code that needs replacement.

2.  **The Strangler Fig Pattern**
    - Don't rewrite the old system from scratch (The "Big Bang" fails).
    - Build new features on the new stack and slowly route traffic away from the old one.

3.  **Mentorship**
    - Your job is to make Senior Engineers into Architects.
    - Explain *why* you made a decision, don't just give orders.

## Red Flags - STOP and Follow Process

If you catch yourself thinking:
- "Let's use Kubernetes because Google uses it." (You are not Google).
- "Microservices will solve our spaghetti code." (Now you have distributed spaghetti).
- "I don't need to write this down, it's in my head." (Bus factor risk).
- "We'll figure out how to migrate the data later." (Data gravity is real).
- "I'll let every team choose their own language." (Operational nightmare).
- **Pushing to deployment without a 'fine-toothed comb' code review.**

**ALL of these mean: STOP. Return to Phase 1.**

## Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Constraints** | Buy vs Build, NFRs | Tech stack aligned with Biz |
| **2. Consensus** | RFCs, Design Review | Team buy-in, clear boundaries |
| **3. Governance** | Tech Radar, Standards | Consistent ecosystem |
| **4. Evolution** | Refactoring, Mentoring | System survives > 5 years |

## 📐 Architecture Decision Record (ADR) Template

Use this for every significant technical decision:

```markdown
# ADR-###: [Title]

**Status:** Proposed | Accepted | Deprecated | Superseded  
**Date:** 2026-01-29  
**Deciders:** [Names]  

## Context

What forces are at play? Business constraints, technical debt, team skillsets.

## Decision

We will [decision]. We chose [option A] over [option B].

## Consequences

### Positive
- What becomes easier?
- What performance/cost benefits?

### Negative
- What becomes harder?
- What new risks?

### Neutral
- What stays the same?

## Alternatives Considered

| Option | Pros | Cons | Why Rejected |
|--------|------|------|---------------|
| Option A (chosen) | ... | ... | N/A |
| Option B | ... | ... | ... |
| Option C | ... | ... | ... |

## References

- [Link to RFC]
- [Link to prototype]
```

## 🛠️ Modern Architecture Stack (2026)

### Compute Patterns
- **Serverless:** AWS Lambda, Cloud Run, Cloudflare Workers
- **Containers:** ECS/Fargate, Cloud Run, Fly.io
- **Edge:** Cloudflare Workers, Vercel Edge, Fastly Compute
- **WASM:** Spin (Fermyon), wasmCloud

### Data Patterns
- **OLTP:** Postgres 17, Neon (serverless), Supabase
- **OLAP:** ClickHouse, BigQuery, DuckDB
- **Caching:** Redis 7, Valkey, Dragonfly
- **Vector:** pgvector, Pinecone, Weaviate
- **Local-First:** Replicache, ElectricSQL, PowerSync

### Observability
- **Standard:** OpenTelemetry
- **APM:** Datadog, New Relic, Honeycomb
- **Logs:** Grafana Loki, CloudWatch, Mezmo
- **Metrics:** Prometheus, Grafana, Datadog

### Cost Optimization
- **FinOps:** Spot instances, reserved capacity
- **Auto-scaling:** KEDA, AWS Auto Scaling
- **Budget Alerts:** CloudWatch billing, Infracost
