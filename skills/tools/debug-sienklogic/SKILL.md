---
name: debug
description: "Systematic debugging with hypothesis testing. Persistent across sessions."
---

## Step 0 — Immediate Output

**Before ANY tool calls**, display this banner:

```
╔══════════════════════════════════════════════════════════════╗
║  PLAN-BUILD-RUN ► DEBUGGING                                  ║
╚══════════════════════════════════════════════════════════════╝
```

Then proceed to Step 1.

# /pbr:debug — Systematic Debugging

You are running the **debug** skill. Your job is to run a structured, hypothesis-driven debugging session that persists across conversations. You track every hypothesis, test, and finding in a debug file so work is never lost.

This skill **spawns Task(subagent_type: "pbr:debugger")** for investigation work.

---

## Context Budget

Reference: `skills/shared/context-budget.md` for the universal orchestrator rules.

Additionally for this skill:
- **Never** perform investigation work yourself — delegate ALL analysis to the debugger subagent
- **Minimize** reading debug file content — read only the latest hypothesis and result section
- **Delegate** all code reading, hypothesis testing, and fix attempts to the debugger subagent

---

## Core Principle

**Debug systematically, not randomly.** Every investigation step must have a hypothesis, a test, and a recorded result. No "let me just try this" — every action has a reason and is documented.

---

## Flow

### Step 1: Ensure Debug Directory Exists

Before any file operations, ensure both directories exist by running:

```bash
mkdir -p .planning/debug
```

This handles the case where neither `.planning/` nor `.planning/debug/` exist yet (debug can be run before other skills that create `.planning/`). Do NOT skip this step — writing files to a non-existent directory will fail.

### Step 2: Check for Active Debug Sessions

**Load depth profile:** Run `node ${PLUGIN_ROOT}/scripts/pbr-tools.js config resolve-depth` to get `debug.max_hypothesis_rounds`. If the command fails (no config.json or CLI error), default to 5 rounds. Initialize a round counter at 0. This counter increments each time a continuation debugger is spawned.

Scan `.planning/debug/` for existing debug files:

```
.planning/debug/
  {NNN}-{slug}.md     # Each debug session is a file
```

Read each file's frontmatter to check status:
- `status: gathering` — collecting symptoms from user
- `status: investigating` — testing hypotheses
- `status: fixing` — applying fix
- `status: verifying` — confirming fix works
- `status: resolved` — session complete

**If active sessions found:**

Use the **debug-session-select** pattern from `skills/shared/gate-prompts.md`:
  question: "Found active debug sessions. Which would you like?"

Generate options dynamically from active sessions:
- Each active session becomes an option: label "#{NNN}: {title}", description "Started {date}, last: {last hypothesis}"
- Always include "New session" as the last option: description "Start a fresh debug investigation"
- If more than 3 active sessions exist, show only the 3 most recent plus "New session" (max 4 options)

Handle responses:
- If user selects an existing session: go to **Resume Flow** (Step 3b)
- If user selects "New session": go to **New Session Flow** (Step 3a)
- If user types a session number not in the list: look it up and resume it

**If no active sessions found:**
- Go to **New Session Flow** (Step 3a)

### Step 3a: New Session Flow

#### Gather Symptoms

If `$ARGUMENTS` is provided and descriptive:
- Use it as the initial issue description
- Still ask targeted follow-up questions

If `$ARGUMENTS` is empty or minimal:
- Ask the user for symptoms

**Symptom gathering questions** (ask as plain text — these are freeform, do NOT use AskUserQuestion):

1. **Expected behavior**: "What should happen?"
2. **Actual behavior**: "What actually happens? Include error messages if any."
3. **Reproduction**: "How do you trigger this? Steps to reproduce?"
4. **Onset**: "When did this start? Did anything change recently (new code, dependency update, config change)?"
5. **Scope**: "Does this affect everything or just specific cases? Any patterns?"

**Optional follow-ups** (ask if relevant):
- "What have you already tried?"
- "Does this happen in all environments (dev, prod, test)?"
- "Any relevant log output?"

#### Generate Session ID

1. Scan `.planning/debug/` for existing files
2. Extract NNN prefixes
3. Next number = highest + 1 (start at 001)
4. Generate slug from issue title (same rules as quick task slugs)

