---
name: semgrep-coderabbit-review
description: Two-stage code review combining fast pattern detection (Semgrep) with AI-powered semantic analysis (CodeRabbit)
license: MIT
compatibility: opencode
metadata:
  audience: developers
  workflow: code-review
  tools: semgrep,coderabbit
  phases: pattern-detection,semantic-review
---

# Semgrep + CodeRabbit Review Skill

A comprehensive two-stage code review workflow that combines **fast pattern detection** with **AI-powered semantic analysis** for maximum coverage and efficiency.

## What I Do

I orchestrate a complete code review process:

### Stage 1: Fast Pattern Detection (10-20 seconds)

- Run Semgrep to catch security patterns, architecture violations, and code quality issues
- Identify hardcoded secrets, SQL injection risks, XSS vulnerabilities
- Check Repository Pattern compliance (API layer)
- Validate authentication guards and error handling patterns
- Flag console.log and weak cryptography patterns

### Stage 2: AI-Powered Semantic Review (5-30 minutes)

- Run CodeRabbit to analyze code logic, architecture, and business context
- Verify OpenAPI contracts and breaking changes
- Validate test coverage and error handling
- Review multi-tenant isolation and data flow
- Provide context-aware recommendations

### Combined Report

- Prioritized findings (CRITICAL → HIGH → MEDIUM → LOW)
- Clear explanation of each issue with fix guidance
- Performance metrics and improvement suggestions
- Compliance verification (security, architecture, code quality)

## When to Use Me

Use this skill when you need to:

- ✅ Review uncommitted changes before committing
- ✅ Audit specific feature implementations
- ✅ Verify API layer compliance
- ✅ Check database query safety
- ✅ Review frontend code (Svelte 5, TypeScript)
- ✅ Pre-merge security and architecture validation
- ✅ Ensure multi-tenant isolation and data protection
- ✅ Validate API contracts (OpenAPI synchronization)

**Do NOT use if you only need:**

- ❌ Quick syntax/linter check (use linters directly)
- ❌ Style or formatting fixes (use Prettier/ESLint)
- ❌ Documentation review (use Codex security review)

## How I Work

### 1. Fast Feedback Loop (Stage 1)

```bash
semgrep scan --config .semgrep.yaml --error
```

**Catches in 10-20 seconds:**

- 🔐 Hardcoded secrets (API keys, tokens, passwords)
- 🔐 SQL injection patterns (string concat, unvalidated input)
- 🔐 XSS vulnerabilities (unsafe {@html}, template injection)
- 🔐 Missing authentication guards on protected routes
- 🏗️ Direct database access (bypassing Repository Pattern)
- 📋 Weak cryptography (MD5, SHA1, deprecated functions)
- 📋 Code quality (console.log, any types, commented code)

**Action:** Fix all Semgrep violations immediately. They are deterministic and indicate real issues.

### 2. Comprehensive Review (Stage 2)

```bash
coderabbit --prompt-only -t uncommitted
```

**Catches in 5-30 minutes:**

- ✅ Logic correctness and edge cases
- ✅ Test coverage gaps on behavioral changes
- ✅ Error handling and resilience
- ✅ Architecture and design patterns
- ✅ API contracts and breaking changes
- ✅ Multi-tenant isolation and security
- ✅ Performance optimization opportunities

**Action:** Fix CRITICAL + HIGH issues. Consider MEDIUM issues if reasonable.

### 3. Verification & Iteration

Re-run both tools after fixes to ensure no new issues introduced:

```bash
semgrep scan --config .semgrep.yaml --error
coderabbit --prompt-only -t uncommitted
```

## Review Focus Areas by Layer

### API Layer (Fastify Routes)

**What Semgrep catches:**

- Missing `preHandler: [fastify.authenticate]` on protected routes
- Direct `db.query()` instead of Repository Pattern
- SQL injection patterns in raw queries
- Hardcoded secrets or API keys in route definitions

**What CodeRabbit catches:**

- OpenAPI schema mismatches
- HTTP status code errors (should be 201 for create, 204 for delete, etc.)
- Inconsistent error response format
- Rate limiting misconfiguration
- Breaking changes to API contracts

