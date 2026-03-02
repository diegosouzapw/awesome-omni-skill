---
name: architecture-discovery
description: Guide users through discovering and defining system architecture through structured conversation. Triggers on "I want to build", "design a system", "architect", "planning a new project", "how should I build X".
---

# Architecture Discovery Workflow

Guide users through structured discovery before design or implementation. Act as a collaborative thinking partner who asks questions, surfaces assumptions, and helps clarify intent.

## Optimized Flow

```
┌─────────────────────────────────────────────────────────────────┐
│ Phase 1: Initial Discovery                                      │
│   Ask universal questions → Gather context                      │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ SUBAGENT: classify-and-expand                                   │
│   Input: User answers + domain-index.md (seed)                  │
│   Output: Domains + GENERATED patterns/questions/considerations │
│   [Skip if: user provided detailed requirements]                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 2: Refinement                                             │
│   Ask generated domain questions → Update requirements          │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 3a: Architecture Questions                                │
│   Scale, availability, operations, integration                  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ SUBAGENT: research-and-propose                                  │
│   PARALLEL BATCHES: db | backend | cloud | specialized         │
│   Output: Research docs + 2-3 architecture options              │
│   [Skip research if: team has production experience]            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 3b: Selection                                             │
│   User picks direction → Refine details                         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ SUBAGENT: generate-artifacts                                    │
│   Output: Single architecture-package.md with ADRs + diagrams   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 4: SUBAGENT: validate-and-synthesize                      │
│   PARALLEL BATCHES: feasibility | performance | compliance      │
│   Output: validation-report.md + spike definitions              │
│   [Skip if: no gaps remaining]                                  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 5: Template Adaptation (Optional)                         │
│   SUBAGENT: template-adapter                                    │
│   Input: Generated artifacts + company templates                │
│   Output: Documents in company format (preserves all content)   │
│   [Skip if: no company templates provided]                      │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 6a: Implementation Planning (Optional)                    │
│   SUBAGENT: plan-implementation                                 │
│   Output: Milestones, tasks, timeline, resources                │
│   [Skip if: user not ready to build]                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 6b: Plan Validation (Optional)                            │
│   SUBAGENT: validate-plan                                       │
│   Input: Plan + normative docs (policies, standards, compliance)│
│   Output: Validation report, gaps, required checkpoints         │
│   [Skip if: no normative docs provided]                         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 6c: CLAUDE.md Generation                                  │
│   SUBAGENT: generate-claude-md                                  │
│   Input: All discovery artifacts + decisions + conventions      │
│   Output: CLAUDE.md for AI-assisted development                 │
│   [Always generate - this is the implementation handoff]        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 6d: CLAUDE.md Validation                                  │
│   SUBAGENT: validate-claude-md                                  │
│   Checks: Completeness, consistency, scale-appropriateness      │
│   Output: Validation report + auto-fixes if needed              │
│   [Always run - ensures quality handoff]                        │
└─────────────────────────────────────────────────────────────────┘
```

## Reference Files

| File | Purpose |
|------|---------|
| **[discovery-state.md](references/discovery-state.md)** | Central state document |
| **[domain-index.md](references/domain-index.md)** | Seed index for domain classification (~5KB) |

---

## Scaling: Minimal to Enterprise

This workflow scales to any project size. Detect complexity early and adjust depth throughout.

### Complexity Levels

| Level | Signals | Discovery Depth | Output Size |
|-------|---------|-----------------|-------------|
| **Minimal** | 1-2 devs, ≤2 weeks, no compliance, internal tool | Light | ~50-150 lines |
| **Standard** | 3-6 devs, 1-3 months, basic compliance | Normal | ~200-400 lines |
| **Complex** | 6-12 devs, 3-6 months, compliance, multi-tenant | Deep | ~400-700 lines |
| **Enterprise** | 12+ devs, 6+ months, strict compliance, multi-team | Comprehensive | ~700-1000 lines |

### Detection Triggers

```yaml
minimal_project:
  any_of:
    - "internal tool"
    - "prototype"
    - "proof of concept"
    - "simple CRUD"
    - "weekend project"
    - team_size: 1-2
    - timeline: "≤2 weeks"
  
enterprise_project:
  any_of:
    - compliance: ["HIPAA", "PCI-DSS", "FedRAMP"]
    - team_size: ">12"
    - timeline: ">6 months"
    - "multi-team"
    - "platform"
    - "mission-critical"
```

