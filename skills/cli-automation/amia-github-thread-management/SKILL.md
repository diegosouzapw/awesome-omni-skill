---
name: amia-github-thread-management
description: "Use when managing PR review threads. Reply does NOT auto-resolve threads. Trigger with /manage-threads."
license: Apache-2.0
compatibility: Requires AI Maestro installed.
metadata:
  version: "1.0.0"
  author: Emasoft
  tags: "github, pull-request, code-review, thread-management"
  triggers: "resolve review thread, unresolve thread, reply to comment, track review comments, unaddressed comments, batch resolve threads"
agent: api-coordinator
context: fork
workflow-instruction: "support"
procedure: "support-skill"
user-invocable: false
---

# GitHub Thread Management Skill

## Overview

This skill teaches you how to manage GitHub Pull Request review threads. Review threads are conversation containers attached to specific lines of code in a PR diff.

**CRITICAL UNDERSTANDING**: Replying to a review thread does NOT automatically resolve it. Resolution is a separate GraphQL mutation that must be explicitly called. Many developers assume replying resolves threads - this is incorrect.

## Prerequisites

- GitHub CLI (`gh`) installed and authenticated
- Python 3.8+ for running automation scripts
- Repository collaborator access (required for resolving threads)
- GraphQL API access

## Instructions

Follow these steps when managing GitHub PR review threads:

1. **Identify threads requiring attention**: Use `amia_get_review_threads.py --unresolved-only` to list all unresolved threads on the PR
2. **Determine the appropriate action**: Consult the Decision Tree section below to decide whether to reply, resolve, or both
3. **Execute the operation**: Run the corresponding script from the Available Scripts section with the required parameters
4. **Verify success**: Check the JSON output's `success` field and verify the thread state changed as expected
5. **Track progress**: Re-run listing commands to confirm all threads have been properly addressed

### Checklist

Copy this checklist and track your progress:

- [ ] Identify unresolved threads using `amia_get_review_threads.py --unresolved-only`
- [ ] Determine action for each thread (reply/resolve/both)
- [ ] For implemented changes: resolve thread (optionally with reply)
- [ ] For clarification needed: reply only (keep thread open)
- [ ] For batch operations: use `amia_resolve_threads_batch.py`
- [ ] Verify each operation succeeded via JSON output
- [ ] Re-run listing to confirm all threads addressed
- [ ] Check for any comments without replies using `amia_get_unaddressed_comments.py`

## Output

All scripts in this skill output JSON to stdout with the following standard structure:

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether the operation completed successfully |
| `threadId` | string | The GraphQL node ID of the affected thread (PRRT_xxx format) |
| `isResolved` | boolean | Current resolution state of the thread after operation |
| `commentId` | string | (Reply operations only) The GraphQL node ID of the created comment |
| `results` | array | (Batch operations only) Array of per-thread success/failure details |
| `error` | string | (On failure) Human-readable error message explaining what went wrong |

See the Script Usage Details section for script-specific output format variations.

## Decision Tree for Thread Operations

```
START: You need to handle a review thread
│
├─► Q: Has the requested change been implemented?
│   │
│   ├─► YES: Do you need to explain what was done?
│   │   │
│   │   ├─► YES: Reply THEN Resolve (two operations)
│   │   │       See: scripts/amia_reply_to_thread.py --and-resolve
│   │   │
│   │   └─► NO: Just Resolve (no reply needed)
│   │           See: scripts/amia_resolve_thread.py
│   │
│   └─► NO: Is clarification needed from the reviewer?
│       │
│       ├─► YES: Reply only (keep thread OPEN)
│       │       See: scripts/amia_reply_to_thread.py (without --and-resolve)
│       │
│       └─► NO: Leave thread untouched until you address it
│
├─► Q: Do you need to resolve MULTIPLE threads at once?
│   │
│   └─► YES: Use batch resolution (1 API call for N threads)
│           See: scripts/amia_resolve_threads_batch.py
│
└─► Q: Do you need to find threads that still need attention?
    │
    ├─► Find all unresolved threads:
    │   See: scripts/amia_get_review_threads.py --unresolved-only
    │
    └─► Find comments without any replies:
        See: scripts/amia_get_unaddressed_comments.py
```

## Key Concepts

