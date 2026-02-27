---
name: linear
description: Manage Linear issues, projects, and cycles. Maintains session memory across conversations for context continuity.
---

# Linear Skill

Manage your Linear workspace from the command line with persistent session memory.

---

## Project Configuration

**Before using this skill, read `config.json`** for project-specific values:

- `team` - The team key to use (e.g., "WIL")
- `defaultProject` - Default project ID for new issues
- `projects` - Named project shortcuts with IDs and descriptions
- `priorityContext` - What priority levels mean for this project's current phase
- `labels` - Label name to ID mappings
- `workflows` - Default states/priorities for different issue types

Use these values instead of the generic `TEAM` placeholders in examples below.

---

## Project Conventions

This section documents agreed-upon conventions for using Linear. Customize these for your project.

### Priority Definitions

| Priority | Value | Meaning | Usage |
|----------|-------|---------|-------|
| **Urgent** | 1 | Hot fix to production | Blocking work on production, drop everything |
| **High** | 2 | Critical for current release | Required before next production deployment |
| **Medium** | 3 | Standard priority | Scheduled alongside feature work |
| **Low** | 4 | Nice-to-have, deferred | Revisit when ahead of schedule |
| None | 0 | Not yet prioritized | Needs review |

**Key behaviors:**
- **Urgent**: Developer patches immediately; stakeholders notified
- **High**: Must be completed before production deployment
- **Medium**: Scheduled alongside feature work
- **Low**: Kept in backlog, won't delay deadlines

### Status Workflow

```
┌─────────────────┐     ┌─────────────┐
│  CLIENT REVIEW  │     │   TRIAGE    │
│  (Client Inbox) │     │ (Dev Inbox) │
└────────┬────────┘     └──────┬──────┘
         │                     │
         └──────────┬──────────┘
                    ▼
              ┌───────────┐
              │   TO DO   │
              └─────┬─────┘
                    ▼
              ┌───────────┐
              │IN PROGRESS│
              └─────┬─────┘
                    ▼
              ┌───────────┐
              │  DEV DONE │
              └─────┬─────┘
                    ▼
            ┌─────────────┐
            │READY TO TEST│
            └──────┬──────┘
                   ▼
              ┌──────────┐
              │   DONE   │
              └──────────┘
```

| Status | Owner | Meaning |
|--------|-------|---------|
| **Client Review** | Client | Developer-created tickets awaiting client approval of priority/details |
| **Triage** | Developer | Client-reported bugs/issues awaiting developer review |
| **To Do** | Both | Approved, prioritized, ready to be worked on |
| **In Progress** | Developer | Actively being developed |
| **Dev Done** | Developer | Code complete (PR merged), not yet deployed to staging |
| **Ready to Test** | Client | Deployed to staging, awaiting client testing |
| **Done** | Both | Client tested and approved |
| **Canceled** | Both | Work determined to be unnecessary |

**Transition responsibilities:**
- `→ Client Review`: Developer creates ticket from planning/meetings
- `Client Review → To Do`: Client approves after reviewing
- `→ Triage`: Client reports bug directly
- `Triage → To Do`: Developer reviews, sets priority
- `To Do → In Progress`: Developer picks up work
- `In Progress → Dev Done`: Code complete
- `Dev Done → Ready to Test`: Deployed to staging
- `Ready to Test → Done`: Client tests successfully

**Inbox goal:** Both `Client Review` and `Triage` should trend toward empty.

### Projects

Organize work into projects based on your needs:

| Project Type | Purpose |
|--------------|---------|
| **Support & Maintenance** | Bug fixes, polish, production support |
| **Feature Development** | New feature work by phase or milestone |
| **Infrastructure** | Technical improvements, refactoring |

**Assignment rules:**
- Production bugs → Support project
- New features → Appropriate feature project
- Urgent hot fixes → May skip project assignment (priority is enough)

### Labels

| Label | Usage |
|-------|-------|
| `bug` | Something broken that needs fixing |
| `feature` | New functionality |
| `improvement` | Enhancement to existing functionality |

### Communication

- **Linear is the primary channel** for project work
- Use **comments** within tickets for discussion
- **@ mentions** notify via inbox
- Check inbox regularly for messages

### Creating Issues from Transcripts

When generating issues from meeting transcripts:

1. **Set status to `Client Review`** (not Triage) - Client needs to approve
2. **Include in description:**
   - Context from discussion
   - Location in app (if applicable)
   - Steps to reproduce (for bugs)
   - Acceptance criteria (for features)
3. **Set priority** based on meeting discussion
4. **Assign to project** based on type
5. **Add appropriate label** (bug/feature/improvement)

### Checking Your Inbox

Your Linear inbox contains notifications about issues you're assigned to, mentioned in, or have subscribed to. This is how client feedback and updates reach you.

**Commands:**
```bash
# View all notifications (most recent first)
npx linear-skill inbox

# View only unread notifications
npx linear-skill inbox --unread

# Limit to specific count
npx linear-skill inbox --unread --limit 10
```

