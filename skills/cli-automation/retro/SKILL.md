---
name: retro
description: "Run an automated session retrospective to evaluate velocity, quality, process, and blockers, then persist durable learnings to MEMORY.md. Use at the end of a work session, after completing an epic, or when you want to reflect on what worked and what did not."
argument-hint: "[focus area or session topic]"
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Grep, Glob, Bash(bd:*), Bash(git:*), Bash(wc:*), Write, Edit
---

# Retro: Automated Session Retrospective

You are running a **session retrospective** -- a structured reflection on the current session to extract durable learnings and update project memory. Focus area (optional): **$ARGUMENTS**

## When to Use

- At the end of a work session before signing off
- After completing an epic or major milestone
- When a session felt unusually productive or unusually rough and you want to capture why
- When the user asks "what did we learn?" or "how did that go?"

## Overview

Retro works in 7 phases:

```
Gather session data (git + backlog + conversation context)
  -> Analyze across 5 dimensions
    -> Extract keep/stop/try learnings
      -> Team learning health (if team exists)
        -> Update MEMORY.md with durable insights
          -> Capture action items as beads
            -> Present structured report
```

---

## Phase 1: Gather Session Data

### 1a. Git Activity

Run:

```bash
git log --oneline --since="8 hours ago"
```

Note which commits were made this session, how many files were touched, and the types of changes.

### 1b. Backlog Activity (conditional)

**If `.beads/` or `.tacks/` exists in the project root**, check backlog activity:

```bash
bd stats
bd list --status=closed
```

Note how many tasks were completed, what types they were, and whether any were closed as duplicates or stale (vs. genuinely completed).

**If neither `.beads/` nor `.tacks/` exists**, skip this step and rely on git activity and conversation context alone.

### 1c. Conversation Context

Review the current conversation for:
- **Pivots**: Points where the plan changed direction
- **Blockers**: Things that stalled progress (API errors, unclear requirements, missing tools)
- **Rework**: Tasks that had to be redone or corrected
- **Surprises**: Unexpected findings, both positive and negative

---

## Phase 2: Analyze

Evaluate the session across these five dimensions:

### Velocity

- How many tasks were completed?
- What was the ratio of planned work to discovered work?
- Were any tasks significantly larger or smaller than expected?
- How much time was spent on overhead (setup, debugging tools, context gathering) vs. actual work?
- **Git activity metrics** (if available from Phase 1a):
  - Commit volume and type distribution (feat/fix/chore/docs/refactor)
  - Fix rate: what percentage of commits were corrections? (High fix rate may indicate insufficient verification before changes)
  - Churn concentration: are changes focused on a few files (targeted work) or scattered (exploratory work)?

### Quality

- Did agents produce CONFIRMED findings or hedge with LIKELY/POSSIBLE?
- Were there rework cycles where output had to be corrected?
- Did commits pass on the first try or were there hook failures?
- Was code read and verified before changes, or were assumptions made?

### Process

- Did the dispatch strategy work? Was serialization effective, or did it bottleneck?
- If teams were used, did parallelization pay off or cause coordination overhead?
- Did the orchestrator stay in orchestration mode or get pulled into implementation?
- Were the right agents/skills used for the right tasks?

### Blockers

- What slowed things down?
- Categories to check: API throttling, unclear requirements, missing tools or skills, context window pressure, dependency chains, bugs in tooling
- Which blockers were resolved vs. worked around vs. still open?

### Discoveries

- What unexpected things were found?
- New patterns or anti-patterns in the codebase or workflow?
- New capabilities or limitations of tools/agents discovered?
- Insights about project architecture or domain that were not documented before?

---

## Phase 3: Extract Learnings

For each dimension from Phase 2, identify:

- **Keep doing**: What worked well and should continue
- **Stop doing**: What was wasteful, harmful, or counterproductive
- **Try next time**: Experiments or changes to test in the next session

