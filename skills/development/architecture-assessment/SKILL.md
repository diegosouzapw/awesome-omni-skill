---
name: architecture-assessment
description: Evaluates codebase architecture for patterns, anti-patterns, coupling, cohesion, scalability, and technical debt. Use when assessing system design, reviewing architecture decisions, identifying improvement areas, or preparing for major refactoring.
---

# Architecture Assessment

Systematic evaluation of codebase architecture including pattern recognition, anti-pattern detection, coupling/cohesion analysis, scalability review, and technical debt identification.

## Quick Start

**Full architecture assessment:**
```
Perform a comprehensive architecture assessment of this codebase, identifying patterns, anti-patterns, and improvement opportunities.
```

**Pattern identification:**
```
Identify the architectural patterns used in this project and evaluate how well they're implemented.
```

**Technical debt analysis:**
```
Analyze this codebase for technical debt and prioritize items for remediation.
```

## Capabilities

### 1. Architecture Evaluation

Assess overall codebase architecture through structured analysis.

#### Evaluation Framework

```
1. STRUCTURE ANALYSIS
   - Directory organization
   - Module boundaries
   - Dependency flow
   - Layer separation

2. DESIGN PRINCIPLES
   - SOLID adherence
   - DRY compliance
   - Separation of concerns
   - Single responsibility

3. QUALITY ATTRIBUTES
   - Maintainability
   - Testability
   - Extensibility
   - Reliability

4. RISK ASSESSMENT
   - Complexity hotspots
   - Coupling issues
   - Change propagation
   - Failure domains
```

#### Architecture Health Score

| Dimension | Weight | Indicators |
|-----------|--------|------------|
| Modularity | 25% | Clear boundaries, low coupling, high cohesion |
| Maintainability | 25% | Complexity metrics, documentation, test coverage |
| Scalability | 20% | Statelessness, horizontal scaling potential |
| Testability | 15% | Dependency injection, isolation, mock-ability |
| Security | 15% | Input validation, auth boundaries, data protection |

**Scoring rubric:**
- 90-100: Excellent - Minor improvements only
- 70-89: Good - Some areas need attention
- 50-69: Fair - Significant improvements needed
- Below 50: Poor - Major refactoring required

---

### 2. Pattern Recognition

Identify and evaluate architectural patterns in the codebase.

#### Pattern Detection Matrix

| Pattern | Directory Indicators | Code Indicators |
|---------|---------------------|-----------------|
| MVC | `models/`, `views/`, `controllers/` | Route handlers, template rendering |
| MVVM | `viewmodels/`, `views/` | Two-way binding, observable state |
| Clean Architecture | `domain/`, `application/`, `infrastructure/` | Dependency inversion, use cases |
| Hexagonal | `ports/`, `adapters/` | Interface-based boundaries |
| Microservices | Multiple `package.json`, Dockerfiles | Service-to-service communication |
| Event-Driven | `events/`, `handlers/`, `subscribers/` | Event bus, message queues |
| CQRS | `commands/`, `queries/` | Separate read/write models |
| Repository | `repositories/` | Data access abstraction |

#### Pattern Evaluation Criteria

For each identified pattern, assess:

```markdown
## Pattern: {Name}

### Implementation Quality
- [ ] Pattern applied consistently across codebase
- [ ] Clear boundaries between pattern components
- [ ] Dependencies flow in correct direction
- [ ] Abstractions used appropriately

### Deviations
- List specific violations or inconsistencies
- Note areas where pattern is partially implemented
- Identify technical debt from pattern drift

### Recommendations
- Specific actions to improve implementation
- Priority (High/Medium/Low)
- Estimated effort
```

See [references/architecture-patterns.md](references/architecture-patterns.md) for detailed pattern analysis.

---

### 3. Anti-Pattern Detection

Identify architectural anti-patterns that harm maintainability and scalability.

#### Critical Anti-Patterns

