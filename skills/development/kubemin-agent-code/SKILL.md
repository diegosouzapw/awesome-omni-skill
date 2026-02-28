---
name: kubemin-agent-code
description: Help users develop the KubeMin-Agent project (Python Agent) in a more standardized manner
metadata:
  short-description: Supporting the development of KubeMin-Agent
---

## Scope

This skill defines the working agreement for assisting with the **KubeMin-Agent** project (Python AI Agent), including communication norms, code change workflow, testing expectations, naming conventions, documentation synchronization, and PR hygiene.

---

## Rule Priority

When rules conflict, apply this priority order:
1. **Security** > Correctness > Performance
2. **Explicit user instructions** > SKILL rules > General best practices
3. **Single-turn delivery** (low risk) > Multi-turn confirmation
4. **Existing project conventions** > General Python conventions

---

## 1. Risk Assessment & Delivery Strategy

### Risk Levels

| Level | Criteria | Delivery Strategy |
|-------|----------|-------------------|
| **LR0** (Pure Internal) | No public API/behavior changes, no persistence/concurrency/security involvement, single module scope, behavior-preserving refactors or non-functional changes | Single-turn delivery |
| **LR1** (Limited Domain) | Touches persistence/concurrency/security BUT satisfies all LR1 constraints below | Single-turn delivery (with verification) |
| **Medium/High** | API/behavior changes, schema changes, new shared concurrency state, security boundary expansion, cross-module impact | Confirm plan first |

### LR1 Constraints (must satisfy ALL)

- **Scope limit**: ≤3 files OR ≤200 LOC changed
- **Localized bugfix/security fix** with no API/contract changes
- **Testable or verifiable** with reproducible steps
- **No schema changes** (migrations, serialization format)
- **No new shared concurrency state** (e.g., new global dict + lock)
- **No permission/access scope expansion**

> **LR1 exclusions**: Schema changes, cross-module protocol/serialization format changes, new shared concurrency state, relaxed auth/access scope.

### Efficiency Rules

- **Ambiguous but low-risk**: Proceed with clearly stated assumptions. Ask only when assumptions may affect correctness, safety, or public behavior.
- **Single-turn preference**: Deliver plan + code + tests/verification + docs/PR in one response unless medium/high-risk confirmation required.
- **Batch processing**: Multiple low-risk changes in same session → one combined delivery.

### Delivery Block Order

```
1. Risk Assessment (LR0/LR1/Medium/High)
2. Assumptions (if any ambiguity)
3. Plan (what will change)
4. Patch (code changes)
5. Tests/Verification (if meaningful)
6. Docs/PR (only if required or requested)
```

---

## 2. Communication

- Communicate with user in **Chinese** by default
- Documentation defaults to Chinese; English only when explicitly requested
- Provide English for specific artifacts only when user requests

---

## 3. Code Standards

### Naming
- Follow Python PEP 8 conventions: `snake_case` for functions/variables/modules, `CamelCase` for class names
- Prefer existing naming patterns in the project
- Names should express intent clearly; avoid abbreviations (except project-standard ones)

### Style
- Favor simplicity; avoid over-engineering
- Small functions, composable, early returns
- Async code uses `async/await`, properly propagate cancellation and timeouts
- Errors use custom exceptions with traceable context information
- Type annotations: add complete Type Hints for all public interfaces

### Constants & Configuration

| Type | Location |
|------|----------|
| Single-package constants | Nearest package (e.g., `agent/constants.py`) |
| Cross-domain constants | `config/constants.py` or project's existing public constants location (avoid reverse-depending on business packages) |
| Runtime-configurable values | `config/` package, using Pydantic Settings |

> **Rule**: Reusable strings/enums should be centralized, avoid scattered hard-coding. Only runtime-configurable items belong in `config/`.

### Project Architecture Conventions

| Module | Responsibility |
|--------|---------------|
| `control/` | Control plane (Scheduler, Validator, AgentRegistry, AuditLog) |
| `agents/` | Sub-agents (BaseAgent, K8sAgent, WorkflowAgent, GeneralAgent, GameTestAgent) |
| `agent/` | Agent runtime infrastructure (Loop, Context, Memory, Skills) |
| `agent/tools/` | Tool registration and execution (Base, Registry, concrete tool implementations) |
| `providers/` | LLM Provider abstraction and concrete implementations |
| `bus/` | Message bus (decouples Channel from Agent) |
| `channels/` | Channel integration (CLI, Telegram, etc.) |
| `session/` | Session management and persistence |
| `config/` | Configuration model and loading |
| `cron/` | Scheduled task scheduling |
| `heartbeat/` | Heartbeat detection and proactive wake-up |
| `skills/` | Built-in skills |
| `cli/` | CLI command entry |
| `utils/` | Common utility functions |

