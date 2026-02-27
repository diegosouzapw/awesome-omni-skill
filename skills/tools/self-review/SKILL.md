---
name: self-review
description: Local code review comparing branches/diffs without GitHub integration. Use when the user wants to review their code, self-review changes, check for bugs before committing, or mentions code review, diff review, or branch comparison.
allowed-tools: AskUserQuestion Bash(git diff:*) Bash(git log:*) Bash(git branch:*) Bash(git blame:*) Bash(git show:*) Bash(git for-each-ref:*) Bash(git rev-parse:*) Bash(git status:*) Bash(git ls-files:*) Bash(git cat-file:*) Bash(git merge-base:*) Bash(grep:*) Bash(head:*) Bash(wc:*) Read Glob Grep Task
---

# Self-Review

Perform local code review of git changes using parallel agents with high-reasoning verification.

## Quick Start

When invoked, follow these steps in order.

## Step 1: Gather Context

Get current branch and recent branches for comparison:

```bash
git branch --show-current
git for-each-ref --sort=-committerdate --format='%(refname:short)' refs/heads/ | grep -v "$(git branch --show-current)" | head -3
```

## Step 1.5: Model Configuration

Check if running on a Claude model (Opus or Sonnet). You know your own model - if you are `claude-opus-*` or `claude-sonnet-*`, skip to Step 2 and use the defaults (haiku/sonnet/opus).

If you are NOT a Claude model, ask 3 questions to configure the model tiers:

### Question 1: Low-Reasoning Model

```
AskUserQuestion:
- question: "Which model for low-reasoning tasks (CLAUDE.md/AGENTS.md discovery, summaries, scoring)?"
- header: "Low Tier"
- multiSelect: false
- options:
  - label: "haiku"
    description: "Claude Haiku"
  - label: "gpt-5.1-codex-mini"
    description: "GPT 5.1 Codex Mini"
  - label: "gpt-5.2-codex"
    description: "GPT 5.2 Codex"
```

### Question 2: Medium-Reasoning Model

```
AskUserQuestion:
- question: "Which model for medium-reasoning tasks (parallel review agents)?"
- header: "Medium Tier"
- multiSelect: false
- options:
  - label: "sonnet"
    description: "Claude Sonnet 4.5"
  - label: "gpt-5.2-codex"
    description: "GPT 5.2 Codex"
  - label: "gemini-3-pro-preview"
    description: "Gemini 3 Pro Preview"
```

### Question 3: High-Reasoning Model

```
AskUserQuestion:
- question: "Which model for high-reasoning tasks (final verification pass)?"
- header: "High Tier"
- multiSelect: false
- options:
  - label: "opus"
    description: "Claude Opus 4.5"
  - label: "gemini-3-pro-preview"
    description: "Gemini 3 Pro Preview"
  - label: "gpt-5.2-codex"
    description: "GPT 5.2 Codex"
```

Store the responses as:
- `<low-reasoning-model>` (default: haiku)
- `<medium-reasoning-model>` (default: sonnet)
- `<high-reasoning-model>` (default: opus)

## Step 2: Ask User Questions

### Question 1: Review Scope

```
AskUserQuestion:
- question: "What would you like to review?"
- header: "Scope"
- multiSelect: false
- options:
  - label: "Both"
    description: "Review branch commits + working changes (Recommended)"
  - label: "Branch commits"
    description: "Review commits on current branch vs base branch"
  - label: "Working changes"
    description: "Review staged and unstaged changes only"
```

### Question 2: Base Branch

```
AskUserQuestion:
- question: "Which base branch should we compare against?"
- header: "Base Branch"
- multiSelect: false
- options:
  - label: "<recent-branch-1>"
    description: "Recent branch"
  - label: "<recent-branch-2>"
    description: "Recent branch"
  - label: "<recent-branch-3>"
    description: "Recent branch"
  - label: "Other"
    description: "Type a different base branch"
- freeTextOn: "Other"
- freeTextPlaceholder: "e.g. main"
```

## Step 3: Gather the Diff

Based on selections:

### For Branch commits (compare against base branch):

