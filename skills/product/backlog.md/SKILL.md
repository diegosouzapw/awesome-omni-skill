---
name: Backlog.md
description: Expert guidance for Backlog.md CLI project management tool including task creation, editing, status management, acceptance criteria, search, and board visualization. Use this when managing project tasks, creating task lists, updating task status, or organizing project work.
---

# Backlog.md

Expert assistance with Backlog.md CLI project management tool.

## Overview

Backlog.md is a CLI-based project management tool that uses markdown files to track tasks, documentation, and decisions. All operations go through the `backlog` CLI command.

**Key Principle**: NEVER edit task files directly. Always use CLI commands.

## Quick Start

```bash
# List tasks
backlog task list --plain

# View task details
backlog task 42 --plain

# Create task
backlog task create "Task title" -d "Description" --ac "Acceptance criterion"

# Edit task
backlog task edit 42 -s "In Progress" -a @me

# Search
backlog search "keyword" --plain
```

## Task Creation

### Basic Task Creation
```bash
# Simple task
backlog task create "Implement user login"

# With description
backlog task create "Implement user login" -d "Add authentication flow"

# With acceptance criteria
backlog task create "Implement user login" \
  --ac "User can login with email and password" \
  --ac "Invalid credentials show error message"

# Complete task creation
backlog task create "Implement user login" \
  -d "Add authentication flow with session management" \
  --ac "User can login with valid credentials" \
  --ac "Session persists across page refreshes" \
  -s "To Do" \
  -a @developer \
  -l backend,auth \
  --priority high
```

### Advanced Creation
```bash
# Create draft task
backlog task create "Research options" --draft

# Create subtask
backlog task create "Implement feature component" -p 42

# With dependencies
backlog task create "Deploy to production" --dep task-40 --dep task-41

# With multiple labels
backlog task create "Fix bug" -l bug,urgent,frontend
```

## Task Modification

### Status Management
```bash
# Change status
backlog task edit 42 -s "In Progress"
backlog task edit 42 -s "Done"
backlog task edit 42 -s "Blocked"

# Common workflow: start work on task
backlog task edit 42 -s "In Progress" -a @me
```

### Basic Fields
```bash
# Update title
backlog task edit 42 -t "New title"

# Update description (with newlines)
backlog task edit 42 -d $'Multi-line\ndescription\nhere'

# Assign task
backlog task edit 42 -a @username

# Add/update labels
backlog task edit 42 -l backend,api,v2

# Set priority
backlog task edit 42 --priority high
backlog task edit 42 --priority medium
backlog task edit 42 --priority low
```

### Acceptance Criteria Management

**IMPORTANT**: Each flag accepts multiple values

```bash
# Add new criteria (multiple flags supported)
backlog task edit 42 --ac "User can login" --ac "Session persists"

# Check criteria by index (multiple flags supported)
backlog task edit 42 --check-ac 1 --check-ac 2 --check-ac 3

# Uncheck criteria
backlog task edit 42 --uncheck-ac 2

# Remove criteria (processed high-to-low)
backlog task edit 42 --remove-ac 3 --remove-ac 2

# Mixed operations in one command
backlog task edit 42 \
  --check-ac 1 \
  --uncheck-ac 2 \
  --remove-ac 4 \
  --ac "New criterion"
```

**Note**:
- ✅ Use multiple flags: `--check-ac 1 --check-ac 2`
- ❌ Don't use commas: `--check-ac 1,2,3`
- ❌ Don't use ranges: `--check-ac 1-3`

### Implementation Plan & Notes

```bash
# Add implementation plan (with newlines using ANSI-C quoting)
backlog task edit 42 --plan $'1. Research codebase\n2. Implement solution\n3. Write tests\n4. Update docs'

# POSIX portable (using printf)
backlog task edit 42 --plan "$(printf '1. Step one\n2. Step two')"

# Add implementation notes (replaces existing)
backlog task edit 42 --notes $'Implemented using strategy pattern\nUpdated all tests\nReady for review'

# Append to existing notes
backlog task edit 42 --append-notes $'- Fixed edge case\n- Added validation'

# Format notes as PR description (best practice)
backlog task edit 42 --notes $'## Changes\n- Implemented X\n- Fixed Y\n\n## Testing\n- Added unit tests\n- Manual testing complete'
```

