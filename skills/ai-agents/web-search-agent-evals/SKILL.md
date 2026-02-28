---
name: web-search-agent-evals
description: Development assistant for web search agent evaluations across multiple CLI agents
compatibility: Bun >= 1.2.9
---

# Web Search Agent Evaluations

Development assistant for running and comparing web search capabilities across CLI agents.

## Overview

Evaluate 4 agents (Claude Code, Gemini, Droid, Codex) with 2 tools (builtin, You.com MCP) = 8 pairings.

**Key Features:**
- **Headless adapters** - Schema-driven CLI agent execution via [@plaited/agent-eval-harness](https://www.npmjs.com/package/@plaited/agent-eval-harness)
- **Flag-based architecture** - Single service per agent, mode selected via environment variables
- **Type-safe constants** - MCP server definitions in TypeScript
- **Isolated execution** - Each pairing runs in its own Docker container

**Architecture:**
- `agent-schemas/` - Headless adapter JSON schemas
- `mcp-servers.ts` - TypeScript MCP server constants
- `docker/entrypoint` - Bun shell script for runtime config
- `scripts/` - Type-safe execution and comparison CLI tools
- `docker/` - Container infrastructure

## Quick Commands

### Run Evaluations

```bash
# Full dataset (151 prompts), k=5 — all 8 agent×provider scenarios
bun run trials

# Quick smoke test (5 random prompts, single trial)
bun run trials -- --count 5 -k 1

# Specific agent or provider
bun run trials -- --agent claude-code --search-provider builtin
bun run trials -- --agent gemini --search-provider you

# Trial type presets
bun run trials -- --trial-type capability   # k=10, deep exploration
bun run trials -- --trial-type regression   # k=3, fast regression check

# Custom k value
bun run trials -- -k 7

# Control parallelism
bun run trials -- -j 4                      # Limit to 4 containers
bun run trials -- --prompt-concurrency 4    # 4 prompts per container

# Direct Docker (manual testing)
docker compose run --rm -e SEARCH_PROVIDER=builtin claude-code
docker compose run --rm -e SEARCH_PROVIDER=you -e PROMPT_COUNT=5 gemini
```

### Compare Results

Comparisons are written to `data/comparisons/YYYY-MM-DD/`.

```bash
# Latest date auto-detected
bun run compare

# Statistical analysis with bootstrap confidence intervals
bun run compare:stat

# Specific date or filter
bun run compare -- --run-date 2026-02-18
bun run compare -- --agent droid
bun run compare -- --search-provider builtin
bun run compare -- --trial-type capability

# View results
cat data/comparisons/2026-02-18/all-builtin-weighted.json | jq '.capability'
cat data/comparisons/2026-02-18/builtin-vs-you-weighted.json | jq '.headToHead.capability'
```

**Comparison strategies:**
- `weighted` (default) - Capability, reliability, and consistency weighted scoring
- `statistical` - Bootstrap sampling with 95% confidence intervals

### Generate Report

Generate a comprehensive `REPORT.md` from comparison results:

```bash
# Latest date auto-detected
bun run report

# Specific date
bun run report -- --run-date 2026-02-18

# Preview without writing
bun run report -- --dry-run
```

**Report includes:**
- Executive summary with best capability, reliability, and performance
- Quality rankings with pass@k and pass^k scores
- Performance rankings (latency P50/P90/P99)
- Flakiness analysis with top flaky prompts
- MCP tool impact analysis (builtin vs MCP comparison)
- Tool call statistics (P50/P90/P99/mean per provider)
- Tool call distribution histograms
- Failing prompts list (pass@k = 0%) with query text

**Output:** `data/comparisons/YYYY-MM-DD/REPORT.md`

### Calibrate Grader

Interactive wizard to sample failures and review grader accuracy. Helps distinguish between agent failures (agent got it wrong) and grader bugs (agent was correct, grader too strict).

```bash
# Interactive calibration (recommended)
bun run calibrate
```

**Interactive prompts:**
1. **Run date** - Select from available dated runs
2. **Agents** - Multi-select via numbers or "all"
3. **Search providers** - Multi-select via numbers or "all"
4. **Sample count** - Number of failures to sample (default: 5)

**Output:** `data/calibration/{date}-{agent}-{provider}.md`

**What calibration reveals:**
- ❌ **Grader too strict** - Agent gave correct answer, grader rejected valid paraphrasing
- ❌ **Hint too vague** - Grader can't tell good from bad answers
- ✅ **Real failures** - Agent genuinely gave wrong/incomplete answer

See [@agent-eval-harness calibration docs](../agent-eval-harness@plaited_agent-eval-harness/SKILL.md#calibrate-command) for grader calibration concepts.

## Parallelization

The evaluation harness supports **two-level parallelization** for optimal performance:

### Container-Level Concurrency (`-j`, `--concurrency`)

Controls how many Docker containers (agent×provider scenarios) run simultaneously.

```bash
bun run trials              # Unlimited (default, all 8 scenarios at once)
bun run trials -- -j 4     # Limit to 4 containers
bun run trials -- -j 1     # Sequential (debugging)
```

**Use cases:**
- Unlimited (default) - All scenarios at once, I/O-bound workload handles it fine
- `-j 4` - Limit concurrency if hitting API rate limits
- `-j 2` - Conservative, for low-resource machines
- `-j 1` - Sequential execution for debugging

### Prompt-Level Concurrency (`--prompt-concurrency`)

Controls how many prompts run in parallel **within each container**.

```bash
bun run trials -- --prompt-concurrency 4    # 4 prompts (moderate parallelism)
bun run trials -- --prompt-concurrency 1    # Sequential (default, safest)
bun run trials -- --prompt-concurrency 8    # 8 prompts (high memory, CI only)
```

**How it works:**
- Uses harness `-j` flag with `--workspace-dir` for isolation
- Each prompt gets its own workspace directory
- Web searches are **I/O-bound** — parallel prompts maximize network bandwidth

**Performance comparison:**

| Config | Containers | Prompts/Container | Full (151 prompts, k=5) |
|--------|-----------|-------------------|------------------------|
| **Default** | **unlimited** | **1** | **~2.5 hrs** |
| Faster | unlimited | 4 | ~40 min |
| CI (high memory) | unlimited | 8 | ~20 min |

**Warning:** Stream-mode agents (claude-code, droid) use ~400-500MB RSS per prompt process. With `--prompt-concurrency 8` that's 3-4GB per container — OOM kills likely in Docker (see [issue #45](https://github.com/plaited/agent-eval-harness/issues/45))

## Prompts

Prompts live in a flat `data/prompts/` directory. The format differs by search provider:
- **Builtin mode**: Just the query (e.g., "What are the best free icon libraries...")
- **MCP mode**: `"Use {server-name} and answer\n{query}"` with MCP metadata

| File | Prompts | Metadata | Use With |
|------|---------|----------|----------|
| `prompts.jsonl` | 151 | No MCP | `SEARCH_PROVIDER=builtin` |
| `prompts-you.jsonl` | 151 | `mcpServer="ydc-server"`, `expectedTools=["you-search"]` | `SEARCH_PROVIDER=you` |

The entrypoint automatically selects the correct prompt file based on `SEARCH_PROVIDER`. To run a random subset, pass `PROMPT_COUNT` (or `--count N` via CLI):

```bash
bun run trials -- --count 5    # 5 random prompts from full dataset
```

## Results

All trial results are written to flat dated directories:

```
data/results/YYYY-MM-DD/
├── claude-code/
│   ├── builtin.jsonl
│   └── you.jsonl
├── gemini/
├── droid/
└── codex/
```

Each `.jsonl` line is a `TrialResult`:
```jsonl
{"id":"websearch-001","input":"...","k":5,"passRate":0.8,"passAtK":0.999,"passExpK":0.328,"trials":[...]}
```

**Versioning:**
```bash
git add data/results/ && git commit -m "feat: trial run YYYY-MM-DD"
```

**Compare runs:**
```bash
bun run compare                             # Latest date auto-detected
bun run compare -- --run-date 2026-02-18   # Specific date
```

## Adding a New Agent

### 1. Create Headless Adapter Schema

Create `agent-schemas/<agent>.json`:

```json
{
  "command": ["<agent-cli>", "--flag", "{input}"],
  "outputEvents": {
    "match": { "path": "$.type" },
    "patterns": {
      "text": { "value": "text" },
      "tool_call": { "value": "tool_call" },
      "tool_result": { "value": "tool_result" }
    }
  },
  "result": {
    "contentPath": "$.output",
    "errorPath": "$.error"
  },
  "mode": "stream",
  "env": ["AGENT_API_KEY"]
}
```

**Key fields:**
- `command` - CLI invocation with `{input}` placeholder
- `outputEvents.match.path` - JSONPath to event type field
- `patterns` - Map event types to standard names
- `result.contentPath` - JSONPath to extract final output
- `mode` - `"stream"` (persistent) or `"iterative"` (new process per turn)
- `env` - Required environment variables

**Test schema:**
```bash
bunx @plaited/agent-eval-harness adapter:check -- \
  bunx @plaited/agent-eval-harness headless --schema agent-schemas/<agent>.json
```

### 2. Create Dockerfile

Create `docker/<agent>.Dockerfile`:

```dockerfile
FROM base

# Install agent CLI
RUN npm install -g <agent-cli>

# Copy entrypoint and MCP config
COPY docker/entrypoint /entrypoint
COPY mcp-servers.ts /eval/mcp-servers.ts

RUN chmod +x /entrypoint

CMD ["/entrypoint"]
```

**Verify installation:**
```bash
docker build -t test-<agent> -f docker/<agent>.Dockerfile .
docker run --rm test-<agent> <agent> --version
```

### 3. Add Docker Compose Service

Add to `docker-compose.yml`:

```yaml
<agent>:
  build:
    context: .
    dockerfile: docker/<agent>.Dockerfile
  volumes:
    - ./agent-schemas:/eval/agent-schemas:ro
    - ./data:/eval/data
    - ./scripts:/eval/scripts:ro
  working_dir: /workspace
  env_file: .env
  environment:
    - AGENT=<agent>
    - SEARCH_PROVIDER=${SEARCH_PROVIDER:-builtin}
```

### 4. Update TypeScript Entrypoint

Edit `docker/entrypoint` to add agent to `configureMcp()` function:

```typescript
const configureMcp = async (agent: string, tool: McpServerKey): Promise<void> => {
  const server = MCP_SERVERS[tool]
  const apiKey = server.auth ? process.env[server.auth.envVar] : undefined

  switch (agent) {
    // ... existing cases ...

    case '<agent>': {
      await $`<agent> mcp add ${server.name} ${server.url} --header "Authorization: Bearer ${apiKey}"`.quiet()
      console.log('✓ Agent MCP server added')
      break
    }
  }
}
```

Add timeout if needed in `buildTrialsCommand()`:

```typescript
switch (AGENT) {
  case '<agent>':
    cmd.push('--timeout', '120000')  // 2 minutes
    break
}
```

### 5. Update Scripts

Edit `scripts/shared/shared.constants.ts` to add agent to `ALL_AGENTS`:

```typescript
export const ALL_AGENTS: Agent[] = ["claude-code", "gemini", "droid", "codex", "<agent>"]
```

Also update the `Agent` type in `scripts/shared/shared.types.ts`:

```typescript
type Agent = "claude-code" | "gemini" | "droid" | "codex" | "<agent>"
```

### 6. Test

```bash
docker compose build <agent>
docker compose run --rm -e SEARCH_PROVIDER=builtin <agent>
docker compose run --rm -e SEARCH_PROVIDER=you <agent>
```

## Adding a New MCP Tool

### 1. Add to mcp-servers.ts

```typescript
export const MCP_SERVERS = {
  you: { /* ... existing */ },
  exa: {
    name: 'exa-server',
    type: 'http' as const,
    url: 'https://api.exa.ai/mcp',
    auth: {
      type: 'bearer' as const,
      envVar: 'EXA_API_KEY',
    },
  },
} as const

export type McpServerKey = keyof typeof MCP_SERVERS
```

### 2. Update docker/entrypoint

Add the new tool case to `configureMcp()` for each agent:

```typescript
case 'claude-code': {
  await $`claude mcp add --transport http ${server.name} ${server.url} --header "Authorization: Bearer ${apiKey}"`.quiet()
  break
}

case 'gemini': {
  await $`gemini mcp add --transport http --header "Authorization: Bearer ${apiKey}" ${server.name} ${server.url}`.quiet()
  break
}

case 'droid': {
  await $`droid mcp add ${server.name} ${server.url} --type http --header "Authorization: Bearer ${apiKey}"`.quiet()
  break
}

case 'codex': {
  const configDir = `${process.env.HOME}/.codex`
  await $`mkdir -p ${configDir}`.quiet()
  const config = `[mcp_servers.${server.name}]
url = "${server.url}"
bearer_token_env_var = "${server.auth?.envVar}"
`
  await Bun.write(`${configDir}/config.toml`, config)
  break
}
```

### 3. Update Environment Files

Add to `.env` and `.env.example`:

```bash
EXA_API_KEY=your_api_key_here
```

### 4. Generate MCP Prompt Sets

Use the generate-mcp-prompts script to create MCP variant files with proper metadata:

```bash
# Generate variants for new MCP server
bun scripts/generate-mcp-prompts.ts --mcp-key exa

# Creates:
# - data/prompts/prompts-exa.jsonl
```

The script prepends `"Use {server-name} and answer\n"` to each query and adds MCP metadata (server name and expected tools).

The entrypoint automatically handles provider-specific prompt files:

```typescript
const promptFile = SEARCH_PROVIDER === "builtin"
  ? `/eval/data/prompts/prompts.jsonl`
  : `/eval/data/prompts/prompts-${SEARCH_PROVIDER}.jsonl`  // e.g., prompts-exa.jsonl
```

**Note:** `scripts/run-trials.ts` automatically picks up new MCP servers from `mcp-servers.ts`, so no manual updates needed.

### 5. Test

```bash
docker compose build
docker compose run --rm -e SEARCH_PROVIDER=exa claude-code
bun run trials -- --search-provider exa --count 5 -k 1
```

## Schema Format Reference

Current agent schemas:

| Schema | Agent | Mode | Status |
|--------|-------|------|--------|
| `claude-code.json` | Claude Code | stream | ✅ Tested |
| `gemini.json` | Gemini CLI | iterative | ✅ Tested |
| `droid.json` | Droid CLI | stream | ✅ Tested |
| `codex.json` | Codex CLI | stream | ✅ Tested |

**Session Modes:**
- `stream` - Process stays alive, multi-turn conversations via stdin
- `iterative` - New process per turn, history passed as context

## Related Skills

- [@agent-eval-harness](../agent-eval-harness@plaited_agent-eval-harness/SKILL.md) - Capture, trials, compare commands
- [@headless-adapters](../headless-adapters@plaited_agent-eval-harness/SKILL.md) - Schema creation and validation