Be specific and actionable. "Communication was good" is not useful. "Sending spike reports via SendMessage with structured format made orchestrator processing 3x faster" is useful.

### Sharpening Gate

Every "could improve" or "try next time" item MUST pass through this gate before being included in the report or becoming a bead:

1. **Name the specific code/file/workflow** where the problem occurred
2. **State what concretely should change** (a function to add, a check to insert, a pattern to adopt)
3. **Make it assignable** — could an agent implement this in one session without design decisions?

If an observation fails the gate, sharpen it:
- "Testing could be better" → fails all 3
- "gl-renderer.ts has 2 manual blend enable/disable blocks that will be missed on new draw calls" → passes #1
- "Add GLRenderer.drawWithAlpha(alpha, drawFn) that wraps blend state, replace the 2 existing manual blocks in gl-renderer.ts" → passes all 3

Drop observations you cannot sharpen — they are not actionable yet. If the root cause is unclear, create an investigation bead instead ("Investigate why X keeps happening in Y").

---

## Phase 4: Team Learning Health (conditional)

**Only run this phase if `.claude/team.yaml` exists.** Skip entirely for non-team sessions.

### 4a. Read Team Learnings

Read all `memory/agents/*/learnings.md` files. For each member, assess:
- **Total entries**: Count of non-empty bullet points
- **Recent additions**: Entries with dates in the last 7 days
- **Staleness**: Days since the last entry was added
- **Size**: Line count vs. the 60-line cap (30 core + 30 task-relevant)
- **Cross-agent notes**: Any pending notes from other members, especially those older than 14 days

### 4b. Prune Bloated Files

If any learnings file exceeds 50 lines (warning threshold) or 60 lines (hard cap):
1. **Merge similar entries** — Combine entries that say the same thing differently
2. **Archive stale entries** — Move entries older than 21 days (with no recent references) to `memory/agents/<name>/archive.md`
3. **Note promotion candidates** — Do not promote entries inline. Instead, note entries that appear durable and cross-agent, and recommend running `/promote` or `/tend` after this retro to evaluate them with full graduation criteria
4. **Validate cross-agent notes** — Notes older than 14 days must be acknowledged (merged), acted upon (integrated), or discarded (moved to archive with rationale)
5. **Apply tiered structure** — Organize remaining entries into Core (30 lines max, high-reuse fundamentals) and Task-Relevant (30 lines max, context-specific)

### 4c. Assess Learning Velocity

For each member:
- **Growing**: 3+ new learnings in last 7 days → agent is actively learning
- **Steady**: 1-2 new learnings in last 7 days → normal pace
- **Stale**: No new learnings in 7+ days → agent may need richer tasks or the role may be inactive
- **Cold**: No learnings at all → role hasn't been dispatched yet

### 4d. Report Team Learning Health

```markdown
### Team Learning Health
| Member | Entries | Recent | Status | Action Needed |
|--------|---------|--------|--------|--------------|
| [name] | [total] | [recent] | [status] | [prune/archive/promote/none] |
| ... | ... | ... | ... | ... |

**Cross-agent notes delivered this session**: [count]
**Entries pruned/archived**: [count]
**Entries promoted to rules**: [count]
```

### 4e. Append to Retro History

Append a summary to `memory/team/retro-history.md`:

```markdown
## Retro: [date]
- Tasks completed: [count]
- New learnings: [count] across [members] members
- Pruned/archived: [count] entries
- Key insight: [most significant learning from this session]
```

---

## Phase 5: Update MEMORY.md

### 4a. Read Current State

Read the current MEMORY.md:

```
~/.claude/projects/<project-path>/memory/MEMORY.md
```

### 4b. Identify Updates

Determine which learnings are **durable** (useful across sessions) vs. **ephemeral** (only relevant to this session). Only persist durable learnings.

Durable examples:
- "Agent teams work well for parallel audits but add overhead for < 5 tasks"
- "Always read existing skill files before writing new ones to match format"
- "Use --parent for epic hierarchy: bd create --parent=<epic-id>, not bd dep add"

