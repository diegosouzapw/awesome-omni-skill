---
name: oatda-video-status
description: Use when the user wants to check the status of an asynchronous video generation task from OATDA. Returns the video URL when generation is complete.
---

# OATDA Video Status

Check the status of asynchronous video generation tasks. This is the companion skill to `oatda-generate-video`.

## When to Use

Use this skill when the user wants to:
- Check if a previously submitted video has finished generating
- Retrieve the video URL after generation completes
- Get the status of a video generation task by its task ID

## Prerequisites

The user needs:
- An OATDA API key (`$OATDA_API_KEY` or `~/.oatda/credentials.json`)
- A task ID from a previous `/oatda:oatda-generate-video` call

## Step-by-Step Instructions

### 1. Resolve the API key

```bash
# Check env var first; if empty, auto-load from credentials file
if [[ -z "$OATDA_API_KEY" ]]; then
  export OATDA_API_KEY=$(cat ~/.oatda/credentials.json 2>/dev/null | jq -r '.profiles[.defaultProfile].apiKey' 2>/dev/null)
fi

# Verify key exists (show first 8 chars only)
echo "${OATDA_API_KEY:0:8}"
```

If the output is empty or `null`, stop and ask the user to configure their API key.

**IMPORTANT**:
- Never print the full API key. Only show the first 8 characters for verification.
- The key resolution script and subsequent `curl` commands **must run in the same shell session**. Each separate bash/terminal invocation starts with an isolated environment where previously exported variables are lost. Either run all commands in one session, or chain them (e.g., `export OATDA_API_KEY=... && curl ...`).

### 2. Get the task ID

The user must provide a task ID from a previous video generation request. If they don't have one, tell them to generate a video first with `/oatda:oatda-generate-video`.

### 3. Make the API call

```bash
curl -s -X GET "https://oatda.com/api/v1/llm/video-status/<TASK_ID>" \
  -H "Authorization: Bearer $OATDA_API_KEY"
```

Replace `<TASK_ID>` with the actual task ID. URL-encode the task ID if it contains special characters.

### 4. Parse the response

```json
{
  "taskId": "minimax-T2V01-abc123def456",
  "status": "completed",
  "videoUrl": "https://cdn.example.com/video.mp4",
  "directVideoUrl": "https://cdn.example.com/video-direct.mp4",
  "provider": "minimax",
  "model": "T2V-01",
  "createdAt": "2025-01-15T10:30:00Z",
  "completedAt": "2025-01-15T10:32:15Z",
  "costs": {
    "totalCost": 0.05,
    "currency": "USD"
  }
}
```

### 5. Present results based on status

| Status | What to tell the user |
|--------|----------------------|
| `pending` | "Your video is queued and hasn't started yet. Check again in a minute." |
| `processing` | "Your video is being generated. Check again in a minute." |
| `completed` | "Your video is ready!" — show `videoUrl` (or `directVideoUrl`). Mention cost if available. |
| `failed` | "Video generation failed." — show `errorMessage` if present. Suggest trying again with a different prompt. |

### 6. Handle errors

| HTTP Status | Meaning | Action |
|-------------|---------|--------|
| 401 | Invalid API key | Tell user to check their key |
| 404 | Task not found | Verify task ID is correct. Tasks may expire. |
| 429 | Rate limited | Wait and retry |

## Full Example

User asks: "Check the status of my video task minimax-T2V01-abc123"

```bash
curl -s -X GET "https://oatda.com/api/v1/llm/video-status/minimax-T2V01-abc123" \
  -H "Authorization: Bearer $OATDA_API_KEY"
```

If completed, tell the user:
> Your video is ready! Download it here: `<videoUrl>`
> Cost: $0.05

If still processing:
> Your video is still being generated. Try checking again in about a minute.

## Tips

- This is a GET request, not POST — no request body needed
- Video generation typically takes 30 seconds to 5 minutes
- Video URLs may be temporary — recommend downloading promptly
- If status is `processing`, suggest waiting 30-60 seconds before checking again
- NEVER expose the full API key in output
- Related skills: `/oatda:oatda-generate-video` (companion for starting generation)