### Dependencies
```bash
# Add dependencies
backlog task edit 42 --dep task-1 --dep task-2

# Remove dependencies (edit task file or recreate without)
```

## Viewing Tasks

### List Tasks
```bash
# List all tasks (always use --plain for AI-readable output)
backlog task list --plain

# Filter by status
backlog task list -s "To Do" --plain
backlog task list -s "In Progress" --plain
backlog task list -s "Done" --plain

# Filter by assignee
backlog task list -a @username --plain
backlog task list -a @me --plain

# Filter by tag
backlog task list --tag backend --plain
backlog task list --tag bug --plain

# Combined filters
backlog task list -s "In Progress" -a @me --plain
```

### View Task Details
```bash
# View single task (always use --plain)
backlog task 42 --plain

# View in browser (if supported)
backlog task 42 --web
```

### Search
```bash
# Search across all content (uses fuzzy matching)
backlog search "authentication" --plain

# Search only tasks
backlog search "login" --type task --plain

# Search with filters
backlog search "api" --status "To Do" --plain
backlog search "bug" --priority high --plain

# Search in specific fields
backlog search "database" --in title --plain
```

## Task Workflow

### Complete Task Lifecycle
```bash
# 1. Create task
backlog task create "Implement feature X" \
  -d "Add new feature to handle Y" \
  --ac "Feature works as expected" \
  --ac "Tests are passing" \
  --ac "Documentation is updated"

# 2. Start work: assign and change status
backlog task edit 42 -s "In Progress" -a @me

# 3. Add implementation plan
backlog task edit 42 --plan $'1. Research existing code\n2. Implement core logic\n3. Add tests\n4. Update docs'

# 4. Work on task (write code, test, etc.)

# 5. Mark acceptance criteria as complete
backlog task edit 42 --check-ac 1 --check-ac 2 --check-ac 3

# 6. Add implementation notes (PR description)
backlog task edit 42 --notes $'## Changes\n- Implemented feature X using approach Y\n- Added comprehensive tests\n\n## Testing\n- Unit tests pass\n- Integration tests pass\n- Manual testing complete'

# 7. Mark task as done
backlog task edit 42 -s Done
```

### Starting a Task (Critical Steps)
```bash
# ALWAYS do these steps when starting a task:
# 1. Set status to In Progress
# 2. Assign to yourself
backlog task edit 42 -s "In Progress" -a @me

# 3. Review the task requirements
backlog task 42 --plain

# 4. Create implementation plan
backlog task edit 42 --plan $'1. Analyze requirements\n2. Design solution\n3. Implement\n4. Test\n5. Document'
```

## Board & Visualization

```bash
# View Kanban board in terminal
backlog board

# Open web UI
backlog browser

# Export board snapshot
backlog board --export board-snapshot.md

# Generate report
backlog report --output report.md
```

## Task Operations

### Archive & Promote
```bash
# Archive completed task
backlog task archive 42

# Demote task to draft
backlog task demote 42

# Promote draft to task (gets new ID)
backlog task promote draft-5
```

### Task History
```bash
# View task history (if supported)
backlog task history 42

# View changes
backlog task diff 42
```

## Documentation & Decisions

### When to Use Docs vs Decisions

**Use `backlog doc` for:**
- General project documentation
- Architecture overviews
- Technical specifications
- Design documents
- User guides
- Reference documentation

**Use `backlog decision` for:**
- Architectural decisions (ADRs)
- Technology choices
- Design pattern selections
- Infrastructure decisions
- Breaking changes that need justification

**Golden Rule for AI Assistants**: When you need to create markdown documentation (like architecture docs, decision records, specifications), ALWAYS use `backlog doc create` or `backlog decision create`. NEVER create standalone .md files in the project root.

