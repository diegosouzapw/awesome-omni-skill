---
name: soulflow
description: General-purpose AI workflow framework for OpenClaw. Build custom multi-step workflows for any task — dev, ops, research, content, or automation. Ships with dev workflow examples.
homepage: https://github.com/0xtommythomas-dev/soulflow
metadata: 
  clawdbot:
    emoji: "⚙️"
    requires:
      bins: ["node"]
      config_files:
        - "~/.openclaw/openclaw.json"
    permissions:
      config: 
        read: ["~/.openclaw/openclaw.json"]
        write: ["~/.openclaw/openclaw.json"]
      gateway: modify
      agents: create
      filesystem: 
        read: ["~/.openclaw/workspace"]
        write: ["~/.openclaw/workspace/.soulflow", "~/.openclaw/agents/soulflow-worker"]
      credentials: inherit
    security_note: "Creates a dedicated 'soulflow-worker' agent with full tool access (read, write, edit, exec, browser) to execute workflow steps. Reads gateway config (~/.openclaw/openclaw.json) for authentication token. Modifies gateway config to add/manage worker agent. Worker inherits authProfiles from existing agents (grants access to external services like GitHub, cloud providers). All operations run locally with your existing OpenClaw permissions. Only install if you trust the skill author and have reviewed the code."
---

# SoulFlow — Workflow Framework for OpenClaw

**A framework for building custom AI workflows.** Each workflow is a series of steps that execute in isolated agent sessions with full tool access. Define your workflow in JSON, invoke it naturally, and let the agents handle the execution.

**What you can build:**
- Development pipelines (security audits, bug fixes, feature development)
- Content workflows (research → draft → edit → publish)
- Operations automation (deploy → verify → rollback-on-fail)
- Research pipelines (gather → analyze → synthesize → report)
- Any multi-step task that benefits from isolated, focused agent sessions

**Ships with 3 example dev workflows** to show you how it works. Build your own for anything.

## Quick Start

**Natural language (easiest):**
Just tell your agent what you need:
- "Run a security audit on my project at ~/myapp"
- "Fix this bug: users can't login with Google OAuth in ~/webapp"
- "Build a referral system for ~/webapp"

Your agent reads this SKILL.md and invokes SoulFlow automatically.

**Command line:**
```bash
cd ~/.openclaw/workspace/soulflow

# Run a security audit
node soulflow.js run security-audit "Audit the codebase at ~/project for vulnerabilities"

# Fix a bug
node soulflow.js run bug-fix "Login returns 500 when email has uppercase letters in ~/myapp"

# Build a feature
node soulflow.js run feature-dev "Add dark mode toggle to the settings page in ~/myapp"
```

## How It Works

SoulFlow connects to your local OpenClaw gateway via WebSocket and runs each workflow step as an isolated agent session. A dedicated `soulflow-worker` agent is auto-created with minimal context — no memory bleed from your main agent.

Each step:
1. Gets a fresh session (no context bloat)
2. Receives the task + output from previous steps
3. Has full tool access (read, write, edit, exec, browser)
4. Must complete its work and report results

**Auto-notifications (v1.1.0+):** When workflows complete, SoulFlow automatically notifies the main agent session with results. No need to manually check status.

## Example Workflows (Included)

**These are examples to show what's possible. Build your own for any domain.**

### security-audit
**Scan → Prioritize → Fix → Verify**
Development example: Reads your source files, identifies vulnerabilities by severity, applies fixes, then verifies them.

### bug-fix
**Triage → Fix → Verify**
Development example: Investigates the root cause by reading code, applies the fix, then verifies it didn't introduce regressions.

### feature-dev
**Plan → Implement → Review**
Development example: Architects the implementation plan, writes the code, then reviews for quality and correctness.

**Want content workflows? Research pipelines? Deploy automation?** Create your own `.workflow.json` — see Custom Workflows below.

## Commands

```bash
node soulflow.js run <workflow> "<task>"    # Run a workflow
node soulflow.js list                       # List available workflows
node soulflow.js runs                       # List past runs
node soulflow.js status [run-id]            # Check run status
node soulflow.js test                       # Test gateway connection
```

## Natural Language (via your agent)

**The agent knows how to invoke SoulFlow for you.** Just describe what you want:

**Security audits:**
- "Audit my app for security issues"
- "Check ~/myapp for vulnerabilities"
- "Scan the codebase for security problems"

**Bug fixes:**
- "Fix this bug: login fails when..."
- "There's a problem with the payment flow"
- "Users are seeing 500 errors when they..."

**Features:**
- "Build a referral system"
- "Add dark mode to the settings page"
- "Implement OAuth login with Google"

**How it works:**
1. You tell your agent what you need
2. Your agent reads this SKILL.md
3. Agent invokes `node soulflow.js run <workflow> "<task>"`
4. SoulFlow runs the workflow and reports back

**Pattern matching:** The agent matches your message to workflows:
- Security audit → keywords: "audit", "security", "scan", "vulnerabilit"
- Bug fix → keywords: "fix", "bug", "broken", "not working", "error"
- Feature dev → keywords: "build", "add", "implement", "create", "feature"

**No workflow matches?** Agent will ask which workflow you want or suggest creating a custom one.

## Custom Workflows

**You can create workflows for ANY task.** Define them in JSON and place in the `workflows/` directory.

### Creating via Chat

Tell your agent:
> "Create a SoulFlow workflow for [your use case]"

Examples:
- "Create a workflow for content publishing: research topic → draft article → edit → publish to blog"
- "Create a workflow for deployment: run tests → build → deploy → verify health checks → rollback if failed"
- "Create a workflow for weekly reports: gather metrics → analyze trends → generate summary → send email"