**Notification types include:**
- `issueAssignedToYou` - You were assigned to an issue
- `issueNewComment` - Someone commented on an issue you follow
- `issueMention` - You were @mentioned in a comment
- `issueStatusChanged` - Status changed on an issue you follow
- And many more...

**Workflow tip:** Check your inbox at the start of each session to see client feedback and updates since your last session. This is especially useful for catching comments on recently completed work.

### Reading Comments & Client Feedback

Comments contain important client feedback, clarifications, and discussion history. Use them strategically to stay context-efficient.

**When to use `--comments` flag or `comments` command:**
- Planning implementation for specific tickets (1-5 tickets)
- Reviewing client feedback before starting work
- Checking for clarifications or scope changes
- Preparing spec documents that need client context

**When NOT to fetch comments:**
- Broad searches across many tickets (`issues --team TEAM`)
- Filtering/triaging tickets by status or priority
- Quick status checks or updates
- Bulk operations

**Commands:**
```bash
# Lightweight - just comments for quick review
npx linear-skill comments TEAM-123

# Full context - issue details + comments for implementation planning
npx linear-skill issue TEAM-123 --comments
```

**Workflow tip:** When moving tickets from `To Do` to `In Progress`, fetch comments first to ensure you have the latest client context before writing implementation specs.

### Viewing Screenshots & Attachments

Issues often include screenshots showing bugs, UI states, or expected behavior. The skill returns **signed URLs** that are publicly accessible for 1 hour.

**Output fields when using `--comments`:**
- `inlineImages` - Images embedded in description or comments (extracted from markdown `![alt](url)`)
- `attachments` - Files attached to the issue

**Workflow to view screenshots:**

1. **Fetch the issue with comments** to get signed URLs:
   ```bash
   npx linear-skill issue TEAM-123 --comments
   ```

2. **Download images** to the skill's cache directory:
   ```bash
   # Create cache directory if needed
   mkdir -p ~/.linear-skill/images

   # Download each image (use the signed URL from output)
   curl -s -o ~/.linear-skill/images/screenshot-1.png "<signed-url>"
   ```

3. **View the image** using the Read tool:
   ```bash
   # Claude can view the downloaded image
   Read ~/.linear-skill/images/screenshot-1.png
   ```

4. **Clean up** after reviewing (images are not auto-deleted):
   ```bash
   rm ~/.linear-skill/images/*.png
   ```

**Important notes:**
- Signed URLs expire after 1 hour - re-fetch the issue if URLs have expired
- macOS does NOT auto-clean `/tmp`, so use `~/.linear-skill/images/` for predictable cleanup
- Always review screenshots before implementing bug fixes to understand the exact issue
- Multiple screenshots may show different states (before/after, different views, etc.)

### Release Workflow

**For staging → production releases:**

1. All `High` priority items must be `Done`
2. `Medium` items are optional for current release
3. `Low` items are explicitly deferred
4. Once all required items are `Done`, deploy to production

**Semantic versioning** may be adopted for tracking releases.

---

## Starting a Session

**Always load context first** to see what you were working on:

```bash
npx linear-skill context
```

This shows:
- Recent session summaries
- Focus tickets from last session
- Recent actions taken
- Any notes you left

## Quick Start

```bash
# Setup (one time)
cp .env.example .env
cp config.example.json config.json
# Edit .env with your LINEAR_API_KEY
npm install && npm run build

# Load previous context
npx linear-skill context

# List your assigned issues
npx linear-skill issues --assignee me

# Create an issue (for transcript-generated tickets, use Client Review state)
npx linear-skill create "Fix login bug" --team TEAM --state "Client Review"

# Update an issue
npx linear-skill update TEAM-123 --state "In Progress"

# Quick actions
npx linear-skill start TEAM-123    # Move to "In Progress"
npx linear-skill done TEAM-123     # Move to "Done"

# Mark tickets as current focus
npx linear-skill focus TEAM-123 TEAM-456

# Save session before ending
npx linear-skill session-save "Completed auth work, PR ready for review"
```

## Session Memory

The skill remembers your recent work across conversations:

- **Auto-tracking**: All issue interactions are logged automatically
- **Focus tickets**: Mark what you're actively working on
- **Session summaries**: 2-3 recent session summaries are kept
- **Context command**: Loads this into new conversations

## Command Categories

### Read Commands
- `teams` - List all teams
- `users` - List users (use `--me` for current user)
- `states --team TEAM` - List workflow states
- `projects` - List projects
- `cycles --team TEAM` - List cycles (`--active` for current)
- `issues` - List issues with filters
- `issue TEAM-123` - Get issue details
- `issue TEAM-123 --comments` - Get issue details with comments
- `comments TEAM-123` - Get comments for an issue
- `search "query"` - Search issues
- `inbox` - List notifications from your inbox
- `inbox --unread` - List only unread notifications

