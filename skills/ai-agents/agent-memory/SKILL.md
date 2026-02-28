---
name: agent-memory
description: Long-term memory store for AI agents - save, search, and manage persistent memories across sessions. Load this skill for complete command reference.
version: 0.4.0
---

# Agent Memory - Full Reference

This skill provides complete documentation for the `agent-memory` CLI. Core behaviors (startup, auto-save, session end) are handled by the rules file which is always loaded.

## Setup

The agent-memory CLI must be installed and accessible:

```bash
# Check if installed
agent-memory --version

# If not in PATH, activate it
source ~/.agent-memory/bin/activate-memory
```

## Memory Scopes (Three-Scope Model)

| Scope | Storage | Visibility | Use Case |
|-------|---------|------------|----------|
| **project** | Project DB | Current project only | Project-specific decisions, patterns (default) |
| **group** | Global DB | Projects with matching `--groups` flag | Team conventions, shared patterns |
| **global** | Global DB | All projects, always | User preferences, cross-cutting concerns |

## Commands

### Save a Memory

```bash
# Save to current project (default scope)
agent-memory save "authentication uses JWT tokens stored in httpOnly cookies"

# Save with group scope (visible to projects using --groups=backend)
agent-memory save --group=backend "API versioning pattern for all services"

# Save globally (visible to all projects always)
agent-memory save --global "user prefers functional components over classes"

# Save and pin (always loaded at startup)
agent-memory save --pin "CRITICAL: never modify the legacy payment module"

# Save with explicit category
agent-memory save --category=decision "rejected Redux, using Zustand instead"

# Save with structured metadata (repeatable --meta key=value)
agent-memory save --meta rationale="simpler API" --meta alternatives="Redux,MobX" \
  --category=decision "Using Zustand for state management"

# Save an error-fix pattern with metadata
agent-memory save --meta error="ECONNREFUSED on port 6379" --meta root_cause="Redis not running" \
  "Fix ECONNREFUSED in tests: run 'docker compose up redis' before test suite"
```

**`--meta` flag:** Attach arbitrary key=value metadata to any memory. Use multiple `--meta` flags for multiple pairs. Metadata is stored as JSON, searchable, and included in all outputs (JSON, startup, get).

### Search Memories

```bash
# Semantic search (if enabled)
agent-memory search "how does authentication work"

# With stricter threshold
agent-memory search "auth" --threshold=0.8

# Include global memories
agent-memory search "coding style" --global

# Include group memories (works from any directory)
agent-memory search "api pattern" --group=backend-team

# Search across all projects (user visibility only)
agent-memory search "api pattern" --all-projects
```

### List Memories

```bash
# List project memories
agent-memory list

# List only pinned memories
agent-memory list --pinned

# List by category
agent-memory list --category=decision

# List global memories only
agent-memory list --global

# List global + group-scoped memories
agent-memory list --global --include-group-owned

# List group-scoped memories only
agent-memory list --group-owned

# List memories owned by a specific group
agent-memory list --owned-by=backend-team

# List memories by group name (works from any directory)
agent-memory list --group=backend-team
agent-memory list --group=all  # all groups

# List from all projects (user visibility only)
agent-memory list --all-projects
```

### Quick Group Access

View group info and memories quickly from anywhere:

```bash
# Quick view of group info + its memories
agent-memory groups backend-team

# View all group memories
agent-memory groups all

# Pinned only
agent-memory groups backend-team --pinned
```

### Manage Memories

```bash
# Get specific memory (verbose, includes metadata)
agent-memory get mem_abc123
agent-memory show mem_abc123  # alias for get

# Pin/unpin a memory
agent-memory pin mem_abc123
agent-memory unpin mem_abc123

# Delete a memory
agent-memory forget mem_abc123

# Delete memories matching a pattern
agent-memory forget --search "old pattern"
```

### Group Management for Memories

Manage owner groups for group-scoped memories:

```bash
# Add owner groups to a group-scoped memory
agent-memory add-groups mem_abc123 backend-team frontend-team

# Remove owner groups from a group-scoped memory
agent-memory remove-groups mem_abc123 frontend-team

# Replace all owner groups
agent-memory set-groups mem_abc123 backend-team devops

# Change scope of a memory
agent-memory set-scope mem_abc123 global                     # → global scope
agent-memory set-scope mem_abc123 group --group=backend      # → group scope
agent-memory set-scope mem_abc123 project --to-project /path # → project scope
```

### Session Management