Ephemeral examples (do NOT persist):
- "Completed 8 tasks today"
- "Session started at 2pm"
- "User seemed happy with the output"

When adding new learnings, include the `dispatch:` field for provenance tracking (optional but recommended). Use `dispatch: retro-session` for learnings discovered during retro, or `dispatch: bead-xyz` if the learning traces to a specific task.

### 4c. Apply Updates

Using Write or Edit, update MEMORY.md:
- Add new durable learnings under appropriate sections with dispatch provenance
- Update existing entries if new evidence confirms, refutes, or refines them
- Remove entries that are no longer accurate
- Keep the file under 200 lines (it is loaded into the system prompt)

Example format for new learnings entries:
```markdown
## Workflow Patterns
- /blossom → /sprint pipeline: spike-driven discovery feeds execution naturally (added: 2026-02-13, dispatch: retro-session)
```

The `dispatch:` field is optional. Use `dispatch: retro-session` for learnings discovered during retro, or `dispatch: bead-xyz` if the learning traces to a specific bead. Existing entries without this field are backward-compatible.

If detailed notes are needed for a topic, create a separate file in the memory directory (e.g., `team-patterns.md`, `skill-authoring.md`) and link to it from MEMORY.md.

### 4d. Verify

Read MEMORY.md after edits to confirm it is well-structured and under the line limit.

---

## Phase 6: Capture Action Items (conditional)

**If `.beads/` or `.tacks/` exists**, create a bead for each sharpened action item from Phase 3 (items that passed the sharpening gate):

```bash
bd create --title="<concrete action from sharpening gate>" --type=task --priority=<2-4> \
  --description="From retro on [date]. Context: [relevant finding from Phase 2]. What to change: [specific code/file/workflow change]."
```

Every item here already passed the sharpening gate, so it should be implementable in one session without design decisions. Typically 1-3 beads per retro; zero is fine if no concrete follow-ups emerged.

**If neither `.beads/` nor `.tacks/` exists**, list action items in the report only.

---

## Phase 7: Report

Present a structured retrospective report:

```markdown
## Session Retrospective

### Summary
[1-2 sentence summary of what this session accomplished and its overall character]

### Git Session Stats
[If git activity data was gathered in Phase 1a, include:]
- **Commits**: [total] ([feat] feat, [fix] fix, [chore] chore, [docs] docs, [refactor] refactor)
- **Fix rate**: [percentage]% [interpretation: low/normal/high]
- **Top churning files**: [list top 2-3 with change volume]

### What Went Well
- [specific item with evidence]
- [specific item with evidence]

### What Could Improve
- [specific item with evidence and suggested change]
- [specific item with evidence and suggested change]

### Action Items
- [ ] [bead ID]: [title] (P[priority])
- [ ] [bead ID]: [title] (P[priority])

### Memory Updates
- [list each change made to MEMORY.md: added, updated, or removed]
```

---

## Guidelines

- **Be honest and specific, not generic.** "Quality was good" tells you nothing. "All 12 findings were CONFIRMED with file:line evidence" tells you something.
- **Focus on actionable learnings.** Every "what could improve" should have a concrete suggestion attached.
- **Update MEMORY.md with durable insights only.** Session-specific details belong in the report, not in persistent memory.
- **Keep it concise.** A retro should take less than 2 minutes. Do not over-analyze.
- **Distinguish root causes from symptoms.** "Agent output was low quality" is a symptom. "Agent was given a vague spike description without specific files to investigate" is a root cause.
- **Credit what worked.** Retros that only focus on problems train you to ignore successes. Capture positive patterns too.

See also: /evolution (track how definitions changed over time — feeds retro analysis with concrete edit history), /handoff (standard session close follows retro — retro extracts learnings, handoff prepares the next session), /curate (optimize learnings generated by retro), /promote (graduate durable learnings to rules — run after retro to evaluate promotion signals), /tend (full lifecycle: curate + promote).
