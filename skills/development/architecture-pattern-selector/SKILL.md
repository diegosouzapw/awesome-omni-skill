---
name: architecture-pattern-selector
description: Recommend architecture patterns (monolith, microservices, serverless, modular monolith) based on scale, team size, and constraints. Use when cto-architect needs to select the right architectural approach for a new system or migration.
---

# Architecture Pattern Selector

Provides systematic framework for selecting the right architecture pattern based on real-world constraints rather than hype or resume-driven development.

## When to Use

- Designing a new system from scratch
- Evaluating whether to migrate from monolith to microservices
- Choosing between serverless vs container-based architecture
- Assessing if current architecture matches actual needs

## Architecture Patterns

### 1. Monolith
**Best for**: Early-stage products, small teams, rapid iteration

**Characteristics**:
- Single deployable unit
- Shared database
- Simple deployment and debugging
- Fastest time to market

**Choose when**:
- Team < 10 engineers
- < 100K users
- Product-market fit not yet proven
- Need to iterate quickly on features
- Limited DevOps expertise

**Avoid when**:
- Different components need independent scaling
- Multiple teams need to deploy independently
- Parts of system have very different reliability requirements

### 2. Modular Monolith
**Best for**: Growing products that need structure without microservices complexity

**Characteristics**:
- Single deployable unit with clear module boundaries
- Modules communicate through defined interfaces
- Can evolve to microservices later
- Best of both worlds for mid-size teams

**Choose when**:
- Team 10-30 engineers
- 100K-1M users
- Need better code organization
- Want microservices benefits without operational overhead
- Planning future extraction to services

**Avoid when**:
- Need independent deployment per module
- Modules have fundamentally different scaling requirements
- Already have microservices expertise and infrastructure

### 3. Microservices
**Best for**: Large organizations with mature DevOps practices

**Characteristics**:
- Independent deployable services
- Service-specific databases
- Complex operational requirements
- High organizational overhead

**Choose when**:
- Team > 30 engineers with multiple squads
- > 1M users with varying load patterns
- Different services need different scaling strategies
- Strong DevOps/Platform team
- Clear bounded contexts

**Avoid when**:
- Team lacks Kubernetes/container orchestration experience
- No dedicated platform/DevOps team
- Bounded contexts are unclear
- Chasing microservices for resume value

### 4. Serverless
**Best for**: Event-driven workloads, variable traffic, cost optimization

**Characteristics**:
- Pay-per-execution pricing
- Auto-scaling to zero
- Vendor lock-in concerns
- Cold start latency

**Choose when**:
- Highly variable or spiky traffic
- Event-driven processing (webhooks, queues)
- Cost-sensitive with unpredictable load
- Simple request-response patterns
- Limited DevOps capacity

**Avoid when**:
- Consistent high-volume traffic (cost inefficient)
- Low-latency requirements (< 100ms consistently)
- Long-running processes
- Complex stateful workflows

### 5. Hybrid
**Best for**: Complex systems with varied requirements

**Characteristics**:
- Mix of patterns for different components
- Monolith core with serverless functions
- Microservices for specific bounded contexts

**Choose when**:
- Different parts of system have different characteristics
- Migrating incrementally from monolith
- Some components need independent scaling
- Event-driven workflows alongside synchronous APIs

---

## Selection Framework

### Step 1: Assess Current State

```
Team Size:        [ ] < 10   [ ] 10-30   [ ] 30-100   [ ] > 100
User Scale:       [ ] < 100K [ ] 100K-1M [ ] 1M-10M   [ ] > 10M
DevOps Maturity:  [ ] None   [ ] Basic   [ ] Intermediate [ ] Advanced
Deployment Freq:  [ ] Monthly [ ] Weekly [ ] Daily    [ ] Multiple/day
```

### Step 2: Evaluate Constraints

| Constraint | Impact on Pattern Choice |
|------------|-------------------------|
| Time to market | Favors monolith |
| Team autonomy | Favors microservices |
| Cost sensitivity | Favors serverless or monolith |
| Latency requirements | Disfavors serverless |
| Compliance/security | May require isolation (microservices) |

### Step 3: Apply Decision Matrix

See [decision-matrix.md](decision-matrix.md) for the scoring framework.

### Step 4: Consider Migration Path

Every architecture should have a migration path:
- Monolith → Modular Monolith → Microservices
- Monolith → Hybrid (extract specific services)
- Serverless → Containers (when scale justifies)

---

## Output Format

When recommending an architecture pattern, provide:

```markdown
## Architecture Recommendation

### Recommended Pattern: [Pattern Name]
**Confidence**: High / Medium / Low

### Why This Pattern
[2-3 specific reasons based on constraints]

### Trade-offs Accepted
- [Trade-off 1 and why it's acceptable]
- [Trade-off 2 and why it's acceptable]

### Migration Path
- **Current**: [Current state]
- **Phase 1**: [Near-term architecture]
- **Phase 2**: [If/when to evolve]
- **Trigger**: [What would cause migration to next phase]

### Patterns Rejected
| Pattern | Reason Not Selected |
|---------|---------------------|
| [Pattern] | [Specific reason] |

### Implementation Considerations
- [Key consideration 1]
- [Key consideration 2]
- [Key consideration 3]
```

---

## Anti-Patterns to Avoid

### Resume-Driven Architecture
Choosing microservices because it looks good on resumes, not because the problem requires it.

**Red flag**: "We should use microservices because that's what Netflix uses."
**Reality check**: You're not Netflix. What's your actual scale and team size?

### Premature Distribution
Distributing a system before understanding the domain boundaries.

**Red flag**: "Let's start with microservices from day one."
**Reality check**: You'll draw the wrong boundaries. Start monolith, extract when clear.

### Complexity Worship
Equating complexity with sophistication.

**Red flag**: "Our architecture needs to be enterprise-grade."
**Reality check**: Simple architectures that work beat complex ones that don't.

---

## References

- [Decision Matrix](decision-matrix.md) - Scoring framework for pattern selection
- [Pattern Comparison](pattern-comparison.md) - Detailed comparison of patterns
