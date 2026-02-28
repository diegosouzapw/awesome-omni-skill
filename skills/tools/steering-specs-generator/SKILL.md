---
name: steering-specs-generator
description: Extract tacit engineering knowledge through guided interviews and generate structured steerings. Use when user mentions "steerings", "tacit knowledge", "conventions", "engineering practices", "interview", or wants to document team/project knowledge. Also activates when user asks for "steerings for X", "document X conventions", "continue steerings", "resume interview", or wants to extract knowledge about a specific topic. Supports reviewing and transforming existing steerings to standard format. Auto-detects existing sessions and offers to continue incomplete ones.
allowed-tools: Read, Write, AskUserQuestion, Glob, Task
---

# Steering Specs Generator

Conducts context-aware interviews to extract tacit engineering knowledge and generate agent-readable steerings. Format: **Intent (Why) → Rules (What) → Practices (How) → Meta**.

Supports **predefined packs** (8 areas) and **custom topics** (user-specified).

**Flow overview:** See [flow-diagram.md](flow-diagram.md) for visual representation.

## Prerequisites

- [pack-reference.md](pack-reference.md) — Topic areas and questions
- [steering-template.md](steering-template.md) — Output format
- Access to AskUserQuestion and Task tools

## Mode Selection

| Keywords | Mode |
|----------|------|
| "review steerings", "transform steerings", "fix format" | → Review Mode (Step R) |
| "continue steerings", "continue session", "resume interview" | → Interview Mode (Step 0) with session check |
| "steerings", "tacit knowledge", "interview", "conventions" | → Interview Mode (Step 0) |

---

## Interview Flow

### Step 0: Check for Existing Sessions

Before configuring paths, check if sessions already exist in the repo:

1. **Scan common session directories:**
   - `.sessions/`
   - `sessions/`
   - `docs/sessions/`

2. **If sessions found**, present AskUserQuestion:
```yaml
questions:
  - question: "Found existing session(s). Continue or start fresh?"
    header: "Session"
    options:
      - label: "Continue {sessionId}"
        description: "Resume incomplete session ({N} of {M} packs done)"
      - label: "Start new session"
        description: "Create fresh session with new ID"
```

3. **If continuing existing session:**
   - Set `sessionsPath` to parent directory of found session
   - Set `sessionId` to selected session name
   - Scan `{sessionsPath}{sessionId}/` for completed pack files
   - **Completed pack detection:** A pack is complete if:
     - File `{packId}.md` exists AND
     - Contains `## Interview` section with at least one `### Q` entry
   - Store `completedPacks[]` list (pack IDs to skip)
   - Read `explore-docs-conventions.md` and `explore-repo-context.md` paths if they exist
   - Skip to Step 3 (Pack Interview Loop), filtering out completed packs

4. **If starting new or no sessions found** → Continue to Step 0a

### Step 0a: Configure Output Paths

Ask user to confirm paths using AskUserQuestion:

```yaml
questions:
  - question: "Where should steering files be saved?"
    header: "Steerings"
    options: ["./steerings/", "docs/steerings/", ".memory-bank/steerings/", "Custom"]
  - question: "Where should session files be saved?"
    header: "Sessions"
    options: ["./sessions/", ".sessions/", "docs/sessions/", "Custom"]
  - question: "Where should action items be saved?"
    header: "Backlog"
    options: ["./backlog/", "Same as steerings parent", ".backlog/", "Custom"]
```


**Store:** `steeringsPath`, `sessionsPath`, `backlogPath`. Create directories if needed.

**Defaults:** `steerings/`, `sessions/`, `backlog/`

### Step 0b: Generate `sessionId` - short words id

### Step 1: Define Topics

**Custom topic detected** (patterns: "steerings for X", "document X conventions"):
- If clear → Generate `packId`, `packName`, `packType: "custom"`, `customTopicDescription`
- If broad → Clarify with AskUserQuestion (aspects, level, scope)

**No custom topic** → Present 8 predefined packs as multi-select:

### Step 1a: Choose Interview Mode

Ask user to select interview mode:

```yaml
questions:
  - question: "Interview mode preference?"
    header: "Mode"
    options:
      - label: "Interactive (default)"
        description: "Answer questions one-by-one with discussion"
      - label: "Fast mode"
        description: "Answer all questions at once, faster execution"
```

**Store:** `interviewMode` ("interactive" or "fast")