### Self-Review
After implementation: eliminate duplication, reduce nesting, remove redundant branches, check type annotation completeness.

---

## 4. Testing Requirements

| Change Type | Test Strategy |
|-------------|---------------|
| LR0 (non-functional) | Tests only if meaningful |
| LR0 (logic changes) | Minimal unit tests covering modified logic |
| LR1 changes | Unit tests OR verification steps |
| Medium/High risk | Full coverage: parameterized tests, edge cases, failure paths |

### Test Exemption & Alternatives

When unit tests are impractical, document:
1. **Why not testable**: e.g., handler layer with complex external dependencies
2. **Alternative verification** (must be **reproducible and copy-pasteable**):
   - Exact commands to run
   - Expected output/behavior

**Handler layer exemption**: If handler layer has no unit tests, must provide:
- Unit test coverage for the underlying service/domain layer (if affected by this change), OR
- Reproducible curl/CLI/integration commands with expected results

Acceptable exemption cases:
- Log/comment/formatting changes (testing adds no value)
- Handler code with complex dependencies (but service layer must be covered)

### Test Organization
- Use `pytest` as the testing framework
- Keep tests for same feature in **one test file**
- Extend existing `test_*.py` files; create new only when no suitable file exists
- Use `conftest.py` to manage shared fixtures

---

## 5. Documentation Updates

### Trigger Conditions

| Change Impact | Requires `docs/`? |
|---------------|-------------------|
| User-visible behavior/API/config/ops | ✅ Yes |
| Contract logs | ✅ Yes |
| Debug/diagnostic logs | ❌ No |
| Error contract changes | ✅ Yes |
| Default value/semantic changes | ✅ Yes |
| Pure internal refactoring / non-functional | ❌ No |

### Documentation Content (when required)
- Summary, motivation, key design decisions
- API/behavior changes
- Test coverage description
- Test execution commands

### Organization Rules
- Merge docs for same PR into **one Markdown file** per topic
- If repository lacks `docs/` directory → confirm before creating; otherwise update existing structure
- Language: Chinese by default

---

## 6. PR Guidelines

### Format
- **Title**: Clear imperative sentence
- **Body**: Summary / Changes / Testing / Notes
- ⚠️ Use real newlines, not `\n` escape sequences

### When to Provide PR Description
- User explicitly requests ("give me PR description" / "preparing to submit PR")
- Change requires docs/examples (complete delivery scenario)
- Default: title + brief summary only

### Updates
Only update PR description when new commits change scope or materially affect user-visible behavior/API, configuration, or risk.

---

## 7. Dependency Management

### Core Dependency Conventions

| Purpose | Recommended Library |
|---------|-------------------|
| CLI framework | `typer` |
| LLM gateway | `litellm` |
| Config model | `pydantic` + `pydantic-settings` |
| HTTP client | `httpx` |
| Logging | `loguru` |
| Cron scheduling | `croniter` |
| Terminal output | `rich` |
| Async WebSocket | `websockets` |

### Rules
- New dependencies must justify their inclusion, prefer reusing existing dependencies
- Use `pyproject.toml` to manage dependencies
- Use `hatchling` as the build backend

---

## 8. Anti-patterns

| ❌ Don't | ✅ Do |
|----------|-------|
| Abuse LR1 for schema/shared-state changes | Use LR1 only within defined constraints |
| Skip tests without reproducible alternative | Provide copy-pasteable verification commands |
| Document every log change | Only document contract logs |
| Put cross-domain constants in random packages | Use `config/constants.py` for shared constants |
| Always generate full docs/PR | Only when required or requested |
| Bypass AgentLoop to modify flow directly | New capabilities must be attached under existing interfaces |
| Expose sensitive info in tool returns | API Keys only read from config or env vars |

---

## 9. Pre-Delivery Checklist

- [ ] Risk level (LR0/LR1/Medium/High) correctly assessed?
- [ ] LR1 constraints verified (≤3 files, no schema/shared-state changes)?
- [ ] Delivery block order followed?
- [ ] Tests provided OR reproducible verification documented?
- [ ] Contract logs declared before requiring docs?
- [ ] Docs/PR only if required?
- [ ] Type annotations complete?
- [ ] async/await usage correct?

---

## Decision Flow