### Scaling Behavior by Phase

| Phase | Minimal | Standard | Enterprise |
|-------|---------|----------|------------|
| **Discovery** | 3-5 questions | Full questions | Full + stakeholder mapping |
| **Classification** | Quick, 1 domain | Normal | Deep, cross-cutting |
| **Research** | Skip (use defaults) | Parallel batches | Comprehensive + edge cases |
| **Architecture** | 1 option, brief | 2-3 options | 3+ options, detailed |
| **Artifacts** | Minimal ADRs | Full package | Full + compliance matrices |
| **Validation** | Skip or minimal | Normal | Comprehensive + compliance |
| **Template Adapt** | Skip | If needed | Usually required |
| **Implementation** | Skip or brief | If ready | Detailed with gates |
| **CLAUDE.md** | 50-150 lines | 200-400 lines | 700-1000 lines |

### Minimal Project Fast Path

For internal tools, prototypes, simple apps:

```
User: "I need a simple tool to track equipment loans internally"

Fast path:
1. Ask: What/Who/When (3 questions max)
2. Skip: Domain expansion (obvious), research (use standards)
3. Generate: Brief architecture (1 paragraph + tech table)
4. Skip: Validation, template adaptation, implementation planning
5. Output: Minimal CLAUDE.md (~80 lines)

Total time: 10-15 minutes
```

### Enterprise Project Full Path

For regulated, multi-team, long-term:

```
User: "Building a patient records platform, HIPAA compliance required"

Full path:
1. Full discovery with stakeholder mapping
2. Deep domain classification (healthcare + compliance + security)
3. Comprehensive research (all batches)
4. Multiple architecture options with detailed comparison
5. Full artifact generation with compliance matrices
6. Complete validation against HIPAA requirements
7. Template adaptation to company formats
8. Detailed implementation planning with gates
9. Plan validation against normative docs
10. Comprehensive CLAUDE.md (~800 lines)

Total time: 2-4 hours across multiple sessions
```

### Scaling Outputs

**Minimal CLAUDE.md includes:**
- Project Context (2-3 sentences)
- Tech Stack (table)
- Code Conventions (essential only)
- Quick Reference (commands)

**Enterprise CLAUDE.md adds:**
- Comprehensive architecture section
- All patterns with examples
- Full anti-patterns list
- Detailed security/compliance
- Multi-team boundaries
- Service ownership
- Incident response
- On-call expectations
- External service matrix
- Comprehensive troubleshooting

---

## Agent Orchestration Autonomy

The main agent has FULL AUTONOMY to scale the workflow. Don't ask permission — detect and adapt.

### Early Detection (Phase 1)

In initial discovery, actively detect complexity:

```yaml
# Ask or infer these signals
detection_questions:
  - "How many people on the team?" → team_size
  - "What's the timeline?" → timeline
  - "Any compliance requirements?" → compliance
  - "Is this internal or customer-facing?" → scope

# Can also infer from project description
inference_signals:
  minimal: ["internal tool", "prototype", "simple", "quick", "basic", "just need"]
  enterprise: ["HIPAA", "PCI", "SOC2", "platform", "multi-team", "regulated"]
```

### Set Complexity Level (After Phase 1)

Determine and store complexity in state:

```yaml
# Set this after initial discovery
state:
  complexity: "minimal|standard|complex|enterprise"
  complexity_signals:
    - "Team size: 2"
    - "Timeline: 2 weeks"
    - "No compliance"
  complexity_reasoning: "Small team, short timeline, internal tool"
```

### Autonomous Phase Decisions

Based on complexity, the agent SKIPS phases without asking:

```yaml
phase_autonomy:
  minimal:
    classify-and-expand: "Run briefly, 1 domain max"
    research-and-propose: "SKIP — use standard stack"
    generate-artifacts: "Run minimal — brief ADR, no diagrams"
    validate-and-synthesize: "SKIP"
    template-adapter: "SKIP"
    plan-implementation: "SKIP"
    validate-plan: "SKIP"
    generate-claude-md: "Run minimal — 50-80 lines"
    validate-claude-md: "Run — verify scale appropriate"
    
  standard:
    classify-and-expand: "Run normal"
    research-and-propose: "Run with parallel batches"
    generate-artifacts: "Run full"
    validate-and-synthesize: "Run if gaps exist"
    template-adapter: "Run if templates provided"
    plan-implementation: "Offer, run if accepted"
    validate-plan: "Run if normative docs provided"
    generate-claude-md: "Run standard — 200-400 lines"
    validate-claude-md: "Run"
    
  complex:
    # Run all phases with depth
    
  enterprise:
    # Run all phases comprehensively
    # Add: stakeholder mapping, team boundaries, compliance matrices
```