### Documents

Documents are stored in `backlog/docs/` with the naming pattern `doc-XXX - Title.md`.

#### Creating Documents

```bash
# Basic document (opens editor for content)
backlog doc create "API Documentation"

# With type
backlog doc create "Architecture Overview" -t architecture

# From file
backlog doc create "README" -p /path/to/readme.md

# Via MCP tools (for AI assistants - preferred method)
# Use mcp__backlog__document_create tool with title and content
```

#### Viewing & Listing Documents

```bash
# List all documents
backlog doc list --plain

# View specific document
backlog doc view doc-001

# Search documents
backlog search "architecture" --type doc --plain
```

#### Document Structure

Documents have YAML frontmatter:
```yaml
---
id: doc-001
title: API Documentation
type: other
created_date: '2025-11-11 15:22'
---
```

Document types:
- `architecture` - System architecture docs
- `specification` - Technical specifications
- `guide` - User or developer guides
- `reference` - Reference documentation
- `other` - General documentation

### Architectural Decisions

Decisions are stored in `backlog/decisions/` with the naming pattern `decision-XXX - Title.md`.

#### Creating Decisions

```bash
# Create decision (opens editor)
backlog decision create "Use PostgreSQL for data storage"

# With status
backlog decision create "Migrate to microservices" --status "proposed"

# Decision statuses:
# - proposed: Decision proposed, under discussion
# - accepted: Decision accepted and implemented
# - rejected: Decision rejected
# - deprecated: Decision no longer valid
# - superseded: Replaced by newer decision
```

#### Decision Document Template

When you create a decision, it uses this template:

```markdown
---
id: decision-XXX
title: Your Decision Title
date: 'YYYY-MM-DD HH:MM'
status: proposed
---
## Context

What is the issue or situation we're addressing?
- What forces are at play?
- What constraints exist?
- What are we trying to achieve?

## Decision

What decision have we made?
- What approach are we taking?
- Why this approach over alternatives?

## Consequences

What are the implications of this decision?
- Positive outcomes
- Negative trade-offs
- What becomes easier/harder?
- Future implications
```

#### Viewing & Listing Decisions

```bash
# List all decisions
backlog decision list --plain

# View specific decision (if supported)
backlog decision view decision-001

# Search decisions
backlog search "database" --type decision --plain
```

#### Writing Good Decisions

**Title**: Clear, concise statement of decision
```bash
# ✅ Good
backlog decision create "Use PostgreSQL for primary database"
backlog decision create "Implement OAuth2 for authentication"
backlog decision create "Adopt microservices architecture"

# ❌ Bad
backlog decision create "Database"
backlog decision create "We should probably use OAuth"
```

**Context**: Explain the problem and constraints
```markdown
## Context

Our current SQLite database cannot handle the increased load from
1000+ concurrent users. We need a production-grade database that:
- Supports high concurrency
- Provides ACID guarantees
- Offers good tooling and community support
- Fits within our budget constraints

We evaluated PostgreSQL, MySQL, and MongoDB.
```

**Decision**: State the decision clearly with rationale
```markdown
## Decision

We will use PostgreSQL as our primary database.

Rationale:
- Excellent ACID compliance and data integrity
- Superior support for complex queries and JSON data
- Strong community and ecosystem
- Our team has PostgreSQL experience
- Free and open source (cost-effective)
- Better performance for our read-heavy workload than MySQL
- More mature than MongoDB for transactional data
```

**Consequences**: Document trade-offs honestly
```markdown
## Consequences

Positive:
- Robust data integrity and ACID compliance
- Better scalability for our use case
- Rich feature set (JSON, full-text search, etc.)
- Excellent monitoring and debugging tools

Negative:
- More complex to set up than SQLite
- Requires separate database server (infrastructure cost)
- Learning curve for team members unfamiliar with PostgreSQL
- Migration effort from existing SQLite database

Trade-offs:
- Operational complexity increases, but data reliability improves
- Higher infrastructure cost, but better performance at scale
```