| Pack | ID |
|------|----|
| Codebase Topology & Ownership | `codebase-topology-ownership` |
| Architecture & Design Invariants | `architecture-design-invariants` |
| Business Domain Contracts | `business-domain-contracts` |
| Quality & Style Assurance | `quality-style-assurance` |
| Testing & Verification Strategy | `testing-verification-strategy` |
| Risk & Historical Landmines | `risk-historical-landmines` |
| Security, Data & Compliance | `security-data-compliance` |
| Delivery Lifecycle & Change Flow | `delivery-lifecycle-change-flow` |

### Step 2: Discovery (Parallel Explore)

Run TWO Explore agents in parallel. Each writes report to `{sessionsPath}{sessionId}` and returns path.

**Explore #1 - Docs & Conventions:**
```
Analyze repository for: steering files, CONVENTIONS.md, ARCHITECTURE.md,
CLAUDE.md, README conventions, eslint/prettier/tsconfig.

OUTPUT: Write to `{sessionsPath}{sessionId}/explore-docs-conventions.md`, return path.
```

**Explore #2 - Repo Context:**
```
Analyze: project purpose, tech stack, directory structure, main modules, patterns.

OUTPUT: Write to `{sessionsPath}{sessionId}/explore-repo-context.md`, return path.
```

**Capture paths:** `docsConventionsReportPath`, `repoContextReportPath`

### Step 3: Pack Interview Loop

**Filter packs:** If `completedPacks[]` exists (from session continuation), exclude those pack IDs from processing.

**Show progress when continuing:**
```
Continuing session: {sessionId}
✅ Completed: {completedPacks.join(', ')}
⏳ Remaining: {remainingPacks.join(', ')}
```

**When running packs in parallel:**
- Launch all pack agents as background tasks
- Capture all task IDs
- Wait for all to complete using TaskOutput tool
- Display progress as packs finish

Spawn a **Task agent per pack** - run all in parallel for faster execution.
Use `run_in_background: true` for each Task call, then wait for all to complete.

```yaml
subagent_type: "general-purpose"
description: "Interview for {packName}"
prompt: |
  Conduct interview for a single pack and save results.

  ## Pack Info
  - Pack ID: {packId}
  - Pack Name: {packName}
  - Pack Type: {packType}  # "predefined" or "custom"
  - Custom Description: {customTopicDescription}  # only if custom

  ## Context Files
  - Pack Reference: .claude/skills/steering-specs-generator/pack-reference.md
  - Repo Context: {repoContextReportPath}
  - Docs & Conventions: {docsConventionsReportPath}

  ## Output
  - Path: {sessionsPath}{sessionId}/{packId}.md

  ## Instructions

  ### 1. Read Context
  Read pack-reference.md to get question themes for this pack.

  Read repoContextReportPath and docsConventionsReportPath for grounding.
  These reports contain ALL necessary findings - do NOT run additional explores.

  ### 2. Generate Questions
  Question count:
  - Predefined: 5
  - Custom (narrow): 3-4
  - Custom (medium): 5
  - Custom (broad): 6-7

  Guidelines:
  - Ground ONLY in the provided explore reports + existing docs
  - Reference actual code: "I see X in Y file..." (from reports)
  - Ask about conventions, not roadmap
  - Offer 4 options (A/B/C/D)
  - Mark one as "⭐ Recommended" at end of description

  Pattern:
  Q: I found {finding}. How should {convention question}?
  A) {Option} — {rationale}
  B) {Option} — {rationale} ⭐ Recommended
  C) {Option} — {rationale}
  D) {Option} — {rationale}

  ### 3. Conduct Interview
  If interviewMode is "fast":
    - Present all questions in a single markdown code block
    - User responds with all answers at once (e.g., "A, B, A, C, B")
  If interviewMode is "interactive":
    - Present via AskUserQuestion (max 4 per call)

  ### 4. Classify Responses
  For each response, classify as:
  - CONVENTION: Timeless, future-focused ("When implementing X, do Y")
  - ACTION_ITEM: Temporal, fixes current state ("Replace X", "Fix X")

  ### 5. Extract and Save Results
  Extract rules immediately (not full Q&A) and write to {sessionsPath}{sessionId}/{packId}.md:
  - CONVENTION items → "## Conventions" section
  - ACTION_ITEM items → "## Action Items" section with severity
  - Optionally preserve raw Q&A in collapsed "## Raw Interview" section
```

**Session structure:**
```
{sessionsPath}
└── {sessionId}/
    ├── codebase-topology-ownership.md
    ├── architecture-design-invariants.md
    ├── {custom-pack-id}.md
    └── ...
```

**Pack file format:**
```markdown
# {Pack Name}
**Pack ID:** {id}

## Conventions
- Q1: {extracted rule}
- Q2: {extracted rule}

## Action Items
- Q3: {action item with severity}

## Raw Interview (optional, for reference)
Preserve Q&A in collapsed detail if needed for debugging.
```