### Autonomous Depth Adjustment

Within phases, adjust depth without asking:

```yaml
depth_autonomy:
  discovery_questions:
    minimal: "3-5 essential questions only"
    standard: "Full question set"
    enterprise: "Full + stakeholder-specific"
    
  architecture_options:
    minimal: "1 recommendation (no comparison needed)"
    standard: "2-3 options with comparison"
    enterprise: "3+ options with detailed analysis"
    
  research_breadth:
    minimal: "Skip web search, use established patterns"
    standard: "Parallel batches, key technologies"
    enterprise: "Comprehensive, edge cases, compliance verification"
    
  validation_depth:
    minimal: "Skip"
    standard: "Key gaps only"
    enterprise: "Full validation with compliance matrix"
```

### Communication Style by Complexity

Adapt communication without asking:

```yaml
communication:
  minimal:
    style: "Brief, direct, no ceremony"
    example: |
      "Got it — simple equipment tracker. Here's what I recommend:
      Django + SQLite, standard patterns. 
      [Shows brief architecture]
      Ready to generate the setup docs?"
    
  standard:
    style: "Collaborative, structured but not heavy"
    example: |
      "Based on your requirements, I've identified this as a 
      [domain] project. Let me walk through a few architecture
      options..."
    
  enterprise:
    style: "Thorough, stakeholder-aware, compliance-conscious"
    example: |
      "Given the HIPAA requirements and multi-team structure,
      I'll need to be comprehensive here. Let's start by mapping
      stakeholders and their concerns..."
```

### Override Rules

User can always override agent decisions:

```yaml
override_handling:
  user_wants_more:
    trigger: "Can you go deeper on the architecture?"
    action: "Increase depth for remaining phases"
    
  user_wants_less:
    trigger: "This is overkill, keep it simple"
    action: "Reduce depth, skip optional phases"
    
  user_specifies:
    trigger: "I want full implementation planning"
    action: "Run requested phase regardless of complexity"
```

### Fast Path Declaration

For minimal projects, agent can declare fast path:

```
Agent (after detecting minimal):

"This sounds like a straightforward internal tool — I'll keep this 
lean. Quick architecture recommendation + minimal CLAUDE.md should 
have you coding in about 10 minutes.

Sound good, or do you want me to go deeper?"
```

### Decision Transparency

When skipping, briefly note why:

```
Agent (skipping research for minimal project):

"Skipping deep technology research since this is a simple internal 
tool — Django + PostgreSQL is the obvious choice for your team's 
experience. Moving straight to setup..."
```

```
Agent (skipping validation for minimal):

"For a 2-week prototype, detailed validation would be overkill. 
If you hit issues, we can revisit. Here's your CLAUDE.md..."
```

### Complexity Can Escalate

If discovery reveals complexity, escalate:

```yaml
escalation:
  trigger: "User mentions compliance/scale/teams mid-conversation"
  action: |
    "Ah, HIPAA compliance changes things significantly. Let me 
    shift to a more thorough approach — we'll need proper security
    controls and compliance documentation."
  update: "complexity: minimal → complex"
```

## Subagents

| Subagent | Purpose | Conditional Skip |
|----------|---------|------------------|
| **[classify-and-expand](subagents/classify-and-expand.md)** | Classify + generate domain content | Skip if detailed requirements provided |
| **[research-and-propose](subagents/research-and-propose.md)** | Parallel research + architecture options | Skip research if production experience |
| **[generate-artifacts](subagents/generate-artifacts.md)** | ADRs, diagrams in single package | Never skip |
| **[validate-and-synthesize](subagents/validate-and-synthesize.md)** | Parallel validation + synthesis | Skip if no gaps |
| **[template-adapter](subagents/template-adapter.md)** | Map to company templates | Skip if no templates provided |
| **[plan-implementation](subagents/plan-implementation.md)** | Milestones, tasks, timeline | Skip if not ready to build |
| **[validate-plan](subagents/validate-plan.md)** | Validate against normative docs | Skip if no normative docs |
| **[generate-claude-md](subagents/generate-claude-md.md)** | Create CLAUDE.md for implementation | Never skip (final handoff) |
| **[validate-claude-md](subagents/validate-claude-md.md)** | Validate CLAUDE.md quality + scale | Never skip (quality gate) |
| **[validate-and-synthesize](subagents/validate-and-synthesize.md)** | Parallel validation + synthesis | Skip if no gaps |

