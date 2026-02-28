---
name: jules
description: Integration with Google's Jules API for agentic coding sessions.
license: UNLICENSED
metadata:
  author: user
  version: "1.0.0"
---

# Jules API Skill

This skill provides instructions for interacting with the Google Jules API.
Use this skill when you need to create coding sessions, manage tasks, or interact with the Jules agent.

## Authentication

1.  Read the API key from `/home/cgasgarth/.config/opencode/jules-secret.json`.
    ```json
    { "julesApiKey": "YOUR_KEY" }
    ```
2.  Use the header `X-Goog-Api-Key: YOUR_KEY` in all requests.

## Base URL

`https://jules.googleapis.com/v1alpha`

## Endpoints

### 1. List Sources
`GET /sources`
Returns available sources (e.g., GitHub repositories) connected to the user.

### 2. Create Session
`POST /sessions`
Creates a new coding session.

**Body:**
```json
{
  "prompt": "Task description",
  "sourceContext": {
    "source": "projects/PROJECT_ID/locations/global/connections/CONNECTION_ID/repositories/REPO_ID",
    "githubRepoContext": {
      "startingBranch": "main"
    }
  },
  "automationMode": "AUTO_CREATE_PR", // Optional
  "requirePlanApproval": true // Optional, defaults to false
}
```

### 3. List Sessions
`GET /sessions`
Lists active and past sessions.

### 4. Get Session Activities
`GET /sessions/{sessionId}/activities`
Returns the history of activities in a session.

**Parameters:**
- `pageSize` (optional): Number of activities to return (e.g., `?pageSize=30`).

**Response Structure:**
Returns an `activities` array. Activity types include:
- `planGenerated`: Agent created a plan.
- `planApproved`: User approved a plan.
- `progressUpdated`: Agent progress message (contains `title`, `description`).
- `sessionCompleted`: Session finished.

**Artifacts:**
Activities may contain an `artifacts` field with data like bash output, git patches, or images.

### 5. Send Message
`POST /sessions/{sessionId}:sendMessage`
Sends a user message to the session.

**Body:**
```json
{
  "prompt": "Your message here"
}
```

### 6. Approve Plan
`POST /sessions/{sessionId}:approvePlan`
Approves the plan proposed by Jules to proceed with implementation.

## Example Usage (Bash)

To list sessions (replace KEY with actual key):
```bash
curl "https://jules.googleapis.com/v1alpha/sessions" \
  -H "X-Goog-Api-Key: $KEY"
```

To get session activities:
```bash
curl "https://jules.googleapis.com/v1alpha/sessions/$SESSION_ID/activities?pageSize=30" \
  -H "X-Goog-Api-Key: $KEY"
```

To create a session:
```bash
curl -X POST "https://jules.googleapis.com/v1alpha/sessions" \
  -H "X-Goog-Api-Key: $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Fix the bug in login",
    "sourceContext": { ... }
  }'
```

## Agent Instructions

- When asked to use Jules, first read the secret file to get the key.
- Construct the appropriate curl command or HTTP request.
- Parse the JSON response.
- Present the result to the user clearly.