### Decision Workflow

#### 1. Proposing a Decision
```bash
# Create decision as "proposed"
backlog decision create "Adopt GraphQL API" --status "proposed"

# Document the decision
# - Add context about the problem
# - Explain the proposed solution
# - List alternatives considered
# - Document trade-offs
```

#### 2. Discussion
```bash
# Team reviews the decision
# - Add comments/feedback
# - Update decision based on feedback
# - Consider alternatives
```

#### 3. Accepting or Rejecting
```bash
# If accepted
# - Update status to "accepted"
# - Document final rationale
# - Link to implementation tasks

# If rejected
# - Update status to "rejected"
# - Document why it was rejected
# - Preserve for future reference
```

#### 4. Implementation
```bash
# Create tasks to implement decision
backlog task create "Implement PostgreSQL migration" \
  -d "Migrate from SQLite to PostgreSQL (per decision-001)" \
  --ac "Database schema migrated" \
  --ac "All data transferred successfully" \
  --ac "Application updated to use PostgreSQL"
```

#### 5. Deprecation or Superseding
```bash
# If decision becomes outdated
# - Create new decision
# - Mark old decision as "deprecated" or "superseded"
# - Reference new decision in old one
```

### Linking Decisions to Tasks

Always reference decisions in related tasks:

```bash
backlog task create "Implement OAuth2 authentication" \
  -d "Implement OAuth2 as decided in decision-002" \
  --ac "OAuth2 flow implemented" \
  --ac "Existing sessions migrated"

# In task notes, reference decision
backlog task edit 42 --notes $'## Decision Reference
This implements decision-002: "Use OAuth2 for authentication"

## Implementation
- Integrated passport-oauth2 library
- Configured Google and GitHub providers
...'
```

### Best Practices for Documentation

#### Document Organization

```bash
# Architecture documents
backlog doc create "System Architecture Overview" -t architecture
backlog doc create "Database Schema Design" -t architecture
backlog doc create "API Architecture" -t architecture

# Specifications
backlog doc create "API Specification v2" -t specification
backlog doc create "Authentication Flow Specification" -t specification

# Guides
backlog doc create "Development Setup Guide" -t guide
backlog doc create "Deployment Guide" -t guide
```

#### Version Control

Documents and decisions are version controlled with Git:
- Keep documents up to date
- Use Git history to track changes
- Reference specific versions when needed

#### Searchability

Use consistent terminology:
```bash
# Good - Searchable terms
backlog doc create "Authentication Architecture"
backlog decision create "Use JWT for API authentication"

# Bad - Hard to search
backlog doc create "How we do auth stuff"
backlog decision create "Token thing"
```

## Best Practices

### Writing Good Tasks

**Title**: Clear, concise, action-oriented
```bash
# ✅ Good
backlog task create "Implement user authentication"
backlog task create "Fix memory leak in image processor"

# ❌ Bad
backlog task create "Users"
backlog task create "There's a problem with the app"
```

**Description**: Explain the "why" and context
```bash
backlog task create "Add rate limiting to API" \
  -d "Current API has no rate limiting, causing server overload during peak hours. Need to implement per-user rate limiting to prevent abuse."
```

**Acceptance Criteria**: Focus on outcomes, not implementation
```bash
# ✅ Good - Testable outcomes
--ac "API rejects requests after 100 requests per minute per user"
--ac "User receives clear error message when rate limited"
--ac "Rate limit resets after 60 seconds"

# ❌ Bad - Implementation details
--ac "Add a rate limiter middleware"
--ac "Use Redis for tracking"
```

### Task Organization

**Use Labels Effectively**
```bash
# Organize by type, area, and priority
backlog task create "Fix login bug" -l bug,urgent,auth
backlog task create "Optimize queries" -l enhancement,backend,performance
backlog task create "Update docs" -l documentation,frontend
```

**Use Tags for Metadata**
```bash
# Tags for filtering and organization
--tag "sprint:23"
--tag "epic:user-management"
--tag "team:backend"
```

### Task Breakdown