---

## Phase 1: Initial Discovery + Complexity Detection

**Primary goals:**
1. Understand what they're building
2. **Detect project complexity** — this drives everything

Ask universal questions (scaled to context):

```
ALWAYS ASK (Essential 5):
1. What are you building? (elevator pitch)
2. Who's it for? How many users?
3. What's your timeline?
4. Who's on the team? How many?
5. Any compliance requirements? (HIPAA, PCI, SOC2)

ASK IF NOT OBVIOUS:
6. What problem does it solve?
7. What exists today?
8. What integrations needed?
9. What does success look like?
```

**While asking, DETECT COMPLEXITY:**

```yaml
listen_for:
  minimal_signals:
    - "just me and one other dev"
    - "couple weeks"
    - "internal tool"
    - "prototype"
    - "simple"
    
  enterprise_signals:
    - "HIPAA" / "PCI" / "SOC2" / "compliance"
    - "multiple teams"
    - "platform"
    - "mission-critical"
    - team_size > 10
    - timeline > 6 months
```

**After questions, SET COMPLEXITY:**

```yaml
# Example: Minimal detected
complexity:
  level: "minimal"
  signals: ["Team: 2", "Timeline: 2 weeks", "Internal tool"]
  reasoning: "Small scope, short timeline, no compliance"
  fast_path: true
```

**Exit:** Enough context to classify domains AND complexity level set.

---

## Phase 1.5: Classification (Subagent)

**Invoke:** `classify-and-expand`

**Send:** User's answers + domain-index.md

**Conditional skip:** If user provided detailed requirements with specific technologies, patterns, and constraints — skip and use their input directly.

**Receive:**
- Domain classification (primary, secondary, cross-cutting)
- **Generated** patterns relevant to this project
- **Generated** questions to ask
- **Generated** considerations and risks
- Execution flags (what to skip later)

**Present:**
```
Based on what you've shared, this looks like a [type] project 
touching these areas:

Primary: [domains]
Also relevant: [domains]
Cross-cutting: [concerns]

Key patterns I'd consider:
- [Pattern]: [Why relevant]
- [Pattern]: [Why relevant]

Before we design, I want to understand:
- [High-priority question]
- [High-priority question]
```

---

## Phase 2: Refinement

Ask the **generated** questions from classify-and-expand.

Focus on:
- Requirements that affect architecture
- Constraints that limit options
- Unknowns that create risk

**Exit:** Requirements specific enough to propose architecture.

---

## Phase 3a: Architecture Questions

Ask scale and operational questions:

**Scale & Performance:**
- Expected load (users, requests, data volume)
- Latency requirements (p50, p99)
- Availability target (99.9%, 99.99%)

**Operations:**
- Who operates this? (your team, dedicated ops, managed)
- Deployment target (cloud, on-prem, hybrid)
- Existing infrastructure/tools

**Integration:**
- How tightly integrated with external systems?
- Real-time or batch?
- What if integration is down?

**Exit:** Enough context for architecture research.

---

## Phase 3b: Research and Proposal (Subagent)

**Invoke:** `research-and-propose`

**Send:** Discovery state with requirements, constraints, domain context

**Conditional skip for research:** If team has production experience with proposed stack, skip web searches and use domain_context patterns directly.

**Parallel research batches:**
```
├── Database: versions, cloud options, performance
├── Backend: frameworks, scaling, ecosystem  
├── Cloud: managed services, compliance certs
├── Specialized: domain-specific (video, payments, etc.)
└── Patterns: current best practices
```

**Receive:**
- technology-research.md
- 2-3 architecture options with:
  - Tech stack (informed by research)
  - Patterns to apply
  - Risks and mitigations
  - Effort estimates
- Comparison table
- Recommendation with rationale

**Present options and get user selection.**

---

## Phase 3c: Selection & Refinement

User picks direction. Facilitate discussion:
- If hybrid: define which aspects from each
- If concerns: address specific risks
- If questions: clarify based on research

**Pre-mortem:**
- If this fails in 6 months, what's the cause?
- What assumption, if wrong, breaks this?
- What's hardest to change later?