| Anti-Pattern | Symptoms | Impact | Fix |
|--------------|----------|--------|-----|
| **God Class** | 1000+ lines, 20+ methods, knows everything | Untestable, change-resistant | Extract classes by responsibility |
| **Spaghetti Code** | No clear structure, tangled dependencies | Impossible to maintain | Introduce layers, extract modules |
| **Big Ball of Mud** | No architecture, everything coupled | Technical bankruptcy | Incremental strangler pattern |
| **Distributed Monolith** | Microservices with shared DB, sync calls | Worst of both worlds | True service boundaries |
| **Anemic Domain** | DTOs with no behavior, logic in services | Fragile, scattered logic | Rich domain models |
| **Circular Dependencies** | A depends on B, B depends on A | Cannot evolve independently | Dependency inversion |

#### Detection Commands

```bash
# Circular dependency detection
npx madge --circular src/
pydeps --show-cycles src/

# God class detection (files > 500 lines)
find src -name "*.ts" -exec wc -l {} \; | awk '$1 > 500'

# Coupling analysis
npx dependency-cruiser --output-type dot src | dot -T svg > deps.svg
```

See [references/anti-patterns.md](references/anti-patterns.md) for comprehensive anti-pattern catalog.

---

### 4. Coupling Analysis

Measure and evaluate coupling between modules.

#### Coupling Types (Worst to Best)

1. **Content Coupling** - Module modifies internal data of another
2. **Common Coupling** - Shared global state
3. **Control Coupling** - One module controls flow of another via flags
4. **Stamp Coupling** - Shared data structures with unused fields
5. **Data Coupling** - Only necessary data passed (ideal)
6. **Message Coupling** - Loosest, communication via messages only

#### Coupling Metrics

```markdown
## Module Coupling Report

### Afferent Coupling (Ca) - Who depends on me?
| Module | Dependents | Risk Level |
|--------|------------|------------|
| core/utils | 45 | HIGH - Changes break many |
| auth/jwt | 12 | MEDIUM |
| ui/Button | 8 | LOW |

### Efferent Coupling (Ce) - Who do I depend on?
| Module | Dependencies | Risk Level |
|--------|--------------|------------|
| pages/Dashboard | 23 | HIGH - Fragile |
| services/order | 8 | MEDIUM |
| utils/format | 2 | LOW |

### Instability Index
I = Ce / (Ca + Ce)
- 0.0 = Maximally stable (many dependents, few dependencies)
- 1.0 = Maximally unstable (few dependents, many dependencies)
```

#### Coupling Reduction Strategies

| Strategy | When to Use | Example |
|----------|-------------|---------|
| Dependency Injection | Control coupling | Pass dependencies via constructor |
| Interface Segregation | Stamp coupling | Create specific interfaces |
| Event Bus | Control coupling | Decouple via events |
| Facade | Multiple dependencies | Single entry point |

See [references/metrics.md](references/metrics.md) for coupling metric calculations.

---

### 5. Cohesion Assessment

Evaluate how well modules focus on single responsibilities.

#### Cohesion Types (Best to Worst)

1. **Functional** - All elements contribute to single task (ideal)
2. **Sequential** - Output of one element is input to next
3. **Communicational** - Elements operate on same data
4. **Procedural** - Elements related by execution order
5. **Temporal** - Elements execute at same time
6. **Logical** - Elements related by category, not function
7. **Coincidental** - No meaningful relationship (worst)

#### LCOM4 Metric

Lack of Cohesion of Methods:
- LCOM = 0: Perfect cohesion
- LCOM = 1: Acceptable
- LCOM > 1: Low cohesion, consider splitting

**Calculation:**
```
LCOM4 = Number of connected components in method-field graph
- Each method is a node
- Edge exists if two methods share a field
- LCOM4 = number of disconnected subgraphs
```

#### Cohesion Assessment Template