**Atomic Tasks**: Each task should be independently deliverable
```bash
# ✅ Good - One PR per task
backlog task create "Add login endpoint"
backlog task create "Add logout endpoint"
backlog task create "Add session refresh endpoint"

# ❌ Bad - Too large for one PR
backlog task create "Implement entire authentication system"
```

**Avoid Future Dependencies**: Never reference tasks that don't exist yet
```bash
# ✅ Good - Reference existing tasks
backlog task create "Deploy auth API" --dep task-40 --dep task-41

# ❌ Bad - Reference future tasks
backlog task create "Add feature A" -d "This will be used by future tasks"
```

### Implementation Notes

**Format as PR Description**: Make notes ready for GitHub
```bash
backlog task edit 42 --notes $'## Summary
- Implemented user authentication with JWT
- Added password hashing with bcrypt
- Created login/logout endpoints

## Changes
- Added auth middleware
- Updated user model
- Added auth routes

## Testing
- Unit tests for auth functions
- Integration tests for endpoints
- Manual testing with Postman

## Breaking Changes
None

## Follow-up
- Add refresh token rotation
- Implement rate limiting'
```

**Progressive Notes**: Append as you work
```bash
# As you make progress
backlog task edit 42 --append-notes "- Implemented core auth logic"
backlog task edit 42 --append-notes "- Added tests"
backlog task edit 42 --append-notes "- Updated documentation"
```

## Definition of Done

**A task is Done only when ALL of these are complete:**

Via CLI:
1. ✅ All acceptance criteria checked: `--check-ac 1 --check-ac 2 ...`
2. ✅ Implementation notes added: `--notes "..."`
3. ✅ Status set to Done: `-s Done`

Via Code/Testing:
4. ✅ Tests pass
5. ✅ Documentation updated
6. ✅ Code reviewed
7. ✅ No regressions

**Never mark task as Done without completing ALL items**

## Common Patterns

### Daily Workflow
```bash
# Morning: Check your tasks
backlog task list -a @me -s "In Progress" --plain

# Start working on next task
backlog task edit 42 -s "In Progress" -a @me
backlog task 42 --plain  # Read requirements

# Add plan
backlog task edit 42 --plan $'1. Analyze\n2. Implement\n3. Test'

# As you work, check off AC
backlog task edit 42 --check-ac 1
# ... continue working ...
backlog task edit 42 --check-ac 2

# End of day: Add notes
backlog task edit 42 --append-notes "- Completed X, Y pending"
```

### Sprint Planning
```bash
# Review backlog
backlog task list -s "To Do" --plain

# Tag tasks for sprint
backlog task edit 42 --tag "sprint:23"
backlog task edit 43 --tag "sprint:23"

# Assign tasks
backlog task edit 42 -a @developer1
backlog task edit 43 -a @developer2

# View sprint board
backlog board
```

### Bug Fix Workflow
```bash
# Create bug task
backlog task create "Fix login timeout issue" \
  -d "Users report login times out after 30 seconds on slow connections" \
  --ac "Login works on slow connections (tested with throttling)" \
  --ac "Timeout increased to 60 seconds" \
  --ac "User sees loading indicator during login" \
  -l bug,urgent,auth \
  --priority high

# Start work
backlog task edit 42 -s "In Progress" -a @me

# Implement fix and add notes
backlog task edit 42 --notes $'## Root Cause
Login timeout was set to 30s, causing issues on slow connections.

## Fix
- Increased timeout to 60s
- Added exponential backoff for retries
- Improved loading indicator visibility

## Testing
- Tested with Chrome network throttling (Slow 3G)
- Verified timeout works correctly
- All existing auth tests pass'

# Complete
backlog task edit 42 --check-ac 1 --check-ac 2 --check-ac 3
backlog task edit 42 -s Done
```

## Troubleshooting

### Task Not Found
```bash
# List all tasks to find ID
backlog task list --plain

# Search for task
backlog search "keyword" --type task --plain
```

