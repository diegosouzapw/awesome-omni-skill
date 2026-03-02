---
name: ashby-api-guide
description: This skill should be used when the user asks about "Ashby API", "how to use Ashby tools", "Ashby authentication", "Ashby MCP tools", "what can I do with Ashby", or needs help understanding available Ashby operations. Provides complete API documentation and tool usage guidance.
version: 0.1.0
---

# Ashby API Guide

Reference guide for working with the Ashby ATS integration. This skill covers authentication, available tools, and common usage patterns.

## Overview

The Ashby plugin provides ~30 MCP tools for interacting with Ashby's Applicant Tracking System (ATS). All operations use Ashby's RPC-style API where every endpoint accepts POST requests.

## Authentication

Ashby uses Basic Authentication with an API key:

1. Generate an API key in Ashby: Settings → API Keys
2. Set the environment variable: `ASHBY_API_KEY=your-api-key`
3. The MCP server handles authentication automatically

API keys have permission scopes. Common permissions needed:
- `candidatesRead` / `candidatesWrite` - Candidate operations
- `jobsRead` / `jobsWrite` - Job operations
- `interviewsWrite` - Interview scheduling

## Available Tools

### Candidate Management

| Tool | Purpose | Required Params |
|------|---------|-----------------|
| `candidate_create` | Create new candidate | name, email |
| `candidate_search` | Find by email/name | email or name |
| `candidate_list` | List all candidates | (optional) cursor, limit |
| `candidate_info` | Get candidate details | candidateId |
| `candidate_update` | Update candidate | candidateId |
| `candidate_add_note` | Add note to profile | candidateId, note |
| `candidate_add_tag` | Tag a candidate | candidateId, tagId |
| `candidate_list_notes` | View all notes | candidateId |

### Job Management

| Tool | Purpose | Required Params |
|------|---------|-----------------|
| `job_create` | Create new job | title |
| `job_search` | Find jobs | (optional) title, status |
| `job_list` | List all jobs | (optional) cursor, limit |
| `job_info` | Get job details | jobId |
| `job_set_status` | Update status | jobId, status |

Job statuses: `Open`, `Closed`, `Draft`, `Archived`

### Application Management

| Tool | Purpose | Required Params |
|------|---------|-----------------|
| `application_create` | Consider candidate for job | candidateId, jobId |
| `application_list` | List applications | (optional) jobId, candidateId, status |
| `application_info` | Get application details | applicationId |
| `application_change_stage` | Move in pipeline | applicationId, interviewStageId |
| `application_change_source` | Update attribution | applicationId, sourceId |
| `application_update` | Update properties | applicationId |

Application statuses: `Active`, `Hired`, `Archived`

### Interview Scheduling

| Tool | Purpose | Required Params |
|------|---------|-----------------|
| `interview_list` | List interviews | (optional) applicationId |
| `interview_schedule_create` | Schedule interview | applicationId, interviewerUserIds, startTime, endTime |
| `interview_schedule_list` | List schedules | (optional) startTimeAfter, startTimeBefore |
| `interview_schedule_update` | Modify schedule | interviewScheduleId |
| `interview_schedule_cancel` | Cancel interview | interviewScheduleId |

### Organization

| Tool | Purpose | Required Params |
|------|---------|-----------------|
| `user_list` | List team members | (optional) includeDeactivated |
| `user_search` | Find user | email or name |
| `department_list` | List departments | (optional) includeArchived |
| `location_list` | List locations | (optional) includeArchived |

### Offers

| Tool | Purpose | Required Params |
|------|---------|-----------------|
| `offer_create` | Create offer | applicationId |
| `offer_list` | List offers | (optional) applicationId |

### Utilities

| Tool | Purpose | Required Params |
|------|---------|-----------------|
| `interview_stage_list` | Get pipeline stages | (optional) jobId |
| `source_list` | Get candidate sources | (optional) cursor |
| `candidate_tag_list` | Get available tags | (optional) cursor |
| `archive_reason_list` | Get rejection reasons | none |

## Common Operations

### Find a Candidate

```
# By email (exact match)
candidate_search(email="jane@example.com")

# By name (partial match)
candidate_search(name="Jane")
```

### Create and Apply Candidate

```
# Step 1: Create candidate
candidate = candidate_create(
    name="Jane Smith",
    email="jane@example.com"
)

# Step 2: Apply to job
application_create(
    candidateId=candidate["id"],
    jobId="target-job-id"
)
```

### Move Candidate Through Pipeline

```
# Get current stage and next stage
stages = interview_stage_list(jobId="...")
next_stage_id = stages["results"][1]["id"]

# Move application
application_change_stage(
    applicationId="app-id",
    interviewStageId=next_stage_id
)
```

### Schedule an Interview

```
interview_schedule_create(
    applicationId="app-id",
    interviewerUserIds=["user-1", "user-2"],
    startTime="2024-01-15T14:00:00Z",
    endTime="2024-01-15T15:00:00Z"
)
```

### Reject a Candidate

```
# Get archive reasons
reasons = archive_reason_list()
reason_id = reasons["results"][0]["id"]  # e.g., "Not qualified"

# Get archived stage
stages = interview_stage_list(jobId="...")
archived_stage = next(s for s in stages["results"] if s["type"] == "Archived")

# Archive
application_change_stage(
    applicationId="app-id",
    interviewStageId=archived_stage["id"],
    archiveReasonId=reason_id
)
```

## Response Format

All tools return JSON. Successful responses have this structure:

```json
{
  "success": true,
  "results": { ... }  // or array for list operations
}
```

List operations include pagination:

```json
{
  "success": true,
  "results": [...],
  "moreDataAvailable": true,
  "nextCursor": "cursor-string"
}
```

Error responses:

```json
{
  "success": false,
  "errors": ["error_code"]
}
```

## Pagination

List operations use cursor-based pagination:

```
# First page
results = candidate_list(limit=50)

# Next page (if moreDataAvailable is true)
results = candidate_list(limit=50, cursor=results["nextCursor"])
```

## Date/Time Format

All timestamps use ISO 8601 format:
- `2024-01-15T14:00:00Z` (UTC)
- `2024-01-15T14:00:00-08:00` (with offset)

## Error Codes

Common error codes and meanings:

| Code | Meaning |
|------|---------|
| `invalid_input` | Missing or malformed parameter |
| `not_found` | Resource doesn't exist |
| `unauthorized` | Permission denied |
| `rate_limited` | Too many requests |
| `already_exists` | Duplicate entry |

## Tips

### Finding IDs

Most operations require resource IDs. Use search/list tools first:

```
# Get candidate ID
candidate = candidate_search(email="...")["results"][0]
candidate_id = candidate["id"]

# Get job ID
job = job_search(title="...")["results"][0]
job_id = job["id"]
```

### Filtering Applications

Filter by multiple criteria:

```
# All active apps for a specific job
application_list(jobId="...", status="Active")

# All apps for a specific candidate
application_list(candidateId="...")
```

### Source Attribution

Always track where candidates come from:

```
# Get available sources
sources = source_list()

# Apply with source
application_create(
    candidateId="...",
    jobId="...",
    sourceId="linkedin-source-id"
)
```

## Additional Resources

### Reference Files

For complete API patterns:
- **`references/tool-reference.md`** - Full parameter details for all tools

### Related Skills

- **ashby-workflows** - Pipeline management and recruiting workflows
