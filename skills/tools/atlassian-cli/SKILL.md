---
name: atlassian-cli
description: Execute Atlassian CLI (acli) commands for Jira work items, projects, sprints, boards, and organization administration. Use when the user wants to manage Jira issues, projects, sprints, or perform Atlassian organization admin tasks. Covers both solo developer workflows and team collaboration patterns.
---

# Atlassian CLI (acli) Skill

This skill provides comprehensive Atlassian CLI integration for managing Jira work items, projects, sprints, boards, and organization administration through the `acli` command-line tool.

## Prerequisites

- Atlassian CLI (`acli`) must be installed and authenticated
- Active internet connection for Atlassian API access
- Appropriate permissions for your Atlassian organization/site
- API token or OAuth credentials

## Quick Command Reference

### Authentication Commands

```bash
# Jira authentication
acli jira auth login --site "mysite.atlassian.net" --email "user@example.com" --token
acli jira auth status                        # Check authentication status
acli jira auth switch                        # Switch between accounts
acli jira auth logout                        # Logout

# Admin authentication (for org management)
acli admin auth login
acli admin auth status
acli admin auth logout

# OAuth authentication (browser-based)
acli jira auth login --web
```

### Work Item Operations

```bash
# Create work items
acli jira workitem create --summary "Task title" --project "PROJ" --type "Task"
acli jira workitem create-bulk               # Bulk create from file

# View and search
acli jira workitem view --key "PROJ-123"
acli jira workitem search --jql "project = PROJ AND status = 'In Progress'"

# Edit and update
acli jira workitem edit --key "PROJ-123" --summary "Updated title"
acli jira workitem assign --key "PROJ-123" --assignee "user@example.com"
acli jira workitem transition --key "PROJ-123" --status "Done"

# Manage relationships
acli jira workitem link --inward "PROJ-123" --outward "PROJ-456" --type "blocks"
acli jira workitem clone --key "PROJ-123"

# Comments and attachments
acli jira workitem comment --key "PROJ-123" --body "Comment text"
acli jira workitem attachment --key "PROJ-123" --file "/path/to/file.pdf"

# Archive operations
acli jira workitem archive --key "PROJ-123"
acli jira workitem unarchive --key "PROJ-123"

# Delete
acli jira workitem delete --key "PROJ-123"
```

### Project Operations

```bash
# List and view projects
acli jira project list                       # List all visible projects
acli jira project view --key "PROJ"          # View project details

# Create and manage
acli jira project create --name "New Project" --key "NEWP" --type "software"
acli jira project update --key "PROJ" --name "Updated Name"

# Archive and restore
acli jira project archive --key "PROJ"
acli jira project restore --key "PROJ"
acli jira project delete --key "PROJ"
```

### Sprint and Board Operations

```bash
# Board management
acli jira board search --query "Team Board"
acli jira board list-sprints --board-id "123"

# Sprint operations
acli jira sprint list-workitems --sprint-id "456"
```

### Filter Operations

```bash
# Manage filters
acli jira filter list                        # List my filters
acli jira filter search --query "Open Issues"
acli jira filter add-favourite --id "12345"
acli jira filter change-owner --id "12345" --owner "user@example.com"
```

### User Administration (Admin)

```bash
# User management
acli admin user activate --email "user@example.com"
acli admin user deactivate --email "user@example.com"
acli admin user delete --email "user@example.com"
acli admin user cancel-delete --email "user@example.com"
```

### Custom Fields

```bash
# Field management
acli jira field create --name "Custom Field" --type "text"
acli jira field delete --id "customfield_12345"
```

## Command Execution Pattern

When executing acli commands:

1. **Check authentication**: Verify you're authenticated with `acli jira auth status`
2. **Use appropriate flags**: Add `--output json` for structured output when parsing is needed
3. **Handle errors gracefully**: Parse stderr and provide clear error messages
4. **Confirm destructive actions**: Always confirm before deleting, archiving, or removing access
5. **Batch operations**: Use bulk commands for multiple items to improve efficiency

Example execution:
```bash
# Get JSON output for parsing
acli jira workitem search --jql "project = PROJ" --output json

# Specify required parameters
acli jira workitem create --summary "New task" --project "PROJ" --type "Task"

# Bulk operations with preview
acli jira workitem create-bulk --file issues.csv --preview
```

## Detailed Guides (Load as Needed)

For comprehensive workflows and advanced usage, refer to these detailed guides:

### Authentication and Setup
See [AUTH.md](AUTH.md) for:
- Installing acli on different platforms
- Setting up authentication (API token, OAuth)
- Managing multiple accounts
- Configuration and troubleshooting

### Work Item Management
See [WORKITEM.md](WORKITEM.md) for:
- Creating and editing work items (issues)
- Searching with JQL
- Transitions and workflows
- Comments, attachments, and watchers
- Linking and cloning issues
- Bulk operations