#### Create Debug File

Create `.planning/debug/{NNN}-{slug}.md`:

```markdown
---
id: "{NNN}"
title: "{issue title}"
status: gathering
created: "{ISO date}"
updated: "{ISO date}"
severity: "{critical|high|medium|low}"
category: "{runtime|build|test|config|integration|unknown}"
---

# Debug Session: {title}

## Symptoms

**Expected:** {expected behavior}
**Actual:** {actual behavior}
**Reproduction:** {steps}
**Onset:** {when it started}
**Scope:** {affected areas}

## Environment

- OS: {detected or reported}
- Runtime: {node version, python version, etc.}
- Relevant config: {any config that matters}

## Investigation Log

### Round 1 (automated)

{This section is filled by debugger}

## Hypotheses

| # | Hypothesis | Status | Evidence |
|---|-----------|--------|----------|
| 1 | {hypothesis} | {testing/confirmed/rejected} | {evidence} |

## Root Cause

{Filled when found}

## Fix Applied

{Filled when fixed}

## Timeline

- {ISO date}: Session created
```

#### Spawn Debugger

Display to the user: `◐ Spawning debugger...`

Spawn `Task(subagent_type: "pbr:debugger")` with the prompt template.

Read `skills/debug/templates/initial-investigation-prompt.md.tmpl` for the spawn prompt. Fill in the `{NNN}`, `{slug}`, and symptom placeholders with values from the debug file created above.

### Step 3b: Resume Flow

1. Read the debug file content
2. Parse the investigation log and hypotheses table
3. Display to user:

```
Resuming debug session #{NNN}: {title}

Last state:
- Hypotheses tested: {N}
- Confirmed: {list or "none yet"}
- Rejected: {list}
- Current lead: {most promising hypothesis}

Continuing investigation...
```

4. **Increment the round counter** (resuming counts as a new round). Display to the user: `◐ Spawning debugger (resuming session #{NNN}, round {N})...`

   Spawn `Task(subagent_type: "pbr:debugger")` with the continuation prompt template.

   Read `skills/debug/templates/continuation-prompt.md.tmpl` for the spawn prompt. Fill in the `{NNN}`, `{slug}`, and `{paste investigation log...}` placeholders with data from the debug file.

### Step 4: Handle Debugger Results

When the debugger agent completes, first check for completion markers in the Task() output before routing:

| Marker in Task() Output | Route To |
|--------------------------|----------|
| `## DEBUG COMPLETE` | ROOT CAUSE FOUND + FIX path |
| `## ROOT CAUSE FOUND` | ROOT CAUSE FOUND (no fix) path |
| `## DEBUG SESSION PAUSED` | CHECKPOINT path |
| No marker found | INCONCLUSIVE path |

**Spot-check:** Before routing, verify `.planning/debug/{NNN}-{slug}.md` exists and was recently updated (modified timestamp is newer than the Task() spawn time). If the debug file was not updated, warn: `⚠ Debug file not updated by agent — results may be incomplete.`

