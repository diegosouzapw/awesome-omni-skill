---
name: agent-dispatch
description: Lightweight agent registry and JIT router. Consult BEFORE performing specialized work such as code review, security audit, debugging, refactoring, performance optimization, infrastructure, data analysis, API design, testing, documentation, or any domain-specific engineering task. Maps tasks to specialized subagents, downloading them on demand if not installed locally.
version: 2.0.0
user-invocable: true
metadata:
  openclaw:
    homepage: https://github.com/userFRM/agent-dispatch
    always: false
    skillKey: agent-dispatch
---

# Agent dispatch

You have access to a registry of 130+ specialized subagents. **Before doing specialized work yourself, check this index and dispatch to the appropriate agent.** If the agent is not installed locally, download it on the fly.

## JIT dispatch procedure

When you encounter a specialized task, follow these steps in order.

### Step 1: index lookup

Scan the agent index at the bottom of this file. Format: `keyword = "agent-name:category"`.
Extract the agent name (before the colon) and category key (after the colon).
If no keyword matches, do the work yourself.

### Step 2: check local cache

Check if the agent file exists locally:
```bash
ls "${AGENTS_DIR:-$HOME/.claude/agents}/AGENT_NAME.md" 2>/dev/null
```
If the file exists, skip to step 4.

### Step 3: download the agent

Construct the download URL from these parts:
- Base: `https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories`
- Directory: look up the category key in the mapping below
- File: `AGENT_NAME.md`

Download:
```bash
mkdir -p "${AGENTS_DIR:-$HOME/.claude/agents}" && curl -sfL "URL" -o "${AGENTS_DIR:-$HOME/.claude/agents}/AGENT_NAME.md"
```

**If download fails** (non-zero exit or empty file):
- Run: `rm -f "${AGENTS_DIR:-$HOME/.claude/agents}/AGENT_NAME.md"`
- Tell the user: "Could not download AGENT_NAME — handling this task directly."
- Do the work yourself. Do not retry.

**Validation**: if the downloaded file does not start with `---` (YAML frontmatter), treat it as corrupt, delete it, and handle the task yourself.

### Step 4: read and dispatch

Read the agent file. Extract everything after the YAML frontmatter (after the second `---` line).
Pass that full text as the prompt to the **Task** tool, prepending the specific work request.
Use a general-purpose subagent with the full prompt inline — do not reference the agent by registered name.

### Step 5: return results

When the Task completes, relay its output to the user in the main conversation.
The downloaded agent file stays cached in the agents directory for future sessions.

### When to skip dispatch

- The task is trivial (one-liner, quick fix, simple question)
- You need tight back-and-forth with the user
- The task spans multiple domains simultaneously
- The user explicitly asks you to handle it directly

## Category directory mapping

| Key | Directory |
|-----|-----------|
| core | 01-core-development |
| languages | 02-language-specialists |
| infra | 03-infrastructure |
| quality | 04-quality-security |
| data | 05-data-ai |
| devex | 06-developer-experience |
| domains | 07-specialized-domains |
| business | 08-business-product |
| meta | 09-meta-orchestration |
| research | 10-research-analysis |

## Platform-specific paths

| Platform | Agent location | Dispatch mechanism |
|----------|---------------|-------------------|
| Claude Code | `~/.claude/agents/` | `Task` tool with inline prompt |
| OpenClaw | `~/.openclaw/workspace/` | `sessions_spawn` tool |
| Cursor | `.cursor/agents/` | Agent invocation |
| Codex | `.codex/agents/` | Agent invocation |

If your platform does not have a programmatic dispatch tool, instruct the user to invoke the agent manually and pause until it completes.

## Known limitations

- Each keyword maps to exactly one agent (TOML requires unique keys)
- Downloaded agents are cached permanently; delete manually to force re-download
- If you are offline, agents not already cached will be unavailable — handle the task yourself

## Agent index

Scan by **keyword**. Format: `keyword = "agent-name:category"`.

