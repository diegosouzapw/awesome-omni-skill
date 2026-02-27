---
name: beads-seed
description: Translate architectural planning documents into actionable beads issue graph structure.
---
# beads-seed

Bridge the gap between architectural planning and implementation tracking by seeding the beads issue graph from architecture documentation.

## When to use

Use this command after completing architectural planning and before starting implementation.
Also apply when new architectural decisions are made mid-project that require new work items.

This transition point occurs when you have stable architecture documents describing WHAT to build and WHY, and need to create the beads structure tracking HOW and WHEN.

## Input: Architecture documentation

Architecture and planning documents serve as source material:

- System design documents describing overall architecture
- Component breakdown documents identifying major subsystems
- Feature specifications detailing user-facing capabilities
- ADRs (Architecture Decision Records) explaining technical choices
- Technical specifications with implementation approaches

These documents typically live in paths like `docs/architecture/`, `docs/notes/development/`, or similar planning directories.

## Output: Beads issue structure

Transform architecture documents into beads work items:

- **Epics** represent major components, features, or subsystems from architecture
- **Stories** break down epics into implementable work units
- **Tasks** further decompose stories when granularity helps
- **Dependencies** wire the graph based on architectural dependencies (what blocks what)

The beads database becomes the living work tracker while architecture docs remain as context and rationale.

## Workflow steps

### 1. Read and analyze architecture documents

Identify all relevant planning documents describing the system to build.
Read through to understand major components, features, and their relationships.
Note explicit dependencies mentioned in the architecture (component A requires component B).

### 2. Identify epic boundaries

Determine natural epic boundaries based on architecture:

- Major system components (auth system, data layer, API gateway)
- User-facing features with multiple implementation steps
- Infrastructure or platform capabilities
- Cross-cutting concerns that span multiple areas

Each epic should represent a cohesive unit of work with clear business or technical value.

### 3. Break down into stories and tasks

For each epic, extract stories representing implementable units:

- Core functionality implementation
- Integration points with other components
- Testing and validation work
- Documentation and deployment tasks

Stories should be independently testable and deliverable.
Add tasks under stories only when additional granularity aids tracking.

### 4. Identify and wire dependencies

Map architectural dependencies to beads dependencies:

- Component A cannot function without component B → `bd dep add story-a story-b --type blocks`
- Feature X builds on feature Y → blocks relationship
- Integration story requires both components ready → two dependencies

Use `bd dep add <blocked-issue> <blocking-issue> --type blocks` syntax.
The first argument is the issue that depends on (is blocked by) the second argument.
The blocking issue must be resolved before the blocked issue can proceed.

**Note**: The `--type` defaults to `blocks` and can be omitted for blocking dependencies.
These are equivalent:
```bash
bd dep add bd-auth-integration-id bd-data-schema-id --type blocks
bd dep add bd-auth-integration-id bd-data-schema-id  # --type blocks is default
```

### 5. Create beads structure

Execute the seeding using bd commands:

```bash
# Create epics for major components
bd create --type epic "Authentication System" \
  --description "JWT-based auth with role-based access control"

bd create --type epic "Data Storage Layer" \
  --description "PostgreSQL schema with migration framework"

# Create stories under epics (use actual bd-* IDs from create output)
bd create "Implement JWT token generation and validation" \
  --parent bd-auth-epic-id \
  --description "Core token operations with expiry and refresh"

bd create "Database schema initial migration" \
  --parent bd-data-epic-id \
  --description "Version 1 schema with user and session tables"

bd create "Integrate auth with database for session storage" \
  --parent bd-auth-epic-id \
  --description "Persist sessions in PostgreSQL"

# Wire dependencies (--type blocks is default, can be omitted)
bd dep add bd-auth-integration-id bd-data-schema-id
bd dep add bd-auth-integration-id bd-jwt-impl-id

# Alternative: wire dependencies during creation
bd create "Integrate auth with database for session storage" \
  --parent bd-auth-epic-id \
  --description "Persist sessions in PostgreSQL" \
  --deps "bd-data-schema-id,bd-jwt-impl-id"
```

### 6. Commit the seeded structure

After creating the initial beads structure, commit the changes:

```bash
git add .beads/
git commit -m "feat: seed beads issues from architecture docs

Translate architecture planning into work items:
- N epics for major components
- M stories implementing features
- Dependencies wired per architectural requirements"
```

Include a summary of what was created in the commit message.

## Example workflow

Starting from architecture document at `docs/architecture/system-design.md`:

