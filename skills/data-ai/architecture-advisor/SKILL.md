---
name: architecture-advisor
description: Helps solo developers with AI agents choose optimal architecture (monolithic/microservices/hybrid)
version: 1.0.0
author: liri-skills
tags: [architecture, decision-support, solo-developer, ai-assisted]
---

# Architecture Advisor

> Evaluates project requirements to recommend the best architecture for solo developers working with AI agents

## When to Use

- Starting a new project and unsure which architecture to choose
- Reconsidering architecture of an existing project
- Need justification for architectural decisions
- Want to understand trade-offs for your specific situation
- Planning future extraction from monolith to microservices

## When NOT to Use

- Already have a team of 5+ developers (traditional frameworks apply)
- Need detailed system design (this is high-level architecture only)
- Looking for technology/framework recommendations (separate concern)

---

## Orchestrator

This skill coordinates the following sub_agents:

| Sub-Agent | Purpose | Invoked During |
|-----------|---------|----------------|
| `input-collector` | Gathers project requirements via structured questions | Step 1 |
| `factor-analyzer` | Evaluates inputs against weighted decision criteria | Step 2 |
| `recommendation-engine` | Determines best architecture with confidence score | Step 3 |
| `rationale-generator` | Produces context-specific pros/cons and justification | Step 4 |

---

## Workflows

### Primary: Evaluate Architecture
**Path**: `workflows/evaluate-architecture.md`

Full evaluation workflow:
1. **Collect Inputs** → Gather project info, scale, constraints
2. **Analyze Factors** → Score against decision framework
3. **Generate Recommendation** → Determine best fit with confidence
4. **Build Rationale** → Create personalized explanation
5. **Present Report** → Formatted decision document

---

## Context Integration

### Target User Profile
- Solo developer (team of 1)
- Uses AI agents for development (Claude, Copilot, Cursor, etc.)
- Needs architecture they can maintain alone with AI assistance

### Key Insight
Traditional architecture decision frameworks assume teams. This skill is calibrated for the reality of one person + AI agents, where:
- Operational simplicity beats theoretical scalability
- AI works better with full codebase context (favors monoliths)
- Deployment complexity directly impacts development velocity

---

## Commands

### `/architecture-advisor`
Run full architecture evaluation workflow.

**Usage**:
```
/architecture-advisor
> Answer questions about your project
> Receive recommendation with rationale
```

### `/architecture-advisor:quick "<description>"`
Quick evaluation with sensible defaults.

**Usage**:
```
/architecture-advisor:quick "e-commerce site, 5K users, bootstrap budget"
```

---

## Implementation

### Entry Point Logic

When `/architecture-advisor` is invoked:

```
1. Initialize evaluation state
2. Run input-collector sub-agent
3. Run factor-analyzer on collected inputs
4. Run recommendation-engine on analysis
5. Run rationale-generator on recommendation
6. Format and present final report
```

### State Management

```yaml
state:
  project_input: null | ProjectInput
  factor_analysis: null | FactorAnalysis
  recommendation: null | Recommendation
  rationale: null | Rationale
```

---

### Decision Framework

#### Factors and Weights (Solo Developer Calibration)

| Factor | Weight | Why This Weight |
|--------|--------|-----------------|
| Team Size | 25% | Solo = always relevant; this is the dominant factor |
| AI-Friendliness | 20% | AI agents are your "team"; architecture affects their effectiveness |
| Deployment Simplicity | 15% | No DevOps team; you manage everything |
| Debugging Ease | 15% | No SRE team; you troubleshoot alone |
| Expected Scale | 10% | Matters less than operational simplicity for solo |
| Domain Complexity | 10% | Clear domains might justify separation eventually |
| Future Flexibility | 5% | Nice to have but don't over-optimize |

#### Score Calculation

```python
def calculate_architecture_scores(inputs):
    factors = [
        ('team_size', 0.25, evaluate_team_size),
        ('ai_friendliness', 0.20, evaluate_ai_fit),
        ('deployment', 0.15, evaluate_deployment),
        ('debugging', 0.15, evaluate_debugging),
        ('scale', 0.10, evaluate_scale),
        ('domain', 0.10, evaluate_domain),
        ('flexibility', 0.05, evaluate_flexibility),
    ]

    scores = {'monolith': 0, 'microservices': 0, 'hybrid': 0}

    for name, weight, evaluator in factors:
        factor_scores = evaluator(inputs)
        for arch in scores:
            scores[arch] += weight * factor_scores[arch]

    return scores
```