```bash
# Start a new session
agent-memory session start

# Add a session summary
agent-memory session summarize "Implemented user authentication with JWT"

# End session
agent-memory session end

# List sessions
agent-memory session list

# Load last session context
agent-memory session load --last
```

### Session Analysis (Error-Fix Pattern Extraction)

Automatically extract error-fix patterns from session content using LLM:

```bash
# Analyze inline text describing errors and fixes
agent-memory session analyze "Hit TypeError in auth.ts, null check missing, fixed with optional chaining"

# Analyze the last session's summaries
agent-memory session analyze --last

# Analyze a specific session
agent-memory session analyze --session sess_abc123

# Preview patterns without saving (dry run)
agent-memory session analyze --last --dry-run

# Output as JSON
agent-memory session analyze --last --json

# Combine dry-run and JSON for preview
agent-memory session analyze --last --dry-run --json
```

Each extracted pattern is saved as a memory with structured metadata:
- `error`: The error message or symptom
- `cause`: The root cause
- `fix`: How it was fixed
- `context`: Where it occurred
- `analyzed_from`: Source (session ID or "text")

### Error-Detection Hooks

When enabled, error-detection hooks monitor command output and remind you to save error-fix patterns:

```bash
# Enable error-detection hooks
agent-memory config set hooks.error_nudge=true

# Disable error-detection hooks
agent-memory config set hooks.error_nudge=false
```

When enabled and an error is detected in a command's output, you'll see:
```
[agent-memory] Error detected in command output. If you resolved this error, consider saving the pattern:
  agent-memory save --meta error="..." --meta root_cause="..." "Description of the fix"
```

The hook never blocks command execution and exits silently when disabled.

### Workspace Groups

**Workspace groups are collections of PROJECTS that can share memories with each other.**

When a user mentions creating a "group with projects", they want to:
1. Create the workspace group
2. Add the listed project directories to it

```bash
# Create a group
agent-memory group create backend-team

# Delete a group
agent-memory group delete backend-team

# Add current project to a group
agent-memory group join backend-team

# Add a specific project to a group
agent-memory group join backend-team --project /path/to/project

# Remove project from a group
agent-memory group leave backend-team

# List all groups
agent-memory group list

# Show group details (includes project list)
agent-memory group show backend-team
```

#### Interpreting User Requests About Groups

| User says | Meaning | Action |
|-----------|---------|--------|
| "Create a group X with projects A, B, C" | Create group and add projects | `group create X` then `group join X --project <path>` for each |
| "Add project Y to group X" | Add a project to existing group | `group join X --project <path>` |
| "Share this memory with group X" | Save with group scope | `save --group=X "content"` |
| "Create a memory for group X" | Save with group scope | `save --group=X "content"` |

### Promote/Unpromote

Move memories between scopes:

```bash
# Promote project memory to global (default)
agent-memory promote mem_abc123

# Promote project memory to group scope
agent-memory promote mem_abc123 --to-group=backend-team

# Promote from a specific project
agent-memory promote mem_abc123 --from-project /path/to/project

# Move a global/group memory to a project (unpromote)
agent-memory unpromote mem_abc123 --to-project /path/to/project
```

### Configuration

```bash
# Show config
agent-memory config show

# Enable/disable semantic search
agent-memory config set semantic.enabled=true

# Set similarity threshold
agent-memory config set semantic.threshold=0.7

# Enable/disable autosave
agent-memory config set autosave.enabled=true

# Enable/disable error-detection hooks
agent-memory config set hooks.error_nudge=true
```

### Cross-Project Visibility (Users Only)

View memories across all projects (for users, not agents):

```bash
# List all tracked projects
agent-memory projects

# List memories from all projects
agent-memory list --all-projects

# Search across all projects
agent-memory search "pattern" --all-projects

# Export from all projects
agent-memory export --all-projects
```

## Startup Behavior

At the beginning of each session:

1. **Pinned project memories are automatically loaded**
2. **Pinned global memories are automatically loaded**
3. **Group-scoped memories are NOT loaded by default** - Use `--groups` to opt-in
4. **Ask the user about previous session** - "Would you like me to load the previous session context?"

Use this command to get startup context:

```bash
# Default: project + global memories only (no groups)
agent-memory startup --json

# Include specific groups
agent-memory startup --json --groups=backend-team

# Include multiple specific groups
agent-memory startup --json --groups=backend-team,shared-libs

# Include all groups
agent-memory startup --json --groups=all

# Include all groups except one
agent-memory startup --json --groups=all --exclude-groups=legacy-team
```

## Memory Categories