### Thread vs Comment

- **Review Thread**: A container that holds one or more comments, anchored to a specific file/line
- **Review Comment**: An individual message within a thread
- **Thread ID**: The GraphQL node ID of the thread (needed for resolution)
- **Comment ID**: The GraphQL node ID of an individual comment

### Thread States

| State | Meaning | When to Use |
|-------|---------|-------------|
| **Unresolved** | Thread requires attention | Default state, indicates pending work |
| **Resolved** | Thread has been addressed | After implementing requested change |

### The Reply-Resolve Separation

GitHub's API separates these operations because:
1. Replying adds a comment to the conversation
2. Resolving changes the thread's status metadata

You might reply without resolving (asking for clarification), or resolve without replying (when the code change speaks for itself).

## Reference Documents

### Thread Resolution Protocol
See [references/thread-resolution-protocol.md](references/thread-resolution-protocol.md)

**Contents:**
- 1.1 Why thread resolution is separate from replying
- 1.2 Single thread resolution workflow
  - 1.2.1 Getting the thread's GraphQL node ID
  - 1.2.2 The resolveReviewThread mutation
  - 1.2.3 Verifying resolution succeeded
- 1.3 Batch thread resolution using GraphQL aliases
  - 1.3.1 Constructing aliased mutations
  - 1.3.2 Handling partial failures
  - 1.3.3 Performance considerations
- 1.4 When to resolve vs when to keep open
  - 1.4.1 Resolve immediately scenarios
  - 1.4.2 Keep open scenarios
- 1.5 Unresolving threads (reopening discussion)

### Thread Conversation Tracking
See [references/thread-conversation-tracking.md](references/thread-conversation-tracking.md)

**Contents:**
- 2.1 Getting thread history via GraphQL
  - 2.1.1 Query structure for review threads
  - 2.1.2 Pagination for threads with many comments
- 2.2 Tracking addressed vs unaddressed comments
  - 2.2.1 Definition of "addressed"
  - 2.2.2 Finding comments without replies
- 2.3 GitHub's comment threading model
  - 2.3.1 Root comments vs reply comments
  - 2.3.2 Outdated comments (when code changes)
- 2.4 Minimized comments handling
  - 2.4.1 What minimized means
  - 2.4.2 When to consider minimized comments

## Available Scripts

All scripts are located in the `scripts/` subdirectory and output JSON to stdout.

| Script | Purpose | Key Parameters |
|--------|---------|----------------|
| `amia_get_review_threads.py` | List all review threads on a PR | `--owner`, `--repo`, `--pr`, `--unresolved-only` |
| `amia_resolve_thread.py` | Resolve a single thread | `--thread-id` |
| `amia_resolve_threads_batch.py` | Resolve multiple threads (1 API call) | `--thread-ids` (comma-separated) |
| `amia_reply_to_thread.py` | Reply to a thread | `--thread-id`, `--body`, `--and-resolve` |
| `amia_get_unaddressed_comments.py` | Find comments without replies | `--owner`, `--repo`, `--pr` |

## Common Workflows

### Workflow 1: Address All Unresolved Threads

```bash
# Step 1: Get all unresolved threads
python3 scripts/amia_get_review_threads.py --owner OWNER --repo REPO --pr 123 --unresolved-only

# Step 2: For each thread, make the code change, then resolve
python3 scripts/amia_resolve_thread.py --thread-id PRRT_xxxxx
```

### Workflow 2: Reply and Resolve in One Command

```bash
# When you want to explain what you did AND resolve
python3 scripts/amia_reply_to_thread.py \
  --thread-id PRRT_xxxxx \
  --body "Fixed by using the recommended approach" \
  --and-resolve
```

### Workflow 3: Batch Resolve After Large Refactor

```bash
# When you've addressed multiple comments in one commit
python3 scripts/amia_resolve_threads_batch.py \
  --thread-ids "PRRT_aaa,PRRT_bbb,PRRT_ccc"
```

### Workflow 4: Find What Still Needs Attention

```bash
# Find comments that haven't received any reply yet
python3 scripts/amia_get_unaddressed_comments.py --owner OWNER --repo REPO --pr 123
```

## Examples

### Example 1: Resolve All Unresolved Threads