---

### Factor Evaluation Details

#### Team Size (25%)
**Solo developer always favors monolith**

| Scenario | Monolith | Microservices | Hybrid |
|----------|----------|---------------|--------|
| Solo + AI agents | 100 | 10 | 50 |

*Rationale*: One person cannot effectively manage distributed system operations. This is the strongest signal.

#### AI-Friendliness (20%)
**AI tools work best with full codebase context**

| Development Style | Monolith | Microservices | Hybrid |
|-------------------|----------|---------------|--------|
| AI-first | 90 | 40 | 70 |
| AI-assisted | 85 | 50 | 70 |
| Traditional | 70 | 60 | 65 |

*Rationale*: Claude/Copilot can see entire monolith; microservices fragment context.

#### Deployment Simplicity (15%)
**Solo devs need simple deployments**

| DevOps Comfort | Monolith | Microservices | Hybrid |
|----------------|----------|---------------|--------|
| Beginner | 100 | 20 | 50 |
| Intermediate | 90 | 40 | 65 |
| Advanced | 80 | 60 | 75 |

*Rationale*: Microservices require K8s, service mesh, etc.—too much for one person.

#### Debugging Ease (15%)
**Stack traces beat distributed tracing for solo devs**

| Domain Complexity | Monolith | Microservices | Hybrid |
|-------------------|----------|---------------|--------|
| Simple | 95 | 50 | 70 |
| Medium | 85 | 45 | 70 |
| Complex | 75 | 40 | 65 |

#### Expected Scale (10%)
**Scale threshold for microservices is high**

| Expected Users | Monolith | Microservices | Hybrid |
|----------------|----------|---------------|--------|
| <10K | 95 | 30 | 50 |
| 10K-100K | 80 | 50 | 70 |
| 100K-1M | 60 | 70 | 80 |
| >1M | 40 | 85 | 75 |

*Rationale*: Microservices overhead only justified at significant scale.

#### Domain Complexity (10%)
**Clear boundaries might justify extraction later**

| Domains | Monolith | Microservices | Hybrid |
|---------|----------|---------------|--------|
| Single | 95 | 30 | 50 |
| 2-3 | 80 | 50 | 75 |
| 4+ | 60 | 70 | 80 |

#### Future Flexibility (5%)
**Plan for success, but don't over-engineer**

| Growth Plan | Monolith | Microservices | Hybrid |
|-------------|----------|---------------|--------|
| Stay solo | 90 | 30 | 60 |
| Maybe hire | 70 | 50 | 80 |
| Team planned | 50 | 70 | 80 |

---

### Architecture Variants

#### Monolith Variants
| Variant | When to Recommend |
|---------|-------------------|
| `simple-monolith` | Simple domain, <10K users, single platform |
| `modular-monolith` | Medium complexity, growth expected |
| `api-first-monolith` | Multiple platforms (web + mobile) |

#### Hybrid Variants
| Variant | When to Recommend |
|---------|-------------------|
| `monolith-plus-service` | Specific high-load component identified |
| `monolith-with-integration-layer` | Heavy third-party integrations |
| `monolith-plus-realtime` | Real-time + CRUD mixed workloads |

---

### Hard Rules (Override Scores)

These conditions override calculated scores:

| Condition | Rule |
|-----------|------|
| DevOps = beginner | Never recommend microservices |
| Budget = bootstrap | Favor simplest deployment |
| Time to MVP < 2 months | Always recommend monolith |
| Solo + microservices score high | Reduce confidence, add warnings |

---

### Output Format

```
╔══════════════════════════════════════════════════════════════╗
║  Architecture Recommendation: {PROJECT_NAME}                  ║
╠══════════════════════════════════════════════════════════════╣

📋 RECOMMENDATION: {ARCHITECTURE} ({VARIANT})
   Confidence: {SCORE}%

📝 SUMMARY
   {Personalized 2-3 sentence summary}

✅ WHY THIS FITS YOUR SITUATION
   • {Dominant factor 1}
   • {Dominant factor 2}
   • {Dominant factor 3}

👍 PROS (Solo Developer Context)
   • {Pro}: {How it helps you specifically}

⚠️  CONS (With Mitigations)
   • {Con}: {How to address it}

🤖 AI AGENT CONSIDERATIONS
   • {How architecture affects AI tool effectiveness}

🔄 ALTERNATIVE: {ALT_ARCHITECTURE}
   Consider if: {Trigger conditions}

📌 NEXT STEPS
   1. {Immediate action}
   2. {Setup action}
   3. {Documentation action}

╚══════════════════════════════════════════════════════════════╝
```

