---
name: architecture
description: Interactive architecture design orchestrator that expands ideas into detailed, operable roadmaps with modules, protocols, tech stack recommendations, and risk analysis.
---

# Architecture Design Orchestrator

Transforms initial ideas into comprehensive, production-ready architecture documents through iterative research, decision gathering, and risk analysis. Outputs to `docs/ARCHITECTURE.md`.

## Architecture Design Checklist

The orchestrator MUST complete each section in order. Each section requires user acceptance before proceeding.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ARCHITECTURE DESIGN CHECKLIST                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  [ ] 1. GOALS & NON-GOALS         ← Define target and boundary          │
│         ↓                                                               │
│  [ ] 2. SYSTEM OVERVIEW           ← Top-level diagram + module map      │
│         ↓                                                               │
│  [ ] 3. DESIGN PRINCIPLES         ← Guiding constraints                 │
│         ↓                                                               │
│  [ ] 4. TOP-LEVEL MODULES         ← Module definitions                  │
│         ↓                                                               │
│  [ ] 5. MODULE STRUCTURE          ← Internal structure per module       │
│         ↓                                                               │
│  [ ] 6. KEY FEATURES              ← Data workflows + core features      │
│         ↓                                                               │
│  [✓] GENERATE docs/ARCHITECTURE.md                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Goals & Non-Goals

**Purpose**: Define the project's target and boundary clearly. This is the MOST IMPORTANT section.

### Actions

1. Ask user about the core problem being solved
2. Identify explicit goals (what the system MUST do)
3. Identify non-goals (what is explicitly OUT OF SCOPE)
4. Define success metrics
5. Present for user acceptance

### Questions to Ask (via AskUserQuestion)

- What is the primary problem this system solves?
- Who are the users/consumers of this system?
- What are the 3-5 MUST-HAVE capabilities?
- What is explicitly NOT in scope for this design?
- How will you measure success?

### Output Format

```markdown
## Goals & Non-Goals

### Goals

| ID | Goal | Success Metric |
|----|------|----------------|
| G1 | [Primary goal] | [Measurable outcome] |
| G2 | [Secondary goal] | [Measurable outcome] |

### Non-Goals

| ID | Non-Goal | Rationale |
|----|----------|-----------|
| NG1 | [What we won't do] | [Why it's out of scope] |
| NG2 | [What we won't do] | [Why it's out of scope] |
```

### 🛑 Checkpoint: Goals Acceptance

Present goals and non-goals to user. MUST receive explicit acceptance before proceeding.

---

## Phase 2: System Overview

**Purpose**: Provide a high-level architecture diagram showing module relationships.

### Actions

1. Identify major system components from goals
2. Create ASCII architecture diagram showing:
   - All top-level modules/layers
   - External dependencies
   - Data flow direction
   - API boundaries
3. Create module relationship table
4. Present for user acceptance

### Output Format

```markdown
## System Overview

[System name] is a [brief description]. It [primary function].

```
                           FRONTENDS / CLIENTS

   +---------------+                         +-------------------+
   |   [Client A]  |                         |   [Client B]      |
   +-------+-------+                         +---------+---------+
           |                                           |
           | [protocol]                                | [protocol]
           |                                           |
           +--------------------+----------------------+
                                |
                                v
+-------------------------------+-------------------------------+
|                        [API Layer]                            |
|              [Description of responsibilities]                |
+---------------+---------------------------+-------------------+
                |                           |
     +----------v----------+     +----------v----------+
     |   [Service Layer]   |     |   [Core Layer]      |
     |   [Description]     +---->|   [Description]     |
     +---------------------+     +---------------------+
```

### Layer/Module Relationships

| Layer | Component | Responsibility |
|-------|-----------|----------------|
| [Layer] | [Component] | [Single-line description] |
```

### 🛑 Checkpoint: Overview Acceptance

Present diagram and relationships. MUST receive explicit acceptance before proceeding.

---

## Phase 3: Design Principles

**Purpose**: Establish guiding constraints that inform all design decisions.

### Actions

1. Identify 5-7 core design principles
2. Each principle should be:
   - Actionable (tells you what to do)
   - Testable (you can verify compliance)
   - Relevant to this specific system
3. Present for user acceptance

### Output Format

```markdown
## Design Principles

1. **[Principle Name]**: [Description of the principle]
2. **[Principle Name]**: [Description of the principle]
3. **[Principle Name]**: [Description of the principle]
...
```

### Example Principles

- **Separation of Concerns**: Each layer has a single responsibility
- **Convention over Configuration**: Predictable structures eliminate boilerplate
- **Trait-based Abstractions**: Interfaces enable testing and flexibility
- **Observable Execution**: Every operation emits events for monitoring
- **Graceful Degradation**: Errors are classified and recovered automatically