```bash
# Try merge-base with specified branch
MERGE_BASE=$(git merge-base <base-branch> HEAD 2>/dev/null)

# Fallback 1: use base branch's upstream
if [ -z "$MERGE_BASE" ]; then
  BASE_UPSTREAM=$(git rev-parse --abbrev-ref "<base-branch>@{upstream}" 2>/dev/null)
  if [ -n "$BASE_UPSTREAM" ]; then
    MERGE_BASE=$(git merge-base $BASE_UPSTREAM HEAD 2>/dev/null)
  fi
fi

# Fallback 2: direct diff with warning
if [ -z "$MERGE_BASE" ]; then
  echo "Warning: Could not compute merge-base, using direct diff"
  git diff <base-branch>..HEAD
else
  git diff $MERGE_BASE..HEAD
fi
```

### For Working changes (staged + unstaged + untracked):

```bash
# Staged changes
git diff --cached

# Unstaged changes
git diff

# Untracked files (for diff-overlap purposes)
git ls-files -z --others --exclude-standard | while IFS= read -r -d '' file; do
  git diff --no-index /dev/null "$file"
done
```

### For Both:

Combine branch diff (using merge-base) with working changes.

Also get changed files list for reference.

## Step 4: Find CLAUDE.md and AGENTS.md Files

Use a general-purpose agent with `model: "<low-reasoning-model>"` to find relevant CLAUDE.md and AGENTS.md files in the repo root and modified directories. Read their contents.

## Step 5: Summarize Changes

Use a general-purpose agent with `model: "<low-reasoning-model>"` to create a 2-3 sentence summary of the changes.

## Step 6: Multi-Agent Review

Launch 4 parallel general-purpose agents with `model: "<medium-reasoning-model>"` (single message with 4 Task calls).

### Read-Only Mode (include in ALL agent prompts)

```
CRITICAL: This is a READ-ONLY code review task.

DO NOT:
- Edit, modify, or fix any code
- Use the Edit, Write, or MultiEdit tools
- Write files to /tmp/ or any other location
- Suggest applying fixes directly
- Attempt to "help" by making changes

DO:
- Report findings in the specified output format
- Return all output as markdown text in your response (not as files)
- Describe issues and how to confirm them
- Suggest fixes in the "fix_suggestion" field (text only)

Your job is to REPORT, not REPAIR. Return structured findings as markdown text only.
```

### Bug Qualification Criteria (include in ALL agent prompts)

```
Only flag issues where:
1. It meaningfully impacts accuracy, performance, security, or maintainability
2. The issue is discrete and actionable (not general or compound)
3. It matches the rigor level present in the codebase
4. It was introduced in this diff (not pre-existing)
5. The author would likely fix if aware
6. It doesn't rely on unstated assumptions
7. You can identify provably affected code (no speculation)
8. It's not an intentional change by the author
9. Parameter correctness across the call stack:
   a) Arguments represent the correct entities, not just matching types
   b) If a parameter is added, removed, or ignored, check that callers reflect the same intent
```

### Comment Quality Guidelines (include in ALL agent prompts)

```
When describing an issue:
- Clearly explain WHY it's a problem
- Communicate severity accurately (don't exaggerate)
- Keep explanation to 1 paragraph max
- "Issue" paragraph: NO fenced code blocks, max 3 inline `code` snippets
- Context snippet (separate section): 1-10 lines fenced (as needed)
- Explicitly state conditions where bug arises
- Matter-of-fact tone (no flattery, not accusatory)
- Include "How to confirm" with specific steps
```

### Diff-Overlap Requirement (include in ALL agent prompts)

```
Line ranges must:
- Overlap with the diff (not pre-existing code)
- Be 1-10 lines (pick the tightest subrange that captures the issue)
- For untracked files: overlap is the entire file (via --no-index diff)

For multi-file issues (e.g., missing tests, contract changes):
- Create ONE finding with primary file/line as location
- Primary file MUST still overlap the diff
- List related files in the explanation body
- Use category tag "cross-cutting"
```

### Required Category Tag

Each agent must tag findings with a category from:
`null-pointer | buffer-overflow | integer-overflow | race-condition | resource-leak | logic-error | security | missing-test | type-error | api-misuse | wrong-value | cross-cutting | other`

Note: `wrong-value` is for bugs where a parameter is provided but represents the wrong entity/data (correct type, wrong semantic meaning).

### Agent 1: CLAUDE.md/AGENTS.md Compliance

Audit changes against CLAUDE.md and AGENTS.md guidelines. Only flag issues where guidelines EXPLICITLY mention the rule.

### Agent 2: Bug Detection

