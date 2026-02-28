---
name: analyzing-branches
description: Use when analyzing another branch's iteration journals to extract findings, decisions, and insights from divergent work
---

# Analyzing Branches

## Overview

Analyze another branch's iteration journals to extract key findings, decisions, and insights from work that diverged from the current branch.

**Core principle:** Learn from parallel explorations. Extract valuable insights from any branch, regardless of success or failure.

## When to Use

Use this skill when:
- User runs `/analyze-branch` command
- Want to review work from parallel experiment branch
- Need to extract lessons from concluded experiments
- Comparing approaches across different branches
- Looking for salvageable ideas from any branch

**DO NOT use for:**
- Analyzing current branch history (use `/review-progress` instead)
- Comparing file changes (use `git diff` directly)
- When branches have no autonomy iterations

## Quick Reference

| Step | Action | Tool |
|------|--------|------|
| 1. Parse arguments | Extract branch name and search criteria | Manual |
| 2. Find divergence | Use git merge-base to find common ancestor | Bash |
| 3. Extract iterations | Get autonomy tags from divergent commits | Bash |
| 4. Read journals | Read iteration files from target branch | Bash, Read |
| 5. Search and filter | Find relevant content based on criteria | Manual |
| 6. Generate report | Create markdown summary | Direct output |

## Process

### Step 1: Parse Arguments

Extract branch name and search criteria from command arguments:

```
Arguments format: "[branch-name] [search-description]"
Example: "experiment-a pricing experiments and revenue optimization"
```

**Parse:**
- First argument: Branch name (e.g., "experiment-a")
- Remaining arguments: Search criteria as free text

**Verify goal exists:**
```bash
# Use Glob to find goal
pattern: "autonomy/*/goal.md"
```

If no goal found:
```
"No autonomy goal found in this project. Branch analysis requires an autonomy goal."
```
Stop here.

Extract goal name from path for use in later steps.

### Step 2: Find Divergence Point

Use git to find where branches diverged:

```bash
# Get current branch name
current_branch=$(git branch --show-current)

# Find common ancestor between current and target branch
merge_base=$(git merge-base "$current_branch" "$target_branch" 2>&1)

# Check if command succeeded
if [ $? -ne 0 ]; then
  # Error handling - see Step 2a
fi
```

**Step 2a: Handle Git Errors**

If `git merge-base` fails, determine cause and prompt user:

**Error: Branch not found**
```markdown
Error: Branch '$target_branch' does not exist.

Available branches:
$(git branch -a | sed 's/^..//; s/ -> .*//')

Please verify the branch name and try again.
```

**Error: No common ancestor**
```markdown
Warning: Cannot find common ancestor between current branch and '$target_branch'.

This may mean:
- Branches have completely independent histories
- Branch was rebased and history was rewritten

Options:
1. Analyze entire branch history (may include duplicate work)
2. Specify iteration range manually
3. Specify common ancestor commit manually

Which would you like?
```

Use AskUserQuestion to let user choose:
- Option 1: Set `merge_base=""` and analyze all iterations
- Option 2: Prompt for iteration range, skip git operations
- Option 3: Prompt for commit hash, use as merge_base

### Step 3: Extract Divergent Iterations

Find autonomy iteration tags on target branch after divergence:

```bash
# Get all autonomy iteration tags on target branch
if [ -n "$merge_base" ]; then
  # Analyze only divergent work
  commit_range="$merge_base..$target_branch"
else
  # Analyze entire branch (fallback)
  commit_range="$target_branch"
fi

# Extract iteration numbers from tags
iterations=$(git log "$commit_range" \
  --pretty=format:"%D" \
  | tr ',' '\n' \
  | grep "tag: autonomy/iteration-" \
  | sed 's/.*autonomy\/iteration-//' \
  | sort -n)

# Count iterations found
iteration_count=$(echo "$iterations" | wc -w)
```

**If no iterations found:**
```markdown
No autonomy iteration tags found on branch '$target_branch' after divergence.

Possible reasons:
- Branch hasn't used autonomy plugin
- All iterations are before the divergence point
- Iterations exist but weren't committed with git integration

Would you like to:
1. Analyze entire branch (all iterations)
2. Specify iteration range manually
```

Use AskUserQuestion for recovery.

### Step 4: Read Journal Files

For each iteration found, read the journal content:

```bash
# For each iteration number
for iter in $iterations; do
  # Find journal file with this iteration number
  # Format: iteration-NNNN-YYYY-MM-DD.md
  journal_file=$(git ls-tree -r --name-only "$target_branch" \
    "autonomy/$goal_name/" \
    | grep "iteration-$(printf '%04d' $iter)-")

  if [ -n "$journal_file" ]; then
    # Read file content from target branch
    journal_content=$(git show "$target_branch:$journal_file")

    # Store for analysis in next step
  fi
done
```

**Build iteration data structure:**
For each iteration, extract:
- Iteration number
- Date (from filename)
- Beginning State section
- Iteration Intention section
- Work Performed subsections
- Ending State section
- Full content for searching