```
Receive request → Assess risk
    │
    ├── LR0 (Pure Internal)
    │     └── Single-turn: Plan + Patch + Tests (if meaningful) + Docs (if required)
    │
    ├── LR1 (Limited Domain) — verify constraints first
    │     ├── ≤3 files, ≤200 LOC?
    │     ├── No schema/shared-state/auth-expansion?
    │     └── Yes → Single-turn: Plan + Patch + Tests/Verification + Docs (if required)
    │
    └── Medium/High
          └── Confirm plan → User approval → Implement
                  │
                  ├── Affects public API? → Generate examples/
                  │     (includes defaults, pagination)
                  │
                  └── User-visible change? → Generate docs/
                        (contract logs only)
```

---

## 10. Iterative Improvement Process

This SKILL.md is a **living document**, continuously improved through real development feedback.

### Improvement Triggers

| Trigger Scenario | Action |
|-----------------|--------|
| User reports recurring issues found in review | → Analyze root cause → Propose new/modified rules → Update SKILL.md |
| Multiple review rounds still produce new issues | → Categorize issues → Add to "Lessons Learned" → Strengthen checklist |
| Existing rules cause low efficiency | → Discuss simplification → Adjust rules → Record change reason |
| Architecture evolution makes conventions outdated | → Update architecture convention table → Sync context-index.md |

### Improvement Flow

```
User reports issue → Analyze if systemic
    │
    ├── One-off issue → Fix code only, don't change SKILL
    │
    └── Systemic/recurring issue → Propose SKILL improvement
          │
          ├── 1. Identify issue category (naming/architecture/security/testing/performance/…)
          ├── 2. Propose specific rule changes (for user confirmation)
          ├── 3. After user confirms, update corresponding SKILL.md section
          ├── 4. Add to "Lessons Learned" section
          └── 5. Record in "Changelog"
```

### Proactive Suggestion Timing

After each code implementation, **proactively suggest** SKILL.md improvements if:
- Repeatedly hesitated on design decisions during implementation (indicates rules are unclear)
- Bypassed a rule in the code (indicates rule may be unreasonable or needs exception clause)
- Discovered quality risk points not covered by existing rules

---

## 11. Lessons Learned (Review Retrospective)

> This section records recurring issues and pitfalls discovered during reviews, for proactive avoidance in future development.
> Each entry includes: **Problem Description**, **Root Cause Analysis**, **Mitigation Measures**.

<!-- Template:
### [LESSON-XXX] Brief Problem Description
- **Date Discovered**: YYYY-MM-DD
- **Problem**: Specific description of what happened
- **Root Cause**: Why this problem occurred
- **Mitigation**: How to avoid this in the future (synced to which SKILL section)
- **Related Rule**: Points to the rule added/modified in SKILL as a result
-->

### [LESSON-001] No Emoji in Documentation
- **Date Discovered**: 2026-02-25
- **Problem**: Design document used emoji characters in module structure comments, reducing professionalism
- **Root Cause**: Default tendency to use emoji for visual categorization in code comments
- **Mitigation**: All documentation and code comments must use plain text descriptions only. No emoji characters in any project artifacts including SKILL.md, design docs, README, and inline comments
- **Related Rule**: Section 5 (Documentation Updates) - implicit formatting standard

### [LESSON-002] Must Read SKILL.md Before Every Task
- **Date Discovered**: 2026-02-25
- **Problem**: Implemented GameTestAgent without re-reading SKILL.md, resulting in: missing type annotations on internal methods, hardcoded magic numbers instead of constants, architecture convention table not updated for new modules
- **Root Cause**: Relied on "memory impression" of SKILL.md instead of actually reading it before implementation
- **Mitigation**: Before every code implementation task, explicitly read SKILL.md first. After implementation, self-review against the Pre-Delivery Checklist (Section 9)
- **Related Rule**: Section 9 (Pre-Delivery Checklist), Section 3 (Code Standards - Type Annotations, Constants)

---

## 12. Changelog

| Version | Date | Changes | Trigger |
|---------|------|---------|---------|
| v1.0 | 2026-02-25 | Initial version: complete development standards | Project initialization |
| v1.1 | 2026-02-25 | Added iterative improvement process, lessons learned section, changelog | User requested continuous improvement capability for Skills |
| v1.2 | 2026-02-25 | Translated entire SKILL.md to English for better readability | User requested English version |
| v1.3 | 2026-02-25 | Added LESSON-001: no emoji in documentation | User feedback during design doc review |
| v1.4 | 2026-02-25 | Added LESSON-002: must read SKILL.md before every task; updated architecture table with control/ and agents/ modules | Self-review after GameTestAgent implementation |