```markdown
## Cohesion Analysis: {ModuleName}

### Responsibility Inventory
1. {Responsibility 1}
2. {Responsibility 2}
...

### Cohesion Evaluation
- **Type**: {Functional|Sequential|etc.}
- **LCOM Score**: {0-N}
- **Field Usage Matrix**: Show which methods use which fields

### Recommendations
- [ ] Extract {Responsibility X} to new module
- [ ] Merge related methods
- [ ] Remove unrelated functionality
```

---

### 6. Scalability Review

Assess architecture for horizontal and vertical scaling capability.

#### Scalability Dimensions

| Dimension | Assessment Questions |
|-----------|---------------------|
| **Statelessness** | Can instances be added/removed freely? Is session state externalized? |
| **Data Partitioning** | Can data be sharded? Are there partition-limiting queries? |
| **Async Processing** | Are long-running tasks queued? Can work be distributed? |
| **Caching Strategy** | What's cached? Cache invalidation approach? |
| **Database Access** | Connection pooling? Read replicas? Query optimization? |
| **Service Dependencies** | Circuit breakers? Timeouts? Graceful degradation? |

#### Scalability Anti-Patterns

| Anti-Pattern | Impact | Detection | Fix |
|--------------|--------|-----------|-----|
| Sticky Sessions | Can't scale horizontally | Session affinity config | Externalize sessions (Redis) |
| Unbounded Queries | Memory exhaustion | `SELECT *` without LIMIT | Pagination, streaming |
| Sync Blocking Calls | Throughput bottleneck | Long-running HTTP calls | Async, message queues |
| Single Write Master | Write bottleneck | All writes to one DB | Sharding, CQRS |
| Memory-Bound Caching | Node memory limits | In-process cache | Distributed cache |

#### Scalability Checklist

See [checklists/scalability-checklist.md](checklists/scalability-checklist.md) for detailed assessment.

---

### 7. Technical Debt Identification

Find, categorize, and prioritize technical debt.

#### Debt Categories

| Category | Examples | Detection |
|----------|----------|-----------|
| **Architecture Debt** | Monolith needing split, wrong pattern | Architecture review |
| **Code Debt** | Complex functions, duplicates, dead code | Static analysis |
| **Dependency Debt** | Outdated packages, security vulns | `npm audit`, `pip-audit` |
| **Test Debt** | Low coverage, missing tests | Coverage reports |
| **Documentation Debt** | Outdated docs, missing API docs | Doc freshness check |
| **Infrastructure Debt** | Manual deployments, no IaC | Infrastructure review |

#### Debt Prioritization Matrix

```
              Impact
         Low    |    High
        +-------+--------+
High    | PLAN  | DO NOW |
Effort  +-------+--------+
Low     | DEFER | DO SOON|
        +-------+--------+
```

#### Debt Inventory Template

```markdown
## Technical Debt Inventory

### Critical (Fix within 2 weeks)
| ID | Description | Category | Impact | Effort | Owner |
|----|-------------|----------|--------|--------|-------|
| TD-001 | Security vulnerabilities in auth | Security | Critical | 2d | @dev |

### High (Fix within quarter)
| ID | Description | Category | Impact | Effort | Owner |
|----|-------------|----------|--------|--------|-------|
| TD-002 | Circular dependency in core | Architecture | High | 1w | @team |

### Medium (Track and address opportunistically)
...

### Low (Accept or defer)
...
```

See [checklists/debt-assessment-checklist.md](checklists/debt-assessment-checklist.md) for systematic debt discovery.

---

## Assessment Workflows

### Full Architecture Assessment

