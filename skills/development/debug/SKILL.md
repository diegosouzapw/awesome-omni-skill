---
name: debug
description: Guide systematic debugging through hypothesis generation, evidence collection, and verification. Uses scientific method for root cause analysis.
license: MIT
compatibility:
  - runtime:any
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(git:log)
  - Bash(git:diff)
  - Bash(git:blame)
  - Bash(git:show)
metadata:
  author: thoreinstein
  version: 1.0.0
---

# Debug

Systematic debugging through hypothesis generation, evidence collection, and verification.

## When to Use This Skill

- When facing a bug with unclear root cause
- When initial debugging attempts haven't resolved the issue
- When you need structured investigation rather than guesswork
- When documenting a debugging session for future reference

## Input

Provide as much of the following as available:

1. **Problem Statement**
   - What's the expected behavior?
   - What's the actual behavior?
   - Steps to reproduce

2. **Context**
   - Recent changes that might be related
   - Error messages or logs
   - Environment details (OS, versions, etc.)

3. **Initial Hypotheses** (if any)
   - What do you think might be causing it?
   - What have you already ruled out?

## Debugging Framework

### Step 1: Problem Definition

Clearly define:
- Expected vs actual behavior
- Reproduction steps (minimal if possible)
- Scope: when does it happen, when doesn't it?
- Impact: severity and affected users/systems

### Step 2: Hypothesis Generation

Generate ranked hypotheses:
- List possible causes
- Assign probability (likely, possible, unlikely)
- Note evidence for and against each
- Identify what would confirm or rule out each

### Step 3: Investigation Plan

For each hypothesis, plan:
- What evidence to look for
- Where to look (files, logs, git history)
- What tests would confirm/deny
- Order of investigation (highest probability first)

### Step 4: Evidence Collection

Investigate systematically:
- Search codebase for relevant patterns
- Review git history for recent changes
- Examine logs and error messages
- Document findings with timestamps

### Step 5: Root Cause Identification

Synthesize findings:
- Identify the root cause (not just symptoms)
- Note contributing factors
- Document code locations involved
- Explain the causal chain

## Investigation Tracks

Run these investigation tracks in parallel where possible:

| Track        | Focus                                        |
| ------------ | -------------------------------------------- |
| **Codebase** | Explore structure, find related code         |
| **History**  | Recent changes, blame, related commits       |
| **Patterns** | Search for similar code, error handling      |
| **External** | Documentation, known issues, similar reports |

## Output

Produce a debug session report following the template in `references/debug-session-template.md`.

The report should include:
- Problem statement with reproduction steps
- Hypotheses with evidence assessment
- Investigation log with timestamps
- Root cause and contributing factors
- Recommended fix and prevention measures

## Constraints

- **Hypotheses first** - generate hypotheses before diving into code
- **Evidence-based** - support conclusions with specific evidence
- **Document as you go** - keep an investigation log
- **Root cause, not symptoms** - dig until you find the actual cause
- **Prevention** - recommend how to prevent recurrence

## Example

### Example: Intermittent API Timeout

**Problem:** `/api/users` endpoint times out ~10% of requests

**Hypotheses:**
```
H1: Database query is slow (70% likely)
    Evidence for: Timeout correlates with high DB load
    Evidence against: None yet
    Test: Check query execution time

H2: Connection pool exhaustion (20% likely)
    Evidence for: Happens under load
    Evidence against: Pool metrics look normal
    Test: Monitor pool during timeout

H3: External service dependency (10% likely)
    Evidence for: None
    Evidence against: No external calls in this endpoint
    Test: Trace request path
```

**Investigation:**
```
10:15 - Checked slow query log → found users query taking 2-5s
10:22 - git blame on users.go → query changed 3 days ago
10:28 - Compared old vs new query → missing index on new column
10:35 - Confirmed: new filter column not indexed
```

**Root Cause:** Query added filter on `last_login` column which lacks an index. Under load, full table scan causes timeouts.

**Fix:** Add index on `users.last_login`

**Prevention:** Add query plan review to PR checklist for database changes.

---

Begin by clearly defining the problem and generating hypotheses before investigating.
