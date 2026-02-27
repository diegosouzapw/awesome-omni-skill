---
name: plan-first-development
description: Break complex projects into detailed multi-phase plans before writing any code. Prevents scope creep, refactoring waste, and missed requirements. Use when starting any project with more than 1000 lines of code or building systems with multiple interacting components.
---

# Plan-First Development

Break complex projects into detailed multi-phase plans before writing any code. Prevents scope creep, refactoring waste, and missed requirements.

## When to use

- Starting any project with >1000 lines of code
- Building systems with multiple interacting components
- Performance-critical applications (HFT, real-time, embedded)
- Projects where requirements are complex but knowable upfront
- When multiple developers need coordination

## When NOT to use

- Exploratory prototypes where requirements are unknown
- Trivial scripts (<100 lines)
- Already well-understood patterns (CRUD apps)
- Research projects where discovery is the goal

## Instructions

### Step 1: Define Project Scope

Create a project overview document with:

```markdown
# [Project Name] - Implementation Plan

## Overview
[2-3 sentence description]

## Core Constraints
- ❌ [What we CANNOT use/do]
- ✅ [What we MUST use/do]

## Success Criteria
- [ ] [Measurable outcome 1]
- [ ] [Measurable outcome 2]
- [ ] [Performance target, if applicable]
```

### Step 2: Break Into Phases

Divide project into 5-10 sequential phases. Each phase should:
- Build on previous phases (dependencies clear)
- Have concrete deliverable (working code + tests)
- Take 1-4 hours to complete
- Be independently testable

**Template:**
```markdown
### Phase N: [Component Name]

**Objective**: [What are we building?]

**Files to create:**
- `path/to/file1.ext` - [Purpose]
- `path/to/file2.ext` - [Purpose]

**Key implementation details:**
- [Technical decision 1 with rationale]
- [Technical decision 2 with rationale]

**Dependencies:**
- Requires Phase [X] to be complete
- Provides [Y] for Phase [Z]

**Testing strategy:**
- Unit tests: [What to test]
- Integration tests: [If applicable]
- Performance validation: [Targets]

**Success criteria:**
- [ ] [Specific measurable outcome]
- [ ] [Tests passing]
- [ ] [Performance target met]
```

### Step 3: Specify Data Structures

For each major type/struct/class:

```markdown
**[StructName]** (`src/module.rs`):
```[language]
struct/class definition with comments
```

**Key implementation details:**
- Size: [X] bytes (if fixed)
- Alignment: [Y] bytes
- Memory: [Stack/Heap]
- Special properties: [Copy, Send, Sync, etc.]

**Compile-time assertions:**
```[language]
// Size verification
// Alignment verification
```
```

### Step 4: Define Critical Paths

Identify hot paths (performance-critical code):

```markdown
## Critical Performance Paths

**Path 1: [Operation Name]**
- Entry point: `function_name()`
- Steps:
  1. [Operation 1] - Target: <X ns/µs/ms
  2. [Operation 2] - Target: <Y ns/µs/ms
- Total target: <Z ns/µs/ms
- Memory access pattern: [Sequential/Random/...]
- Constraints: [No heap allocation, No locks, etc.]
```

### Step 5: Create Verification Checklist

End plan with measurable validation:

```markdown
## Verification Checklist

### Correctness
- [ ] All unit tests passing (target: X tests)
- [ ] Property tests verify invariants
- [ ] Concurrency tests pass (if applicable)

### Performance
- [ ] Operation A: <X ns/µs (measured)
- [ ] Operation B: <Y ns/µs (measured)
- [ ] Throughput: >Z ops/sec (sustained)

### Code Quality
- [ ] No heap allocations on hot path (verified with profiler)
- [ ] Zero unsafe warnings
- [ ] Documentation complete

### Platform Support
- [ ] Builds on [Platform 1]
- [ ] Tests pass on [Platform 2]
```

### Step 6: Get Approval & Execute

1. **Review plan** - Share with team/stakeholders
2. **Refine** - Adjust based on feedback
3. **Lock & Execute** - No major scope changes during implementation
4. **Validate per phase** - Check success criteria after each phase

## Examples

### Example 1: Planning a Lock-Free Queue