**Exit:** User confirms architecture direction.

---

## Phase 3d: Artifact Generation (Subagent)

**Invoke:** `generate-artifacts`

**Send:** State + selected option + decisions

**Receive:** Single `architecture-package.md` containing:
- Executive summary
- All ADRs
- C4 diagrams (context, container)
- Key sequence diagrams
- Tech stack table
- Risks and mitigations
- Next steps

**Present to user for review.**

---

## Phase 4: Validation (Subagent)

**Invoke:** `validate-and-synthesize`

**Conditional skip:** If no gaps remaining after architecture phase, skip entirely.

**Send:** State + architecture package

**Parallel research batches:**
```
├── Feasibility: can X do Y?
├── Performance: scale validation
├── Compliance: certifications
└── Integration: API capabilities
```

**Receive:**
- validation-report.md
- Spike definitions (if needed)
- Updated risk assessments
- Resolved/remaining gaps

**Present findings:**
```
Validation complete.

✅ Confirmed: [findings]
⚠️ Needs attention: [findings]
🔬 Spike recommended: [if any]

[Updated next steps]
```

---

## Phase 5: Template Adaptation (Optional)

**Skip if:** No company templates provided.

**Invoke:** `template-adapter`

**Purpose:** Map generated artifacts to company-specific document formats without losing any content.

**Send:**
- Generated artifacts (architecture-package.md, research docs, etc.)
- Company template(s) (user provides)
- Mapping hints (optional)

