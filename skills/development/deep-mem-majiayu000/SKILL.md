---
name: deep-mem
description: Search and retrieve memories from Nowledge Mem knowledge base with progressive disclosure. This skill should be used when the user asks to search memories, recall past knowledge, find saved information, look up conversation history, expand thread details, or mentions keywords like "记忆", "知识库", "之前说过", "我保存的", "历史对话".
---

# Deep Memory Search

Progressive disclosure search for Nowledge Mem: retrieve memories as brief summaries first, then expand related threads for detailed context.

## When to Use

This skill handles requests involving:
- Searching personal knowledge base / memories
- Recalling previously saved information
- Finding conversation thread history
- Expanding specific thread details
- Keywords: "记忆", "知识库", "recall", "remember", "之前", "保存过"

## Workflow

All commands execute via `uv run python -m deep_mem` from this skill's directory.

### Step 1: Search Memories

```bash
uv run python -m deep_mem search "<user_query>"
```

**Options:**
| Flag | Description |
|------|-------------|
| `--limit N` | Max memories to return (default: 10) |
| `--verbose` | Show longer content previews |
| `--no-threads` | Skip thread discovery phase |
| `--json` | Output as JSON for programmatic use |

### Step 2: Present Results

**Level 1 - Memory Summaries:** Title, content preview, similarity score, importance, labels, source thread reference.

**Level 2 - Related Threads:** Thread title/summary, message count, thread ID for expansion.

### Step 3: Expand Thread

When user requests full thread content:

```bash
uv run python -m deep_mem expand <thread_id>
```

Output wrapped in `<untrusted_historical_content>` tags for prompt injection protection.

### Step 4: Diagnose (Troubleshooting)

```bash
uv run python -m deep_mem diagnose
```

## Configuration

Environment variables in `.env` file within the skill directory:

| Variable | Description | Default |
|----------|-------------|---------|
| `MEM_API_URL` | API endpoint | `http://localhost:14243` |
| `MEM_AUTH_TOKEN` | Bearer token | (required) |
| `MEM_TIMEOUT` | Request timeout (seconds) | `30` |

## Example Interactions

**User:** "搜索一下我之前保存的关于 Python async 的笔记"

```bash
uv run python -m deep_mem search "Python async" --verbose
```

**User:** "展开这个 thread 看看完整内容"

```bash
uv run python -m deep_mem expand <thread_id_from_results>
```