| Category | When to Use |
|----------|-------------|
| `factual` | Facts about codebase architecture, patterns, how things work |
| `decision` | User preferences, rejected options, chosen approaches |
| `task_history` | What was completed, implementation details |
| `session_summary` | Condensed summaries of work sessions |

## Structured Metadata

Use `--meta key=value` to attach structured data to memories. This is especially useful for:

### Decision Records (ADR-lite)

Record architectural decisions with full context:

```bash
agent-memory save --category=decision \
  --meta rationale="Need real-time updates without polling overhead" \
  --meta alternatives="polling,SSE,WebSockets" \
  --meta status=accepted \
  "Use WebSockets via Socket.IO for real-time dashboard updates"

agent-memory save --category=decision \
  --meta rationale="Team familiar with PostgreSQL, need JSONB support" \
  --meta alternatives="MongoDB,DynamoDB" \
  --meta decided_by=user \
  "Use PostgreSQL as primary database"
```

### Error-Fix Patterns

Save debugging knowledge so it's never lost:

```bash
agent-memory save \
  --meta error="TypeError: Cannot read properties of undefined (reading 'map')" \
  --meta root_cause="API returns null instead of empty array when no results" \
  --meta file="src/components/UserList.tsx" \
  "UserList crashes on empty search — API returns null for items, fix: use items ?? [] before .map()"

agent-memory save \
  --meta error="ECONNREFUSED 127.0.0.1:5432" \
  --meta root_cause="PostgreSQL not started" \
  "Database connection fails in dev — run 'brew services start postgresql@16' first"
```

### Workflow Annotations

```bash
agent-memory save --pin \
  --meta trigger="after changing .proto files" \
  --meta command="make proto" \
  "Run 'make proto' after modifying any .proto file to regenerate Go bindings"
```

## Writing Good Memories

### Be Specific and Searchable

| Quality | Example |
|---------|---------|
| Good | `"Auth middleware in src/middleware/auth.ts validates JWT from Authorization header, returns 401 on expiry"` |
| Bad | `"There's auth stuff somewhere"` |

### Include the Why

| Quality | Example |
|---------|---------|
| Good | `"Using Zustand over Redux — simpler API, less boilerplate for our small state"` |
| Bad | `"We use Zustand"` |

### Save Error-Fix Patterns with Context

| Quality | Example |
|---------|---------|
| Good | `"'Cannot read property of undefined' in UserProfile — API returns null when user has no avatar. Fix: optional chaining user.avatar?.url"` |
| Bad | `"Fixed a bug"` |

### What NOT to Save

- **Transient state** — Current branch name, temporary debugging flags
- **Obvious facts** — Things clear from package.json, README, or file structure
- **Exact code snippets** — Describe the pattern; code will change but patterns persist
- **Every small action** — Save learnings and decisions, not a log of every command run

## Best Practices

1. **Search before starting work** — Check memory for prior decisions, patterns, and context
2. **Save decisions with rationale** — "Chose X because Y, rejected Z" is more valuable than just "using X"
3. **Use `--meta` for structured data** — Especially for decisions (alternatives, rationale) and errors (error, root_cause, file)
4. **Pin critical memories** — Things that should always be loaded at startup
5. **Save error-fix patterns** — Debugging knowledge is expensive to re-derive
6. **Be specific** — Include file paths, function names, exact error messages
7. **Maintain memory hygiene** — Forget outdated memories when you notice them
8. **Summarize sessions** — Good summaries create searchable history for future sessions
9. **Don't over-save** — Quality over quantity; every memory costs context window space
10. **Don't create groups without permission** — Only manage groups when user explicitly asks

## Example Workflow

```bash
# At session start
agent-memory startup --json

# Search for relevant context before starting work
agent-memory search "payments webhook"

# During work - save learnings
agent-memory save "The billing service uses Stripe webhooks at /api/webhooks/stripe"
agent-memory save --category=decision --meta rationale="cleaner error boundaries" \
  "User prefers error handling with Result types over exceptions"

# Save debugging discovery
agent-memory save --meta error="webhook signature verification failed" \
  --meta root_cause="clock skew in Docker container" \
  "Stripe webhook sig verification fails in Docker — fix: sync container clock with host"

# If user asks to share across team projects:
agent-memory save --pin --group=backend-team "All services must use structured logging with correlation IDs"

# Before ending
agent-memory session summarize "Implemented Stripe webhook handler with signature verification. \
Chose Result types for error handling (cleaner boundaries). \
Discovered Docker clock skew issue with webhook verification — documented fix."
```