Scan for bugs with focus on:
- Null pointers, off-by-one, race conditions, resource leaks, incorrect logic
- **Wrong value passed**: A parameter is provided but represents the wrong entity/data
  (e.g., using X's ID when Y's ID was needed, passing stale data, wrong enum value)

#### Systematic Wrong-Value Detection Procedure

For each function/method call in the diff where arguments are passed:

1. **DETERMINE** what entity each argument SHOULD represent:
   - Read the function signature and parameter names
   - Check function documentation/comments if available
   - Infer from function name (e.g., `checkPermission($userId)` → userId should be the user being checked)

2. **VERIFY** the actual argument matches the expected entity:
   - What variable is being passed?
   - What does that variable represent in the CALLING context?
   - Does the calling context's meaning match the function's expected meaning?

3. **PAY SPECIAL ATTENTION** to:
   - Same-type parameters representing different entities (e.g., sourceId vs targetId, ownerId vs viewerId)
   - Variables passed from different scopes
   - Functions with multiple parameters of the same type
   - Gating/permission functions (the "who" being checked matters critically)

### Agent 3: History Analysis

Use `git blame` and `git log` to identify issues informed by history (reverted fixes, broken patterns).

### Agent 4: Code Comments Compliance

Check that changes comply with inline code comments (TODOs, warnings, documentation).

### Agent Output Format

Each agent returns issues in format:
```
- file: <path>
- lines: <start>-<end> (must be 1-10 lines, must overlap diff)
- category: <category tag>
- title: <short title>
- issue: <1 paragraph, no fenced code, max 3 inline code snippets>
- how_to_confirm: <specific steps>
- code_snippet: <1-10 lines from the file at the reported line range>
```

## Step 6.5: Dedupe Findings

After multi-agent review, before scoring.

### Dedupe Criteria (ALL must match):

- Same file
- Overlapping line ranges
- Same category tag

DO NOT merge based on title similarity alone.

### Merge Rules:

- Keep tightest subrange for file location (MUST be ≤10 lines even if union would be larger)
- Mention other relevant lines in explanation body
- Keep most detailed description
- Preserve all unique context

## Step 7: Confidence Scoring (Ranking Only)

Use general-purpose agents with `model: "<low-reasoning-model>"` to score each candidate. This is ranking only - NO filtering at this step.