**Checklist:**

- [ ] All protected routes have authentication guards
- [ ] All DB ops use `await request.repos.entity.method(ctx)`
- [ ] Correct HTTP status codes for all scenarios
- [ ] Error responses follow consistent format
- [ ] OpenAPI schema synchronized with implementation

### Database Layer (Repository Pattern)

**What Semgrep catches:**

- Raw SQL queries (should use Oracle parameterized queries)
- Missing error handling
- Direct database access in routes

**What CodeRabbit catches:**

- Parameterization correctness
- User context scoping (multi-tenant isolation)
- Error recovery logic
- Query performance patterns

**Checklist:**

- [ ] All queries use Oracle parameterized bindings (no string concat)
- [ ] User context (DbContext) properly scoped
- [ ] Errors thrown with ApplicationError(code, message)
- [ ] 100% test coverage for security-critical queries
- [ ] Multi-tenant isolation verified

### Frontend Layer (Svelte 5, TypeScript)

**What Semgrep catches:**

- Legacy Svelte 4 syntax (`$:`, `export let`, `bind:`)
- `{@html}` without sanitization (XSS risk)
- `any` types (type safety lost)
- console.log statements

**What CodeRabbit catches:**

- Svelte 5 runes compliance (proper use of $state, $derived, etc.)
- Component logic correctness
- Performance (N+1 queries, unnecessary re-renders)
- Accessibility issues
- Mobile responsiveness

**Checklist:**