```toml
# -- Code quality -----------------------------------------------
review          = "code-reviewer:quality"
refactor        = "refactoring-specialist:devex"
lint            = "code-reviewer:quality"
code-quality    = "code-reviewer:quality"
simplify        = "refactoring-specialist:devex"
dead-code       = "refactoring-specialist:devex"

# -- Security ---------------------------------------------------
security        = "security-auditor:quality"
vulnerability   = "security-auditor:quality"
owasp           = "security-auditor:quality"
secrets         = "security-auditor:quality"
penetration     = "penetration-tester:quality"
audit           = "security-auditor:quality"
compliance      = "compliance-auditor:quality"
ad-security     = "ad-security-reviewer:quality"

# -- Debugging and errors ---------------------------------------
debug           = "debugger:quality"
error           = "error-detective:quality"
stacktrace      = "debugger:quality"
crash           = "debugger:quality"
troubleshoot    = "debugger:quality"

# -- Testing ----------------------------------------------------
test            = "qa-expert:quality"
e2e             = "test-automator:quality"
unit-test       = "test-automator:quality"
integration     = "test-automator:quality"
accessibility   = "accessibility-tester:quality"

# -- Performance ------------------------------------------------
performance     = "performance-engineer:quality"
optimize        = "performance-engineer:quality"
profiling       = "performance-engineer:quality"
bottleneck      = "performance-engineer:quality"
chaos           = "chaos-engineer:quality"

# -- Architecture and design ------------------------------------
api-design      = "api-designer:core"
architecture    = "architect-reviewer:quality"
microservices   = "microservices-architect:core"
graphql         = "graphql-architect:core"
websocket       = "websocket-engineer:core"

# -- Frontend ---------------------------------------------------
react           = "react-specialist:languages"
nextjs          = "nextjs-developer:languages"
vue             = "vue-expert:languages"
angular         = "angular-architect:languages"
ui              = "ui-designer:core"
frontend        = "frontend-developer:core"
electron        = "electron-pro:core"

# -- Backend ----------------------------------------------------
backend         = "backend-developer:core"
django          = "django-developer:languages"
rails           = "rails-expert:languages"
spring          = "spring-boot-engineer:languages"
laravel         = "laravel-specialist:languages"
dotnet          = "dotnet-core-expert:languages"

# -- Languages --------------------------------------------------
typescript      = "typescript-pro:languages"
javascript      = "javascript-pro:languages"
python          = "python-pro:languages"
rust            = "rust-engineer:languages"
golang          = "golang-pro:languages"
java            = "java-architect:languages"
kotlin          = "kotlin-specialist:languages"
swift           = "swift-expert:languages"
cpp             = "cpp-pro:languages"
csharp          = "csharp-developer:languages"
elixir          = "elixir-expert:languages"
php             = "php-pro:languages"
sql             = "sql-pro:languages"
flutter         = "flutter-expert:languages"
powershell      = "powershell-7-expert:languages"

# -- Infrastructure ---------------------------------------------
docker          = "docker-expert:infra"
kubernetes      = "kubernetes-specialist:infra"
terraform       = "terraform-engineer:infra"
terragrunt      = "terragrunt-expert:infra"
cloud           = "cloud-architect:infra"
aws             = "cloud-architect:infra"
azure           = "azure-infra-engineer:infra"
cicd            = "deployment-engineer:infra"
devops          = "devops-engineer:infra"
sre             = "sre-engineer:infra"
platform        = "platform-engineer:infra"
network         = "network-engineer:infra"
incident        = "devops-incident-responder:infra"
database        = "database-administrator:infra"

# -- Data and AI ------------------------------------------------
data-analysis   = "data-analyst:data"
data-eng        = "data-engineer:data"
data-science    = "data-scientist:data"
machine-learning = "machine-learning-engineer:data"
ml-ops          = "mlops-engineer:data"
llm             = "llm-architect:data"
nlp             = "nlp-engineer:data"
prompt-eng      = "prompt-engineer:data"
postgres        = "postgres-pro:data"
db-optimize     = "database-optimizer:data"

# -- Developer experience ---------------------------------------
documentation   = "documentation-engineer:devex"
cli             = "cli-developer:devex"
build           = "build-engineer:devex"
dependencies    = "dependency-manager:devex"
git-workflow    = "git-workflow-manager:devex"
dx              = "dx-optimizer:devex"
legacy          = "legacy-modernizer:devex"
mcp             = "mcp-developer:devex"
tooling         = "tooling-engineer:devex"

# -- Specialized domains ----------------------------------------
blockchain      = "blockchain-developer:domains"
fintech         = "fintech-engineer:domains"
gamedev         = "game-developer:domains"
iot             = "iot-engineer:domains"
embedded        = "embedded-systems:domains"
payments        = "payment-integration:domains"
seo             = "seo-specialist:domains"
mobile          = "mobile-app-developer:domains"

# -- Business and product ---------------------------------------
product         = "product-manager:business"
project         = "project-manager:business"
technical-write = "technical-writer:business"
ux-research     = "ux-researcher:business"
scrum           = "scrum-master:business"
business        = "business-analyst:business"

# -- Orchestration ----------------------------------------------
coordinate      = "multi-agent-coordinator:meta"
organize        = "agent-organizer:meta"
workflow        = "workflow-orchestrator:meta"
distribute      = "task-distributor:meta"
```
