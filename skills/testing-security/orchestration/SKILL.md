---
name: orchestration
description: "Multi-phase AI workflow pipeline with quality gates, cognitive tiering, context scoping, and mandatory security remediation loops. Use when orchestrating idea-to-ship workflows: arm (brief), design, adversarial review, plan, drift detection, build, static/test gates, post-build quality, and release-readiness. Choose configuration from user prompt."
---

# orchestration (Playbook)

Quality-gated pipeline from idea to shipped code. Each phase produces a validated artifact that feeds the next. Context is scoped per phase; no phase inherits prior conversation history. **Choose one configuration** from the user prompt.

## When to use (triggers)
- Fuzzy idea, requirements, brief → **arm**
- First-principles design, constraints, research → **design**
- Multi-perspective critique, adversarial review → **adversarial-review**
- Execution plan, task groups, atomic planning → **plan**
- Drift detection, plan-vs-design, claim verification → **pmatch**
- Parallel build, agent coordination, implementation → **build**
- Static quality gate (lint/format/type) → **quality-static**
- Test quality gate (verification commands) → **quality-tests**
- Dead code, noise, cleanup → **denoise**
- Frontend style audit → **quality-frontend**
- Backend style audit → **quality-backend**
- Documentation freshness → **quality-docs**
- OWASP, vulnerability scan, security → **security-review**
- Final release decision gate → **release-readiness**
- Full pipeline, end-to-end orchestration → **pipeline**

## Choosing configuration from user prompt

| Prompt / theme | Configuration |
|----------------|----------------|
| idea, requirements, brief, arm, crystallize | arm |
| design, first-principles, constraints, approach, research | design |
| adversarial review, multi-perspective, critique, ar | adversarial-review |
| execution plan, task groups, atomic planning, deterministic | plan |
| drift, pmatch, plan-vs-design, claim verification | pmatch |
| build, parallel, coordinate, implement, agent teams | build |
| static checks, lint, format, typecheck | quality-static |
| test gate, verification tests, quality tests | quality-tests |
| denoise, dead code, unused, cleanup | denoise |
| frontend quality, frontend style, qf | quality-frontend |
| backend quality, backend style, qb | quality-backend |
| docs freshness, documentation quality, qd | quality-docs |
| security review, OWASP, vulnerabilities | security-review |
| release readiness, ship gate, go/no-go | release-readiness |
| full pipeline, end-to-end, orchestrate all phases | pipeline |

## Cognitive tiering

| Tier | Role | Phases |
|------|------|--------|
| Lead (high-reasoning) | Strategy, design, coordination | arm, design, adversarial-review (lead), plan, build (lead), release-readiness |
| Worker (fast) | Implementation, audit, review | build (workers), adversarial-review (reviewers), quality-static, quality-tests, denoise, quality-*, security-review, pmatch |

## Artifact contracts

All artifacts validate against schemas in `contracts/artifacts/` (including `execution-trace` and `evaluation-report`). Quality gates validate against `contracts/quality-gate.schema.json`. Pipeline state lives in `.pipeline/` (gitignored).

## Adapter mapping (source of truth)

Canonical stage adapters are defined under `adapters/<runner>/skills/`.

- Codex: `adapters/codex/skills/orchestration-*/SKILL.md`
- Cursor: `adapters/cursor/skills/orchestration-*/SKILL.md`
- Claude: `adapters/claude/skills/orchestration-*/SKILL.md`
- Gemini: `adapters/gemini/skills/orchestration-*/SKILL.md`
- Kilo: `adapters/kilo/skills/orchestration-*/SKILL.md`

## Semantic anchors

Use these terms consistently. Each term carries specific operational meaning:
- **Context minimization**: excessive context degrades signal quality; scope context tightly per phase.
- **Capability allocation**: reserve deep reasoning for strategy and use fast models for execution/audit throughput.
- **Separation of duties**: producers do not self-certify; validation must run from independent context.
- **Parallel challenge**: independent reviewers pressure-test assumptions to surface blind spots.
- **Evidence-first guidance**: recommendations must trace to verifiable sources and repository facts.
- **Predeclared verification**: tests and acceptance checks are defined before implementation begins.
- **Human control points**: automation assists execution, but critical gate progression requires explicit approval.
- **Fix-first security closure**: critical/high security findings are fixed and re-verified before progression; risk acceptance is explicit and owned.
- **Traceable intent preservation**: requirements and constraints keep stable trace IDs through plan, tests, drift, and gate evidence.

## Configurations

### arm
Requirements crystallization. Input: fuzzy idea (user prompt). Model: lead (high-reasoning). Steps: Extract requirements, constraints, non-goals, style, key concepts via conversational Q&A → Force remaining decisions in single structured checkpoint → Validate against `contracts/artifacts/brief.schema.json` → Write to `.pipeline/runs/<run-id>/brief.json`. Gate: `open_questions` array is empty; all constraints have explicit type and source. Output: Brief artifact. Hard rules: never guess a requirement; if ambiguous, ask; checkpoint is one round, not iterative.

