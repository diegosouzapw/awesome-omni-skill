---
name: jira-automation
description: Complete Jira automation toolkit using REST API - create, update, search, transition issues, manage sprints, add comments, link issues, and more
---

# Jira Automation Skill

## When to Use This Skill

Use this skill when you need to:

**Issue Management:**
- Create single or multiple Jira issues from specifications
- Update existing issues (fields, status, assignee, etc.)
- Search and filter issues using JQL (Jira Query Language)
- Get detailed issue information
- Delete issues (with caution)

**Workflow & Status:**
- Transition issues through workflow states (To Do → In Progress → Done)
- Get available transitions for an issue
- Track issue status across projects

**Collaboration:**
- Add comments to issues
- Mention users in comments with @username
- Update issue descriptions with findings

**Agile & Sprint Management:**
- Create and update sprints
- Link issues to epics
- Get board and sprint information
- Manage sprint issues and backlog

**Issue Relationships:**
- Link related issues (blocks, relates to, duplicates, etc.)
- Remove issue links
- Create parent-child relationships

**Project & Version Management:**
- Get all projects and their details
- Create and manage project versions
- Get project-specific issues

**Time Tracking:**
- Add work logs to issues
- Track time spent on tasks

## Prerequisites

Before using this skill, ensure you have:
- ✅ Jira API token generated (https://id.atlassian.com/manage-profile/security/api-tokens)
- ✅ Jira email/username for authentication
- ✅ Epic keys that exist in Jira
- ✅ Task definitions structured with required fields (summary, description)
- ✅ Python 3 and `requests` library installed

## Skill Workflow

### Step 1: Prepare Task Definitions

Structure your tasks as a Python dictionary with this format:

```python
EPIC_TASKS = {
    "EPIC-KEY": [
        {
            "summary": "Task title (required)",
            "description": "Task description (required)",
            "labels": ["label1", "label2"],  # optional
            "priority": "Major",  # optional: Highest, High, Medium, Low, Lowest
            "timeEstimate": "2h"  # optional: e.g., "30m", "2h", "1d"
        },
        # ... more tasks
    ]
}
```

**Required fields:**
- `summary`: Short task title
- `description`: Detailed task description (supports markdown)

**Optional fields:**
- `labels`: Array of label strings
- `priority`: Jira priority name (default: "Major")
- `timeEstimate`: Time estimate in Jira format (e.g., "30m", "2h", "1d")

### Step 2: Create the Python Script

Use this template as the base for your bulk creation script:

```python
#!/usr/bin/env python3
"""
Bulk create Jira tasks for epics.

Usage:
    python create-jira-tasks.py --email YOUR_EMAIL --api-token YOUR_API_TOKEN
"""

import argparse
import sys
import requests
from requests.auth import HTTPBasicAuth

# Jira configuration
JIRA_BASE_URL = "https://your-company.atlassian.net"
JIRA_API_URL = f"{JIRA_BASE_URL}/rest/api/3"
PROJECT_KEY = "YOUR_PROJECT_KEY"

# Task definitions
EPIC_TASKS = {
    # Add your epic tasks here
}

def create_jira_task(auth, epic_key, task):
    """Create a single Jira task under the specified epic."""
    payload = {
        "fields": {
            "project": {"key": PROJECT_KEY},
            "summary": task["summary"],
            "description": {
                "type": "doc",
                "version": 1,
                "content": [{
                    "type": "paragraph",
                    "content": [{"type": "text", "text": task["description"]}]
                }]
            },
            "issuetype": {"name": "Task"},
            "parent": {"key": epic_key},
            "labels": task.get("labels", []),
            "priority": {"name": task.get("priority", "Major")}
        }
    }

    if "timeEstimate" in task:
        payload["fields"]["timetracking"] = {
            "originalEstimate": task["timeEstimate"]
        }

    response = requests.post(
        f"{JIRA_API_URL}/issue",
        auth=auth,
        headers={"Content-Type": "application/json"},
        json=payload
    )

    if response.status_code == 201:
        result = response.json()
        print(f"[OK] Created {result['key']}: {task['summary']}")
        return result
    else:
        print(f"[ERROR] Failed to create task: {task['summary']}")
        print(f"   Status: {response.status_code}")
        print(f"   Response: {response.text}")
        return None

def main():
    parser = argparse.ArgumentParser(description="Bulk create Jira tasks")
    parser.add_argument("--email", required=True, help="Your Atlassian account email")
    parser.add_argument("--api-token", required=True, help="Your Atlassian API token")
    parser.add_argument("--epic", help="Specific epic to create tasks for")
    parser.add_argument("--dry-run", action="store_true", help="Print tasks without creating them")

    args = parser.parse_args()
    auth = HTTPBasicAuth(args.email, args.api_token)

    # Verify authentication
    response = requests.get(f"{JIRA_API_URL}/myself", auth=auth)
    if response.status_code != 200:
        print(f"[ERROR] Authentication failed: {response.status_code}")
        sys.exit(1)

    user_info = response.json()
    print(f"[OK] Authenticated as: {user_info['displayName']} ({user_info['emailAddress']})")
    print()

    # Determine which epics to process
    epics_to_process = [args.epic] if args.epic else EPIC_TASKS.keys()

    total_created = 0
    total_failed = 0

    for epic_key in epics_to_process:
        if epic_key not in EPIC_TASKS:
            print(f"[WARN] Unknown epic: {epic_key}")
            continue

        tasks = EPIC_TASKS[epic_key]
        print(f"[EPIC] {epic_key}: {len(tasks)} tasks")
        print()

        if args.dry_run:
            for task in tasks:
                print(f"   - {task['summary']}")
            print()
            continue

        for task in tasks:
            result = create_jira_task(auth, epic_key, task)
            if result:
                total_created += 1
            else:
                total_failed += 1

        print()

    if args.dry_run:
        print(f"[DRY-RUN] Complete - no tasks created")
    else:
        print(f"[SUMMARY] Created {total_created} tasks")
        if total_failed > 0:
            print(f"[SUMMARY] Failed to create {total_failed} tasks")

if __name__ == "__main__":
    main()
```

### Step 3: Windows Encoding Fix

**IMPORTANT FOR WINDOWS:** Remove emoji characters from print statements to avoid encoding errors.

Replace emojis with ASCII prefixes:
- ✅ → `[OK]`
- ❌ → `[ERROR]`
- ⚠️ → `[WARN]`
- 📋 → `[EPIC]`
- 🔍 → `[DRY-RUN]`

### Step 4: Execute the Script

**Dry run (preview only):**
```bash
python create-jira-tasks.py --email your.email@company.com --api-token YOUR_TOKEN --dry-run
```

**Create tasks for specific epic:**
```bash
python create-jira-tasks.py --email your.email@company.com --api-token YOUR_TOKEN --epic DP01-21
```

**Create all tasks:**
```bash
python create-jira-tasks.py --email your.email@company.com --api-token YOUR_TOKEN
```

## Best Practices

### 1. Always Dry Run First
Run with `--dry-run` to verify task definitions before creating them in Jira.

### 2. Test with One Epic
Use `--epic EPIC-KEY` to test with a single epic before bulk creating all tasks.

### 3. Validate Epic Keys Exist
Ensure all parent epic keys exist in Jira before running the script. The script will fail if parent epic doesn't exist.

### 4. Use Meaningful Labels
Add labels for filtering and organization:
- `Track-1`, `Track-2`, `Track-3` for project tracks
- `frontend`, `backend`, `database` for component types
- `Day-1-90`, `Day-91-180` for MVP phasing

### 5. Set Realistic Time Estimates
Use Jira time format:
- Minutes: `30m`, `45m`
- Hours: `1h`, `2h`, `4h`
- Days: `1d`, `2d` (1 day = 8 hours by default)

### 6. Structure Task Descriptions
Include in each task description:
- **Autonomy Level:** HIGH/MEDIUM/LOW (for Claude Code execution)
- **Claude Code Prompt:** Ready-to-use prompt for implementation
- **Deliverables:** Specific files or features to create
- **Validation:** How to verify the task is complete

### 7. Handle Errors Gracefully
The script continues even if individual tasks fail. Review the summary to identify failed tasks.

## Common Issues and Solutions

### Issue: Authentication Failed (401)
**Cause:** Wrong email or expired API token
**Solution:**
- Verify email matches your Atlassian account
- Generate new API token at https://id.atlassian.com/manage-profile/security/api-tokens
- For organization accounts (e.g., `vividcg.atlassian.net`), use your organization email

### Issue: Epic Not Found (404)
**Cause:** Parent epic key doesn't exist
**Solution:** Create the epic in Jira first, or verify the epic key is correct

### Issue: Unicode Encoding Error (Windows)
**Cause:** Emoji characters in print statements
**Solution:** Replace all emojis with ASCII equivalents (see Step 3)

### Issue: Time Estimate Not Set
**Cause:** Invalid time format
**Solution:** Use Jira format: `30m`, `2h`, `1d` (not `30 minutes`, `2 hours`)

### Issue: Tasks Created in Wrong Project
**Cause:** Incorrect `PROJECT_KEY`
**Solution:** Update `PROJECT_KEY` constant to match your Jira project

## Example: Epic Tasking Guide to Jira Tasks

This skill was used to create 74 tasks from the Epic Tasking Guide:

```bash
python scripts/create-jira-tasks.py \
  --email clay.campbell@vividcg.com \
  --api-token YOUR_TOKEN

# Output:
# [OK] Authenticated as: Clay Campbell (clay.campbell@vividcg.com)
#
# [EPIC] DP01-21: 11 tasks
# [OK] Created DP01-74: Configure AWS Organizations and account structure
# [OK] Created DP01-75: Setup IAM roles for CI/CD pipeline
# ...
#
# [SUMMARY] Created 74 tasks
```

## Customization Options

### Add Custom Fields

To add custom Jira fields, update the `payload["fields"]` dictionary:

```python
payload["fields"]["customfield_10001"] = "Custom value"
payload["fields"]["assignee"] = {"accountId": "user-account-id"}
payload["fields"]["duedate"] = "2025-12-31"
```

### Filter by Labels

Create tasks only for specific labels:

```python
if args.label_filter:
    tasks = [t for t in tasks if args.label_filter in t.get("labels", [])]
```

### Batch Creation with Rate Limiting

Add rate limiting for large batches:

```python
import time

for task in tasks:
    result = create_jira_task(auth, epic_key, task)
    time.sleep(0.5)  # 500ms delay between tasks
```

## Integration with Claude Code

When breaking down epics for Claude Code execution:

1. **Use this skill** to create the task structure
2. **Add autonomy levels** to descriptions (HIGH/MEDIUM/LOW)
3. **Include Claude Code prompts** in each task description
4. **Link related tasks** using Jira issue links
5. **Track progress** as Claude Code completes tasks

## Security Notes

- ⚠️ **Never commit API tokens** to version control
- ⚠️ **Use environment variables** for sensitive data
- ⚠️ **Rotate tokens regularly** (every 90 days recommended)
- ⚠️ **Restrict token permissions** to minimum required scope

## Success Criteria

You've successfully used this skill when:
- ✅ All tasks are created in Jira under correct epics
- ✅ Task descriptions include autonomy levels and prompts
- ✅ Labels and priorities are set correctly
- ✅ Time estimates are realistic and useful for planning
- ✅ No duplicate tasks were created
- ✅ Error handling caught and reported any failures

## Advanced Jira Operations

Beyond bulk task creation, this skill covers all Jira REST API operations. Here are code examples for common workflows:

### 1. Search Issues with JQL

```python
def search_issues(auth, jql, max_results=50):
    """Search Jira issues using JQL (Jira Query Language)."""
    params = {
        "jql": jql,
        "maxResults": max_results,
        "fields": "summary,status,assignee,priority,created"
    }

    response = requests.get(
        f"{JIRA_API_URL}/search",
        auth=auth,
        params=params
    )

    if response.status_code == 200:
        data = response.json()
        print(f"[OK] Found {data['total']} issues")
        for issue in data['issues']:
            print(f"  {issue['key']}: {issue['fields']['summary']}")
        return data['issues']
    else:
        print(f"[ERROR] Search failed: {response.status_code}")
        return []

# Example usage:
# search_issues(auth, "project = DP01 AND status = 'In Progress'")
# search_issues(auth, "assignee = currentUser() AND status != Done")
# search_issues(auth, "created >= -7d AND labels = Track-3-Platform")
```

### 2. Get Issue Details

```python
def get_issue(auth, issue_key):
    """Get complete details of a specific issue."""
    response = requests.get(
        f"{JIRA_API_URL}/issue/{issue_key}",
        auth=auth
    )

    if response.status_code == 200:
        issue = response.json()
        print(f"[OK] {issue_key}: {issue['fields']['summary']}")
        print(f"     Status: {issue['fields']['status']['name']}")
        print(f"     Assignee: {issue['fields']['assignee']['displayName'] if issue['fields']['assignee'] else 'Unassigned'}")
        return issue
    else:
        print(f"[ERROR] Failed to get issue: {response.status_code}")
        return None
```

### 3. Update Issue Fields

```python
def update_issue(auth, issue_key, fields):
    """Update fields on an existing issue."""
    payload = {"fields": fields}

    response = requests.put(
        f"{JIRA_API_URL}/issue/{issue_key}",
        auth=auth,
        headers={"Content-Type": "application/json"},
        json=payload
    )

    if response.status_code == 204:
        print(f"[OK] Updated {issue_key}")
        return True
    else:
        print(f"[ERROR] Failed to update: {response.status_code}")
        print(f"   Response: {response.text}")
        return False

# Example usage:
# update_issue(auth, "DP01-74", {
#     "summary": "New task title",
#     "description": {...},  # ADF format
#     "priority": {"name": "High"},
#     "labels": ["urgent", "backend"]
# })
```

### 4. Transition Issue (Change Status)

```python
def get_transitions(auth, issue_key):
    """Get available transitions for an issue."""
    response = requests.get(
        f"{JIRA_API_URL}/issue/{issue_key}/transitions",
        auth=auth
    )

    if response.status_code == 200:
        transitions = response.json()['transitions']
        print(f"[OK] Available transitions for {issue_key}:")
        for t in transitions:
            print(f"     {t['id']}: {t['name']}")
        return transitions
    else:
        print(f"[ERROR] Failed to get transitions: {response.status_code}")
        return []

def transition_issue(auth, issue_key, transition_id):
    """Transition an issue to a new status."""
    payload = {
        "transition": {"id": transition_id}
    }

    response = requests.post(
        f"{JIRA_API_URL}/issue/{issue_key}/transitions",
        auth=auth,
        headers={"Content-Type": "application/json"},
        json=payload
    )

    if response.status_code == 204:
        print(f"[OK] Transitioned {issue_key}")
        return True
    else:
        print(f"[ERROR] Failed to transition: {response.status_code}")
        return False

# Example workflow:
# transitions = get_transitions(auth, "DP01-74")
# # Find "In Progress" transition ID from the list
# transition_issue(auth, "DP01-74", "21")  # ID for "In Progress"
```

### 5. Add Comments

```python
def add_comment(auth, issue_key, comment_text):
    """Add a comment to an issue."""
    payload = {
        "body": {
            "type": "doc",
            "version": 1,
            "content": [{
                "type": "paragraph",
                "content": [{
                    "type": "text",
                    "text": comment_text
                }]
            }]
        }
    }

    response = requests.post(
        f"{JIRA_API_URL}/issue/{issue_key}/comment",
        auth=auth,
        headers={"Content-Type": "application/json"},
        json=payload
    )

    if response.status_code == 201:
        print(f"[OK] Added comment to {issue_key}")
        return True
    else:
        print(f"[ERROR] Failed to add comment: {response.status_code}")
        return False

# Example usage:
# add_comment(auth, "DP01-74", "Implementation started. Setting up AWS Organizations.")
```

### 6. Link Issues

```python
def link_issues(auth, inward_issue, outward_issue, link_type="Relates"):
    """Create a link between two issues."""
    payload = {
        "type": {"name": link_type},  # "Relates", "Blocks", "Duplicate", etc.
        "inwardIssue": {"key": inward_issue},
        "outwardIssue": {"key": outward_issue}
    }

    response = requests.post(
        f"{JIRA_API_URL}/issueLink",
        auth=auth,
        headers={"Content-Type": "application/json"},
        json=payload
    )

    if response.status_code == 201:
        print(f"[OK] Linked {inward_issue} {link_type} {outward_issue}")
        return True
    else:
        print(f"[ERROR] Failed to link issues: {response.status_code}")
        return False

# Example usage:
# link_issues(auth, "DP01-74", "DP01-75", "Blocks")  # DP01-74 blocks DP01-75
# link_issues(auth, "DP01-85", "DP01-86", "Relates")  # DP01-85 relates to DP01-86
```

### 7. Link Issue to Epic

```python
def link_to_epic(auth, issue_key, epic_key):
    """Link an issue to an epic (parent)."""
    payload = {
        "fields": {
            "parent": {"key": epic_key}
        }
    }

    response = requests.put(
        f"{JIRA_API_URL}/issue/{issue_key}",
        auth=auth,
        headers={"Content-Type": "application/json"},
        json=payload
    )

    if response.status_code == 204:
        print(f"[OK] Linked {issue_key} to epic {epic_key}")
        return True
    else:
        print(f"[ERROR] Failed to link to epic: {response.status_code}")
        return False
```

### 8. Add Work Log (Time Tracking)

```python
def add_worklog(auth, issue_key, time_spent, comment=""):
    """Log time spent on an issue."""
    payload = {
        "timeSpent": time_spent,  # e.g., "3h 30m", "1d", "45m"
        "comment": {
            "type": "doc",
            "version": 1,
            "content": [{
                "type": "paragraph",
                "content": [{
                    "type": "text",
                    "text": comment
                }]
            }]
        } if comment else None
    }

    response = requests.post(
        f"{JIRA_API_URL}/issue/{issue_key}/worklog",
        auth=auth,
        headers={"Content-Type": "application/json"},
        json=payload
    )

    if response.status_code == 201:
        print(f"[OK] Logged {time_spent} on {issue_key}")
        return True
    else:
        print(f"[ERROR] Failed to log work: {response.status_code}")
        return False

# Example usage:
# add_worklog(auth, "DP01-74", "2h 30m", "Configured AWS Organizations")
```

### 9. Get All Projects

```python
def get_all_projects(auth):
    """Get list of all accessible Jira projects."""
    response = requests.get(
        f"{JIRA_API_URL}/project",
        auth=auth
    )

    if response.status_code == 200:
        projects = response.json()
        print(f"[OK] Found {len(projects)} projects:")
        for project in projects:
            print(f"     {project['key']}: {project['name']}")
        return projects
    else:
        print(f"[ERROR] Failed to get projects: {response.status_code}")
        return []
```

### 10. Batch Create Issues

```python
def batch_create_issues(auth, issues):
    """Create multiple issues in a single API call."""
    payload = {
        "issueUpdates": [
            {"fields": issue} for issue in issues
        ]
    }

    response = requests.post(
        f"{JIRA_API_URL}/issue/bulk",
        auth=auth,
        headers={"Content-Type": "application/json"},
        json=payload
    )

    if response.status_code == 201:
        results = response.json()
        print(f"[OK] Created {len(results['issues'])} issues")
        for issue in results['issues']:
            print(f"     {issue['key']}")
        return results
    else:
        print(f"[ERROR] Batch creation failed: {response.status_code}")
        return None

# Example usage:
# batch_create_issues(auth, [
#     {
#         "project": {"key": "DP01"},
#         "summary": "Task 1",
#         "issuetype": {"name": "Task"},
#         "parent": {"key": "DP01-21"}
#     },
#     {
#         "project": {"key": "DP01"},
#         "summary": "Task 2",
#         "issuetype": {"name": "Task"},
#         "parent": {"key": "DP01-21"}
#     }
# ])
```

### 11. Create Sprint

```python
def create_sprint(auth, board_id, sprint_name, start_date=None, end_date=None):
    """Create a new sprint for an agile board."""
    payload = {
        "name": sprint_name,
        "originBoardId": board_id,
    }

    if start_date:
        payload["startDate"] = start_date  # ISO 8601 format: "2025-01-20T10:00:00.000Z"
    if end_date:
        payload["endDate"] = end_date

    response = requests.post(
        f"{JIRA_BASE_URL}/rest/agile/1.0/sprint",
        auth=auth,
        headers={"Content-Type": "application/json"},
        json=payload
    )

    if response.status_code == 201:
        sprint = response.json()
        print(f"[OK] Created sprint: {sprint['name']} (ID: {sprint['id']})")
        return sprint
    else:
        print(f"[ERROR] Failed to create sprint: {response.status_code}")
        return None
```

### 12. Get Board Issues

```python
def get_board_issues(auth, board_id, jql_filter=""):
    """Get issues from a specific agile board."""
    params = {"maxResults": 100}
    if jql_filter:
        params["jql"] = jql_filter

    response = requests.get(
        f"{JIRA_BASE_URL}/rest/agile/1.0/board/{board_id}/issue",
        auth=auth,
        params=params
    )

    if response.status_code == 200:
        data = response.json()
        print(f"[OK] Found {data['total']} issues on board")
        return data['issues']
    else:
        print(f"[ERROR] Failed to get board issues: {response.status_code}")
        return []
```

## Common JQL Query Examples

Jira Query Language (JQL) is powerful for filtering issues. Here are common queries:

```python
# Issues in specific project
"project = DP01"

# Issues assigned to you
"assignee = currentUser()"

# Issues in progress
"project = DP01 AND status = 'In Progress'"

# Recent issues (last 7 days)
"created >= -7d"

# High priority bugs
"project = DP01 AND issuetype = Bug AND priority in (High, Highest)"

# Issues with specific label
"labels = Track-3-Platform"

# Overdue issues
"duedate < now() AND status != Done"

# Issues updated recently
"updated >= -1d"

# Complex query with multiple conditions
"project = DP01 AND assignee = currentUser() AND status in ('To Do', 'In Progress') AND labels = urgent ORDER BY priority DESC"

# Epic and its children
"'Epic Link' = DP01-21"

# Unassigned issues in current sprint
"sprint in openSprints() AND assignee is EMPTY"

# Issues blocking others
"issueFunction in linkedIssuesOf('project = DP01', 'blocks')"
```

## Integration with Claude Code Workflows

### Automated Task Status Updates

When Claude Code completes a task, automatically update Jira:

```python
def complete_claude_task(auth, issue_key, time_spent, implementation_notes):
    """Mark a Claude Code task as complete in Jira."""
    # 1. Add work log
    add_worklog(auth, issue_key, time_spent, "Implementation completed by Claude Code")

    # 2. Add comment with results
    add_comment(auth, issue_key, f"Implementation complete.\n\n{implementation_notes}")

    # 3. Get available transitions
    transitions = get_transitions(auth, issue_key)
    done_transition = next((t for t in transitions if t['name'].lower() == 'done'), None)

    # 4. Transition to Done
    if done_transition:
        transition_issue(auth, issue_key, done_transition['id'])
        print(f"[OK] Task {issue_key} marked as complete")
    else:
        print(f"[WARN] Could not find 'Done' transition for {issue_key}")
```

### Sprint Planning Automation

Automate sprint creation and issue assignment:

```python
def setup_sprint(auth, board_id, sprint_name, epic_key, num_days=14):
    """Create sprint and add epic issues to it."""
    from datetime import datetime, timedelta

    # 1. Create sprint
    start_date = datetime.now().isoformat() + "Z"
    end_date = (datetime.now() + timedelta(days=num_days)).isoformat() + "Z"
    sprint = create_sprint(auth, board_id, sprint_name, start_date, end_date)

    if not sprint:
        return

    # 2. Get issues from epic
    jql = f"'Epic Link' = {epic_key} AND status = 'To Do'"
    issues = search_issues(auth, jql)

    # 3. Move issues to sprint
    for issue in issues:
        move_to_sprint(auth, issue['key'], sprint['id'])

    print(f"[OK] Sprint '{sprint_name}' created with {len(issues)} issues")
```

## Related Skills

- **brainstorming** - Use before this skill to refine epic breakdown
- **test-driven-development** - Use after task creation for implementation
- **finishing-a-development-branch** - Use when completing tasks

## Version History

- **v2.0** (2025-01-14): Expanded to full Jira automation toolkit
  - Added all MCP-equivalent operations (search, update, transition, comments, links)
  - Sprint and board management
  - Work logging and time tracking
  - Issue relationships and linking
  - JQL query examples
  - Claude Code workflow integration patterns

- **v1.0** (2025-01-14): Initial skill creation
  - Basic task creation workflow
  - Windows encoding fixes
  - Dry-run and single-epic modes
  - Error handling and validation
