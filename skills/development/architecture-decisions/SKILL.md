---
name: architecture-decisions
description: "Architecture decision guides for Kailash SDK including framework selection (Core SDK vs DataFlow vs Nexus vs Kaizen), runtime selection (Async vs Sync), database selection (PostgreSQL vs SQLite), node selection, and test tier selection. Use when asking about 'which framework', 'choose framework', 'which runtime', 'which database', 'which node', 'architecture decision', 'when to use', 'Core SDK vs DataFlow', 'PostgreSQL vs SQLite', 'AsyncLocalRuntime vs LocalRuntime', or 'test tier selection'."
---

# Kailash Architecture Decisions

Decision guides for selecting the right frameworks, runtimes, databases, nodes, and testing strategies for your Kailash application.

## Overview

Comprehensive decision guides for:
- Framework selection (Core SDK, DataFlow, Nexus, Kaizen)
- Runtime selection (AsyncLocalRuntime vs LocalRuntime)
- Database selection (PostgreSQL vs SQLite)
- Node selection for specific tasks
- Test tier selection (Unit, Integration, E2E)

## Reference Documentation

### Framework Selection
- **[decide-framework](decide-framework.md)** - Choose the right framework
  - Core SDK: Custom workflows with full control
  - DataFlow: Database-first applications
  - Nexus: Multi-channel platforms
  - Kaizen: AI agent systems
  - When to use each
  - Combining frameworks

### Runtime Selection
- **[decide-runtime](decide-runtime.md)** - AsyncLocalRuntime vs LocalRuntime
  - Docker/FastAPI → AsyncLocalRuntime
  - CLI/Scripts → LocalRuntime
  - Performance implications
  - Threading considerations
  - Auto-detection with get_runtime()

### Database Selection
- **[decide-database-postgresql-sqlite](decide-database-postgresql-sqlite.md)** - PostgreSQL vs SQLite
  - Production → PostgreSQL
  - Development/Testing → SQLite
  - Feature comparison
  - Migration strategies
  - Multi-database support

### Node Selection
- **[decide-node-for-task](decide-node-for-task.md)** - Choose the right node
  - AI tasks → AI nodes
  - API calls → API nodes
  - Custom logic → PythonCodeNode
  - Database → Database nodes or DataFlow
  - File operations → File nodes
  - Conditional logic → SwitchNode

### Test Tier Selection
- **[decide-test-tier](decide-test-tier.md)** - Unit vs Integration vs E2E
  - Tier 1: Unit tests (fast, mocking allowed)
  - Tier 2: Integration tests (real infrastructure)
  - Tier 3: End-to-end tests (full system)
  - When to use each tier
  - Coverage targets

## Key Decision Frameworks

### Framework Selection Matrix

| Need | Framework | Why |
|------|-----------|-----|
| **Custom workflows** | Core SDK | Full control, 110+ nodes |
| **Database CRUD** | DataFlow | Auto-generated nodes |
| **Multi-channel API** | Nexus | API + CLI + MCP instantly |
| **AI agents** | Kaizen | Signature-based agents |
| **All of above** | Combine them | They work together |

### Runtime Selection Flow

```
Are you deploying to Docker/FastAPI/Kubernetes?
  ├─ YES → AsyncLocalRuntime (async-first, no threads)
  └─ NO → Is this a CLI/script?
       ├─ YES → LocalRuntime (sync execution)
       └─ NO → Use get_runtime() for auto-detection
```

### Database Selection Flow

```
What's your use case?
  ├─ Production deployment?
  │   └─ YES → PostgreSQL (scalable, enterprise)
  ├─ Development/testing?
  │   └─ YES → SQLite (simple, fast setup)
  └─ High concurrency?
      └─ YES → PostgreSQL (better concurrency)
```

### Node Selection Flow

```
What task are you doing?
  ├─ Custom Python logic → PythonCodeNode
  ├─ LLM/AI tasks → LLMNode, OpenAINode, AnthropicNode
  ├─ Database operations → DataFlow auto-generated nodes
  ├─ HTTP API calls → APICallNode
  ├─ File reading → FileReaderNode
  ├─ Conditional routing → SwitchNode
  └─ Not sure? → Check nodes-quick-index
```

### Test Tier Flow

```
What are you testing?
  ├─ Individual function → Tier 1 (Unit)
  ├─ Workflow execution → Tier 2 (Integration)
  ├─ Complete user flow → Tier 3 (E2E)
  └─ All of above → Use all tiers
```

## Critical Decision Rules

### Framework Decisions
- ✅ Use Core SDK for custom workflows
- ✅ Use DataFlow for database operations (don't use SQLAlchemy/Django ORM)
- ✅ Use Nexus for multi-channel platforms (don't use FastAPI directly)
- ✅ Use Kaizen for AI agents (don't build from scratch)
- ✅ Combine frameworks as needed
- ❌ NEVER use ORM when DataFlow can generate nodes
- ❌ NEVER build API/CLI/MCP manually when Nexus can do it
- ❌ NEVER skip framework evaluation

### Runtime Decisions
- ✅ Docker/FastAPI → AsyncLocalRuntime (mandatory)
- ✅ CLI/Scripts → LocalRuntime
- ✅ Use get_runtime() when unsure
- ❌ NEVER use LocalRuntime in Docker (causes hangs)
- ❌ NEVER mix runtimes in same application

### Database Decisions
- ✅ Production → PostgreSQL
- ✅ Development → SQLite (for speed)
- ✅ Testing → SQLite in Docker (for isolation)
- ✅ Multi-instance → One DataFlow per database
- ❌ NEVER use SQLite for production high-concurrency
- ❌ NEVER skip connection pooling config

## When to Use This Skill

Use this skill when you need to:
- Choose between Core SDK, DataFlow, Nexus, or Kaizen
- Select AsyncLocalRuntime vs LocalRuntime
- Decide between PostgreSQL and SQLite
- Find the right node for a task
- Determine test tier for a test case
- Make architecture decisions
- Understand trade-offs between options

## Decision Templates

### Starting a New Project
```
1. What's the primary use case?
   - Database CRUD → Start with DataFlow
   - Multi-channel API → Start with Nexus
   - AI agents → Start with Kaizen
   - Custom workflows → Start with Core SDK

2. What's the deployment target?
   - Docker/K8s → Use AsyncLocalRuntime
   - CLI tool → Use LocalRuntime

3. What's the database?
   - Production → PostgreSQL
   - Dev/Test → SQLite

4. How to test?
   - Tier 1: Fast unit tests
   - Tier 2: Real infrastructure integration
   - Tier 3: Full system E2E
```

## Related Skills

- **[01-core-sdk](../../01-core-sdk/SKILL.md)** - Core SDK fundamentals
- **[02-dataflow](../../02-dataflow/SKILL.md)** - DataFlow framework
- **[03-nexus](../../03-nexus/SKILL.md)** - Nexus framework
- **[04-kaizen](../../04-kaizen/SKILL.md)** - Kaizen framework
- **[08-nodes-reference](../../08-nodes-reference/SKILL.md)** - Node reference
- **[12-testing-strategies](../../12-testing-strategies/SKILL.md)** - Testing strategies

## Support

For architecture decisions, invoke:
- `framework-advisor` - Framework selection and architecture
- `deep-analyst` - Deep analysis for complex decisions
- `requirements-analyst` - Requirements breakdown
- `pattern-expert` - Pattern recommendations