### design
First-principles design. Input: Brief artifact. Model: lead (high-reasoning). Steps: Classify every constraint (hard/soft, flag misclassifications) → Reconstruct approach from validated truths only (no assumptions from training data) → Research via MCP-backed live docs (e.g. Context7) + web search → Align with codebase patterns (grep for conventions) → Iterate with user until alignment → Validate against `contracts/artifacts/design-document.schema.json` → Write to `.pipeline/runs/<run-id>/design.json`. Gate: user approves alignment; all research citations have verified_at timestamps; no unclassified constraints remain. Tools: filesystem, MCP docs, web search. Semantic intent: evidence-first guidance.

### adversarial-review
Multi-perspective critique. Input: Design Document artifact. Model: 3 independent reviewer subagents with different expert perspectives — architect-reviewer (architecture, scalability, coupling), security-engineer (OWASP, auth, trust boundaries), performance-engineer (bottlenecks, resource usage, caching). Lead (high-reasoning) orchestrates. Steps: Fan out design to 3 reviewer subagents in parallel (each gets filesystem + MCP docs access, each returns findings in Finding[] format) → Collect findings and pipe through `multi-model-review` skill for deduplication and cost/benefit analysis → Lead fact-checks each finding against actual codebase → Structured report for human review → Loop until no findings warrant mitigation per cost/benefit → Validate against `contracts/artifacts/review-report.schema.json` → Write to `.pipeline/runs/<run-id>/review.json`. Gate: all critical/high findings mitigated or accepted with justification; `remaining_unmitigated` contains only low/info items. Hard rules: the lead never dismisses a finding without evidence; each reviewer works independently (no shared context between reviewers). Semantic intent: parallel challenge + separation of duties.

### plan
Atomic execution planning. Input: Design Document + Review Report artifacts. Model: lead (high-reasoning). Steps: Transform design into task groups (target 3-6 tasks per builder, hard max 8) → Assign file ownership (no file appears in more than one group) → Include exact file paths, complete code patterns showing the approach, named test cases with setup and assertions → Define non-negotiable acceptance criteria per task → Link every task and test to covered requirement IDs (`covers_requirement_ids`) → If a group intentionally falls outside the 3-6 target, include `scope_override.reason` in that group → Validate against `contracts/artifacts/execution-plan.schema.json` → Write to `.pipeline/runs/<run-id>/plan.json`. Gate: every task has at least one test case; `file_ownership` has no conflicts; all code patterns are complete (no TODOs or placeholders); verification commands are executable; MUST requirements are covered by >=1 test link (`coverage-min` gate criterion); groups outside 3-6 have explicit `scope_override.reason`. Hard rules: if the builder would have to guess, the plan failed. Semantic intent: predeclared verification.

### pmatch
Drift detection. Input: source artifact (design or plan) + target artifact (plan or implementation). Model: two independent Task subagents (e.g. refactoring-specialist + architect-reviewer) working in parallel. Steps: Each agent independently extracts claims from source document → Verifies each claim against target → Feed both claim sets to `multi-model-review` with `action.type="drift-detect"` and `drift_config.mode="dual-extractor"` for adjudication (heuristic mode remains fallback-only) → Emit drift claim taxonomy classes (`interface`, `invariant`, `security`, `performance`, `docs`) for quality measurement → Lead validates conflicts and mitigations → Validate against `contracts/artifacts/drift-report.schema.json` → Write to `.pipeline/runs/<run-id>/drift-reports/pmatch.json`. Gate: no unmitigated structural drift (severity critical or high); all violated claims have mitigation or justification; adjudication metadata present (`mode`, `extractors`, `conflicts_resolved`, `resolution_policy`). Hard rules: extractors work independently (no shared context); lead must verify before accepting a finding.

### build
Coordinated parallel build. Input: Execution Plan artifact. Model: lead (high-reasoning) + workers (fast). Steps: Lead distributes task groups to builder agents (each gets own terminal/context scoped to its task group only) → Builders implement independently → Lead coordinates, unblocks, monitors progress → After all builders finish, lead runs pmatch (plan vs implementation) → Gate: all acceptance criteria pass; pmatch clean; verification commands succeed. Hard rules: lead never writes code; builders see only their task group and file paths; no builder context leaks to another builder. Semantic intent: context minimization + separation of duties.

### quality-static
Static quality gate. Input: implementation (changed files) + verification commands. Model: worker (fast). Steps: Execute lint, format-check, and typecheck/build commands (no auto-fix) → Capture command evidence and failures → Validate report against `contracts/artifacts/quality-report.schema.json` (`audit_type: static`) → Write `.pipeline/runs/<run-id>/quality-reports/static.json`. Gate: all static commands exit 0; report schema-valid.

### quality-tests
Test quality gate. Input: implementation + execution plan verification commands. Model: worker (fast). Steps: Execute required test commands independently of build coordination → Capture failures with reproduction commands → Validate report against `contracts/artifacts/quality-report.schema.json` (`audit_type: tests`) → Write `.pipeline/runs/<run-id>/quality-reports/tests.json`. Gate: all required tests pass; no unresolved critical/high test violations.

