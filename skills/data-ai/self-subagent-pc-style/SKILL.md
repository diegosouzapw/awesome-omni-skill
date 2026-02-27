---
name: self-subagent
description: Orchestrate parallel sub-tasks by spawning non-interactive instances of your own CLI as subagents. Use when you need to parallelize work across multiple files, run independent investigations simultaneously, or delegate heavy multi-step tasks. Works with ANY AI coding CLI agent (Amp, Claude Code, Codex, Cursor, OpenCode, aider, Cline, Roo, goose, Windsurf, Copilot CLI, pi, etc.). Triggers on "run in parallel", "subagent", "delegate", "fan out", "concurrent tasks", or any complex task that benefits from parallel execution.
---

# Self-Subagent Orchestration

Spawn parallel copies of yourself in non-interactive mode to do work concurrently.

```
YOU (parent, interactive)
 ├─ spawn ──→ [self --exec "task A"]  ──→ result A ─┐
 ├─ spawn ──→ [self --exec "task B"]  ──→ result B ─┼─→ collect → verify → done
 └─ spawn ──→ [self --exec "task C"]  ──→ result C ─┘
```

Each subagent is **fire-and-forget**: receives a complete prompt, does the work, exits. No follow-ups.

## Phase 1: Discover Your Execute Mode

You must figure out how to invoke yourself non-interactively. **Do not assume** — discover.

### 1a. Identify what CLI you are

```bash
# Check parent process
ps -p $PPID -o comm= 2>/dev/null

# Check known agent CLIs on PATH
for cmd in amp claude codex cursor opencode aider pi goose cline roo windsurf copilot; do
  command -v "$cmd" &>/dev/null && echo "$cmd"
done
```

### 1b. Read your own --help

Once identified, **read the help** to find the non-interactive/execute/print mode:

```bash
# Replace YOUR_CLI with the identified binary
YOUR_CLI --help 2>&1 | grep -iE 'exec|non.?interactive|print|batch|run|pipe|headless|-p |-x '
YOUR_CLI exec --help 2>&1   # some CLIs nest it under a subcommand
YOUR_CLI run --help 2>&1
```

Look for flags that indicate:
- **Non-interactive execution**: `exec`, `run`, `-x`, `-p`, `--print`, `--batch`, `--headless`
- **Auto-approve / skip permissions**: `--yes`, `--auto`, `--full-auto`, `--dangerously-*`, `--no-confirm`
- **Structured output**: `--json`, `--output-format`, `--stream-json`
- **Stdin support**: `--stdin`, `-` as argument, pipe support

### 1c. If unknown, use web search or tool docs

If `--help` is insufficient, search for documentation:
- Search: `"<cli-name> non-interactive mode"` or `"<cli-name> exec mode"`
- Check the CLI's GitHub README
- Look for `AGENTS.md`, `CLAUDE.md`, or similar instruction files in the project root

### 1d. Known profiles (quick reference)

| CLI | Execute command | Auto-approve | JSON output |
|-----|----------------|--------------|-------------|
| amp | `amp -x "prompt"` | `--dangerously-allow-all` | `--stream-json` |
| claude | `claude -p "prompt"` | `--dangerously-skip-permissions` | `--output-format json` |
| codex | `codex exec "prompt"` | `--full-auto` | `--json` |
| aider | `echo "prompt" \| aider --yes-always` | built-in | — |
| opencode | `opencode run "prompt"` | — | — |
| pi | `pi -p "prompt"` | — | — |
| goose | `goose session --non-interactive "prompt"` | — | — |

**Full details, edge cases, and output capture**: see [references/cli-profiles.md](references/cli-profiles.md)

### 1e. Test it

Before spawning real work, validate with a trivial prompt:

```bash
AGENT_CMD="claude -p --dangerously-skip-permissions"  # or whatever you discovered
echo "Reply with exactly: PING" | timeout 30 $AGENT_CMD 2>&1
# Should output something containing "PING"
```

### 1f. Fallback

If no non-interactive mode exists, fall back to shell scripts with standard tools:

```bash
bash -c 'cat src/auth.ts | head -50 && echo "ANALYSIS: ..."'
```