---

### Common Recommendations

#### 90%+ of Solo Projects: Modular Monolith

```yaml
recommendation: modular-monolith
confidence: 0.85
summary: >
  For most solo developers with AI agents, a modular monolith
  provides the best balance of simplicity and future flexibility.
  AI tools work effectively with full codebase context, deployment
  is simple, and you can extract services later if needed.

next_steps:
  - Design clear module boundaries based on domain concepts
  - Set up single deployment (Vercel, Railway, Fly.io)
  - Create CLAUDE.md documenting patterns for AI assistants
  - Implement feature flags for safe deployments
  - Add monitoring to detect when extraction might help
```

#### When Hybrid Makes Sense

```yaml
triggers:
  - Specific component with 10x higher load than others
  - Real-time requirements (chat, live updates) alongside CRUD
  - Third-party integration that benefits from isolation

recommendation: hybrid
variant: monolith-plus-service
next_steps:
  - Keep core business logic in monolith
  - Extract only the specific high-load/real-time component
  - Use simple service communication (HTTP, not message queues)
  - Share authentication between monolith and service
```

#### Rare: When Microservices Might Apply

```yaml
triggers:
  - Proven scale >1M users (not projected)
  - Advanced DevOps experience
  - Multiple clearly independent products
  - Team growth imminent

recommendation: microservices
warning: >
  Even with these triggers, consider starting with modular monolith
  and extracting. Microservices add significant operational burden
  that's hard to manage solo.
```

---

### Example Evaluations

#### Example 1: Indie SaaS Product

**Input**:
```yaml
project:
  name: "TaskFlow"
  description: "Project management tool for freelancers"
  platforms: [web]
  domain: "productivity-saas"
scale:
  expected_users: 5000
  expected_growth: "moderate"
solo_context:
  ai_tools_used: [claude, cursor]
  development_style: "ai-first"
  devops_comfort: "intermediate"
constraints:
  time_to_mvp: "3 months"
  budget: "bootstrap"
```

**Analysis**:
```
Team Size (25%):     Monolith: 100 | Microservices: 10 | Hybrid: 50
AI-Friendliness (20%): Monolith: 90 | Microservices: 40 | Hybrid: 70
Deployment (15%):    Monolith: 90 | Microservices: 40 | Hybrid: 65
Debugging (15%):     Monolith: 85 | Microservices: 45 | Hybrid: 70
Scale (10%):         Monolith: 95 | Microservices: 30 | Hybrid: 50
Domain (10%):        Monolith: 95 | Microservices: 30 | Hybrid: 50
Flexibility (5%):    Monolith: 90 | Microservices: 30 | Hybrid: 60
─────────────────────────────────────────────────────────────────
TOTAL:               Monolith: 92 | Microservices: 31 | Hybrid: 59
```

**Recommendation**: `modular-monolith` (Confidence: 92%)

---

#### Example 2: Real-Time Collaboration Tool

**Input**:
```yaml
project:
  name: "CollabDocs"
  description: "Real-time document editing with AI assistance"
  platforms: [web, desktop]
  domain: "collaboration"
scale:
  expected_users: 50000
  real_time_requirements: true
solo_context:
  ai_tools_used: [claude]
  development_style: "ai-assisted"
  devops_comfort: "intermediate"
constraints:
  time_to_mvp: "6 months"
  budget: "moderate"
```

**Recommendation**: `monolith-plus-realtime` (Hybrid, Confidence: 78%)

*Rationale*: Real-time collaboration (WebSocket connections) benefits from a dedicated service, while core document CRUD stays in monolith. This keeps most code in full AI context while isolating the scaling-sensitive real-time component.

---

#### Example 3: E-Commerce Platform

**Input**:
```yaml
project:
  name: "ArtisanMarket"
  description: "Marketplace for handmade goods"
  platforms: [web, ios, android]
  domain: "e-commerce"
scale:
  expected_users: 100000
  expected_growth: "aggressive"
solo_context:
  ai_tools_used: [claude, copilot]
  development_style: "ai-first"
  devops_comfort: "advanced"
constraints:
  time_to_mvp: "4 months"
  budget: "moderate"
  compliance: ["PCI-DSS"]
```

