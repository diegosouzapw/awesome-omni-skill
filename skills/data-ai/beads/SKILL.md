---
name: beads
description: Work with Beads issue tracker for AI agent memory. Use when managing tasks, tracking dependencies, filing issues, or maintaining context across coding sessions. Handles bd commands, issue graphs, and persistent agent memory.
license: MIT
compatibility: Requires beads CLI (bd) v0.21.5+ in PATH. Git repository required for issue storage.
metadata:
  version: "0.21.5"
  author: Mark Ferree
  repository: https://github.com/mrf/beads-skill
  skill-author: Mark Ferree
---

# Beads: Memory System for Coding Agents

Beads is a lightweight, git-backed issue tracking system that provides persistent memory for AI coding agents across sessions.

## CRITICAL: Initialization Check

**BEFORE doing ANY beads operations, you MUST:**

1. Check if beads is initialized in the current project using the Glob tool:
   ```
   Glob pattern=".beads"
   ```

   If the .beads directory is found in the results, beads is initialized.

2. If .beads is not found, you MUST immediately run:
   ```
   /beads:init
   ```

   This will:
   - Initialize the beads database in the project
   - Set up proper permissions in `.claude/settings.local.json`
   - Enable all beads commands to run without permission prompts

3. Only after beads is confirmed initialized should you proceed with any `bd` commands.

**DO NOT skip this check.** Running `bd` commands without initialization will fail.

## Core Capabilities

### Issue Management
- Create, update, and close issues with rich metadata
- **Hash-based IDs** (bd-a1b2, bd-f14c) eliminate merge conflicts and ID collisions
- **Hierarchical child IDs** (bd-a3f8e9.1, bd-a3f8e9.3.1) support nested work breakdown
- Track dependencies between issues (blocks, discovered-from, etc.)
- Automatically identify "ready" work (issues with no open blockers)
- Filter by status, priority, assignee, labels, and type

### Agent Memory Integration
- Persist discovered work across conversation sessions
- Build dependency graphs for complex nested tasks
- File issues automatically during exploration
- Resume long-horizon tasks with full context

### Git-Based Sync
- Issues stored as JSONL files in `.beads/` directory
- Committed to git like any other code artifact
- Local SQLite cache for fast queries
- Optional daemon for background sync
- No external servers required

## Setup

### Slash Commands Installation
The beads skill provides slash commands like `/beads:ready`, `/beads:create-issue`, etc. If these commands are not available:

1. Create a symlink from the skill's commands to Claude Code's commands directory:
```bash
ln -s ~/.claude/skills/beads/commands/beads ~/.claude/commands/beads
```

2. Restart your Claude Code session to pick up the new commands

### Permissions Configuration
When first using Beads commands, configure project permissions to allow all `bd` commands without prompting:

1. Check if settings.json exists and has permissions configured
2. If not, add the following to the project's `.claude/settings.json`:
```json
{
  "permissions": {
    "allow": [
      "Bash(bd:*)"
    ]
  }
}
```

This ensures smooth operation of all Beads commands without repeated permission prompts.

## Common Workflows

### Understanding Issue IDs

Beads uses **hash-based IDs** to eliminate merge conflicts:
- Format: `bd-` followed by 4-6 hex characters (e.g., `bd-a1b2`, `bd-f14c`)
- **Hierarchical IDs** for nested work: `bd-a3f8e9.1`, `bd-a3f8e9.2`, `bd-a3f8e9.3.1`
- Auto-generated on creation - collision-resistant even with multiple agents
- Migration from old sequential IDs: use `bd migrate`

### Starting a Session
1. Check for ready work: `bd ready --json`
2. Review issue details: `bd show bd-a1b2`
3. Assign to yourself: `bd update bd-a1b2 -a @me`
4. Mark in progress: `bd update bd-a1b2 -s in_progress`

