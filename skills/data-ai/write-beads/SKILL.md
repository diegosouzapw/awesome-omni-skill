---
name: write-beads
description: Convert a Technical Design Document into a BEADS.md file for agent implementation
---

# Write Beads

Convert a Technical Design Document (TDD) into a BEADS.md markdown file containing atomic implementation beads.

**This skill outputs a markdown file only.** To create actual beads in bd, use `/create-beads` after review.

## When to use

Use when you have:
- A polished Technical Design Document from Stage 2
- Need to break down work for autonomous coding agents
- Want to create trackable, independently-executable tasks

## Role

You are a task decomposition engine preparing work for autonomous coding agents.

## Bead definition

A bead is the smallest unit of work that:
- Has a single clear objective (one responsibility)
- Is self-contained (no missing context)
- Has explicit inputs/outputs
- Has testable completion criteria
- Can be completed in one focused session (target < 30 minutes)

## Rules

- Prefer *more beads* over fewer
- **DECISION gating**: If a bead depends on a decision that is not yet resolved, create a separate **DECISION** bead that BLOCKS the dependent beads. Unresolved decisions MUST NOT be left implicit.
- Use dependencies to express ordering; do not assume strict sequence
- Assume agents will not infer context beyond what is written
- Include parallelism and confidence metadata where applicable

## Context Verification Rules

Before writing a bead's Context and Specification sections, verify the following. If you cannot verify, mark with UNVERIFIED.

1. **File paths**: Every file in the Files section must be a real path you have confirmed exists (for Modify/Reference) or a path in an existing directory (for Create). Do not write "find X in src/" — write the exact path.
2. **Import paths**: If a bead says "import X from Y", verify that Y actually exports X. If unverifiable, mark UNVERIFIED.
3. **Function references**: If a bead references a function from another module, confirm it exists and note its signature. Do not use speculative language ("may reuse", "might need").
4. **Line numbers**: When referencing specific locations in existing files, include line numbers or anchoring context (function name, class name).
5. **No investigation deferred to implementation**: Phrases like "verify if file exists", "check which export to use", "find the right component" are NOT acceptable. The planning phase must resolve these.

## Output Format

Output a SINGLE markdown file (`BEADS.md`) that can be directly imported into bd:

```bash
bd create -f docs/plans/<project>/BEADS.md
```

### File Structure

The output file MUST have these sections in order:

1. **Header** - Project name and link to TDD
2. **Dependency Graph** - Mermaid diagram showing relationships
3. **Summary Tables** - Beads grouped by priority with dependencies
4. **Dependency Commands** - Ready-to-run `bd dep add` commands
5. **Full Bead Definitions** - One `## ` section per bead

### Bead Format (bd-compatible)

Each bead MUST use an H2 title plus H3 metadata sections that `bd create -f` can reliably ingest.

```markdown
## {ID}: {Title}

### Priority
P0|P1|P2

### Type
task|bug|feature|decision

### Labels
{project-label}, {phase-label}

### Blocked by
{ID}, {ID} (optional; informational only)

### Parallelism
parallelizable|requires-sequence|decision-gated
(Optional. Hints for swarm orchestrator: can this run with other beads?)

### Confidence
verified|assumed|unverified
(Optional. How confident is this specification? Unverified = likely needs iteration)

### Context
Why this exists, what already exists, what this creates/modifies.

### Specification
Exact requirements, inputs, outputs, interface contracts, error handling.

### Files (optional but recommended)
- Create: {filepaths}
- Modify: {filepaths}
- Reference: {filepaths}

### Common Failure Modes (optional)
What implementers commonly get wrong.

### Acceptance Criteria
- [ ] Specific verifiable condition
- [ ] Passes: {test command}
```

**CRITICAL FORMAT RULES**:
1. Do NOT use `## ` (H2) anywhere except bead titles - `bd create -f` treats EVERY H2 as a bead
2. Do NOT add YAML frontmatter (`---` block at top) - the plan validator rejects it
3. Do NOT rely on `**Blocked by:**` for bd dependencies (bd cannot resolve symbolic deps during batch creation)
4. Dependencies are added AFTER creation using a runnable `bd dep add` script
5. Do NOT use `bd update --blocked-by` (unsupported by bd CLI)

**COMMON MISTAKES TO AVOID**:
```markdown
# WRONG - Will create unwanted beads and fail validation:
---
schemaVersion: 1
artifactType: beads
---

# Project Name

## Summary           <-- WRONG: H2 creates a bead named "Summary"
## Dependency Graph  <-- WRONG: H2 creates a bead named "Dependency Graph"

## CORE-01: Real bead
...

# CORRECT - Only beads use H2:
# Project Name

**Summary:** Brief description here.

### Dependency Graph   <-- H3 is safe, not parsed as bead
\`\`\`mermaid
...
\`\`\`

---

## CORE-01: Real bead  <-- Only beads get H2
...
```