### Acceptance Criteria Issues
```bash
# View task to see AC numbers
backlog task 42 --plain

# AC numbering starts at 1
backlog task edit 42 --check-ac 1  # First AC

# Check multiple at once
backlog task edit 42 --check-ac 1 --check-ac 2 --check-ac 3
```

### Metadata Out of Sync
```bash
# Re-edit via CLI to fix
backlog task edit 42 -s "In Progress"

# If persistent, check file permissions
ls -la backlog/tasks/
```

### Multiline Input
```bash
# Use ANSI-C quoting (bash/zsh)
backlog task edit 42 --notes $'Line 1\nLine 2'

# Use printf (POSIX portable)
backlog task edit 42 --notes "$(printf 'Line 1\nLine 2')"

# PowerShell
backlog task edit 42 --notes "Line 1`nLine 2"

# Don't use literal \n - it won't work
# ❌ backlog task edit 42 --notes "Line 1\nLine 2"
```

## Command Reference

### Core Commands
```bash
# Tasks
backlog task create <title> [options]
backlog task list [filters] --plain
backlog task <id> --plain
backlog task edit <id> [options]
backlog task archive <id>
backlog task demote <id>

# Search
backlog search <query> [filters] --plain

# Board & Reports
backlog board
backlog browser
backlog report --output <file>

# Documents
backlog doc create <title>
backlog doc list --plain
backlog doc edit <id>

# Decisions
backlog decision create <title>
backlog decision list --plain
backlog decision <id> --plain
```

### Common Options
```bash
# Task creation/editing
-t, --title           Task title
-d, --description     Task description
-s, --status          Status (To Do, In Progress, Done, Blocked)
-a, --assignee        Assignee (@username)
-l, --labels          Comma-separated labels
--priority            Priority (low, medium, high)
--ac                  Add acceptance criterion
--check-ac            Check AC by index
--uncheck-ac          Uncheck AC by index
--remove-ac           Remove AC by index
--plan                Implementation plan
--notes               Implementation notes
--append-notes        Append to notes
--dep                 Add dependency (task-id)
--draft               Create as draft
-p, --parent          Parent task ID

# Filters
--plain               Plain text output (AI-friendly)
--status              Filter by status
--assignee            Filter by assignee
--tag                 Filter by tag
--priority            Filter by priority
--type                Filter by type (task, doc, decision)
```

## Tips

1. **Always use `--plain`** when listing or viewing for AI processing
2. **For AI: Use backlog for docs**: Create architecture/decision docs with `backlog doc create` or `backlog decision create`, NEVER create standalone .md files
3. **Start tasks properly**: Set In Progress and assign to yourself
4. **Check AC as you go**: Don't wait until end to mark them complete
5. **Use multiline properly**: Use `$'...\n...'` syntax for newlines
6. **Multiple flags work**: `--check-ac 1 --check-ac 2 --check-ac 3`
7. **Organize with labels**: Use consistent labeling scheme
8. **Atomic tasks**: One PR = One task
9. **PR-ready notes**: Format notes as GitHub PR description
10. **Never edit files directly**: Always use CLI commands
11. **Search is fuzzy**: "auth" finds "authentication"
12. **Document decisions**: Use decision records for architectural choices
13. **Link decisions to tasks**: Reference decision IDs in related task descriptions

## Integration with Git

```bash
# Workflow
git checkout -b task-42-implement-feature
# ... implement task 42 ...
backlog task edit 42 --check-ac 1 --check-ac 2
backlog task edit 42 --notes "Implementation complete"
backlog task edit 42 -s Done
git add .
git commit -m "Implement feature X

Refs: task-42"
git push
```

## Remember

**🎯 Golden Rule**: Always use CLI commands. Never edit markdown files directly.

**📋 Task Quality**: Good AC = Testable outcomes, not implementation steps.

**✅ Definition of Done**: All AC checked + notes + tests passing + status Done.

**📝 Documentation Rule for AI**: Always use `backlog doc create` or `backlog decision create` for project documentation. NEVER create standalone .md files in the project root.