### denoise
Dead code and noise removal. Input: implementation (changed files). Model: worker (fast). Steps: Identify dead imports, unused variables, debug artifacts (console.log, TODO comments from build), orphaned code → Remove with evidence for each removal → Run tests to confirm no regressions → Validate against `contracts/artifacts/quality-report.schema.json` (audit_type: denoise). Gate: tests still pass after cleanup; no functional changes.

### quality-frontend
Frontend style audit. Input: implementation (frontend files). Model: worker (fast). Steps: Audit against project-specific frontend style guide (component patterns, naming, accessibility, performance) → Report violations with file, line, evidence, remediation → Validate against `contracts/artifacts/quality-report.schema.json` (audit_type: frontend). Gate: no critical/high violations.

### quality-backend
Backend style audit. Input: implementation (backend files). Model: worker (fast). Steps: Audit against project-specific backend style guide (API patterns, error handling, security, logging) → Report violations with file, line, evidence, remediation → Validate against `contracts/artifacts/quality-report.schema.json` (audit_type: backend). Gate: no critical/high violations.

### quality-docs
Documentation freshness audit. Input: implementation. Model: worker (fast). Steps: Cross-reference docs (README, API docs, inline docs) with changed code → Flag stale documentation (functions renamed, parameters changed, behavior altered without doc update) → Validate against `contracts/artifacts/quality-report.schema.json` (audit_type: docs). Gate: all changed public APIs have matching documentation.

### security-review
Comprehensive security audit and remediation loop. Input: implementation + runtime/deploy config. Model: worker (fast) for scanning, lead verification for closure.

Mandatory coverage checklist:
1. Access control and tenancy isolation (IDOR, horizontal/vertical privilege escalation, mass assignment).
2. Client-side injection defenses (XSS sources/sinks, output encoding, CSP effectiveness).
3. CSRF defenses on every state-changing endpoint (token enforcement + SameSite strategy).
4. Secret exposure and sensitive-data leakage (hardcoded keys, tokens, credentials, signing secrets, client bundle leaks).
5. HTTP security hardening (CSP, HSTS, X-Frame-Options/frame-ancestors, X-Content-Type-Options, Referrer-Policy, cache policy for sensitive pages).
6. Session and cookie hardening (`Secure`, `HttpOnly`, `SameSite`, auth token storage policy).
7. Production information exposure (server banners/versions, debug mode, stack traces, verbose errors).
8. Dependency vulnerabilities and patching (lockfile-aware updates + rescan).
9. Server-side injection and input abuse (SQL/NoSQL injection, command injection, SSRF).
10. File and path handling (insecure upload validation, path traversal, archive extraction safety).
11. Redirect and token issues (open redirects, JWT algorithm/expiry/storage validation).

Execution steps:
1. Run all checklist categories and capture evidence per finding.
2. Fix every critical/high finding.
3. Rerun the same checks and confirm closures.
4. Repeat until `critical/high open = 0`.
5. For any accepted risk, record owner, justification, expiry, and follow-up ticket.

Validate against `contracts/artifacts/quality-report.schema.json` (audit_type: security). Gate: no open critical/high findings; mandatory checklist fully executed; accepted risks explicitly signed off.

### release-readiness
Final ship gate. Input: completed gate artifacts + quality reports + release docs. Model: lead (high-reasoning). Steps: Determine release decision (`go`, `no-go`, `conditional`) → Assess semver impact, changelog updates, migration readiness, rollback readiness, open risks, and approvals → Validate against `contracts/artifacts/release-readiness.schema.json` → Write `.pipeline/runs/<run-id>/release-readiness.json`. Gate: schema-valid artifact; changelog updated; major version impacts include validated migration docs; rollback tested with owner; open risks have owner and due date; conditional decisions include explicit conditions.

### pipeline
Full pipeline orchestrator. Meta-configuration that runs phases in sequence: arm → design → adversarial-review → plan → pmatch → build → quality-static → quality-tests → post-build → release-readiness. Steps: Initialize `.pipeline/runs/<run-id>/` → Run each phase, write artifacts, validate gates between phases → Enforce per-phase context budgets using `context_manifest` + quality-gate criteria (`count-max`, `number-max`) → Apply `config.orchestration_policy` to choose reviewer/builder fan-out only when marginal gain clears threshold → Emit append-only trace events to `.pipeline/runs/<run-id>/trace.jsonl` and summarize with `trace-collector` → Block on gate failure (human must resolve before proceeding) → Post-build: run denoise, then quality-frontend + quality-backend + quality-docs + security-review (parallel or sequential depending on file independence) → For security-review specifically, run mandatory remediation loop (fix + rescan) with coverage of all required categories before final post-build gate close → Run release-readiness gate and block closure on `no-go` or unfulfilled `conditional` requirements → Write pipeline-state.json after each phase transition → Optional matrix evaluation writes `.pipeline/evaluations/<eval-id>/evaluation-report.json`. Hard rules: never auto-advance past a failed gate; human approval required at arm checkpoint, design alignment, adversarial review report, conditional/no-go release decisions, and any accepted security risk exception. Semantic intent: human control points + capability allocation.