### During Development
1. Discover new issues: `bd create "Title" -d "Details" -t bug -p 1`
2. Link dependencies: `bd dep add bd-f3a1 bd-a1b2` (bd-f3a1 depends on bd-a1b2)
3. Update status: `bd update bd-a1b2 -s blocked`
4. Add labels: `bd label add bd-a1b2 security`

### Completing Work
1. Mark done: `bd close bd-a1b2`
2. Commit changes (includes Beads metadata in .beads/ directory)
3. Check for newly unblocked work: `bd ready --json`

### Complex Task Planning
1. Create parent epic: `bd create "Epic: Feature Name" -t epic -p 0`
2. Create child tasks with hierarchical IDs: `bd create "Subtask 1" -t task --parent bd-a3f8` (creates bd-a3f8.1)
3. Add blockers: `bd dep add bd-f3a1 bd-a1b2` (bd-f3a1 depends on bd-a1b2)
4. Visualize: `bd dep tree bd-a3f8`

## Best Practices for Agents

### When to File Issues
- Discovered technical debt during exploration
- Found bugs or edge cases while implementing features
- Identified related work that's out of current scope
- Need to remember context for future sessions

### Dependency Modeling
When using `bd dep add [dependent] [dependency]`:
- The first issue depends on the second
- The second issue must be done before the first
- Example: `bd dep add bd-f3a1 bd-a1b2` means "bd-f3a1 depends on bd-a1b2"

Use `--deps` flag during creation:
- `bd create "Task" --deps "discovered-from:bd-a3f8,blocks:bd-b2c4"`
- Or simple format: `--deps "bd-a3f8,bd-b2c4"`

Use `--parent` flag for hierarchical tasks:
- `bd create "Subtask" --parent bd-a3f8` creates bd-a3f8.1
- Supports up to 3 nesting levels (epic > feature > task)

### Working with Ready Issues
```bash
# Get machine-readable list of unblocked work
bd ready --json --limit 20

# Filter by priority
bd ready --priority 0  # Critical only

# Filter by assignee
bd ready --assignee alice

# Show all open issues
bd list --status open --json
```

### Maintaining Clean State
- Close completed issues: `bd close bd-1 bd-2 bd-3`
- Remove invalid blockers: `bd dep remove bd-2 bd-1`
- Reopen if needed: `bd reopen bd-1`
- Use labels for categorization: `bd label add bd-1 refactor security`

## CLI Reference

### Core Commands

#### Initialization & Maintenance
```bash
bd init                    # Initialize bd in current project
bd onboard                 # Interactive agent onboarding
bd quickstart              # Interactive tutorial
bd migrate                 # Migrate database to hash-based IDs
bd migrate --inspect       # Preview migration without changes
bd migrate --dry-run       # Test migration safety
bd doctor                  # Health checks and validation
bd doctor --fix            # Attempt automatic fixes
```

#### Creating Issues
```bash
bd create "Title"                              # Basic creation (auto-generates hash ID)
bd create "Title" -d "Description"             # With description
bd create "Title" -t bug -p 1                  # Set type and priority
bd create "Title" -a alice -l backend,urgent   # Assign and label
bd create "Title" --deps "bd-a3f8,bd-b2c4"     # With dependencies
bd create "Title" --parent bd-a3f8             # Create child issue (bd-a3f8.1)
bd create -f plan.md                           # From markdown file
bd create "Title" --json                       # JSON output
```

**Types**: bug, feature, task, epic, chore
**Priorities**: 0=critical, 1=high, 2=medium (default), 3=low, 4=backlog
**Note**: Hash-based IDs are auto-generated; explicit IDs are no longer supported

#### Viewing Issues
```bash
bd show bd-a1b2                        # Full details of one issue
bd show bd-a1b2 bd-f3a1 bd-c5d6        # Multiple issues
bd list                                # All issues
bd list --status open                  # Filter by status
bd list --priority 1                   # Filter by priority (0-4)
bd list --assignee alice               # Filter by assignee
bd list --label backend,urgent         # AND logic (all labels)
bd list --label-any frontend,backend   # OR logic (any label)
bd list --type bug                     # Filter by type
bd list --title "auth"                 # Text search in title
bd list --limit 50                     # Limit results
bd list --json                         # JSON output
```