**Good plan structure:**
```markdown
## Phase 1: Core Types
- `src/types.rs` - Element wrapper (32 bytes, repr(C))
- Compile-time size assertions
- Zero-copy serialization
- Tests: size, alignment, serialization round-trip

## Phase 2: SPSC Ring Buffer
- `src/ring.rs` - Single-producer, single-consumer queue
- Cache-line padded atomics (64 bytes)
- Power-of-two size for fast modulo
- Release/Acquire memory ordering
- Tests: push/pop, wrap-around, FIFO order, concurrent access (Loom)
- Target: <50ns per push+pop

## Phase 3: Benchmarking
- `benches/ring_bench.rs` - Criterion benchmarks
- Measure P50, P99, P99.9 latency
- Validate <50ns target met
```

**Why this works:**
- Phase 1 validates data layout before building on it
- Phase 2 completes core algorithm with tests
- Phase 3 verifies performance before adding features

### Example 2: Planning a REST API

```markdown
## Phase 1: Database Schema
- migrations/001_users.sql
- migrations/002_posts.sql
- Seed data for testing
- Tests: schema validation, constraints

## Phase 2: Repository Layer
- src/repositories/user.rs
- src/repositories/post.rs
- CRUD operations
- Tests: all operations, error cases

## Phase 3: Service Layer
- src/services/auth.rs (JWT)
- src/services/posts.rs (business logic)
- Tests: authorization, validation

## Phase 4: HTTP Handlers
- src/handlers/auth.rs
- src/handlers/posts.rs
- Tests: integration tests, E2E

## Phase 5: Performance & Security
- Benchmarks: target <10ms P99
- Security audit: SQL injection, XSS, CSRF
- Load test: 1000 req/sec sustained
```

## Best Practices

### ✅ Do

- **Write plan in markdown** - Easy to read, version control, share
- **Include "why" not just "what"** - Document rationale for decisions
- **Set measurable targets** - "Fast" → "<50ns P99 latency"
- **Phase dependencies clear** - Use arrows/diagrams
- **Test strategy per phase** - Don't leave testing to end
- **Review before coding** - 30 min planning saves hours debugging

### ❌ Don't

- **Don't plan implementation details** - Plan structure, not every function
- **Don't skip performance targets** - You'll optimize wrong things later
- **Don't make phases too large** - Max 4 hours each
- **Don't change plan mid-execution** - Finish phase, then reassess
- **Don't skip validation checklist** - How will you know you're done?

## Common Pitfalls

### Pitfall 1: "I'll figure it out as I go"
**Problem:** Leads to refactoring entire codebase when you discover architectural mismatch

**Solution:** Spend 10-15% of project time on planning upfront

### Pitfall 2: Too much detail
**Problem:** Planning every function becomes maintenance burden

**Solution:** Plan modules/components, not individual functions

### Pitfall 3: Ignoring dependencies
**Problem:** Build Phase 5 before Phase 2, realize Phase 2 API doesn't work

**Solution:** Draw dependency graph, execute in topological order

### Pitfall 4: No performance targets
**Problem:** Build entire system, discover it's 10x too slow, no idea where bottleneck is

**Solution:** Set targets per phase, measure immediately

## Integration with Development Tools

### With GitHub Projects
- Create one issue per phase
- Label: `phase-1`, `phase-2`, etc.
- Each issue has checklist from success criteria

### With Git
```bash
# Branch per phase
git checkout -b phase-1-core-types
# ... implement & test ...
git checkout main
git merge phase-1-core-types

git checkout -b phase-2-ring-buffer
# ... etc
```

### With CI/CD
- Run tests after each phase merge
- Benchmark comparisons in PR comments
- Block merge if phase success criteria not met

## Measuring Success

### Indicators plan worked:
- ✅ Few/no major refactorings during implementation
- ✅ All tests written concurrently with code
- ✅ Performance targets met in final product
- ✅ Schedule variance <20% (planned vs actual time)
- ✅ Team members understand architecture

### Indicators plan failed:
- ❌ Constant "back to the drawing board" moments
- ❌ Tests written as afterthought
- ❌ Performance surprises at end
- ❌ Schedule blown (2x+ planned time)
- ❌ Confusion about component responsibilities

## Related skills

- incremental-validation - Test after each phase
- parallel-subagent-orchestration - Execute validation in parallel
- documentation-while-fresh - Document decisions in plan

---

**Skill Version:** 1.0
**Last Updated:** 2025-01-06