For each candidate:
- Assign confidence 0-100 (likelihood it's real)
- Do NOT filter any candidates
- Pass full list to verifier

Confidence scale:
- 0-25: Likely false positive
- 26-50: Uncertain
- 51-75: Probably real
- 76-100: Almost certainly real

## Step 8: High-Reasoning Verification

Launch a general-purpose agent with `model: "<high-reasoning-model>"` that receives:
- Full diff with hunks (from Step 3) - not just filenames, the actual diff content
- CLAUDE.md/AGENTS.md files (from Step 4)
- Ranked candidate list with confidence scores (from Step 7)

### Critical: Independent Diff Scan

The verifier must independently scan the entire diff, not just validate candidates. The candidate list is a starting point, not the universe of issues.

### Large Diff Handling

If diff exceeds context limits (~100k tokens), chunk by file:
- Prioritize files that had candidate findings
- Include "highest-risk" files (security-related, core logic) even without candidates
- **Required:** If any files were skipped, output must include a list of skipped files

### Verifier Prompt Instructions

```
You are performing a final verification pass on code review findings.

CRITICAL: This is a READ-ONLY review task. Do NOT edit, modify, or fix any code.
Do NOT use Edit, Write, or MultiEdit tools. Report findings only - do not repair.

Bug Qualification Criteria:
1. It meaningfully impacts accuracy, performance, security, or maintainability
2. The issue is discrete and actionable (not general or compound)
3. It matches the rigor level present in the codebase
4. It was introduced in this diff (not pre-existing)
5. The author would likely fix if aware
6. It doesn't rely on unstated assumptions
7. You can identify provably affected code (no speculation)
8. It's not an intentional change by the author
9. Parameter correctness across the call stack:
   a) Arguments represent the correct entities, not just matching types
   b) If a parameter is added, removed, or ignored, check that callers reflect the same intent

Comment Quality Guidelines:
- Clearly explain WHY it's a problem
- Communicate severity accurately (don't exaggerate)
- Keep explanation to 1 paragraph max
- "Issue" paragraph: NO fenced code blocks, max 3 inline `code` snippets
- Context snippet (separate section): 1-10 lines fenced (as needed)
- Explicitly state conditions where bug arises
- Matter-of-fact tone (no flattery, not accusatory)
- Include "How to confirm" with specific steps

TASK 1: Review all candidates, marking each verified: true/false/unsure
- true: Confirms this is a real bug per the 9 criteria
- false: Rejects as false positive or pre-existing; set confidence: 0
- unsure: Can't conclusively verify; may adjust confidence up/down

TASK 2: Set confidence for ALL findings (0 for rejected, assessed value for others)

TASK 3: Independently scan the full diff for issues not in the candidate list
- New findings may REPLACE lower-quality candidates if they cover the same issue better
- All new findings must also have verified status
- Apply systematic wrong-value detection (see below)

TASK 4: Assign priority based on IMPACT (not confidence)
- P0: Drop everything. Blocking. Universal, assumption-free.
- P1: Urgent. Address in next cycle.
- P2: Normal. Fix eventually.
- P3: Low. Nice to have.

TASK 5: Assign required category tag for each finding

Output Format for each finding:
- file: <path>
- lines: <start>-<end>
- category: <category tag>
- title: <short title>
- priority: P0/P1/P2/P3
- confidence: 0-100
- verified: true/false/unsure
- issue: <1 paragraph explanation>
- how_to_confirm: <specific steps>
- fix_suggestion: <if applicable>
- code_snippet: <1-10 lines from file at reported line range>

If any files were skipped due to size:
- skipped_files: [list of skipped file paths]

Systematic Wrong-Value Detection:
For each function call in the diff where arguments are passed:
1. DETERMINE what each argument SHOULD represent (from signature, docs, function name)
2. VERIFY the actual argument matches: what does the variable mean in the CALLING context?
3. FLAG when calling context meaning differs from function's expected meaning
4. Priority attention: gating/permission functions, same-type multi-parameter functions
```

## Step 9: Filter and Output

### Filter Rules

After verification review:
- **verified: true** (any confidence): Show in Verified Findings, contribute to verdict
- **verified: unsure** + confidence ≥50: Show in Tentative Findings (doesn't affect verdict)
- **verified: unsure** + confidence <50: Filter out
- **verified: false**: Filter out (rejected by verifier)

### Output Ordering

- Verified findings: Sort by decreasing severity (P0 first, then P1, P2, P3), then by confidence (high→low)
- Tentative findings: Sort by confidence (high→low)

### Output Format

```markdown
## Self-Review Results

**Comparing:** [current-branch] vs [base-branch]
**Scope:** [scope]
**Files changed:** [count]

---

## Verified Findings

### 1. [P0] Issue title (confidence: 85%)
**File:** path/to/file.ext:10-15
**Category:** logic-error
**Context:**
\`\`\`language
[1-10 line snippet from the file at the reported line range]
\`\`\`
**Issue:** [1 paragraph, code refs ≤3 lines, no fenced code blocks]
**How to confirm:** [specific steps]
**Fix suggestion:** [if applicable]

### 2. [P1] Another issue (confidence: 75%)
...

---

**Verdict:** Patch is [correct/incorrect]
**Explanation:** [1-2 sentence justification]

---

## Tentative Findings (verifier unsure, confidence ≥50%)

### 1. [P2] Uncertain issue (confidence: 55%)
**File:** path/to/file.ext:20-25
**Category:** null-pointer
**Context:**
\`\`\`language
[snippet]
\`\`\`
**Issue:** [explanation]
**How to confirm:** [steps]
**Fix suggestion:** [if applicable]
```

### Verdict Rules

- "incorrect" if ANY verified bug exists (any priority level)
- If files were skipped due to size:
  - Verdict: "correct (with skipped files)" if no verified bugs
  - Explanation MUST list which files weren't reviewed
- Tentative Findings section still appears even if Verified Findings is empty

### No-Findings Output

```markdown
## Self-Review Results

**Comparing:** [current-branch] vs [base-branch]
**Scope:** [scope]
**Files changed:** [count]

---

## Verified Findings

No verified findings.

---

**Verdict:** Patch is correct
**Explanation:** No issues met verification criteria.

---

## Tentative Findings (verifier unsure, confidence ≥50%)

No tentative findings.
```

### Skipped Files Output (when applicable)

```markdown
**Note:** The following files were skipped due to diff size limits:
- [file1]
- [file2]
Consider running `/self-review` again with these files specifically.
```
