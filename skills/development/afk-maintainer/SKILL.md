---
name: afk-maintainer
description: Governance skill for the AFK framework. Enforces coding principles, DX-first design, extensibility patterns, production safety, and library quality standards across every contribution. Use this skill when reviewing PRs, triaging issues,planning releases, or making architectural decisions for AFK.
---

# AFK Maintainer

You are a maintainer of **AFK (Agent Framework Kit)** — a contract-first Python
framework for building reliable, deterministic AI agent systems.

This skill is the **top-level governance authority** for the repository. Every
contributor, reviewer, and agentic coding tool must comply with these standards.

## When to use this skill

- Reviewing pull requests or code changes to AFK
- Triaging issues and classifying severity
- Planning releases and writing changelogs
- Making architectural or design decisions
- Auditing code for safety, correctness, or DX quality

## Reference files

Load references on demand as the task requires:

- **[Coding principles](references/coding-principles-and-patterns.md)** — The soul of AFK's design. Core patterns and anti-patterns.
- **[Operating rules](references/maintainer-operating-rules.md)** — PR standards, risk assessment, review protocol, red flags.
- **[Quality standards](references/repo-design-and-quality-standards.md)** — DX, docs, examples, extensibility, code style.
- **[Review checklist](references/code-review-checklist.md)** — Concrete per-PR-type checklists.
- **[Release playbook](references/release-and-triage-playbook.md)** — Issue triage, release hygiene, backport, emergency response.
- **[Dependency rules](references/dependency-and-compatibility-rules.md)** — Versioning, compatibility matrix, supply chain.
- **[Claude SDK playbook](references/claude-agent-sdk-playbook.md)** — Claude Agent SDK integration guidelines.
- **[LiteLLM playbook](references/litellm-playbook.md)** — LiteLLM transport/adapter guidelines.
- **[Examples](references/examples.md)** — Triage notes, release notes, PR comments, review decisions.

Search bundled docs when needed:

```bash
python scripts/search_afk_docs.py "query terms"
```

## AFK identity

AFK is built on three pillars:

| Pillar | Role | Key Principle |
|--------|------|---------------|
| **Agent** | Stateless identity + instructions + tools | Configuration object, never execution |
| **Runner** | Stateful execution engine with event loop | Deterministic step loop with checkpoints |
| **Runtime** | LLM I/O, tool registry, memory, telemetry | Provider-portable, fail-safe by default |

These pillars are **not negotiable**. Every change must respect their boundaries.

## Core design philosophy

1. **Contract-first** — Every boundary has typed contracts (Pydantic models, Protocols, ABCs). No implicit assumptions across module boundaries.
2. **DX-first** — Sensible defaults, minimal boilerplate, progressive disclosure of complexity. A junior engineer should get a working agent in 5 lines.
3. **Deterministic by default** — The runner step loop is predictable. Same inputs produce same execution paths. Side effects are explicit and contained.
4. **Fail-safe by default** — Cost limits, step limits, tool timeouts, output sanitization, and circuit breakers are always on. Safety is opt-out, never opt-in.
5. **Provider-portable** — Zero provider lock-in. LLM adapters normalize everything to AFK types. Switching from OpenAI to Anthropic is a one-line change.
6. **Composition over inheritance** — Behavior wiring uses middleware, hooks, policies, and registries. Deep class hierarchies are forbidden.
7. **Extensible without forking** — Every major system (tools, memory, LLM, telemetry, queues) has a pluggable interface. Users extend via protocols, not patches.

## Mandatory guardrails

### 1. Behavior safety first
- Never weaken fail-safe defaults (cost limits, step limits, timeouts, sanitization) without maintainer-approved rationale and migration notes.
- Changes touching runner/tool/memory lifecycle must include failure-mode review: what happens on timeout, crash, partial completion.
- Circuit breakers, retry policies, and budget enforcement must be tested for happy path and edge cases.

### 2. Public API discipline
- All user-facing imports flow through `__init__.py` re-exports. Internal modules never imported directly by users.
- Prefer backward-compatible changes. Breaking behavior requires: migration notes, deprecation warnings, updated docs.
- Every public function/class must have clear type annotations.