### Project Management
See [PROJECT.md](PROJECT.md) for:
- Creating and configuring projects
- Project settings and administration
- Archiving and restoring projects
- Project permissions and roles

### Sprint and Board Management
See [SPRINT.md](SPRINT.md) for:
- Managing agile boards
- Sprint planning and execution
- Viewing sprint work items
- Board configuration

### Team Collaboration
See [TEAM.md](TEAM.md) for:
- Multi-developer workflows
- Sprint planning with acli
- Team permission management
- Bulk operations for teams
- Code review integration

### Solo Developer Workflows
See [SOLO-DEV.md](SOLO-DEV.md) for:
- Personal task management
- Quick issue creation patterns
- Efficient solo development with acli
- Personal automation tips

## Helper Scripts

The `scripts/` directory contains helper scripts for common operations:

- `create-issue.sh` - Quick issue creation workflow
- `sprint-report.sh` - Generate sprint reports
- `bulk-assign.sh` - Bulk assign issues to team members
- `daily-standup.sh` - Generate daily standup summaries

Execute scripts as needed:
```bash
bash scripts/create-issue.sh "Fix bug in login"
```

## Best Practices

### 1. Always verify authentication
```bash
# Check authentication status
acli jira auth status

# Login if needed
echo $JIRA_API_TOKEN | acli jira auth login --site "mysite.atlassian.net" --email "user@example.com" --token
```

### 2. Use structured output for automation
```bash
# JSON output for parsing
acli jira workitem search --jql "project = PROJ" --output json | jq '.issues[] | .key'

# CSV output for spreadsheets
acli jira workitem search --jql "project = PROJ" --output csv
```

### 3. Use JQL effectively
```bash
# Find issues assigned to you
acli jira workitem search --jql "assignee = currentUser() AND status != Done"

# Find issues updated this week
acli jira workitem search --jql "project = PROJ AND updated >= -7d"

# Complex queries
acli jira workitem search --jql "project = PROJ AND status = 'In Progress' AND assignee = currentUser() ORDER BY priority DESC"
```

### 4. Batch operations for efficiency
```bash
# Edit multiple issues at once
acli jira workitem edit --key "PROJ-1,PROJ-2,PROJ-3" --label "urgent"

# Bulk create from CSV
acli jira workitem create-bulk --file issues.csv
```

## Error Handling

Common errors and solutions:

**Authentication errors**:
```bash
# Check authentication status
acli jira auth status

# Re-authenticate
echo $JIRA_API_TOKEN | acli jira auth login --site "mysite.atlassian.net" --email "user@example.com" --token
```

**Permission errors**:
- Verify you have appropriate access to the project/issue
- Check if you're using the correct authentication method (API token vs OAuth)
- Ensure your API token has the required scopes

**Invalid JQL errors**:
- Validate JQL syntax in Jira web UI first
- Use quotes around field values with spaces
- Check field names are correct

## Output Formatting

The acli CLI supports multiple output formats:

```bash
# JSON output (best for parsing)
acli jira workitem view --key "PROJ-123" --output json

# Table output (human-readable)
acli jira workitem search --jql "project = PROJ" --output table

# CSV output (for spreadsheets)
acli jira project list --output csv
```

## Tips and Tricks

1. **Environment variables**: Store credentials in environment variables
   ```bash
   export JIRA_SITE="mysite.atlassian.net"
   export JIRA_EMAIL="user@example.com"
   export JIRA_API_TOKEN="your-token-here"
   ```

2. **Shell aliases**: Create shortcuts for common commands
   ```bash
   alias jira-mine='acli jira workitem search --jql "assignee = currentUser() AND status != Done"'
   alias jira-create='acli jira workitem create --project PROJ --type Task'
   ```

3. **Piping and chaining**: Combine with other tools
   ```bash
   # Export to file
   acli jira workitem search --jql "project = PROJ" --output csv > issues.csv

   # Parse with jq
   acli jira workitem view --key "PROJ-123" --output json | jq '.fields.summary'
   ```

4. **Preview mode**: Use preview for bulk operations
   ```bash
   acli jira workitem create-bulk --file issues.csv --preview
   ```

## When to Use This Skill

Use this skill when the user wants to:
- Create or manage Jira work items (issues, tasks, bugs, stories)
- Manage Jira projects
- Work with sprints and agile boards
- Perform organization administration tasks
- Automate Jira workflows
- Generate reports from Jira data
- Bulk operations on issues
- Integrate Jira with CI/CD pipelines
- Follow team collaboration patterns
- Implement solo developer workflows

## Next Steps

Based on the user's request, load the appropriate detailed guide:
- Authentication and setup → Load AUTH.md
- Work item operations → Load WORKITEM.md
- Project management → Load PROJECT.md
- Sprint/board operations → Load SPRINT.md
- Team collaboration → Load TEAM.md
- Solo development → Load SOLO-DEV.md
