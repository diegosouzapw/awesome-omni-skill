---
name: analyzing-branch-status
description: Use when user wants detailed status report for single autonomy branch including iteration timeline and metrics progression
---

# Analyzing Branch Status

## Overview

Provide comprehensive status report for a single autonomy branch by analyzing all journal commits and extracting timeline, metrics, and state evolution.

**Core principle:** Dispatch branch-analyzer agent for computational analysis. Never manually review commits.

## When to Use

Use this skill when:
- User runs `/branch-status` command
- User wants deep dive into one branch
- User wants to see metrics progression over time
- User wants blocker history for a branch

**DO NOT use for:**
- Listing all branches (use listing-branches instead)
- Comparing two branches (use comparing-branches instead)
- Current branch review (use reviewing-progress instead)

## Quick Reference

| Step | Action | Tool |
|------|--------|------|
| 1. Parse and validate | Normalize branch name, check exists | Bash |
| 2. Dispatch agent | Send branch to branch-analyzer | Task |
| 3. Present report | Display comprehensive status | Direct output |

## Process

### Step 1: Parse and Validate Branch Name

Normalize and validate the branch name:

**Normalize:**
```bash
# If user provided name without autonomy/ prefix, add it
if [[ "$branch_name" != autonomy/* ]]; then
  branch_name="autonomy/$branch_name"
fi
```

**Validate exists:**
```bash
# Check if branch exists (local or remote)
if ! git branch -a | grep -q "$branch_name\$"; then
  echo "Error: Branch '$branch_name' not found among autonomy branches."
  echo ""
  echo "Available autonomy branches:"
  git branch -a | grep 'autonomy/' | sed 's/^..//; s/ -> .*//'
  echo ""
  echo "Run '/list-branches' to see all autonomy branches."
  exit 1
fi
```

**Validate is autonomy branch:**
```bash
# Verify it's an autonomy branch
if [[ "$branch_name" != autonomy/* ]]; then
  echo "Error: Branch '$branch_name' is not an autonomy branch."
  echo ""
  echo "These commands only operate on autonomy/* branches."
  echo ""
  echo "To analyze this branch's iterations:"
  echo "- Run '/review-progress' (works on any branch)"
  echo ""
  echo "To convert to autonomy workflow:"
  echo "- Run '/fork-iteration <strategy-name>' to create autonomy branch from current state"
  exit 1
fi
```

### Step 2: Dispatch Branch-Analyzer Agent

Dispatch the `branch-analyzer` agent with detailed instructions:

```bash
Task tool with subagent_type: "autonomy:branch-analyzer"
Model: haiku
Prompt: "Analyze autonomy branch '$branch_name' and provide comprehensive status report.

Tasks:
1. Read all journal commits on branch (commits starting with 'journal: ')
2. Parse each commit message for:
   - Iteration number
   - Date
   - Status (active/blocked/concluded/dead-end)
   - Metrics
   - Blockers
   - Next steps
3. Generate Python script to analyze:
   - Complete iteration timeline (chronological)
   - Status changes over time
   - Metrics progression (if metrics exist)
   - Blocker history
   - Current state from most recent commit
4. Execute Python script
5. Output comprehensive markdown report

Use computational methods (Python scripts), do not eyeball the analysis.

Report format:
- Iteration Timeline section
- Metrics Over Time section (if metrics exist)
- Status Evolution section
- Blocker History section
- Current State and Recommendations section"
```

**Agent will:**
1. List all commits on branch: `git log autonomy/<branch-name>`
2. Filter for journal commits (start with "journal: ")
3. Parse each commit message metadata
4. Generate Python script for analysis
5. Execute script to produce timeline, metrics, blockers
6. Return formatted markdown report

### Step 3: Present Report

Display agent's comprehensive report to user.

**Example output format:**
```markdown
# Branch Status: autonomy/experiment-a

**Current Status:** blocked
**Latest Iteration:** 0028
**Last Updated:** 2026-01-02
**Total Iterations:** 28

---

## Iteration Timeline

| Iteration | Date | Status | Summary |
|-----------|------|--------|---------|
| 0001 | 2025-11-15 | active | Initial setup of usage-based pricing model |
| 0002 | 2025-11-16 | active | Implemented tier calculations |
| ... | ... | ... | ... |
| 0027 | 2026-01-01 | active | Stripe API integration progress |
| 0028 | 2026-01-02 | blocked | Awaiting Stripe webhook documentation |

---

## Metrics Over Time

MRR progression:
- Iteration 0001: $45k (baseline)
- Iteration 0010: $52k (+15.6%)
- Iteration 0020: $58k (+28.9%)
- Iteration 0028: $62k (+37.8%)

Build time:
- Iteration 0015: 5.2min (baseline)
- Iteration 0028: 3.2min (-38.5%)

---

## Status Evolution

- Iterations 0001-0027: active (normal progression)
- Iteration 0028: blocked (current)

---

## Blocker History

**Current Blockers (Iteration 0028):**
- Stripe webhook integration unclear: need updated API docs
- Finance team approval pending for pricing structure

**Resolved Blockers:**
- Iteration 0015: Build performance (resolved at 0016)
- Iteration 0022: User feedback collection (resolved at 0024)

---

## Current State and Recommendations

**Where we are:**
Branch has made substantial progress over 28 iterations. MRR increased 37.8%, build time reduced 38.5%. Currently blocked on external dependencies.

**Recommended actions:**
1. Escalate Stripe API documentation request
2. Schedule finance team review meeting
3. Consider parallel work on pricing page UI while blocked
4. Review iteration 0027 for alternative integration approaches

**Branch health:** Active exploration, currently blocked but making good progress
```

## Important Notes

### Only Autonomy Branches

This skill ONLY analyzes `autonomy/*` branches:
- Validates branch has `autonomy/` prefix
- Will not analyze non-autonomy branches
- For general iteration review, user should use `/review-progress`

### Computational Analysis Required

**DO NOT:**
- Manually read through commits
- "Eyeball" metrics progression
- Guess at patterns or trends

**DO:**
- Dispatch branch-analyzer agent
- Let agent generate Python scripts
- Use computational methods for precision

### Read-Only Operations

All analysis happens via git commands:
- Never checkout the branch
- Read commits via `git log <branch-name>`
- Branch-analyzer uses read-only operations
- No modifications to any files

### Metrics May Not Exist

Not all goals have quantitative metrics:
- Some goals are qualitative
- Metrics section may be "None" in commit messages
- Report should handle missing metrics gracefully
- Don't force metrics where they don't exist

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| "I'll read the journal files to get status" | NO. Read commit messages via git log. Don't checkout branch. |
| "Only 10 iterations, I can review manually" | NO. Always dispatch branch-analyzer for computational analysis. |
| "Branch is on remote, I can't analyze it" | YES YOU CAN. Use git log origin/branch-name to read commits. |
| "No metrics in some commits, report is incomplete" | OK. Not all iterations have metrics. Report what exists. |
| "I'll checkout branch to read latest journal" | NO. Read commit message via git log. Never checkout. |

## After Analyzing

Once analysis is complete:
- Report displayed to user
- No files created or modified
- User can fork from any iteration: `/fork-iteration <iteration> <strategy-name>`
- User can compare with another branch: `/compare-branches <branch-a> <branch-b>`