```
1. DISCOVERY (1-2 hours)
   [ ] Clone/access repository
   [ ] Review documentation (README, ADRs, wiki)
   [ ] Interview stakeholders on pain points
   [ ] Identify key business domains

2. STATIC ANALYSIS (2-4 hours)
   [ ] Run complexity analysis (lizard, escomplex)
   [ ] Run dependency analysis (madge, pydeps)
   [ ] Check test coverage
   [ ] Audit dependencies

3. PATTERN ANALYSIS (1-2 hours)
   [ ] Identify primary architecture pattern
   [ ] Evaluate pattern adherence
   [ ] Document deviations

4. COUPLING/COHESION (1-2 hours)
   [ ] Map module dependencies
   [ ] Calculate coupling metrics
   [ ] Assess module cohesion

5. ANTI-PATTERN SCAN (1-2 hours)
   [ ] Check for god classes
   [ ] Find circular dependencies
   [ ] Identify distributed monolith signs

6. SCALABILITY REVIEW (1-2 hours)
   [ ] Assess statelessness
   [ ] Review data access patterns
   [ ] Check async processing

7. DEBT INVENTORY (2-4 hours)
   [ ] Catalog all debt
   [ ] Prioritize by impact/effort
   [ ] Estimate remediation costs

8. REPORT & RECOMMENDATIONS (2-4 hours)
   [ ] Compile findings
   [ ] Prioritize recommendations
   [ ] Create roadmap
```

### Quick Health Check (30 min)

```
1. Structure scan - directory layout, layers
2. Complexity hotspots - top 5 complex files
3. Dependency health - vulnerabilities, outdated
4. Test coverage - overall percentage
5. Quick recommendations - top 3 issues
```

---

## Output Templates

### Architecture Assessment Report

```markdown
# Architecture Assessment: {Project Name}

## Executive Summary
- **Overall Health Score**: {X}/100
- **Primary Pattern**: {Pattern Name}
- **Critical Issues**: {Count}
- **Recommended Actions**: {Top 3}

## Assessment Details

### 1. Architecture Overview
- Pattern: {Identified pattern}
- Layers: {Layer breakdown}
- Key Components: {List}

### 2. Strengths
- {Strength 1}
- {Strength 2}

### 3. Concerns
| Priority | Issue | Impact | Recommendation |
|----------|-------|--------|----------------|
| Critical | ... | ... | ... |
| High | ... | ... | ... |

### 4. Metrics Summary
| Metric | Current | Target | Status |
|--------|---------|--------|--------|
| Avg Cyclomatic Complexity | 12 | <10 | WARN |
| Test Coverage | 65% | >80% | WARN |
| Dependency Health | 3 vulns | 0 | FAIL |

### 5. Recommendations Roadmap
#### Immediate (0-2 weeks)
1. ...

#### Short-term (1-3 months)
1. ...

#### Long-term (3-12 months)
1. ...

### 6. Technical Debt Summary
- Total Items: {count}
- Estimated Remediation: {time}
- Priority Distribution: {Critical: X, High: Y, Medium: Z}
```

---

## Tool Integration

### Static Analysis Commands

```bash
# JavaScript/TypeScript
npx escomplex src/ --format json > complexity.json
npx madge --circular --extensions ts src/
npx dependency-cruiser --output-type json src/ > deps.json

# Python
radon cc src/ -a -j > complexity.json
radon mi src/ -j > maintainability.json
pydeps src/ --cluster --max-bacon 2

# Go
gocyclo -over 10 ./...
gocognit -over 15 ./...

# Multi-language
lizard src/ -l python -l javascript --CCN 15

# Dependency health
npm audit --json
pip-audit --format=json
```

### Visualization

```bash
# Dependency graphs
npx madge --image deps.png src/
npx dependency-cruiser --output-type dot src | dot -T svg > deps.svg

# Architecture diagrams
npx arkit -o architecture.svg
```

---

## Reference Files

- [references/architecture-patterns.md](references/architecture-patterns.md) - Detailed pattern catalog and evaluation criteria
- [references/anti-patterns.md](references/anti-patterns.md) - Anti-pattern detection and remediation
- [references/metrics.md](references/metrics.md) - Coupling, cohesion, and complexity metrics
- [checklists/scalability-checklist.md](checklists/scalability-checklist.md) - Scalability assessment checklist
- [checklists/debt-assessment-checklist.md](checklists/debt-assessment-checklist.md) - Technical debt discovery checklist