This loses AI reasoning but still enables parallel scripted work.

## Phase 2: Decompose Into a Task Graph

Do NOT just list tasks. Build a **dependency graph** — this is what enables maximum parallelism.

### 2a. Identify tasks and their write targets

For each task, declare:
- `id`: short identifier
- `writes`: files this task will create or modify
- `reads`: files this task needs (read-only)
- `depends_on`: task IDs that must complete first

### 2b. Build the graph

```
Example: "Add logging and tests to auth + payments modules"

  task1: {id: "log-auth",     writes: [src/auth.ts],              depends_on: []}
  task2: {id: "log-payments",  writes: [src/payments.ts],          depends_on: []}
  task3: {id: "test-auth",     writes: [tests/auth.test.ts],       depends_on: ["log-auth"]}
  task4: {id: "test-payments", writes: [tests/payments.test.ts],   depends_on: ["log-payments"]}
  task5: {id: "update-ci",     writes: [.github/workflows/ci.yml], depends_on: ["test-auth", "test-payments"]}

  Wave 1: [task1, task2]           ← parallel (disjoint writes, no deps)
  Wave 2: [task3, task4]           ← parallel (disjoint writes, wave 1 done)
  Wave 3: [task5]                  ← serial (depends on wave 2)
```

### 2c. Scheduling rules

| Condition | Action |
|-----------|--------|
| No dependency + disjoint writes | Parallel |
| Depends on another task's output | Wait for dependency |
| Two tasks write the same file | Serialize OR use git worktrees |
| Read-only task (research, review) | Always parallelizable |
| > 6 tasks ready simultaneously | Throttle to 6 concurrent |

### 2d. Wave execution

Group tasks into **waves** — each wave is a set of tasks that can all run in parallel:

```
Wave 1: all tasks with 0 unmet dependencies    → spawn all, wait all
Wave 2: tasks whose deps were all in wave 1     → spawn all, wait all
...repeat until all tasks complete
```

## Phase 3: Write Subagent Prompts

Each prompt must be **completely self-contained**. The subagent knows nothing about your session.

### Template

```
ROLE: You are a focused code executor. Do exactly what is asked. Do not explore beyond scope.
GOAL: [one sentence]
WORKING DIRECTORY: [absolute path]
READ FIRST: [file list — the subagent should read these to understand context]
MODIFY: [exact file list — the ONLY files the subagent may write to]
DO NOT MODIFY: anything not listed above
CONSTRAINTS:
- [coding style, framework, patterns to follow]
- [specific things to avoid]
DELIVERABLES:
- [what each output file should contain]
VALIDATION:
- [command to run, e.g. "npx tsc --noEmit && npm test -- --testPathPattern=auth"]
CONTEXT:
[paste relevant code snippets, types, interfaces — anything the subagent needs]
```

### Prompt size guidelines

- Include **all** necessary context inline — file contents, type definitions, examples
- For large contexts (>4K chars), write to a temp file and instruct the subagent to read it
- Be extremely specific about constraints — the subagent will improvise if you're vague

### Role prefixes

| Role | Prefix | Use for |
|------|--------|---------|
| **Executor** | "You are a focused code executor." | Implementation, refactors, migrations |
| **Researcher** | "You are a codebase researcher. Do NOT edit any files." | Code search, architecture analysis |
| **Reviewer** | "You are a senior code reviewer. Do NOT edit any files." | Code review, security audit |
| **Planner** | "You are a technical planner. Do NOT edit any files." | Architecture decisions, migration plans |

## Phase 4: Spawn, Collect, Verify

### 4a. Spawn a wave

```bash
AGENT_CMD="claude -p --dangerously-skip-permissions"  # from Phase 1
TMPDIR=$(mktemp -d)
PIDS=()
TASK_NAMES=()

spawn_task() {
  local id="$1" prompt="$2"
  timeout 300 $AGENT_CMD "$prompt" > "$TMPDIR/$id.out" 2>&1 &
  PIDS+=($!)
  TASK_NAMES+=("$id")
}

# Wave 1
spawn_task "log-auth" "$(cat <<'EOF'
ROLE: You are a focused code executor.
GOAL: Add structured logging to src/auth.ts
...
EOF
)"

spawn_task "log-payments" "$(cat <<'EOF'
ROLE: You are a focused code executor.
GOAL: Add structured logging to src/payments.ts
...
EOF
)"

# Wait for wave
FAILED=()
for i in "${!PIDS[@]}"; do
  if ! wait "${PIDS[$i]}"; then
    FAILED+=("${TASK_NAMES[$i]}")
  fi
done
```