Your agent will:
1. Design the workflow steps
2. Write the `.workflow.json` file to `workflows/`
3. Show you how to run it

### Manual Creation

Create a `.workflow.json` file in the `workflows/` directory:

```json
{
  "id": "my-workflow",
  "name": "My Custom Workflow",
  "version": 1,
  "description": "What this workflow does",
  "steps": [
    {
      "id": "step1",
      "name": "First Step",
      "input": "Do this thing: {{task}}",
      "expects": "STATUS: done",
      "maxRetries": 1
    },
    {
      "id": "step2",
      "name": "Second Step",
      "input": "Now do this based on step 1:\n\n{{step1_output}}\n\nOriginal task: {{task}}",
      "expects": "STATUS: done",
      "maxRetries": 1
    }
  ]
}
```

### Variables

- `{{task}}` — The user's original task description
- `{{stepid_output}}` — Full output from a previous step (e.g. `{{scan_output}}`)
- Any `KEY: value` lines in step output become variables (e.g. `ROOT_CAUSE: ...` → `{{root_cause}}`)

### Prompt Tips

For best results, write prompts that:
- Explicitly tell the agent to use tools: "Use `read` to examine the file", "Use `edit` to apply the fix"
- Say "Do NOT just describe — actually do it"
- End with "When done, end with: STATUS: done"

## Architecture

- **Zero dependencies** — Pure Node.js 22 (native WebSocket)
- **Gateway-native** — Connects via WebSocket with challenge-response auth
- **Session isolation** — Each step in a fresh session
- **Dedicated worker** — Auto-creates `soulflow-worker` agent with minimal brain files
- **JSON state** — Run history saved to `~/.openclaw/workspace/.soulflow/runs/`
- **10-minute timeout** per step (configurable)

## Requirements

- OpenClaw 2026.2.x or later
- Node.js 22+ (for native WebSocket)
- Gateway with token auth configured

## Security & Permissions

**What SoulFlow does to your OpenClaw instance:**

1. **Reads your gateway config** (`~/.openclaw/openclaw.json`) to obtain the authentication token needed to connect via WebSocket
2. **Modifies your gateway config** (`~/.openclaw/openclaw.json`) via `config.patch` to register the soulflow-worker agent
3. **Creates a dedicated worker agent** (`soulflow-worker`) with minimal brain files (SOUL.md only, no memory/history)
4. **Copies authProfiles from existing agents** — Worker inherits credentials for external services (GitHub, cloud providers, etc.) that your other agents use
5. **Grants the worker full tool access** (read, write, edit, exec, browser) — this is required for workflows to actually perform tasks
6. **Writes run state** to `~/.openclaw/workspace/.soulflow/runs/` as JSON files

**Why these permissions are needed:**
- **Config read/write**: Required to authenticate with the gateway and register the worker agent (same as `openclaw` CLI tool)
- **Agent creation**: Each workflow step runs in an isolated session to prevent context bleed
- **authProfiles inheritance**: Allows workflows to interact with external services (e.g., git push, cloud API calls) using your existing credentials
- **Full tools**: Workflows need real capabilities (e.g., security-audit reads files, bug-fix edits code, deploy-pipeline pushes to git)
- **Filesystem write**: Stores workflow history and allows workflows to create/modify files

**Security considerations:**
- Worker agent has NO access to your main agent's memory or history
- Worker DOES inherit your external service credentials (authProfiles) — can access GitHub, cloud APIs, etc.
- Workflows run with YOUR permissions (same as running commands yourself)
- Malicious workflows could read/modify files, run commands, or access external services
- **Only install SoulFlow if you trust the skill author** (review code on GitHub first)
- **Only run workflows you trust** — custom workflows from untrusted sources could exfiltrate data or misuse credentials
- Run SoulFlow in isolated/sandboxed environments if processing untrusted workflows

**Recommended practices:**
- Review built-in workflows before first use (especially security-audit and bug-fix)
- Inspect custom `.workflow.json` files before running
- Review GitHub repo (https://github.com/0xtommythomas-dev/soulflow) before installation
- Run on non-production OpenClaw instances when testing new workflows
- Back up important files before running workflows that modify code
- Use BYOK (bring your own keys) mode if you want isolated credentials per workflow
- Monitor `~/.openclaw/workspace/.soulflow/runs/` for workflow execution logs

---

## For Agents: How to Invoke SoulFlow

When the user requests a workflow (security audit, bug fix, feature build, etc.), you should:

1. **Identify the workflow** by matching keywords:
   - Security audit: "audit", "security", "scan", "vulnerabilit"
   - Bug fix: "fix", "bug", "broken", "not working", "error"
   - Feature dev: "build", "add", "implement", "create", "feature"
   - Custom: check `workflows/*.workflow.json` for other options

2. **Extract the task description** — the user's description of what they want done

3. **Invoke SoulFlow** using exec:
   ```bash
   cd /root/.openclaw/workspace/soulflow && node soulflow.js run <workflow> "<task>"
   ```

4. **Monitor the run** — SoulFlow will output the run ID, then show progress as each step completes

5. **Report results** — When complete, relay the final status to the user

**Example:**
```
User: "Run a security audit on ~/myapp"
You: [exec] cd /root/.openclaw/workspace/soulflow && node soulflow.js run security-audit "Audit ~/myapp for vulnerabilities"
```

**Creating workflows for users:**
If the user asks you to create a custom workflow:
1. Design the workflow steps based on their requirements
2. Write a `.workflow.json` file to `/root/.openclaw/workspace/soulflow/workflows/`
3. Show them how to run it

See CONTRIBUTING.md for workflow design best practices.