### 🛑 Checkpoint: Principles Acceptance

Present principles to user. MUST receive explicit acceptance before proceeding.

---

## Phase 4: Top-Level Modules

**Purpose**: Define each top-level module's purpose and boundaries.

### Actions

1. List all modules identified in System Overview
2. For each module, define:
   - Name and layer
   - Primary responsibility (single sentence)
   - Key abstractions/interfaces
3. Create dependency diagram
4. Present for user acceptance

### Output Format

```markdown
## Top-Level Modules

### [Module Name] ([Layer])

[Description of what this module does and why it exists]

**Key Abstractions**:
- `[AbstractionName]`: [Purpose]
- `[AbstractionName]`: [Purpose]

### [Module Name] ([Layer])
...
```

### Dependency Direction

```
[Module A] ----+
               |
               +---> [Module B]
               |           |
[Module C] ----+           +---> [Module D] ---> [Module E]
                           |
                           +---> [Module F]
```

**Key constraint**: [State the dependency rule, e.g., "Lower layers never depend on higher layers"]

### 🛑 Checkpoint: Module Definitions Acceptance

Present module definitions to user. MUST receive explicit acceptance before proceeding.

---

## Phase 5: Module Structure

**Purpose**: Define the internal structure of each module.

### Actions

1. For each module from Phase 4:
   - Define directory/file structure
   - List public API surface
   - Document internal components
2. MUST use AskUserQuestion for any ambiguity
3. Present for user acceptance

### Output Format

```markdown
## Module Structure

### [Module Name]

```
[module-path]/
├── mod.rs / index.ts / __init__.py    # Module exports
├── error.rs                            # Error types
├── [submodule]/                        # Submodule
│   ├── mod.rs
│   ├── model.rs                        # Data models
│   ├── loader.rs                       # Loading logic
│   └── parser.rs                       # Parsing logic
└── [another_submodule]/
    └── ...
```

**Key Abstractions**:
- `[TraitName]`: [What it abstracts]
- `[StructName]`: [What it represents]
```

### 🛑 Checkpoint: Structure Acceptance

Present module structures to user. MUST receive explicit acceptance before proceeding.

---

## Phase 6: Key Features

**Purpose**: Design core features and data workflows in detail.

### Execution Protocol

This phase is ITERATIVE. For each key feature:

1. **List all key features first**
   - Present numbered list of features to design
   - User confirms or modifies list

2. **Design each feature one-by-one**
   - Present feature design
   - Use AskUserQuestion for ANY ambiguity
   - User accepts or requests changes
   - Only proceed to next feature after acceptance

3. **MANDATORY RESEARCH** for features involving:
   - Third-party dependencies (APIs, SDKs, libraries)
   - Infrastructure (Docker, K8s, cloud services)
   - Protocols (gRPC, WebSocket, message queues)
   - Databases (specific DB features, query patterns)

### Research Protocol

When a feature depends on external tech, you MUST:

1. Use **WebFetch** to get official documentation
2. Use **WebSearch** for best practices and gotchas
3. Document findings in the feature design
4. Include version constraints and compatibility notes

```markdown
### Research: [Technology Name]

**Source**: [Official docs URL]
**Version**: [Specific version being designed for]

**Key Findings**:
1. [Finding relevant to this design]
2. [Finding relevant to this design]

**Risks/Gotchas**:
- [Known issue or limitation]
- [Compatibility concern]

**Recommendation**: [How to use it in this system]
```

### Feature Design Format

```markdown
## Key Features

### [Feature Name]

**Purpose**: [What this feature accomplishes]

**Data Flow**:
```
[Step-by-step data flow diagram]
```

**Components Involved**:
| Component | Role | Input | Output |
|-----------|------|-------|--------|
| [Name] | [What it does] | [Data in] | [Data out] |

**Implementation Notes**:
- [Key implementation detail]
- [Key implementation detail]

**Error Handling**:
- [Error case]: [How it's handled]
```

### Example: Execution Pipeline Feature

```markdown
### Execution Pipeline

Handles AI agent execution with observable progress tracking.

**Data Flow**:
```
CLI/Service Request
        |
        v
+-------------------+
| ExecutionService  |
| .run(context)     |
+--------+----------+
         |
         v
+-----------------------------------------------------------+
| WorkflowEngine.run_with_progress()                        |
|   1. Load configuration                                   |
|   2. Build ExecutionContext                               |
|   3. Call ExecutionRunner.run_with_observer()             |
|   4. Emit progress events                                 |
|   5. Return ExecutionResult                               |
+-----------------------------------------------------------+
```

**Observer Pattern**:
```rust
#[async_trait]
pub trait ExecutionObserver: Send + Sync {
    async fn on_event(&self, event: ExecutionEvent);
}