**Statuses**: open, in_progress, blocked, closed

#### Updating Issues
```bash
bd update bd-a1b2 -s in_progress       # Change status
bd update bd-a1b2 -p 0                 # Change priority
bd update bd-a1b2 -a bob               # Reassign
bd update bd-a1b2 --title "New Title"  # Update title
bd update bd-a1b2 -d "New description" # Update description
bd update bd-a1b2 bd-f3a1 bd-c5d6 -s closed  # Bulk update
bd update bd-a1b2 --json               # JSON output
```

#### Closing Issues
```bash
bd close bd-a1b2                       # Close one issue
bd close bd-a1b2 bd-f3a1 bd-c5d6       # Close multiple
bd close bd-a1b2 --reason "Completed"  # With reason
bd reopen bd-a1b2                      # Reopen closed issue
```

#### Dependencies
```bash
bd dep add bd-f3a1 bd-a1b2             # bd-f3a1 depends on bd-a1b2
bd dep add bd-f3a1 bd-a1b2 --type blocks  # Explicit type
bd dep remove bd-f3a1 bd-a1b2          # Remove dependency
bd dep tree bd-a1b2                    # Show dependency tree
bd dep cycles                          # Detect circular deps
```

**Note**: In `bd dep add [A] [B]`, issue A depends on issue B (B must finish first)

#### Finding Work
```bash
bd ready                               # Issues with no blockers
bd ready --limit 20                    # Limit results
bd ready --priority 1                  # High priority only
bd ready --assignee alice              # Assigned to alice
bd ready --json                        # JSON output
bd blocked                             # Show blocked issues
bd stats                               # Statistics
```

#### Labels
```bash
bd label add bd-a1b2 security          # Add single label
bd label add bd-a1b2 bug urgent        # Add multiple labels
bd label remove bd-a1b2 urgent         # Remove label
bd label list bd-a1b2                  # Labels on issue
bd label list-all                      # All labels with counts
```

#### Daemon Management
```bash
bd daemons                             # List all running daemons
bd daemons health                      # Check daemon health status
bd daemons stop <pid>                  # Stop specific daemon
bd daemons logs <pid>                  # View daemon logs
bd daemons killall                     # Stop all daemons
bd daemon &                            # Start daemon in background
```

#### Deletion
```bash
bd delete bd-a1b2                      # Preview mode
bd delete bd-a1b2 --force              # Force delete
bd delete bd-a1b2 bd-f3a1 bd-c5d6 --force  # Batch delete
bd delete bd-a1b2 --cascade --force    # Delete with dependents
```

#### Comments
```bash
bd comments bd-a1b2                    # View comments
bd comments bd-a1b2 "Add comment"      # Add comment
```

#### Configuration
```bash
bd config set jira.url "https://..."   # Set config value
bd config get jira.url                 # Get value
bd config list --json                  # List all config
bd config unset jira.url               # Remove config
```

#### Export/Import
```bash
bd export -o issues.jsonl              # Export to JSONL
bd import -i issues.jsonl              # Import from JSONL
bd compact --days 90                   # Compact old issues
bd sync                                # Manual git sync
```

### Global Flags

Available on all commands:
```bash
--json                  # JSON output
--actor "name"          # Override actor name
--db "/path/to/db"      # Override database path
--no-daemon             # Bypass daemon
--no-auto-flush         # Disable auto-sync
--no-auto-import        # Disable auto-import
--sandbox               # All no-* flags combined
```

## JSON Output Format

All commands support `--json` flag:

```json
{
  "id": "bd-a1b2",
  "title": "Implement OAuth login",
  "description": "Add OAuth 2.0 support",
  "status": "in_progress",
  "priority": 1,
  "type": "feature",
  "assignee": "alice",
  "labels": ["auth", "security"],
  "created_at": "2025-01-15T10:30:00Z",
  "updated_at": "2025-01-15T14:22:00Z",
  "closed_at": null,
  "dependencies": ["bd-c5d6"],
  "dependents": ["bd-f3a1", "bd-d7e8"],
  "external_ref": "gh-123"
}
```