### 4b. Collect results

```bash
for id in "${TASK_NAMES[@]}"; do
  echo "=== $id (exit: $(wait ${PIDS[$i]}; echo $?)) ==="
  tail -20 "$TMPDIR/$id.out"  # last 20 lines usually have the summary
done

# See what actually changed on disk
git diff --stat
```

### 4c. Retry failures

If a task failed, retry it once with the error output appended:

```bash
for id in "${FAILED[@]}"; do
  ERROR=$(tail -50 "$TMPDIR/$id.out")
  RETRY_PROMPT="$ORIGINAL_PROMPT

PREVIOUS ATTEMPT FAILED. Error output:
$ERROR

Fix the issue and try again."
  timeout 300 $AGENT_CMD "$RETRY_PROMPT" > "$TMPDIR/$id.retry.out" 2>&1
done
```

After 1 retry, do the task yourself — don't loop.

### 4d. Verify the wave

Run project-wide validation after each wave:

```bash
# Adapt to your project
npx tsc --noEmit && npm test && npm run lint
```

Only proceed to the next wave if validation passes.

### 4e. Diff-Based Verification (Pre-Merge)

**Before running the quality gate, verify the diff itself is safe to merge.**

The `diff-verify.sh` script performs three checks on the raw `git diff`:

| Check | What | Action on Failure |
|-------|------|-------------------|
| **Secret Scan** | Scans added lines for API keys, tokens, credentials | Hard block (exit 2) + auto-revert |
| **Rogue Edit Detection** | Flags files modified outside declared targets | Reject (exit 1) + auto-revert |
| **Diff Proportionality** | Checks total changes vs task complexity threshold | Reject (exit 1) + auto-revert |

#### Secret Detection Patterns

Scans for 25+ patterns across:
- **Generic**: `api_key`, `secret`, `password`, `token`, `private_key`
- **Provider-specific**: OpenAI (`sk-`), GitHub (`ghp_`, `gho_`, `ghs_`), AWS (`AKIA`), Stripe (`sk_live_`), Slack (`xox`), Google (`AIza`), SendGrid (`SG.`)
- **Structural**: PEM private keys, connection strings with embedded credentials
- **Suspicious markers**: `TODO.*remove.*key`, `FIXME.*secret`

Allowlist exempts: `process.env`, `os.environ`, shell variable refs, `placeholder`, `test_key`, `mock_secret`

#### Usage

```bash
# Standalone
./skill/diff-verify.sh <subagent_dir> <results_dir> <expected_files> [complexity]

# Exit codes: 0=PASS, 1=FAIL, 2=SECRETS_FOUND

# Integrated (called automatically by quality-gate.sh Phase 0)
./skill/quality-gate.sh <subagent_dir> <results_dir> <expected_files> [complexity]
# Now runs diff verification FIRST, then quality scoring
```

#### Auto-Revert

On any verification failure, changes are automatically reverted:
```bash
git checkout -- .   # revert tracked changes
git clean -fd       # remove untracked files
```

This ensures the working directory is clean before:
- Retrying the subagent with error context
- Proceeding to the next wave
- Falling back to inline execution

#### Exit Code Chain

| quality-gate.sh exit | Meaning | What Happened |
|---------------------|---------|---------------|
| 0 | ACCEPT | Diff clean + score >= 6 |
| 1 | REJECT | Diff failed OR score < 6 |
| 2 | SECRETS_FOUND | Hard block, auto-reverted |

### 4f. Quality Gate

**Before accepting any subagent's output, score it 0-10 on three criteria:**

#### Gate Criteria

