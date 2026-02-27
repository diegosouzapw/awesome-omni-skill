---
name: debugging
description: "Debug failures systematically: reproduce, hypothesize, bisect, and fix. Use when the user reports a bug, asks why something fails, or wants to find the root cause."
triggers:
  - "/debug"
  - "fix this bug"
  - "why does this fail"
  - "root cause"
  - "reproduce the bug"
  - "this is broken"
  - "investigate"
---

# Debugging Skill

## Core Philosophy

**"Reproduce first; then narrow down."**

**"Write a failing test before fixing."** (TDD for Bug Fixes)

Get a reliable repro, form a hypothesis, test it, then fix. Avoid changing code until the cause is understood.

**⚠️ MANDATORY:** Always write a failing test that reproduces the bug BEFORE attempting to fix it. This ensures:
- The bug is truly understood and reproducible
- The fix can be verified automatically
- The bug cannot regress in the future

---

## Protocol

### 1. Reproduce (with Test)

- **Steps**: Minimal steps to see the failure (code path, input, env).
- **Environment**: Runtime, OS, versions, config, env vars. Note if it's intermittent.
- **Output**: Exact error message, stack trace, logs, or observable misbehavior.

**⚠️ MANDATORY: Write a Failing Test**

Once you understand the reproduction steps, **write a test that reproduces the bug**:

1. Create a test case that exercises the buggy code path
2. Assert the expected (correct) behavior
3. Run the test and verify it FAILS (reproducing the bug)
4. This test becomes your verification that the fix works

```
# TDD Bug Fix Flow
1. Write test → 2. Verify test fails → 3. Fix code → 4. Verify test passes
```

If the user didn't provide steps, ask or infer and state assumptions. If you can't reproduce, say so and suggest what's needed (e.g. logs, repro repo).

### 2. Hypothesize

- **Where**: Likely component, file, or function from stack trace, logs, or code flow.
- **Why**: Possible cause (wrong input, bad state, race, wrong assumption, missing check).
- **One at a time**: Test the most likely hypothesis first; use logs, asserts, or a minimal repro.

### 3. Narrow Down

- **Bisect**: If the failure is recent, narrow the change that introduced it (e.g. `git bisect`, or disable/revert changes).
- **Isolate**: Simplify input or code path (smaller test case, unit test) to confirm the cause.
- **Instrument**: Add logging or assertions to verify hypothesis; remove or reduce before final fix.

### 4. Fix and Verify

- **Fix**: Minimal change that addresses the root cause (not only the symptom).
- **Verify**:
  - **Reproduction test now passes** (the bug is fixed)
  - Run all existing tests; check for regressions
  - The reproduction test serves as permanent regression protection
- **Document**: The reproduction test itself documents the bug. Add a comment in the test explaining what bug it prevents.

### 5. Commands

| Action                | Examples                                                                       |
| --------------------- | ------------------------------------------------------------------------------ |
| Run failing test/code | `npm test -- --grep "..."`, `pytest path::test_name`, run main with same input |
| Inspect logs/errors   | Read stack trace, stderr, log file                                             |
| Bisect history        | `git bisect start/bad/good/run`                                                |
| Search for usage      | `grep` for the failing symbol or error string                                  |

### 6. MCP (Datadog)

When the failure involves production or staged environments, use the **Datadog MCP** (after **/setup**) to search logs, inspect trace details, and query metrics. Use `search_logs`, `get_log_details`, `query_traces`, and `query_metrics` to correlate errors and narrow down the cause. Ensure **/setup** has been run so Datadog MCP is configured.

### 7. Flaky Test Handling

When a test failure appears intermittent (flaky):

1. **Identify the flake** - Run the failing test in a loop (e.g., `pytest --count=20`, `npx jest --repeat=10`) to confirm flakiness.
2. **Common causes:**
   - **Timing/race conditions** - Tests depending on setTimeout, async operations, or external services.
   - **Shared state** - Tests leaking state to each other (global variables, database rows, file artifacts).
   - **Non-deterministic input** - Using `Date.now()`, `Math.random()`, or system-dependent values.
   - **Resource contention** - Ports, files, or connections not properly cleaned up.
3. **Fix before proceeding** - Flaky tests MUST be fixed, not skipped. A flaky test masks real failures.
4. **Write deterministic tests** - Use fixed seeds, mock time/date, isolate test state.

### 8. Production vs Local Debugging

| Aspect | Local Debugging | Production Debugging |
|--------|----------------|---------------------|
| **Access** | Full code, debugger, breakpoints | Logs, metrics, traces only |
| **Reproduce** | Run test directly | Analyze logs, attempt local repro with prod data shape |
| **Tools** | IDE debugger, `console.log`, `pdb` | Datadog MCP (`search_logs`, `query_traces`), error tracking |
| **Safety** | No risk | Read-only access; never modify prod data to debug |
| **Strategy** | Step through code | Correlate logs → traces → metrics to find the failure point |

### 9. Log Analysis Patterns

When debugging with logs (local or production):

1. **Find the error** - Search for ERROR/FATAL level entries around the incident timestamp.
2. **Trace the request** - Use correlation/request ID to follow the full request path.
3. **Check surrounding context** - Look at WARN entries before the error for early indicators.
4. **Compare with success** - Find a successful request and diff the log patterns to spot divergence.
5. **Use structured search** - With Datadog MCP: `search_logs` with filters for service, status, and time range.

### 10. Cross-Skill Integration

| Situation | Skill to invoke |
|-----------|----------------|
| Performance-related bug | **performance** skill (profile first) |
| Security-related bug | **security-reviewer** skill |
| Bug in CI/CD pipeline | **ci-cd** skill |
| Bug caused by dependency update | **dependencies** skill |

**See also:** **testing** skill for writing reproduction tests (TDD approach for bug fixes).

---

## Checklist

- [ ] Reproducible (steps and environment clear).
- [ ] **Failing reproduction test written** (TDD - MANDATORY).
- [ ] Hypothesis tested (log/assert/bisect) before coding the fix.
- [ ] Fix targets root cause; **reproduction test now passes**.
- [ ] All existing tests still pass (no regressions).
- [ ] No unrelated changes in the fix.
- [ ] Flaky tests identified and fixed (not skipped).
- [ ] Production debugging done via logs/traces only (no prod data modification).