### Dependency Script Section

After the summary tables, include a **complete executable script** that adds all dependencies.

bd cannot resolve symbolic dependencies during batch creation, so dependencies MUST be added after beads are created.

```markdown
## Add Dependencies

After creating beads, run this script to add all dependencies:

\`\`\`bash
#!/bin/bash
# Auto-generated dependency script for {project-name}
# Usage: bd create -f BEADS.md && bash add-deps.sh

set -e

# Get bead ID mapping
echo "Fetching bead IDs..."
declare -A IDS
while IFS=': ' read -r key value; do
  IDS[$key]=$value
done < <(bd list -l {label} --json | jq -r '.[] | "\(.title | split(":")[0]): \(.id)"')

# Add dependencies (child depends on parent)
bd dep add ${IDS[CHILD-01]} ${IDS[PARENT-01]}
bd dep add ${IDS[CHILD-02]} ${IDS[PARENT-01]}
bd dep add ${IDS[CHILD-02]} ${IDS[PARENT-02]}
# ... continue for all dependencies

echo "Dependencies added. Syncing..."
bd sync
\`\`\`
```

**The script MUST be complete and runnable** - include every dependency from the graph.

## ID Conventions

Use category prefixes:
- `SETUP-NN` - Foundation/configuration
- `SCHEMA-NN` - Database schemas/migrations
- `CORE-NN` - Core logic/utilities
- `JOB-NN` - Background jobs
- `API-NN` - API endpoints
- `UI-NN` - User interface
- `TEST-NN` - Tests
- `INTEG-NN` - Integration work
- `DOCS-NN` - Documentation
- `CLEAN-NN` - Cleanup/refactoring
- `DECISION-NN` - Decisions requiring input (see DECISION Beads section)

## DECISION Beads (Gating Mechanism)

**CRITICAL**: Unresolved decisions MUST NOT be left implicit in implementation beads.

When to create a DECISION bead:
- Architecture choices not yet finalized (e.g., "use Redis vs DynamoDB")
- API design decisions (e.g., "REST vs GraphQL for this endpoint")
- Data model choices that affect multiple beads
- Third-party service selection
- Any "TBD" or "TODO: decide" in the TDD

DECISION bead structure:
```markdown
## DECISION-NN: {Decision Title}

### Priority
P0 (decisions are always high priority - they block work)

### Type
decision

### Labels
{project-label}, decision-gate

### Options
1. **Option A**: Description, pros, cons
2. **Option B**: Description, pros, cons
3. **Option C**: Description, pros, cons

### Recommendation
Recommended option with rationale (if any)

### Decision Needed By
Who/what role needs to make this decision

### Blocks
List of bead IDs that cannot proceed until this is resolved:
- CORE-02
- API-01

### Acceptance Criteria
- [ ] Decision documented in TDD/ADR
- [ ] Dependent beads updated with decision outcome
```

**Gating rule**: Any bead that references an unresolved decision MUST have that DECISION bead as a blocker. Agents CANNOT proceed with implementation until the DECISION bead is closed.

## Priority Guidelines

- **P0**: Blocks critical path, must be done first
- **P1**: Core functionality, MVP requirements
- **P2**: Polish, cleanup, nice-to-have

## Phase Labels

Assign phase labels based on category:
- `phase-0`: SETUP, SCHEMA (foundation)
- `phase-1`: CORE (infrastructure)
- `phase-2`: JOB (background processing)
- `phase-3`: API (endpoints)
- `phase-4`: UI (user interface)
- `phase-5`: TEST (testing)
- `phase-6`: CLEAN, DOCS (polish)

## Ordering Heuristics

- Foundation/setup before core logic
- Core logic before integrations
- Tests alongside or immediately after the behavior they validate
- Integration/system beads last

## Example Output