| Criterion | Weight | Check |
|-----------|--------|-------|
| **File Scope** | 4 pts | Only modified declared files, no rogue edits |
| **Validation** | 4 pts | Typecheck/lint/tests pass |
| **Diff Size** | 2 pts | Diff proportional to task complexity |

#### Scoring Algorithm

```bash
#!/usr/bin/env bash
# quality-gate.sh - Score subagent output before merging

SUBAGENT_DIR="$1"      # Temp worktree or directory
RESULTS_DIR="$2"       # Where to save scores
EXPECTED_FILES="$3"    # Space-separated list of expected modified files
TASK_COMPLEXITY="${4:-medium}"  # small|medium|large

cd "$SUBAGENT_DIR"

SCORE=10
FAILURES=""

# 1. File Scope Check (4 points)
MODIFIED=$(git diff --name-only 2>/dev/null | sort)
UNEXPECTED=0
MISSING=0

for file in $MODIFIED; do
  if [[ ! " $EXPECTED_FILES " =~ " $file " ]]; then
    UNEXPECTED=$((UNEXPECTED + 1))
    FAILURES="${FAILURES}UNEXPECTED: $file\n"
  fi
done

for expected in $EXPECTED_FILES; do
  if ! echo "$MODIFIED" | grep -q "^$expected$"; then
    MISSING=$((MISSING + 1))
    FAILURES="${FAILURES}MISSING: $expected\n"
  fi
done

if [[ $UNEXPECTED -gt 0 || $MISSING -gt 0 ]]; then
  SCORE=$((SCORE - 4))
  echo "⚠️  File scope violation: $UNEXPECTED unexpected, $MISSING missing"
fi

# 2. Validation Check (4 points)
VALIDATION_FAILED=0

# TypeScript
if command -v npx >/dev/null 2>&1; then
  if ! npx tsc --noEmit 2>&1 | head -20; then
    SCORE=$((SCORE - 2))
    VALIDATION_FAILED=1
    FAILURES="${FAILURES}TypeScript compilation failed\n"
    echo "❌ TypeScript compilation failed"
  fi
fi

# Lint
if [[ -f "package.json" ]] && grep -q '"lint"' package.json 2>/dev/null; then
  if ! npm run lint 2>&1 | tail -10; then
    SCORE=$((SCORE - 1))
    VALIDATION_FAILED=1
    FAILURES="${FAILURES}Lint failed\n"
    echo "❌ Lint failed"
  fi
fi

# Tests
if [[ -f "package.json" ]] && grep -q '"test"' package.json 2>/dev/null; then
  if ! npm test 2>&1 | tail -10; then
    SCORE=$((SCORE - 1))
    VALIDATION_FAILED=1
    FAILURES="${FAILURES}Tests failed\n"
    echo "❌ Tests failed"
  fi
fi

# 3. Diff Size Check (2 points)
DIFF_LINES=$(git diff --stat 2>/dev/null | tail -1 | awk '{print $1}')

# Thresholds by complexity
if [[ "$TASK_COMPLEXITY" == "small" ]]; then
  MAX_LINES=50
elif [[ "$TASK_COMPLEXITY" == "large" ]]; then
  MAX_LINES=500
else
  MAX_LINES=200  # medium
fi

if [[ $DIFF_LINES -gt $MAX_LINES ]]; then
  SCORE=$((SCORE - 2))
  FAILURES="${FAILURES}Diff too large: $DIFF_LINES lines (max $MAX_LINES for $TASK_COMPLEXITY task)\n"
  echo "⚠️  Diff size: $DIFF_LINES lines exceeds threshold ($MAX_LINES)"
fi

# Clamp to 0-10
[[ $SCORE -lt 0 ]] && SCORE=0
[[ $SCORE -gt 10 ]] && SCORE=10

# Save results
echo "$SCORE" > "$RESULTS_DIR/quality_score"
cat > "$RESULTS_DIR/quality_report.txt" << EOF
Quality Gate Report
===================
Score: $SCORE/10

Criteria:
- File Scope: $([[ $UNEXPECTED -eq 0 && $MISSING -eq 0 ]] && echo "PASS" || echo "FAIL") ($UNEXPECTED unexpected, $MISSING missing)
- Validation: $([[ $VALIDATION_FAILED -eq 0 ]] && echo "PASS" || echo "FAIL")
- Diff Size: $DIFF_LINES lines $([[ $DIFF_LINES -le $MAX_LINES ]] && echo "(PASS)" || echo "(FAIL - max $MAX_LINES)")

Failures:
${FAILURES:-None}

Modified Files:
$(git diff --name-only 2>/dev/null || echo "N/A")
EOF

# Decision
echo ""
echo "╔════════════════════════════════════╗"
echo "║     QUALITY GATE: $SCORE/10        ║"
echo "╚════════════════════════════════════╝"

if [[ $SCORE -ge 6 ]]; then
  echo "✅ ACCEPT: Changes meet quality threshold"
  exit 0
else
  echo "❌ REJECT: Changes below quality threshold"
  echo "   Options:"
  echo "   1. Retry subagent with error context"
  echo "   2. Do task inline (parent handles it)"
  exit 1
fi
```