### Write Commands
- `create "title" --team TEAM` - Create issue
- `update TEAM-123 --state "Done"` - Update issue
- `assign TEAM-123 me` - Assign issue
- `comment TEAM-123 "comment text"` - Add comment
- `start TEAM-123` - Move to "In Progress"
- `done TEAM-123` - Move to "Done"

### Session Commands
- `context` - Load session context (use at start)
- `focus TEAM-123 TEAM-456` - Set focus tickets
- `notes "reminder text"` - Set session notes
- `session-save "summary"` - Save session summary
- `sessions` - View session history
- `session-clear --older-than 7` - Clear old data

### Config Commands
- `init` - Initialize configuration
- `config` - Show current config
- `cache-refresh` - Refresh teams/users/states cache
- `cache-status` - Show cache status

## Issue Filtering

The `issues` command supports flexible filtering with multiple values for most parameters.

### Available Filter Options

| Option | Alias | Description | Multiple Values |
|--------|-------|-------------|-----------------|
| `--team` | `-t` | Filter by team key or ID | No |
| `--project` | `-p` | Filter by project ID(s) | Yes (comma-separated) |
| `--cycle` | `-c` | Filter by cycle ID | No |
| `--assignee` | `-a` | Filter by assignee (ID, email, or "me") | No |
| `--state` | `-s` | Filter by state name(s) | Yes (repeat flag or comma) |
| `--priority` | `-P` | Filter by priority (1-4 or names) | Yes (repeat flag or comma) |
| `--label` | `-L` | Filter by label name(s) | Yes (repeat flag) |
| `--limit` | `-l` | Maximum issues to return (default: 50) | No |

### Priority Values

| Value | Name | Meaning |
|-------|------|---------|
| 1 | urgent | Hot fix to production |
| 2 | high | Critical for current release |
| 3 | medium | Standard priority |
| 4 | low | Nice-to-have |
| 0 | none | Not yet prioritized |

### Filtering Examples

```bash
# Basic filters
npx linear-skill issues --team TEAM
npx linear-skill issues --assignee me
npx linear-skill issues --state "To Do"

# Priority filtering (CRITICAL for hotfix workflow)
npx linear-skill issues --team TEAM --priority 1              # Urgent only
npx linear-skill issues --team TEAM --priority urgent         # Same as above
npx linear-skill issues --team TEAM --priority 1,2            # Urgent and High
npx linear-skill issues --team TEAM -P urgent -P high         # Same as above

# Multiple states
npx linear-skill issues --team TEAM -s "To Do" -s "In Progress"
npx linear-skill issues --state "Triage,Client Review"        # Comma syntax

# Label filtering (matches issues with ANY of the labels)
npx linear-skill issues --team TEAM --label "Bug"
npx linear-skill issues --team TEAM -L "Bug" -L "Feature"

# Combined filters
npx linear-skill issues --team TEAM --priority 1 --state "To Do"
npx linear-skill issues --team TEAM --assignee me --priority 1,2 --limit 20

# Find all inbox items (Triage + Client Review)
npx linear-skill issues --team TEAM -s "Triage" -s "Client Review"

# Production hotfix query
npx linear-skill issues --team TEAM --priority urgent --state "To Do"
```

## Bulk Operations for Transcript Processing

When creating multiple issues from a meeting transcript:

```bash
# Create each ticket with Client Review status
npx linear-skill create "Ticket title" --team TEAM --state "Client Review" --priority 2 -d "Description here"

# After client reviews, they move approved tickets to To Do
npx linear-skill update TEAM-123 --state "To Do"
```

## Setup

1. Copy `.env.example` to `.env`
2. Add your Linear API key: `LINEAR_API_KEY=lin_api_xxxx`
3. Copy `config.example.json` to `config.json`
4. Set your default team in `config.json`
5. Run `npm install && npm run build`
6. Test: `npx linear-skill teams`

## Skill Initialization

If this skill has not been configured for the current project, help the user set it up:

### Check if initialized

```bash
npx linear-skill config
```

If `team` is "YOUR_TEAM_KEY" or missing, initialization is needed.

### Initialization steps

1. **Verify `.env`** has `LINEAR_API_KEY` set

2. **Discover workspace:**
   ```bash
   npx linear-skill teams      # Get team key
   npx linear-skill projects   # Get project IDs and names
   npx linear-skill states --team TEAM_KEY  # Check workflow states
   ```

3. **Ask the user:**
   - Which team to use as default?
   - How to categorize their projects (support, feature phases)?
   - What does "High priority" mean for their current work?
   - What is their current project phase?

4. **Update `config.json`** with discovered values

5. **Check for missing workflow states** (Client Review, Dev Done, Ready to Test) and guide user to create them in Linear UI

6. **Refresh cache:**
   ```bash
   npx linear-skill cache-refresh
   ```

## Documentation

- [README.md](./README.md) - Developer setup guide
- [REFERENCE.md](./skill/REFERENCE.md) - Full command reference
- [CONFIG.md](./skill/CONFIG.md) - Configuration guide
- [SESSION.md](./skill/SESSION.md) - Session memory details