```bash
# Get all unresolved threads
python3 scripts/amia_get_review_threads.py --owner myorg --repo myrepo --pr 123 --unresolved-only
# Output: [{"id": "PRRT_abc", "path": "src/main.py", "line": 42, ...}]

# Resolve each thread
python3 scripts/amia_resolve_thread.py --thread-id PRRT_abc
# Output: {"success": true, "threadId": "PRRT_abc", "isResolved": true}
```

### Example 2: Reply and Resolve in One Command

```bash
python3 scripts/amia_reply_to_thread.py \
  --thread-id PRRT_xyz \
  --body "Fixed by refactoring the validation logic" \
  --and-resolve
# Output: {"success": true, "commentId": "...", "resolved": true}
```

## Error Handling

### "Thread not found" Error

**Cause**: The thread ID is incorrect or the thread was deleted.

**Solution**: Re-fetch thread IDs using `amia_get_review_threads.py`. Thread IDs start with `PRRT_` for review threads.

### Resolution Appears to Fail Silently

**Cause**: The mutation succeeded but the response wasn't checked properly.

**Solution**: The script verifies resolution by checking the `isResolved` field in the response. If it returns `false`, the resolution didn't take effect - check permissions.

### Cannot Resolve Thread - Permission Denied

**Cause**: Only the PR author and repository collaborators can resolve threads.

**Solution**: Ensure you're authenticated as a user with write access to the repository.

### Reply Added But Thread Still Unresolved

**Cause**: This is expected behavior! Replying does not resolve.

**Solution**: Use `--and-resolve` flag with `amia_reply_to_thread.py`, or call `amia_resolve_thread.py` separately.

### GraphQL Rate Limiting

**Cause**: Too many API calls in short succession.

**Solution**: Use batch operations (`amia_resolve_threads_batch.py`) to resolve multiple threads in a single API call instead of individual calls.

## Script Usage Details

### amia_get_review_threads.py

```bash
python3 scripts/amia_get_review_threads.py \
  --owner <repository_owner> \
  --repo <repository_name> \
  --pr <pull_request_number> \
  [--unresolved-only]
```

**Output**: JSON array of thread objects with `id`, `isResolved`, `path`, `line`, `body` (first comment).

### amia_resolve_thread.py

```bash
python3 scripts/amia_resolve_thread.py --thread-id <PRRT_xxx>
```

**Output**: JSON object with `success`, `threadId`, `isResolved`.

### amia_resolve_threads_batch.py

```bash
python3 scripts/amia_resolve_threads_batch.py \
  --thread-ids "PRRT_aaa,PRRT_bbb,PRRT_ccc"
```

**Output**: JSON object with `results` array containing per-thread success/failure.

### amia_reply_to_thread.py

```bash
python3 scripts/amia_reply_to_thread.py \
  --thread-id <PRRT_xxx> \
  --body "Your reply message" \
  [--and-resolve]
```

**Output**: JSON object with `success`, `commentId`, `resolved` (if --and-resolve used).

### amia_get_unaddressed_comments.py

```bash
python3 scripts/amia_get_unaddressed_comments.py \
  --owner <repository_owner> \
  --repo <repository_name> \
  --pr <pull_request_number>
```

**Output**: JSON array of comments that have no replies, with `threadId`, `commentId`, `author`, `body`, `path`, `line`.

## Exit Codes (Standardized)

All scripts use standardized exit codes for consistent error handling:

| Code | Meaning | Description |
|------|---------|-------------|
| 0 | Success | Operation completed successfully |
| 1 | Invalid parameters | Bad thread ID format, missing required args |
| 2 | Resource not found | Thread or PR does not exist |
| 3 | API error | Network, rate limit, timeout |
| 4 | Not authenticated | gh CLI not logged in |
| 5 | Idempotency skip | Thread already resolved (for resolve scripts) |
| 6 | Not mergeable | N/A for these scripts |

**Note:** `amia_resolve_threads_batch.py` returns exit code 0 for partial success. Check the JSON output's `summary.failed` field for individual failures.

## Resources

- [references/thread-resolution-protocol.md](references/thread-resolution-protocol.md) - Thread resolution workflow
- [references/thread-conversation-tracking.md](references/thread-conversation-tracking.md) - Conversation tracking patterns
