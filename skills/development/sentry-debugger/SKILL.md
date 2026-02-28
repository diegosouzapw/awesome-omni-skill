---
name: sentry-debugger
description: Debug production issues using Sentry error tracking API. Use when Claude needs to investigate production errors, crashes, or issues by fetching error data from Sentry - including stack traces, user context, breadcrumbs, and error frequency. Triggers on requests like "check Sentry for errors", "debug this production issue", "what's causing crashes", "investigate errors in [project]", or when users share Sentry issue URLs.
---

# Sentry Debugger

Debug production issues by fetching and analyzing error data from Sentry's API.

## Prerequisites

- Sentry API auth token with `project:read`, `event:read`, `issue:read` scopes
- Organization slug and project slug(s)

## Authentication

Uses the same auth as `sentry-cli`. Priority order:
1. `~/.sentryclirc` file (INI format with `[auth]` section)
2. `SENTRY_AUTH_TOKEN` environment variable

To extract token from sentryclirc:
```bash
SENTRY_AUTH_TOKEN=$(grep -A1 '^\[auth\]' ~/.sentryclirc | grep '^token=' | cut -d'=' -f2)
```

Or use sentry-cli directly for auth verification:
```bash
sentry-cli info  # Verify auth is working
```

## Core Workflow

### 1. Fetch Recent Issues

```bash
# Get token from sentryclirc (same as sentry-cli)
SENTRY_TOKEN=$(grep -A1 '^\[auth\]' ~/.sentryclirc 2>/dev/null | grep '^token=' | cut -d'=' -f2)
SENTRY_TOKEN=${SENTRY_TOKEN:-$SENTRY_AUTH_TOKEN}

curl -s "https://sentry.io/api/0/projects/{org_slug}/{project_slug}/issues/?query=is:unresolved&statsPeriod=24h" \
  -H "Authorization: Bearer $SENTRY_TOKEN" | jq '.[] | {id, title, count, userCount, firstSeen, lastSeen}'
```

### 2. Get Issue Details with Latest Event

```bash
curl -s "https://sentry.io/api/0/issues/{issue_id}/events/latest/" \
  -H "Authorization: Bearer $SENTRY_TOKEN" | jq '{
    message: .message,
    timestamp: .dateCreated,
    user: .user,
    tags: .tags,
    contexts: .contexts,
    exception: .entries[] | select(.type == "exception"),
    breadcrumbs: .entries[] | select(.type == "breadcrumbs")
  }'
```

### 3. Analyze and Cross-Reference with Codebase

After fetching error data:
1. Extract file paths and line numbers from stack traces
2. Read the relevant source files
3. Understand the execution flow from breadcrumbs
4. Identify root cause and suggest fix

## API Reference

See [references/sentry-api.md](references/sentry-api.md) for complete endpoint documentation.

## Debugging Patterns

### Pattern 1: High-Frequency Errors
Query by event count to prioritize:
```bash
curl -s "https://sentry.io/api/0/projects/{org}/{project}/issues/?query=is:unresolved&sort=freq" \
  -H "Authorization: Bearer $SENTRY_TOKEN"
```

### Pattern 2: User-Reported Issues
Filter by specific user or search query:
```bash
curl -s "https://sentry.io/api/0/projects/{org}/{project}/issues/?query=user.email:user@example.com" \
  -H "Authorization: Bearer $SENTRY_TOKEN"
```

### Pattern 3: Specific Error Types
```bash
# JavaScript errors
curl -s "https://sentry.io/api/0/projects/{org}/{project}/issues/?query=error.type:TypeError"

# HTTP errors  
curl -s "https://sentry.io/api/0/projects/{org}/{project}/issues/?query=http.status_code:500"
```

### Pattern 4: Time-Based Investigation
```bash
# Last hour
?statsPeriod=1h

# Specific date range
?start=2024-01-01T00:00:00&end=2024-01-02T00:00:00
```

## Output Format

When presenting findings to user:

1. **Summary**: Error type, frequency, affected users
2. **Stack Trace Analysis**: Key frames with file:line references
3. **Context**: User info, browser/device, request data
4. **Breadcrumbs**: Sequence of events leading to error
5. **Root Cause**: Analysis based on code review
6. **Recommended Fix**: Code changes with rationale

## Tips

- Always check `contexts.browser` and `contexts.os` for environment-specific issues
- Breadcrumbs show the user journey - crucial for reproducing issues
- `tags` often contain custom business context (user tier, feature flags, etc.)
- For React/frontend: check `contexts.react` for component hierarchy
- Event `extra` field may contain custom debug data from the app