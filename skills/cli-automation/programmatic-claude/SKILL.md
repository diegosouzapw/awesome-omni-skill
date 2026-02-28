---
name: programmatic-claude
description: Run Claude Code programmatically via CLI (-p flag), Python SDK, or TypeScript SDK. For automation, CI/CD, session chaining, headless execution, and orchestrating multiple Claude instances. Triggers: programmatic, headless, CLI automation, session chaining, -p flag, Python SDK, orchestrate Claude, CI/CD Claude, run Claude automatically.
---

# Programmatic Claude Code

Run Claude non-interactively for automation workflows.

## Three Interfaces

| Interface | Install | Best For |
|-----------|---------|----------|
| CLI (`-p`) | Built-in | Scripts, CI/CD, quick tasks |
| Python SDK | `pip install claude-agent-sdk` | Custom agents, hooks, multi-turn |
| TypeScript SDK | `npm install @anthropic-ai/claude-agent-sdk` | Type-safe apps, Node.js |

## Core Patterns

### One-Shot Task
```bash
claude -p "Fix TypeScript errors in src/" \
  --allowedTools "Read,Edit,Bash" \
  --output-format json
```

### Session Chaining
```bash
# Start task, capture session ID
result=$(claude -p "Create auth system" --output-format json)
session_id=$(echo "$result" | jq -r '.session_id')

# Continue in same session
claude -p "Add tests for auth" --resume "$session_id"
```

### Structured Output
```bash
claude -p "List all API endpoints" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"endpoints":{"type":"array"}}}'
```

### Python Multi-Turn
```python
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions

options = ClaudeAgentOptions(
    allowed_tools=["Read", "Write", "Edit", "Bash"],
    permission_mode="acceptEdits"
)

async with ClaudeSDKClient(options=options) as client:
    await client.query("Build the component")
    async for msg in client.receive_response():
        print(msg)

    await client.query("Now add tests")
    async for msg in client.receive_response():
        print(msg)
```

## Key Flags

| Flag | Purpose |
|------|---------|
| `-p "prompt"` | Non-interactive mode |
| `--allowedTools "X,Y"` | Auto-approve tools (no prompts) |
| `--output-format json` | Structured output with session_id |
| `--json-schema '{...}'` | Enforce response format |
| `--resume <id>` | Continue specific session |
| `--continue` | Continue most recent session |

## Files

- `README.md` - Overview and quick examples
- `harvest-and-build.md` - Website cloning automation pipeline

## Use Cases

1. **CI/CD** - Run in GitHub Actions for automated fixes
2. **Orchestration** - Chain multiple Claude sessions
3. **Pipelines** - Harvest → Analyse → Build → Validate workflows
4. **Scheduled Tasks** - Daily security scans, dependency updates