```bash
# Read the architecture
cat docs/architecture/system-design.md

# Create top-level epics
bd create --type epic "User Management" \
  --description "Complete user lifecycle and authentication"

bd create --type epic "Content Pipeline" \
  --description "Ingest, process, and serve content"

# Break down user management epic
bd create "User registration with email verification" \
  --parent bd-user-mgmt-epic

bd create "Password reset flow" \
  --parent bd-user-mgmt-epic

bd create "Session management and logout" \
  --parent bd-user-mgmt-epic

# Break down content pipeline epic
bd create "Content upload API endpoint" \
  --parent bd-content-epic

bd create "Processing worker for uploaded content" \
  --parent bd-content-epic

bd create "CDN integration for serving processed content" \
  --parent bd-content-epic

# Wire dependencies (processing worker needs upload API first)
bd dep add bd-processing-worker bd-upload-api

# Verify structure
bd status                                      # Quick health check
bd list --pretty                               # Tree view with status symbols
bd dep tree bd-user-mgmt-epic --direction both # Full dependency graph for epic
bd dep tree bd-content-epic --direction both   # Full dependency graph for epic

# Commit
git add .beads/
git commit -m "feat: seed beads from system-design.md architecture"
```

## Relationship between architecture docs and beads

Architecture documentation and beads issues serve complementary roles:

**Architecture docs answer WHY and WHAT**:
- Why this design approach was chosen
- What the system should accomplish
- What constraints and tradeoffs exist
- What alternatives were considered

**Beads issues answer HOW and WHEN**:
- How to implement the architecture
- When work items are started and completed
- How work items depend on each other
- How progress flows through the dependency graph

Architecture docs remain as permanent context and rationale.
Beads issues are the living work tracker that evolves as implementation proceeds.
Both are version controlled and evolve together but serve different purposes.

## Best practices

**Keep architecture docs stable**: Once seeded, architecture docs should change only for genuine architectural revisions, not implementation details.

**Let beads evolve**: Use beads-evolve patterns to refine work items as understanding deepens during implementation.

**Link bidirectionally**: Reference architecture docs in epic descriptions, reference epics from architecture docs for traceability.

**Seed incrementally**: For large systems, seed one epic at a time rather than attempting to create the entire structure upfront.

**Validate dependencies**: After seeding, run `bd list --pretty` to review the structure, and `bd dep tree <epic-id> --direction both` for each epic to verify dependencies match architectural understanding.

**Note**: Use `--direction both` with `bd dep tree` to see the full graph in both directions (what blocks this issue and what this issue blocks).
Use `--format mermaid` to generate Mermaid.js diagrams for visualization.

## Scriptable seeding

For automated seeding from scripts, use the `--silent` flag to capture issue IDs:

```bash
#!/usr/bin/env bash
# Create epics and capture IDs
AUTH_EPIC=$(bd create --type epic "Authentication System" \
  --description "JWT-based auth with RBAC" --silent)

DATA_EPIC=$(bd create --type epic "Data Storage Layer" \
  --description "PostgreSQL schema with migration framework" --silent)

# Create stories under epics with captured parent IDs
JWT_STORY=$(bd create "JWT token generation" \
  --parent "$AUTH_EPIC" \
  --description "Core token operations with expiry and refresh" --silent)

SCHEMA_STORY=$(bd create "Database schema initial migration" \
  --parent "$DATA_EPIC" \
  --description "Version 1 schema with user and session tables" --silent)

# Create integration story with inline dependencies
SESSION_STORY=$(bd create "Session storage" \
  --parent "$AUTH_EPIC" \
  --description "Persist sessions in PostgreSQL" \
  --deps "$SCHEMA_STORY,$JWT_STORY" --silent)

echo "Created epics: $AUTH_EPIC, $DATA_EPIC"
echo "Created stories: $JWT_STORY, $SCHEMA_STORY, $SESSION_STORY"
```

The `--silent` flag outputs only the issue ID, making it suitable for scripting and automation.

## Available dependency types

When wiring dependencies with `bd dep add`, the following types are available:

- `blocks` (default) - blocking dependency
- `tracks` - tracking relationship
- `related` - related issues
- `parent-child` - hierarchical relationship
- `discovered-from` - found during implementation
- `until` - temporal dependency
- `caused-by` - causal relationship
- `validates` - validation relationship
- `relates-to` - general relation
- `supersedes` - replacement relationship

The `blocks` type is the default and can be omitted.

## Common patterns

**Layered architecture seeding**:
- Create epics for each layer (data, domain, API, UI)
- Wire dependencies bottom-up (UI depends on API depends on domain depends on data)

**Feature-oriented seeding**:
- Create epics for each user-facing feature
- Stories represent implementation slices through the stack
- Dependencies between features based on shared infrastructure

**Infrastructure-first seeding**:
- Create foundation epic with core platform capabilities
- Create feature epics that depend on foundation stories
- Ensures platform readiness before feature work

## Integration with beads-evolve

Seeding creates the initial structure from architecture.
Use beads-evolve during implementation to refine as you learn:

- Split stories that prove too large
- Add missing dependencies discovered during work
- Create new stories for unanticipated requirements
- Adjust epic boundaries if initial decomposition was suboptimal

Seeding is the bridge from planning to execution.
Evolution is the adaptive refinement during execution.