```markdown
# Project Name - Implementation Beads

Derived from [TECHNICAL_DESIGN.md](./TECHNICAL_DESIGN.md).

### Dependency Graph

\`\`\`mermaid
graph TD
    SETUP-01 --> DECISION-01
    DECISION-01 --> CORE-01
    CORE-01 --> API-01
    API-01 --> UI-01
\`\`\`

### Summary

| ID | Title | Priority | Dependencies | Parallelism |
|----|-------|----------|--------------|-------------|
| SETUP-01 | Add provider constant | P0 | - | parallelizable |
| DECISION-01 | Choose caching strategy | P0 | SETUP-01 | decision-gated |
| CORE-01 | Implement client | P0 | DECISION-01 | requires-sequence |
| API-01 | Create endpoint | P1 | CORE-01 | parallelizable |
| UI-01 | Build picker UI | P1 | API-01 | parallelizable |

### Dependency Commands

After `bd create -f BEADS.md`, add dependencies:

\`\`\`bash
bd list -l my-project --json | jq -r '.[] | "\(.title | split(":")[0]): \(.id)"' | sort
# Then for each dependency:
# bd dep add <DECISION-01-id> <SETUP-01-id>
# bd dep add <CORE-01-id> <DECISION-01-id>
# bd dep add <API-01-id> <CORE-01-id>
# bd dep add <UI-01-id> <API-01-id>
\`\`\`

---

## SETUP-01: Add provider constant

### Priority
P0

### Type
task

### Labels
my-project, phase-0

### Parallelism
parallelizable

### Confidence
verified

### Context
All providers must be declared in a central constant before use.

### Specification
Add `MY_PROVIDER: 'my_provider'` to the PROVIDERS object in `src/constants.ts`.

### Files
- Modify: src/constants.ts

### Acceptance Criteria
- [ ] PROVIDERS.MY_PROVIDER === 'my_provider'
- [ ] Passes: pnpm typecheck

## DECISION-01: Choose caching strategy

### Priority
P0

### Type
decision

### Labels
my-project, decision-gate

### Options
1. **Redis**: Fast, distributed, but requires infrastructure
2. **In-memory LRU**: Simple, no external deps, but no persistence
3. **SQLite**: Persistent, no infrastructure, but slower for high-throughput

### Recommendation
Redis is recommended for production scalability, but in-memory LRU is acceptable for MVP.

### Decision Needed By
Tech lead or architect

### Blocks
- CORE-01 (client implementation depends on caching choice)

### Acceptance Criteria
- [ ] Decision documented in TDD/ADR
- [ ] Dependent beads updated with decision outcome

## CORE-01: Implement client

### Priority
P0

### Type
task

### Labels
my-project, phase-1

### Blocked by
DECISION-01

### Parallelism
requires-sequence

### Confidence
verified

### Context
Need a typed client to interact with the external API.

### Specification
Create `src/lib/my-client.ts` with methods for list, get, create operations.
Handle rate limiting with exponential backoff.
Use the caching strategy chosen in DECISION-01.

### Files
- Create: src/lib/my-client.ts
- Reference: src/lib/other-client.ts

### Acceptance Criteria
- [ ] Client handles rate limits
- [ ] All methods return typed responses
- [ ] Passes: pnpm typecheck
```

### Add Dependencies

After creating beads, run this script:

\`\`\`bash
#!/bin/bash
set -e

declare -A IDS
while IFS=': ' read -r key value; do
  IDS[$key]=$value
done < <(bd list -l my-project --json | jq -r '.[] | "\(.title | split(":")[0]): \(.id)"')

# CORE-01 depends on SETUP-01
bd dep add ${IDS[CORE-01]} ${IDS[SETUP-01]}
# API-01 depends on CORE-01
bd dep add ${IDS[API-01]} ${IDS[CORE-01]}
# UI-01 depends on API-01
bd dep add ${IDS[UI-01]} ${IDS[API-01]}

bd sync
echo "Done!"
\`\`\`

### Verification Checklist

Before finalizing, verify:
- [ ] **No YAML frontmatter** - file starts with `# Title`, not `---`
- [ ] **No H2 except beads** - Summary, Dependency Graph, etc. use H3 (`###`)
- [ ] Every TDD requirement maps to ≥1 bead
- [ ] No orphan beads (all map back to TDD)
- [ ] Each bead has ONE responsibility
- [ ] All dependencies listed in Deps section
- [ ] Dependency commands section is complete
- [ ] Labels include project name and phase
- [ ] **DECISION gating**: All unresolved decisions have DECISION beads
- [ ] **DECISION gating**: All beads referencing unresolved decisions are blocked by the corresponding DECISION bead
- [ ] **Parallelism metadata**: Beads that can run concurrently are marked `parallelizable`
- [ ] **Confidence metadata**: Unverified beads are identified for potential iteration
- [ ] **No speculative references**: No "may", "might", "verify if", "find the" in Context/Specification
- [ ] **All file paths verified**: Every Modify/Reference path confirmed to exist
- [ ] **Coverage matrix produced**: Before finalizing, create a TDD-to-bead mapping table:
  | TDD Section/Requirement | Covered by Bead(s) | Status |
  |---|---|---|
  This table should appear as an H3 section (`### Coverage Matrix`) after the Summary Tables.
- [ ] **Run validator**: `pnpm validate:plans --path docs/plans/<project>` passes