### Step 4: Await Pack Interviews

Wait for all pack interview agents to complete. Each writes its results to `{sessionsPath}{sessionId}/{packId}.md` with classifications already included.

**Note:** Pack agents use existing explore reports - no additional explores are run.

### Step 5: Generate Outputs

Delegate to general-purpose subagent, choose Opus model:

```yaml
subagent_type: "general-purpose"
description: "Generate steerings and action items"
prompt: |
  Generate steerings AND action items from interview sessions.

  Session directory: {sessionsPath}{sessionId}/
  (contains one .md file per pack with classifications)

  Docs report: {docsConventionsReportPath}
  Repo report: {repoContextReportPath}
  Template: .claude/skills/steering-specs-generator/steering-template.md

  Output paths: {steeringsPath}, {backlogPath}

  Instructions:
  1. Read all pack files from session directory
  2. Extract CONVENTION items → generate steering files
  3. Extract ACTION_ITEM items → generate backlog file
  4. Generate index.md for steerings
```

**Steering filenames:**

| Pack ID | Filename |
|---------|----------|
| codebase-topology-ownership | code-ownership.md |
| architecture-design-invariants | architecture-invariants.md |
| business-domain-contracts | domain-invariants.md |
| quality-style-assurance | quality-and-style.md |
| testing-verification-strategy | testing-strategy.md |
| risk-historical-landmines | risk-registry.md |
| security-data-compliance | security-and-compliance.md |
| delivery-lifecycle-change-flow | delivery-lifecycle.md |
| {custom-pack-id} | {custom-pack-id}.md |

**Rule style:**
- ✅ "When implementing X, do Y" / "Use X for Y" / "New features require X"
- ❌ "Proactively refactor X" / "Continue using X" / "Add X to existing code"

**Action items file:** `{backlogPath}steering-specs-action-items.md`

Severity: 🔴 CRITICAL (data loss, security) → 🟡 HIGH → 🟢 MEDIUM → 🔵 LOW → ⏸️ DEFERRED

### Step 6: Present Results

```
Generated steerings:
- {steeringsPath}*.md (N rules, M practices each)
- {steeringsPath}index.md

Action Items: {backlogPath}steering-specs-action-items.md
- 🔴 Critical: N | 🟡 High: N | 🟢 Medium: N

Session: {sessionsPath}{sessionId}/
- {packId-1}.md
- {packId-2}.md
- ...
```

---

## Review Mode (Step R)

Transform existing steerings to standard format.

### R1: Locate Steerings

Auto-detect or ask: `steerings/`, `docs/steerings/`, `.steerings/`

### R2: Analyze with Explore

```yaml
subagent_type: "Explore"
prompt: |
  Analyze steering files in {steeringsPath}:
  - Structure: Intent → Rules → Practices → Meta?
  - Rules: numbered, prescriptive, no metadata?
  - Code examples: 5-15 lines?
  - File size: <200 lines?

  Report compliant vs non-compliant files with specific issues.
```

### R3: Present Summary

```
✅ Compliant: file1.md, file2.md
⚠️ Need transformation:
- file3.md: Missing Intent, verbose rules
- file4.md: Code examples too long
```

### R4: Ask Action

Options: Transform all | Transform specific | Backup and transform | Plan only | Skip

### R5: Transform

Delegate to general-purpose subagent with transformation guidelines from steering-template.md.

### R6: Present Results

Show changes per file, regenerate index.md.

---

## Quick Reference

**Activation keywords:** steerings, tacit knowledge, interview, conventions, "steerings for X", "continue steerings"

**Session continuation:** Auto-detects existing sessions in `.sessions/`, `sessions/`, `docs/sessions/`. Offers to resume incomplete sessions, skipping completed packs.

**8 Predefined Packs:** Topology, Architecture, Domain, Quality, Testing, Risk, Security, Delivery

**Output format:** Intent (1 sentence) → Rules (numbered) → Practices (heading + explanation) → Meta

**Files generated:**
- `{steeringsPath}*.md` — Steering files
- `{steeringsPath}index.md` — Table of contents
- `{backlogPath}steering-specs-action-items.md` — Action items
- `{sessionsPath}{sessionId}/{packId}.md` — Interview responses per pack
- `{sessionsPath}explore-*.md` — Discovery reports

**Reference files:**
- [flow-diagram.md](flow-diagram.md) — Visual flow
- [pack-reference.md](pack-reference.md) — Pack definitions and Explore prompts
- [steering-template.md](steering-template.md) — Output format validation