**Note**: IDs use hash-based format (bd-a1b2) instead of sequential numbers

## Agent Integration Patterns

### Session Initialization
When starting a session, check if Beads is initialized using the Glob tool:

```
Glob pattern=".beads"
```

If the .beads directory is found:
- Beads memory is available
- Run `bd ready --json --limit 10` to see available work

If not found:
- No Beads database exists
- Run `/beads:init` to initialize

### Auto-Filing Discovered Work
```bash
# When finding a bug during implementation
bd create "Fix null pointer in auth handler" \
    -d "Found in auth.go:142 during login feature work" \
    -t bug \
    -p 1 \
    -l "bug,backend" \
    --deps "discovered-from:bd-a3f8" \
    --json
```

### Task Planning Template
```bash
# Break down large feature using hierarchical IDs
EPIC_ID=$(bd create "Epic: User Dashboard" -t epic -p 1 --json | jq -r '.id')
# Creates bd-a3f8.1, bd-a3f8.2, etc.
bd create "Design dashboard layout" -t task --parent "$EPIC_ID"
bd create "Implement data fetching" -t task --parent "$EPIC_ID"
bd create "Add filtering controls" -t task --parent "$EPIC_ID"
bd create "Write integration tests" -t task --parent "$EPIC_ID"

# Add inter-task dependencies
LAYOUT_ID=$(bd list --title "Design dashboard" --json | jq -r '.[0].id')
DATA_ID=$(bd list --title "Implement data" --json | jq -r '.[0].id')
bd dep add "$DATA_ID" "$LAYOUT_ID"  # Data fetching depends on layout
```

### Querying Ready Work
```bash
# Get top priority unblocked work
READY=$(bd ready --priority 0 --json --limit 5)
echo "$READY" | jq -r '.[] | "\(.id): \(.title)"'

# Get assigned work for agent
bd list --assignee "@agent" --status in_progress --json
```

## Installation

```bash
curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash
```

Or install from source (requires Go):
```bash
git clone https://github.com/steveyegge/beads.git
cd beads
go install
```

Verify installation:
```bash
bd version
bd quickstart
```

## Troubleshooting

### Migration from Sequential IDs
If you have an old database with sequential IDs (bd-1, bd-2):
```bash
# Inspect migration plan
bd migrate --inspect

# Test migration safety
bd migrate --dry-run

# Perform migration (creates backup)
bd migrate
```

### Health Checks
Run diagnostic checks for common issues:
```bash
# Check setup and database health
bd doctor

# Attempt automatic fixes
bd doctor --fix
```

### Daemon Management
```bash
# Check running daemons
bd daemons

# Check daemon health
bd daemons health

# Stop all daemons if experiencing issues
bd daemons killall
```

### Sync Conflicts
If git conflicts occur in `.beads/` directory:
```bash
# Beads auto-merges on next operation
bd list  # Triggers merge
```

### Missing Dependencies
```bash
# Check Go installation (if building from source)
go version

# Verify bd in PATH
which bd

# Check version and ensure it's up to date
bd version  # Should be v0.21.5 or later for hash IDs
```

### Performance Issues
```bash
# Use daemon for better performance
bd daemon &

# Check database path
bd config get db.path

# Compact old issues
bd compact --days 90 --dry-run
```

### Common Errors

**"Issue not found"**: Check ID format (should be hash-based like `bd-a1b2`, `bd-f14c`)
**"Dependency cycle detected"**: Use `bd dep cycles` to find circular dependencies
**"Database not found"**: Run `bd init` in project root
**"Old ID format detected"**: Run `bd migrate` to upgrade to hash-based IDs

## References

- GitHub: https://github.com/steveyegge/beads
- Installation: https://github.com/steveyegge/beads#installation
- CLI Reference: `bd --help`
- Command Help: `bd [command] --help`