### Step 5: Search and Filter

Apply search criteria to find relevant iterations and content:

**Search strategy:**
```bash
# Convert search criteria to grep pattern
# Example: "pricing experiments" → "pricing|experiments"
search_pattern=$(echo "$search_criteria" | tr ' ' '|')

# For each iteration's content
# Score by relevance (number of search term matches)
# Prioritize sections: Key Decisions, Ending State, Reasoning & Strategy
```

**Extract relevant sections:**
- **Key Findings:** Extract from Ending State + Work Performed
- **Decisions:** Extract from Key Decisions Made subsection
- **Ideas/Experiments:** Extract from Reasoning & Strategy Changes
- **Outcomes:** Extract from Ending State

**If search finds nothing:**
```markdown
No iterations matched search criteria: "$search_criteria"

Analyzed $iteration_count iterations on '$target_branch'.

Would you like to:
1. Broaden search (show all iterations)
2. Refine search criteria
```

Use AskUserQuestion to offer alternatives.

### Step 6: Generate Report

Produce markdown report formatted for journal inclusion:

```markdown
## Analysis of Branch: $target_branch

**Analyzed range:** Iterations $first_iter-$last_iter (diverged from iteration $divergence_iter)
**Search focus:** $search_criteria
**Common ancestor:** $merge_base
**Branch status:** [Active/Merged/Concluded - from latest iteration if stated]

---

### Key Findings

[For each significant finding extracted from journals:]
- **Iteration NNNN:** [Finding or insight]
  - **Context:** [What problem was being addressed]
  - **Approach taken:** [How it was implemented or explored]
  - **Outcome:** [Results from Ending State]
  - **Reference:** `$target_branch @ iteration-NNNN`

---

### Decisions and Rationale

[For each major decision from Key Decisions Made:]
- **Iteration NNNN:** [Decision]
  - **Rationale:** [Why this choice was made]
  - **Outcome:** [Result - success/failure/inconclusive/ongoing]
  - **Takeaway:** [What this tells us]

---

### Ideas and Experiments

[For each experimental approach from Reasoning & Strategy Changes:]
- **Iteration NNNN:** [What was tried]
  - **Results:** [What was learned]
  - **Status:** [Validated/Invalidated/Needs more testing/Inconclusive]
  - **Applicability:** [Could this apply to current branch?]

---

### Timeline of Branch Activity

| Iteration | Date | Summary | Status |
|-----------|------|---------|--------|
| NNNN | YYYY-MM-DD | [From Iteration Intention or Ending State] | [From Ending State] |
| NNNN | YYYY-MM-DD | [Summary] | [Status] |

---

**Analysis Summary:** [Neutral assessment of what was learned, key patterns, notable outcomes]
```

**Output to user:**
Display the complete report. User can copy relevant sections into their current journal's "External Context Gathered" section.

## Important Notes

### Neutral Framing

**Do NOT assume branch status:**
- Branch may be active, merged, concluded, or abandoned
- Don't label as "failed" unless journal explicitly states that
- Let the journal entries speak for themselves
- Use neutral language: "findings", "outcomes", not "failures"

### Git Safety

**Read-only operations:**
- Never checkout or switch branches
- Use `git show` to read historical files
- Don't modify target branch
- Don't create commits

### Search Flexibility

**Interpreting search criteria:**
- Free-text is intentionally loose
- Don't require exact matches
- Look for semantic relevance
- Prioritize iteration sections mentioned in criteria

**If criteria is very specific:**
- "pricing tier experiments" → Focus on those specific words
- "revenue optimization" → Include related terms (profit, MRR, monetization)

**If criteria is broad:**
- "general findings" → Show all significant content
- "lessons learned" → Focus on Ending State assessments

### Report Quality

**Characteristics of good report:**
- **Self-contained:** Readable without accessing original branch
- **Actionable:** Clear what to do with findings
- **Referenced:** Links to specific iterations for deep dives
- **Concise:** Extract signal, avoid noise
- **Balanced:** Show successes and failures equally

### Performance Considerations

**For branches with many iterations:**
- Reading 50+ iterations may be slow
- Consider limiting to most recent N iterations if search is broad
- User can refine criteria to narrow scope

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| "Branch is abandoned, so it failed" | NO. Don't assume status. Read journals for actual outcomes. |
| "I'll modify the target branch" | NO. Read-only operations only. Never checkout or modify target. |
| "Search found nothing, report empty" | NO. Offer to broaden search or show all iterations. |
| "I'll analyze current branch" | NO. Use /review-progress for current branch analysis. |
| "No common ancestor means I should fail" | NO. Offer alternatives: analyze all, manual range, manual ancestor. |
| "Report should only include successes" | NO. Balanced view - show what worked AND what didn't. |

## After Analyzing

Once analysis is complete:
- Report displayed to user
- User copies relevant sections into current journal
- No files created or modified by this skill
- Can analyze multiple branches in same conversation
- Skill usage logged in journal's "Skills & Workflows Used" section