#### Usage in Wave Execution

```bash
# After collecting subagent results
for id in "${TASK_NAMES[@]}"; do
  # Run quality gate on each subagent's output
  if quality-gate.sh "$TMPDIR/$id-worktree" "$RESULTS_DIR" "${TASK_WRITES[$id]}" "medium"; then
    # Merge changes
    git merge "subagent/$id" --no-edit
  else
    # Retry once, then abandon
    FAILED_TASKS+=("$id")
  fi
done

# Only proceed if all tasks passed quality gate
if [[ ${#FAILED_TASKS[@]} -gt 0 ]]; then
  echo "❌ Wave failed quality gate for: ${FAILED_TASKS[*]}"
  # Retry or handle inline
fi
```

#### Decision Matrix

| Score | Action |
|-------|--------|
| 10 | Perfect - merge immediately |
| 8-9 | Good - merge with note |
| 6-7 | Acceptable - merge, monitor next wave |
| 5 | Borderline - retry with context |
| <5 | Reject - retry once, then do inline |

#### Integration with Retry

When retrying a failed quality gate:

```bash
retry_with_quality_context() {
  local id="$1"
  local original_prompt="$2"
  local quality_report=$(cat "$RESULTS_DIR/$id/quality_report.txt")
  
  RETRY_PROMPT="$original_prompt

QUALITY GATE FAILED (Score: $(cat $RESULTS_DIR/$id/quality_score)/10)

Issues to fix:
$quality_report

Please address these issues and ensure:
1. Only modify the declared target files
2. All validation passes (typecheck, lint, tests)
3. Keep changes focused and proportional to the task"

  # Retry with enhanced prompt
  timeout 300 $AGENT_CMD "$RETRY_PROMPT" > "$TMPDIR/$id.retry.out" 2>&1
}
```

#### Rules

1. **Score < 6 = Reject** - Don't merge low-quality changes
2. **Retry once** - Give subagent a chance to fix with context
3. **After 2 failures, do inline** - Parent takes over
4. **Log all scores** - Track quality trends across waves
5. **Fail fast** - Reject early, don't waste time on bad output

### 4g. Proceed to next wave

Clear PIDs, spawn the next wave's tasks (whose dependencies are now met), repeat 4a-4d.

## Advanced Patterns

See [references/orchestration.md](references/orchestration.md) for:
- Git worktree isolation (parallel writes to overlapping files)
- Chained pipelines (researcher → planner → executor)
- Structured JSON output collection and manifests
- Throttle governors and resource limits
- Streaming results as they complete

## Rules

1. **Discover, don't assume** — always check `--help` before using any CLI flags.
2. **Max 6 concurrent subagents** — more causes resource contention.
3. **Always timeout** — `timeout 300` (5 min) default, adjust per task complexity.
4. **Disjoint writes only** — never let two subagents write the same file in the same wave.
5. **Verify every wave** — run typecheck/tests/lint before proceeding.
6. **Full context in every prompt** — subagents have zero memory of the parent session.
7. **Retry once, then do it yourself** — don't retry-loop.
8. **Prefer fewer, larger tasks** — process spawn overhead is real.