**Process:**
1. Analyze company template structure
2. Map our content to template sections
3. Generate adapted documents
4. Track where each piece of content went
5. Flag gaps (template sections we can't fill)

**Receive:**
- Documents in company format
- Preservation report (what went where)
- Gaps requiring user input

**Key principle:** Never lose information. If template has no place for content, add appendix or supplementary doc.

**Example adaptations:**
- Our ADRs → Company's RFC format
- Architecture package → Company's Design Doc template
- Tech research → Company's Tech Evaluation form

**Present:**
```
I've adapted our architecture work to your company templates:

Generated:
- design-doc-[project].md (Company Design Doc format)
- adr-001-[decision].md (Company ADR format)

All original content preserved. [N] sections need your input:
- [Section]: [What's needed]

Want to review the adapted documents?
```

---

## Phase 6a: Implementation Planning (Optional)

**Skip if:** User not ready to build, or spikes needed first.

**Invoke:** `plan-implementation`

**Send:**
- Architecture package
- Constraints (timeline, team, skills)
- Risks and spikes from validation
- Stakeholder context

**Receive:**
- Workstream breakdown
- Milestone definitions with exit criteria
- Task breakdown with estimates
- Build order (what to build first)
- Resource requirements
- Risk-adjusted timeline

**Receive:** `implementation-plan.md` containing:
- Milestones with deliverables and exit criteria
- Task breakdown by workstream
- Gantt-style timeline
- Resource requirements
- Critical path identification
- Risk factors

**Present:**
```
Here's the implementation plan:

Milestones:
- M0: Foundation (Week 2)
- M1: Walking Skeleton (Week 4)
- M2: Core Features (Week 8)
- M3: MVP Launch (Week 10)

Critical path: [Key dependencies]
Total effort: [X person-weeks]
Timeline risk: [Assessment]

Want to review the detailed breakdown?
```

---

## Phase 6b: Plan Validation (Optional)

**Skip if:** No normative documents provided.

**Invoke:** `validate-plan`

**Purpose:** Validate implementation plan against company standards, policies, and compliance requirements.

**Send:**
- Implementation plan
- Normative documents (user provides):
  - Security policies
  - Development standards
  - Release management process
  - Compliance requirements (HIPAA, SOC2, etc.)
- Architecture package (for context)

**Process:**
1. Extract requirements from normative docs
2. Map requirements to plan elements
3. Identify gaps and missing checkpoints
4. Check compliance control coverage
5. Generate validation report

**Receive:**
- Validation report with:
  - Coverage analysis
  - Critical gaps
  - Missing checkpoints/gates
  - Compliance matrix
- Required plan adjustments
- Timeline impact

**Present:**
```
Plan validation against your standards:

✅ Covered: [N] requirements
⚠️ Gaps found: [N] ([Critical count] critical)

Critical gaps:
1. [Gap]: [Required by] - [Remediation]
2. [Gap]: [Required by] - [Remediation]

Missing checkpoints:
- [Checkpoint] before [Phase]

Timeline impact: +[X weeks] to address gaps

Want to review the full validation report?
```

**After validation:**
- Update implementation plan to address gaps
- Re-run validation to confirm compliance
- Proceed to CLAUDE.md generation

---

## Phase 6c: CLAUDE.md Generation

**Always run** — this is the implementation handoff.

**Invoke:** `generate-claude-md`

**Purpose:** Create a comprehensive CLAUDE.md that captures everything an AI assistant needs to help build the project.

**Send:**
- Discovery state (full context)
- Architecture package (ADRs, diagrams, tech stack)
- Implementation plan (if created)
- Validation reports (compliance requirements)
- Company standards (if adapted)

**CLAUDE.md includes:**

| Section | Content |
|---------|---------|
| Project Context | What we're building, constraints, success criteria |
| Architecture | Style, components, data flow |
| Tech Stack | Technologies with versions and notes |
| Code Conventions | File structure, naming, style |
| Patterns to Follow | With code examples |
| Anti-Patterns to Avoid | With bad examples and fixes |
| Key Decisions | ADR summaries with implications |
| API Design | Conventions, formats, auth |
| Database | Schema conventions, migrations, multi-tenancy |
| Testing | Structure, commands, coverage expectations |
| Security | Sensitive data, auth, compliance reminders |
| Observability | Logging, metrics, tracing patterns |
| Development Workflow | Setup, branches, PRs, deployment |
| Common Tasks | Step-by-step for frequent operations |
| External Services | Table of integrations |
| Troubleshooting | Common issues and fixes |
| Quick Reference | Essential commands |

**Output:** `CLAUDE.md` ready to add to repository root

**Present:**
```
I've generated CLAUDE.md — the implementation handoff document.

It captures:
- Architecture decisions and rationale
- Tech stack with conventions
- Code patterns (with examples)
- Anti-patterns to avoid
- Security/compliance reminders
- Development workflow

This goes in your repo root. Claude (and developers) can reference it 
when writing code to ensure consistency with your architecture.

Ready to review?
```

---

## Phase 6d: CLAUDE.md Validation

**Always run** — quality gate for implementation handoff.

**Invoke:** `validate-claude-md`

**Purpose:** Ensure CLAUDE.md is complete, consistent, and appropriately scaled for project complexity.

**Checks performed:**

| Check | What It Validates |
|-------|-------------------|
| **Completeness** | All decisions covered, required sections present |
| **Consistency** | Code examples match tech stack, patterns don't conflict |
| **Scale** | Document size appropriate for project complexity |
| **Actionability** | Examples present, commands complete, steps specific |
| **Safety** | Security/compliance covered if required |

**Receive:**
- Validation report with scores
- Issues found (by severity)
- Auto-fix suggestions
- Scale assessment

**Present:**
```
CLAUDE.md validation complete.

✅ Completeness: 95%
✅ Consistency: 100%
✅ Scale: Appropriate (320 lines for standard project)
⚠️ Actionability: 90% (1 placeholder command)

Issues:
1. Migration command is placeholder — [auto-fixable]

Apply auto-fix? Or review full validation report?
```

**Auto-fix capability:**
- Replace placeholder commands with actual commands
- Generate missing code examples
- Add/remove sections for scale match
- Fix inconsistent tech references

**After validation:**
- Apply fixes if needed
- Re-validate if significant changes
- CLAUDE.md ready for repo

---

## Reference: Implementation Guidance

These sections provide detailed guidance for implementation planning and architecture artifacts.

### Observability Strategy

Generated as part of architecture package:

### What to Instrument

```yaml
observability:
  metrics:
    business:
      - "[Key business metric]"
      - "[Conversion/success rate]"
    technical:
      - "Request latency (p50, p95, p99)"
      - "Error rate by endpoint"
      - "Database query time"
      - "Queue depth"
      
  logging:
    structured: true
    correlation_id: "request-scoped"
    levels:
      - ERROR: "All errors with stack traces"
      - WARN: "Degraded performance, retries"
      - INFO: "Request/response, key events"
      - DEBUG: "Detailed flow (off in prod)"
    sensitive_fields: ["redact these"]
    
  tracing:
    enabled: true
    sampling: "10% normal, 100% errors"
    spans:
      - "HTTP requests"
      - "Database queries"
      - "External API calls"
      - "Queue operations"
      
  alerting:
    critical:
      - condition: "Error rate > 5%"
        action: "Page on-call"
      - condition: "P99 latency > 2s"
        action: "Page on-call"
    warning:
      - condition: "Error rate > 1%"
        action: "Slack notification"
```

### Recommended Stack

Based on cloud/constraints:
```
AWS: CloudWatch + X-Ray + CloudWatch Logs
GCP: Cloud Monitoring + Cloud Trace + Cloud Logging
Azure: Application Insights
Self-hosted: Prometheus + Grafana + Jaeger + Loki
```

---

## Testing Strategy

Generated based on architecture:

```yaml
testing:
  unit:
    coverage_target: "80% on business logic"
    focus:
      - "Domain logic"
      - "Validation rules"
      - "Edge cases"
    skip:
      - "Simple getters/setters"
      - "Framework code"
      
  integration:
    scope:
      - "Database operations"
      - "External API calls"
      - "Message queue interactions"
    approach: "Testcontainers / Docker Compose"
    
  e2e:
    critical_paths:
      - "[User journey 1]"
      - "[User journey 2]"
    tools: "Playwright / Cypress"
    
  contract:
    needed_for:
      - "[API consumed by others]"
      - "[Integration with external system]"
    tools: "Pact / OpenAPI validation"
    
  load:
    scenarios:
      - "Normal load: [X] users"
      - "Peak load: [Y] users"
      - "Stress test: find breaking point"
    tools: "k6 / Locust"
    when: "Before launch, after major changes"
    
  security:
    - "SAST in CI pipeline"
    - "Dependency scanning"
    - "DAST on staging"
    - "Penetration test before launch (if compliance requires)"
```

---

## Security Deep-Dive

When security-critical or compliance-heavy:

### Threat Modeling (STRIDE)

```yaml
threats:
  spoofing:
    - threat: "[Identity threat]"
      mitigation: "[Control]"
  tampering:
    - threat: "[Data integrity threat]"
      mitigation: "[Control]"
  repudiation:
    - threat: "[Audit threat]"
      mitigation: "[Control]"
  information_disclosure:
    - threat: "[Data leak threat]"
      mitigation: "[Control]"
  denial_of_service:
    - threat: "[Availability threat]"
      mitigation: "[Control]"
  elevation_of_privilege:
    - threat: "[Access control threat]"
      mitigation: "[Control]"
```

### Security Controls Checklist

```yaml
authentication:
  - "MFA available/required"
  - "Session management"
  - "Password policy"
  
authorization:
  - "RBAC/ABAC implemented"
  - "Least privilege principle"
  - "Resource-level permissions"
  
data_protection:
  - "Encryption at rest"
  - "Encryption in transit"
  - "Key management"
  - "PII handling"
  
infrastructure:
  - "Network segmentation"
  - "WAF/DDoS protection"
  - "Secrets management"
  
operational:
  - "Audit logging"
  - "Incident response plan"
  - "Security monitoring"
```

---

## Multi-Stakeholder Handling

When multiple stakeholders with different concerns:

### Stakeholder Map
```yaml
stakeholders:
  - role: "[Role]"
    concerns: ["[Concern 1]", "[Concern 2]"]
    success_criteria: "[What they care about]"
    involvement: "decision-maker | consulted | informed"
    
  - role: "[Another role]"
    concerns: ["[Different concerns]"]
    success_criteria: "[Their success metric]"
    involvement: "[Level]"
```

### Conflict Resolution
```
When stakeholders disagree:
1. Surface the conflict explicitly
2. Identify underlying concerns (not positions)
3. Find solutions that address both concerns
4. If irreconcilable, escalate to decision-maker
5. Document decision and rationale
```

### Communication Strategy
```yaml
updates:
  - audience: "Technical team"
    frequency: "Daily/standup"
    format: "Sync meeting"
    
  - audience: "Product stakeholders"
    frequency: "Weekly"
    format: "Status update + demo"
    
  - audience: "Leadership"
    frequency: "Bi-weekly"
    format: "Dashboard + summary"
```

---

## Workflow Guidance

**Pacing:**
- One phase per session is fine
- Let user set pace
- Don't rush through phases

**Conditional Execution:**
- Skip classify-and-expand if detailed requirements provided
- Skip research if team has production experience
- Skip validation if no gaps remain
- Skip template adaptation if no company templates provided
- Skip implementation planning if not ready to build
- Skip plan validation if no normative docs provided

**Parallel Research:**
- Batch searches by category
- Don't wait for one batch before starting next
- Reduces total research time significantly

**Tone:**
- Curious, not interrogating
- Collaborative, not prescriptive
- Direct about gaps and concerns

**Outputs:**
- technology-research.md (if research performed)
- architecture-package.md (always)
- validation-report.md (if validation performed)
- spike-*.md (only if spikes needed)
- [company-format]-*.md (if templates adapted)
- implementation-plan.md (if planning performed)
- plan-validation-report.md (if plan validated)
- **CLAUDE.md** (always — implementation handoff)
- claude-md-validation.md (always — quality confirmation)

---

## Error Handling & Fallbacks

### Search Returns Nothing
```
If web search yields no results:
1. Broaden query (remove version, year)
2. Try alternative terms
3. Fall back to domain_context patterns
4. Be transparent: "I couldn't find current data on X, 
   proceeding with established patterns"
```

### User Says "I Don't Know"
```
For critical questions:
  → Offer reasonable defaults with rationale
  → Flag as assumption in state
  → Add to gaps for later validation

For non-critical questions:
  → Skip and proceed
  → Note as open question
```

### Malformed Subagent Output
```
If subagent output is incomplete:
1. Identify what's missing
2. Re-run specific section
3. If still failing, generate manually with simpler approach
4. Never block the user - graceful degradation
```

### Recovery Prompts
```
"I hit a snag with [X]. Let me try a different approach..."
"I couldn't determine [X] automatically. Can you help clarify [specific question]?"
"My research on [X] was inconclusive. Want me to proceed with [assumption] or dig deeper?"
```

---

## Iteration & Backtracking

Users can revisit any phase. Handle with:

### Explicit Backtrack Triggers
```
User says: "Actually, let's reconsider the database choice"
→ Identify affected decisions
→ Show what would change
→ Re-run relevant subagent with new constraints
→ Cascade updates through state
```

### State Versioning
```yaml
state:
  version: 3
  history:
    - version: 1
      timestamp: "..."
      phase: "classification"
    - version: 2
      timestamp: "..."
      phase: "architecture"
      change: "Added compliance requirement"
  rollback_available: true
```

### Iteration Commands
```
"Let's go back to [phase]"
"What if we changed [decision]?"
"Reconsider [component] with [new constraint]"
"Show me alternatives to [choice]"
```

### Cascade Logic
```
When decision changes:
1. Identify downstream dependencies
2. Mark affected decisions as "needs review"
3. Re-validate impacted sections
4. Present summary of changes
```

---

## Multi-Session Support

### Save Checkpoint
```
At end of any phase:
"Want me to save progress? You can resume later with:
 'Continue architecture discovery for [project name]'"

Checkpoint includes:
- Full state YAML
- Phase completed
- Documents generated
- Open questions
```

### Resume Session
```
When user returns:
1. Load checkpoint
2. Summarize where we left off
3. Confirm context still valid
4. Continue from next phase

"Welcome back! We were working on [project]. 
 Last time we completed [phase] and decided on [key decisions].
 Ready to continue with [next phase]?"
```

### Checkpoint Triggers
- End of each phase
- Before major subagent invocation
- User requests save
- Session timeout warning

### State Persistence Format
```yaml
checkpoint:
  project: "[name]"
  saved_at: "[timestamp]"
  phase_completed: "refinement"
  next_phase: "architecture"
  
  state: { ... full state ... }
  
  documents_generated:
    - filename: "..."
      content: "..."
      
  resume_prompt: "We identified [domains] and gathered requirements. 
                  Ready to discuss architecture options?"
```

---

## Handling Different Project Types

### Greenfield (default)
- Full discovery flow
- Open technology choices
- Clean-slate patterns

### Migration
- Add migration-specific questions
- Include strangler fig, parallel run patterns
- Risk mitigation focus
- Legacy integration considerations
- See: migration-patterns in domain-index.md

### Enhancement
- Understand existing architecture first
- Constraint-heavy (must fit existing)
- Incremental change patterns
- Backward compatibility focus

### Integration
- API-first discovery
- Contract negotiation
- Error handling focus
- Sync vs async patterns

Detection:
```
"What exists today?" answer determines type:
- "Nothing" → greenfield
- "Old system we're replacing" → migration  
- "Existing system to extend" → enhancement
- "Need to connect systems" → integration
```