**Recommendation**: `api-first-monolith` (Confidence: 75%)

*Rationale*: Multiple platforms require clean API boundaries. Even with aggressive growth expectations, start monolithic with API-first design. Extract payment processing to separate service only if PCI compliance scope reduction is needed.

---

### Input Collection

Questions gathered by `input-collector` sub-agent:
1. Project identity (name, description, platforms, domain)
2. Scale expectations (users, growth, real-time needs)
3. Solo context (AI tools, development style, DevOps comfort)
4. Constraints (timeline, budget, compliance)

See `sub_agents/input-collector.md` for detailed question flow.

---

### Error Handling

#### Input Validation Errors

| Error | Detection | Resolution |
|-------|-----------|------------|
| Missing project name | Required field empty | Prompt for name |
| Invalid scale | Non-numeric or out of range | Show valid options |
| Contradictory inputs | Bootstrap budget + >1M users | Flag and discuss expectations |
| Incomplete context | <50% questions answered | Use defaults, reduce confidence |

#### Analysis Errors

| Error | Detection | Resolution |
|-------|-----------|------------|
| Scores too close | Top 2 within 5 points | Present both options, explain trade-offs |
| No clear winner | All scores within 15 points | Gather more context or recommend safest (monolith) |
| Hard rule conflict | Calculated score vs hard rule mismatch | Hard rule wins, explain override |

#### Output Errors

| Error | Detection | Resolution |
|-------|-----------|------------|
| Confidence too low | < 0.5 | Add uncertainty disclaimer, suggest revisiting inputs |
| Missing rationale | Sub-agent returned incomplete | Generate fallback rationale from factors |

---

### Technology Stack Suggestions

Based on architecture recommendation, suggest stacks optimized for solo + AI development:

#### Monolith Stacks

| Stack | Best For | AI-Friendly Features |
|-------|----------|---------------------|
| **Next.js + Prisma** | Web apps, full-stack | Single repo, great TypeScript support |
| **Django + HTMX** | Content-heavy, rapid prototyping | Excellent ORM, admin panel |
| **Rails** | CRUD-heavy apps | Convention over configuration |
| **Laravel** | PHP ecosystem | Eloquent ORM, built-in auth |
| **Go + Templ** | Performance-critical | Single binary deployment |

#### Hybrid Stacks

| Component | Monolith | Extracted Service |
|-----------|----------|-------------------|
| Core app | Next.js, Django, Rails | - |
| Real-time | - | Elixir/Phoenix, Go, Rust |
| Heavy processing | - | Python worker, Go service |
| Search | - | Elasticsearch, Typesense |

---

## Research Basis

This decision framework is based on:

- [2025 industry consensus](https://medium.com/@kittikawin_ball/microservices-vs-monoliths-architecture-decision-framework-for-2025-98c8ff2ec484): Below 10 devs, monoliths perform better
- [Amazon Prime Video case](https://foojay.io/today/monolith-vs-microservices-2025/): Cut costs 90% returning to monolith
- [AI-first development patterns](https://www.makingdatamistakes.com/ai-first-development/): Simple architectures work best with AI
- [Solo AI startup best practices](https://www.nucamp.co/blog/solo-ai-tech-entrepreneur-2025-setting-up-a-selfhosted-solo-ai-startup-infrastructure-best-practices): Solo founders can build at scale with right architecture

---

## Configuration

| Setting | Default | Override |
|---------|---------|----------|
| detailed_output | true | `--brief` |
| include_alternatives | true | `--no-alternatives` |
| factor_weights | solo-calibrated | `--weights=traditional` |

---

## Orchestrator Agent

This skill has an associated orchestrator agent at `.claude/agents/architecture-advisor.md` that coordinates the sub-agents. The orchestrator:

- Runs input-collector to gather project requirements
- Runs factor-analyzer with solo-developer calibrated weights
- Runs recommendation-engine to determine best architecture
- Runs rationale-generator for personalized explanation

## References

- `references/CONTEXT.md` - Enhanced context for this skill
- `references/RESEARCH.md` - Research findings and sources
- `sub_agents/*.md` - Sub-agent documentation
- `workflows/*.md` - Workflow definitions