pub enum ExecutionEvent {
    Started { worker: String },
    Progress { message: String },
    Completed { metrics: Metrics },
    Failed { error: String },
}
```
```

### 🛑 Checkpoint: Feature Design Acceptance (per feature)

Present each feature design individually. MUST receive explicit acceptance before proceeding to next feature.

---

## Document Generation

After all phases are complete, generate `docs/ARCHITECTURE.md` with this structure:

```markdown
# [System Name] Architecture

[Brief description of the system and its purpose]

## Table of Contents

- [System Overview](#system-overview)
- [Design Principles](#design-principles)
- [Module Structure](#module-structure)
- [Core Domain Models](#core-domain-models)
- [Key Features](#key-features)
- [Error Handling](#error-handling)
- [Security Considerations](#security-considerations)

## System Overview

[From Phase 2 - diagram and layer descriptions]

### Design Principles

[From Phase 3]

## Module Structure

### [Module Name]

[From Phase 5 - directory structure and abstractions]

## Core Domain Models

[Key data structures with code examples]

## Key Features

### [Feature Name]

[From Phase 6 - data flows and implementation details]

## Error Handling

[Error classification and recovery strategies]

## Security Considerations

[Security-relevant design decisions]

## Summary

[Key takeaways and architectural achievements]
```

---

## Interactive Behavior

### Using AskUserQuestion

MUST use AskUserQuestion tool for:

1. **Ambiguous requirements** - When user intent is unclear
2. **Design trade-offs** - When multiple valid approaches exist
3. **Technology choices** - When selecting between options
4. **Scope decisions** - When feature boundaries are unclear

Example:
```
AskUserQuestion:
  question: "For the event streaming component, which pattern fits better?"
  header: "Event Pattern"
  options:
    - label: "Event Sourcing"
      description: "Store all events, reconstruct state. Better for audit trails."
    - label: "CQRS"
      description: "Separate read/write models. Better for read-heavy workloads."
    - label: "Simple Pub/Sub"
      description: "Fire and forget events. Simpler but less traceable."
```

### Mandatory Research Triggers

MUST perform web research when user mentions:

| Trigger | Research Target |
|---------|-----------------|
| Docker, container | Container best practices, Dockerfile patterns |
| Kubernetes, K8s | K8s resource patterns, operator patterns |
| AWS, GCP, Azure | Cloud service docs, pricing, limits |
| gRPC | Protocol buffers, streaming patterns |
| Kafka, NATS, RabbitMQ | Message queue patterns, partitioning |
| PostgreSQL, MySQL, MongoDB | Schema design, indexing strategies |
| Redis, Memcached | Caching patterns, eviction policies |
| GraphQL | Schema design, N+1 prevention |
| WebSocket, SSE | Real-time communication patterns |
| OAuth, JWT, OIDC | Auth implementation patterns |

### Research Execution

```bash
# Step 1: Fetch official docs
WebFetch(url="https://docs.example.com/guide", prompt="Extract key patterns for [use case]")

# Step 2: Search for best practices
WebSearch(query="[technology] best practices [use case] 2024")

# Step 3: Document findings in architecture
```

---

## Usage Examples

```bash
# Start new architecture design
architecture: Build a recommendation engine for e-commerce

# With specific constraints
architecture: Design a video transcoding pipeline using FFmpeg and AWS

# Resume previous session
architecture resume

# Research-only mode
architecture research: Best practices for Kafka consumer groups
```

---

## Triggers

| Trigger | Action |
|---------|--------|
| `architecture:` | Full interactive workflow |
| `arch:` | Alias for architecture |
| `design:` | Alias for architecture |
| `architecture research:` | Research-only, no document generation |
| `architecture resume` | Resume previous session |

---

## Constraints

**MUST**:
- Complete checklist phases in order
- Get user acceptance at each checkpoint
- Use AskUserQuestion for ANY ambiguity
- Research external dependencies before designing
- Design key features one-by-one with individual acceptance
- Generate comprehensive docs/ARCHITECTURE.md at the end

**MUST NOT**:
- Skip any checklist phase
- Proceed without user acceptance
- Assume technology choices without research
- Design all features at once without checkpoints
- Generate document before all phases complete

---

## Output Artifact

**Location**: `docs/ARCHITECTURE.md`

The final document should be:
- Self-contained and readable without external context
- Include ASCII diagrams for visual clarity
- Have code examples where appropriate
- Follow the structure shown in Document Generation section
- Be production-ready as a team reference document

---

*Architecture Design Orchestrator - Transform ideas into production-ready architecture through guided, interactive design with mandatory research*