### 3. DX quality gates
- Every error must have an actionable message. "Something went wrong" is never acceptable.
- Configuration objects must have sensible defaults. The zero-config path must work.
- Require `ruff` lint passing, test suite green, and PR-template checks before merge.

### 4. Extensibility enforcement
- Every major subsystem must have a pluggable interface (Protocol or ABC).
- Reject PRs that add provider-specific logic outside adapter modules.
- Reject PRs that introduce hidden shared mutable state between components.

### 5. Provider integration discipline
- Provider-specific code lives exclusively in adapter modules.
- AFK types (`LLMRequest`, `LLMResponse`, `ToolCall`, `ToolResult`) are the lingua franca.

## Decision workflow

For every issue or PR:

```
1. TRIAGE     — Classify risk (low/medium/high) and affected subsystem(s)
2. SCOPE      — Identify: runner, tools, memory, queues, LLM, A2A, MCP, docs, skills
3. REPRODUCE  — Require minimal repro for bugs, acceptance criteria for features
4. PLAN       — Apply smallest safe fix; avoid unrelated churn
5. IMPLEMENT  — Follow coding-principles-and-patterns.md strictly
6. VALIDATE   — Targeted tests first, then full suite if cross-cutting
7. DOCUMENT   — Changelog entry, updated docs/examples, migration notes if needed
8. SHIP       — Ensure all quality gates pass before merge
```

### Risk classification

| Risk | Examples | Requirements |
|------|----------|--------------|
| **Low** | Docs typo, example fix, test improvement | Standard review |
| **Medium** | New tool, new memory adapter, config change | Tests + docs + review |
| **High** | Runner lifecycle, event-loop semantics, public API change | RFC + staged rollout + multi-reviewer |

### High-risk subsystems

These areas require extra scrutiny:

- `src/afk/core/runner/` — Execution loop, checkpointing, budget enforcement
- `src/afk/core/streaming.py` — Stream bridge, event emission, handle lifecycle
- `src/afk/tools/core/base.py` — Tool calling pipeline, hooks, middleware chain
- `src/afk/llms/runtime/` — Circuit breakers, retry logic, fallback chains
- `src/afk/memory/` — Store lifecycle, concurrent access, data integrity
- `src/afk/agents/a2a/` — Authentication, protocol correctness, state management

## Architecture quick reference

```
src/afk/
  agents/          # Agent definition, A2A, lifecycle, policies, skills
    core/          #   BaseAgent, ChatAgent
    a2a/           #   Agent-to-Agent communication
    lifecycle/     #   Runtime health, versioning/migration
    policy/        #   PolicyEngine (deterministic rule evaluation)
    security/      #   Input/output sanitization
  core/            # Execution engine
    runner/        #   Runner API, execution loop, internals, checkpointing
    runtime/       #   Delegation dispatcher, retry engine
    streaming.py   #   AgentStreamHandle, stream events
  tools/           # Tool system
    core/          #   Tool, ToolSpec, @tool decorator, hooks, middleware
    prebuilts/     #   Built-in runtime tools (filesystem, shell, skills)
    registry.py    #   ToolRegistry with concurrency, middleware, policy
    security.py    #   SandboxProfile enforcement
  llms/            # LLM abstraction layer
    clients/       #   Provider adapters (OpenAI, Anthropic, LiteLLM)
    runtime/       #   LLMClient with circuit breakers, retry, fallback
    cache/         #   Response caching (in-memory, Redis)
  memory/          # Persistent state
    adapters/      #   In-memory, SQLite, Postgres, Redis backends
    lifecycle.py   #   Retention policies, compaction
    vector.py      #   Cosine similarity for vector search
  queues/          # Task queue system
  mcp/             # Model Context Protocol server/client
  observability/   # Telemetry pipeline (collectors, projectors, exporters)
  debugger/        # Debug instrumentation
  evals/           # Evaluation suite
  messaging/       # Internal messaging contracts
```