- [ ] Only Svelte 5 runes used ($state, $derived, $props, $effect, {#snippet})
- [ ] HTML output sanitized or text interpolation used
- [ ] No `any` types (explicit types or `unknown` with guards)
- [ ] Mobile-first responsive design
- [ ] Accessibility verified (ARIA labels, semantic HTML)

### Shared Types & Contracts

**What Semgrep catches:**

- `any` types in shared definitions
- Hardcoded values (should be constants or env vars)

**What CodeRabbit catches:**

- Zod schema completeness (match OpenAPI definitions)
- Type inference correctness
- Breaking change documentation

**Checklist:**

- [ ] Zod schemas match OpenAPI definitions
- [ ] Types exported from schemas (type User = z.infer<...>)
- [ ] Discriminated unions for error types
- [ ] No `any` types (use `unknown` with guards)
- [ ] Breaking changes documented in JSDoc

## Priority & Workflow

### Execution Sequence

1. **Stage 1: Semgrep** (10-20 seconds)
   - Run scan
   - If FAIL → Fix ALL violations → Re-run until PASS
   - If PASS → Proceed to Stage 2

2. **Stage 2: CodeRabbit** (5-30 minutes)
   - Run review
   - Evaluate findings

3. **Fix Issues**
   - CRITICAL: Fix immediately (security, auth, data corruption)
   - HIGH: Fix before merge (architecture, breaking changes)
   - MEDIUM: Fix if reasonable (code quality)
   - LOW: Consider optional (suggestions)

4. **Verify**
   - Re-run both tools
   - Confirm no new issues

### Issue Priority Reference

| Priority    | Level         | Examples                                                           | Deadline              |
| ----------- | ------------- | ------------------------------------------------------------------ | --------------------- |
| 🚫 CRITICAL | Blocking      | Secrets, auth bypass, SQL injection, XSS, data leakage             | MUST fix before merge |
| 🚫 HIGH     | Architectural | Missing guards, schema mismatch, breaking changes, race conditions | Fix before merge      |
| ⚠️ MEDIUM   | Quality       | Weak crypto, poor error handling, type safety, duplication         | Fix if reasonable     |
| 💡 LOW      | Polish        | Suggestions, optimization, style                                   | Consider optional     |

## Common Issues & Fixes

### Security Issues

| Pattern                          | Fix                                                  |
| -------------------------------- | ---------------------------------------------------- |
| `const secret = "hardcoded-key"` | Move to `process.env.SECRET` or OCI Vault            |
| SQL string concat                | Use `SELECT * FROM users WHERE id = ?` with bindings |
| `{@html userContent}`            | Use `{@html DOMPurify.sanitize(userContent)}`        |
| No auth guard                    | Add `preHandler: [fastify.authenticate]`             |
| `console.log(token)`             | Remove or use safe logger without secrets            |

### Architecture Issues

| Pattern                       | Fix                                                |
| ----------------------------- | -------------------------------------------------- |
| `await fastify.db.query(...)` | Use `await request.repos.entity.method(ctx)`       |
| Missing DbContext             | Pass user ID context to all DB operations          |
| `export let prop` (Svelte)    | Use `let { prop } = $props()` (Svelte 5)           |
| Wrong status code             | 201 for create, 204 for delete, 400 for validation |
| `any` type                    | Define explicit type or `unknown` with type guard  |

### Code Quality Issues

| Pattern                 | Fix                                        |
| ----------------------- | ------------------------------------------ |
| `console.log()` in prod | Remove or use proper logging layer         |
| `MD5.hash()`            | Use `crypto.subtle.digest('SHA-256', ...)` |
| No test coverage        | Add test for new behavior/branch           |
| Duplicate logic         | Extract to reusable function/component     |
| N+1 queries             | Batch queries or use database join         |

## Tips for Efficiency

1. **Fix Semgrep First**
   - Patterns are deterministic and quick to verify
   - No ambiguity in fixes
   - Fail fast mentality

2. **Read CodeRabbit Carefully**
   - It provides context and reasoning
   - Use inline comments for clarification
   - Ask it to explain if unclear

3. **Batch Similar Fixes**
   - Don't context-switch between issue types
   - Fix all secret issues, then auth issues, etc.
   - Run unit tests after each batch

4. **Maximum 3 Passes**
   - If you need more than 3 review cycles, break into smaller PRs
   - Each cycle should be more targeted

5. **Test as You Go**
   - Run unit tests after each fix batch
   - Verify Semgrep passes immediately
   - Run CodeRabbit once before committing

## Related Skills & References

- **Plugin:** `opencode-semgrep-coderabbit-plugin` (provides tools for this skill)
- **Docs:** `AGENTS.md`, `.semgrep.yaml`, `.coderabbit.yaml`
- **Repository Pattern:** `packages/database/src/repositories/`
- **API Examples:** `apps/api/src/routes/`

## Troubleshooting

| Issue                     | Cause                                | Solution                                                        |
| ------------------------- | ------------------------------------ | --------------------------------------------------------------- |
| Semgrep takes >30s        | Large change set or complex patterns | Run on smaller subset or filter by rule                         |
| CodeRabbit takes >30min   | Very large changes (>500 lines)      | Break into smaller PRs                                          |
| Too many findings         | Code quality debt accumulation       | Fix critical/high, schedule follow-up for medium/low            |
| Same issue on re-run      | Fix incomplete or misunderstood      | Read feedback again, ask CodeRabbit for clarification           |
| Semgrep false positives   | Rule too broad or context-specific   | Review rule definition, can mark as intended if safe            |
| Conflicting tool feedback | Tools have different perspectives    | Trust Semgrep (deterministic), validate CodeRabbit with context |

## Next Steps

1. **Load this skill:** Use `/skill semgrep-coderabbit-review` in OpenCode
2. **Install plugin:** Install `opencode-semgrep-coderabbit-plugin` for tools
3. **Run review:** Use one of the provided commands
4. **Follow workflow:** Stage 1 (Semgrep) → Stage 2 (CodeRabbit) → Fix → Verify
5. **Share feedback:** Help improve this skill for your team

## Questions?

- Check the plugin README for detailed usage
- Review `.semgrep.yaml` for specific rule definitions
- Check `.coderabbit.yaml` for CodeRabbit configuration
- Ask CodeRabbit inline if feedback is unclear
- Open an issue on the plugin repository

---

**Version:** 1.0  
**Last Updated:** Jan 22, 2026  
**Status:** Production Ready  
**Plugin Required:** opencode-semgrep-coderabbit-plugin >= 1.0.0