Display: `✓ Debug session complete — {N} hypotheses tested` (read the hypothesis count from the debug file's Hypotheses table).

The debugger returns one of four outcomes:

#### ROOT CAUSE FOUND + FIX

```
Root cause identified: {cause}
Fix applied: {description}
Commit: {hash}
```

Actions:
1. Update debug file:
   - Set `status: resolved`
   - Fill "Root Cause" section
   - Fill "Fix Applied" section
   - Add timeline entry
2. Update STATE.md if it has a Debug Sessions section
3. Report to user with branded output:

```
╔══════════════════════════════════════════════════════════════╗
║  PLAN-BUILD-RUN ► BUG RESOLVED ✓                             ║
╚══════════════════════════════════════════════════════════════╝

**Session #{NNN}:** {title}
**Root cause:** {cause}
**Fix:** {description}
**Commit:** {hash}

╔══════════════════════════════════════════════════════════════╗
║  ▶ NEXT UP                                                   ║
╚══════════════════════════════════════════════════════════════╝

**Continue your workflow** — the bug is fixed

`/pbr:status`

<sub>`/clear` first → fresh context window</sub>

**Also available:**
- `/pbr:continue` — execute next logical step
- `/pbr:review {N}` — verify the current phase

```

#### ROOT CAUSE FOUND (no fix)

Used when the debugger was invoked with `find_root_cause_only` or when the fix is too complex for auto-application.

```
Root cause identified: {cause}
Suggested fix: {approach}
```

Actions:
1. Update debug file:
   - Set `status: resolved`
   - Fill "Root Cause" section
   - Add suggested fix to notes
2. Suggest next steps to user:

```

╔══════════════════════════════════════════════════════════════╗
║  ▶ NEXT UP                                                   ║
╚══════════════════════════════════════════════════════════════╝

**Apply the fix** — root cause identified, fix needed

`/pbr:quick {fix description}`

<sub>`/clear` first → fresh context window</sub>

**Also available:**
- `/pbr:plan` — for complex fixes that need planning
- `/pbr:status` — see project status

```

#### CHECKPOINT

The debugger found something but needs user input or more investigation.

```
Investigation progress:
- Tested: {hypotheses tested}
- Found: {key finding}
- Need: {what's needed to continue}
```

Actions:
1. Update debug file with findings so far
2. Present checkpoint to user
3. Use the **debug-checkpoint** pattern from `skills/shared/gate-prompts.md`:
   question: "Investigation has reached a checkpoint. How should we proceed?"

Handle responses:
- "Continue": **Increment the round counter** (e.g., round 1 becomes round 2). Then display `◐ Spawning debugger (continuing investigation, round {N})...` and spawn another `Task(subagent_type: "pbr:debugger")` with updated context from the debug file
- "More info": **Increment the round counter.** Ask the user freeform what additional context they have, then update the debug file and spawn another debugger
- "New approach": **Increment the round counter.** Ask the user freeform what alternative angle to try, then update hypotheses and spawn another debugger

#### INCONCLUSIVE

The debugger exhausted its hypotheses without finding the root cause.

```
Investigation exhausted:
- Tested: {all hypotheses}
- Rejected: {list}
- Remaining unknowns: {list}
```

Actions:
1. Update debug file with all findings
2. Report to user:
   - What was tested and eliminated
   - What remains unknown
   - Suggested next investigation approaches:
     - Different reproduction steps
     - Log-level debugging
     - Environment comparison
     - Bisect (git bisect to find the breaking commit)
     - External help (stack overflow, docs)
3. Keep session active for future resumption

---

## Debugger Investigation Protocol

The debugger agent follows this protocol internally:

### Hypothesis-Driven Investigation

```
1. OBSERVE: Read error messages, logs, code around the failure point
2. HYPOTHESIZE: "The most likely cause is X because Y"
3. PREDICT: "If X is the cause, then test Z should show W"
4. TEST: Execute test Z
5. EVALUATE:
   - Result matches prediction → hypothesis supported → investigate deeper
   - Result contradicts → hypothesis rejected → try next hypothesis
   - Result is unexpected → new information → form new hypothesis
```

### Investigation Techniques

| Technique | When to Use |
|-----------|------------|
| **Stack trace analysis** | Error with stack trace available |
| **Code path tracing** | Logic error, wrong behavior |
| **Log injection** | Need to see runtime values |
| **Binary search** | Know it worked before, need to find when it broke |
| **Isolation** | Complex system, need to narrow scope |
| **Comparison** | Works in one case, fails in another |
| **Dependency audit** | Recent dependency changes |
| **Config diff** | Works in one environment, not another |

### Evidence Quality

| Quality | Description | Action |
|---------|-------------|--------|
| **Strong** | Directly proves/disproves hypothesis | Record and move on |
| **Moderate** | Suggests but doesn't prove | Record, seek corroboration |
| **Weak** | Tangentially related | Note but don't base decisions on it |
| **Misleading** | Red herring | Record as eliminated, explain why |

### Hypothesis Round Limit

The maximum number of investigation rounds is controlled by the depth profile's `debug.max_hypothesis_rounds` setting:
- `quick`: 3 rounds (fast, surface-level investigation)
- `standard`: 5 rounds (default)
- `comprehensive`: 10 rounds (deep investigation)

The orchestrator tracks the round count. Before spawning each continuation debugger (Step 4 "CHECKPOINT" -> "Continue"), increment the round counter. If the counter reaches the limit:
- Do NOT spawn another debugger
- Present to user: "Debug session has reached the hypothesis round limit ({N} rounds for {depth} mode). Options:"

Use AskUserQuestion:
  question: "Reached {N}-round hypothesis limit. How should we proceed?"
  header: "Debug Limit"
  options:
    - label: "Extend"    description: "Allow {N} more rounds (doubles the limit)"
    - label: "Wrap up"   description: "Record findings so far and close the session"
    - label: "Escalate"  description: "Save context for manual debugging"

- If "Extend": double the limit and continue
- If "Wrap up": update debug file `status: resolved` with `resolution: abandoned`, record all findings, suggest next steps
- If "Escalate": write a detailed handoff document to the debug file with all hypotheses, evidence, and suggested manual investigation steps

---

## Debug File Management

### Lifecycle

```
gathering → investigating → fixing → verifying → resolved
(any non-resolved) → resolved  (with resolution: abandoned, if 7+ days stale)
(any non-resolved) → (same)    (resumed after pause)
```

### Staleness Detection

When scanning for active sessions, check the `updated` date. If more than 7 days old:
- Set `status: resolved` with `resolution: abandoned` in frontmatter
- Still offer to resume, but warn: "This session is {N} days old. Context may have changed."

### Cleanup

Old resolved debug files can accumulate. They serve as a knowledge base for similar issues. Do NOT auto-delete them.

---

## State Integration

Update STATE.md Debug Sessions section (create if needed):

```markdown
### Debug Sessions

| # | Issue | Status | Root Cause |
|---|-------|--------|------------|
| 001 | Login timeout | resolved | DB connection pool exhausted |
| 002 | CSS not loading | active | investigating |
```

---

## Git Integration

Reference: `skills/shared/commit-planning-docs.md` for the standard commit pattern.

If `planning.commit_docs: true` in config.json:
- New session: `docs(planning): open debug session {NNN} - {slug}`
- Resolution: `docs(planning): resolve debug session {NNN} - {root cause summary}`
- Fix commits use standard format: `fix({scope}): {description}`

---

## Edge Cases

### User provides a stack trace or error in arguments
- Parse it as the "Actual behavior" symptom
- Extract key information: error type, file, line number
- Use this to form initial hypotheses immediately

### Debugger agent fails
If the debugger Task() fails or returns an error, display:
```
╔══════════════════════════════════════════════════════════════╗
║  ERROR                                                       ║
╚══════════════════════════════════════════════════════════════╝

Debugger agent failed for session #{NNN}.

**To fix:**
- Check the debug file at `.planning/debug/{NNN}-{slug}.md` for partial findings
- Re-run with `/pbr:debug` to resume the session
- If the issue persists, try a fresh session with different symptom details
```

### Issue is in a dependency (not user code)
- Document which dependency and version
- Check if there's a known issue (search patterns in node_modules, site-packages, etc.)
- Suggest: update dependency, pin version, or work around

### Issue is intermittent
- Note intermittency in symptoms
- Suggest: run multiple times, add timing/logging, check for race conditions
- Investigation must account for non-deterministic reproduction

### Multiple issues interacting
- If investigation reveals multiple separate issues, split into separate debug sessions
- Create additional debug files
- Track each independently

### Fix would be a breaking change
- Report the root cause but DO NOT auto-apply the fix
- Present the trade-offs to the user
- Let the user decide how to proceed

---

## Anti-Patterns

1. **DO NOT** skip hypothesis formation — every test must have a reason
2. **DO NOT** make random changes hoping something works
3. **DO NOT** ignore failed hypotheses — record why they failed
4. **DO NOT** exceed the depth profile's `debug.max_hypothesis_rounds` limit without user confirmation (default: 5 for standard mode)
5. **DO NOT** fix the symptom instead of the root cause
6. **DO NOT** auto-apply fixes for breaking changes
7. **DO NOT** delete debug files — they're a knowledge base
8. **DO NOT** combine multiple bugs into one debug session
9. **DO NOT** skip updating the debug file after each investigation round
10. **DO NOT** start a new session when an active one covers the same issue
