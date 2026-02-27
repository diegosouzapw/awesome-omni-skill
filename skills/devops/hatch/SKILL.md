---
name: hatch
description: Provision exe.dev cloud VMs for development. Use when user wants to create projects, feature branches, spike prototypes, manage VMs, or deploy to Vercel/Convex. Triggers on "new project", "feature branch", "spike", "VM", "exe.dev", "cloud development".
compatibility: Designed for Claude Code. Requires pnpm installed locally.
metadata: {"openclaw": {"requires": {"bins": ["pnpm"]}, "emoji": "🐣"}, "author": "hatch", "version": "1.0"}
---

# Hatch - Cloud Development CLI

Hatch provisions exe.dev VMs with GitHub, Vercel, and Convex integration.

Hatch is installed at `~/.hatch-cli`. All commands must be run from that directory using `pnpm dev`.

## Commands

### List projects and VMs
```bash
cd ~/.hatch-cli && pnpm dev list --json
```
Use first to find project names. Returns JSON with projects array and vms array.

### Check status of VMs, spikes, and PRs
```bash
cd ~/.hatch-cli && pnpm dev status --json
```
Returns dashboard with VM liveness, spike progress, and PR review/CI status.

**Options:**
- `--json` - Output as JSON (recommended for agents)
- `--project <name>` - Filter to a specific project

Use after starting a spike to check progress, PR review status, and CI checks.
Automatically detects if a "running" spike has actually completed.

### Create new project
```bash
cd ~/.hatch-cli && pnpm dev new <project-name> --dry-run
cd ~/.hatch-cli && pnpm dev new <project-name> --confirm <token>
```
Creates a new project with GitHub repo, Vercel deployment, and Convex backend.
Takes 5-10 minutes. Returns Vercel URL when complete.

**Options:**
- `--dry-run` - Show what will be created and get a confirmation token
- `--confirm <token>` - Execute with a token from `--dry-run`
- `-f, --force` - Skip confirmation (interactive terminal only)

### Create feature VM (interactive development)
```bash
cd ~/.hatch-cli && pnpm dev feature <name> --project <project> --dry-run
cd ~/.hatch-cli && pnpm dev feature <name> --project <project> --confirm <token>
```
Creates isolated VM with git branch and a separate Convex project for isolation. Returns SSH host and preview URL.
User will SSH in and drive development with Claude Code.

**Options:**
- `--dry-run` - Show what will be created and get a confirmation token
- `--confirm <token>` - Execute with a token from `--dry-run`
- `-f, --force` - Skip confirmation (interactive terminal only)

### Autonomous spike (fire and forget)
```bash
cd ~/.hatch-cli && pnpm dev spike <name> --project <project> --prompt "<instructions>" --dry-run
cd ~/.hatch-cli && pnpm dev spike <name> --project <project> --prompt "<instructions>" --confirm <token>
```
Creates VM, runs Claude Agent SDK autonomously, creates PR when done.

**Options:**
- `--wait` - Block until spike completes (default: return immediately)
- `--timeout <minutes>` - Max time when using `--wait` (default: 240)
- `--json` - Output result as JSON
- `--dry-run` - Show what will be created and get a confirmation token
- `--confirm <token>` - Execute with a token from `--dry-run`
- `-f, --force` - Skip confirmation (interactive terminal only)

### Show detailed spike progress
```bash
cd ~/.hatch-cli && pnpm dev progress <feature> --project <project>
```
Shows detailed spike progress for a feature VM including plan steps checklist and recent log activity.

**Options:**
- `--json` - Output as JSON (recommended for agents)

Use after starting a spike to see plan step progress and recent activity without SSH.

### Clean up
```bash
cd ~/.hatch-cli && pnpm dev clean <name> --project <project> --dry-run
cd ~/.hatch-cli && pnpm dev clean <name> --project <project> --confirm <token>
```
Deletes VM and Convex feature project after PR is merged.

**Options:**
- `--dry-run` - Show what will be deleted and get a confirmation token
- `--confirm <token>` - Execute with a token from `--dry-run`
- `-f, --force` - Skip confirmation (interactive terminal only)

### Add existing project
```bash
cd ~/.hatch-cli && pnpm dev add <project-name>
```
Adds an existing GitHub/Vercel/Convex project to Hatch tracking.

### Clone project repo locally
```bash
cd ~/.hatch-cli && pnpm dev clone --project <name> [--path <dir>] [--pull] [--json]
```
Clones a project's GitHub repo to `~/projects/<name>/repo/`. If already cloned, pulls latest changes. Use `--pull` to only pull (skip clone logic). Use before spikes to give the agent fresh codebase context.

### Show VM connection info
```bash
cd ~/.hatch-cli && pnpm dev connect
```
Shows SSH connection details for active VMs.

### Generate config file
```bash
cd ~/.hatch-cli && pnpm dev config
```
Generates `hatch.json` config file with credentials and defaults.

Hatch uses an Anthropic API key (`anthropicApiKey` in the config JSON) for spike agent authentication. The config wizard prompts for this key during setup.

**Options:**
- `--project <name>` - Create per-project config at `~/.hatch/configs/<name>.json`
- `--refresh` - Refresh GitHub token (preserves orgs/teams/env vars; API keys don't expire)

### Per-project configuration
Per-project configs live at `~/.hatch/configs/<project-name>.json`. Commands with `--project` auto-resolve the right config.

```bash
# Create config for a specific project
cd ~/.hatch-cli && pnpm dev config --project my-app

# List all project configs
cd ~/.hatch-cli && pnpm dev config list --json

# Validate a project's tokens before use
cd ~/.hatch-cli && pnpm dev config check --project my-app --json

# Push project config to remote VM
cd ~/.hatch-cli && pnpm dev config-push <ssh-host> --project my-app
```

When `--project` is provided on feature/spike/clean commands, the matching config is used automatically. Falls back to `~/.hatch.json` if no project-specific config exists.

### Update hatch
```bash
cd ~/.hatch-cli && pnpm dev update
```
Pulls latest code, reinstalls dependencies, rebuilds, and updates OpenClaw skills if installed.

### Destroy project (HUMAN ONLY)
**DO NOT USE THIS COMMAND.** Project destruction must be performed manually by a human operator. If asked to destroy a project, instruct the user to run `hatch destroy <project-name>` themselves. The destroy command now requires a two-step `--dry-run` → `--confirm <token>` flow as an additional safeguard.

## Execution Plans

Spikes always create a structured execution plan before writing any code.

### How it works

1. The agent reads `docs/plans/_template.md` and project context (`docs/architecture.md`, `docs/patterns.md`)
2. Creates `docs/plans/<spike-name>.md` with goal, approach, and step-by-step checklist
3. Commits the plan as the first commit on the branch
4. Executes each step in order, checking boxes and logging decisions as it goes
5. Final commit marks the plan status as "completed"

### Continuing a spike

When using `--continue`, the agent reads the existing plan and resumes from the first unchecked step.

### Plan progress in status

`hatch status` shows plan progress when available:

```
    Plan:   3/5 steps completed
```

## When to Use Feature vs Spike

### Use `feature` when:
- The task is complex or requires exploration
- The user wants to learn the codebase
- Requirements are unclear and need iteration
- Multiple back-and-forth interactions are expected
- The user explicitly asks for interactive development

### Use `spike` when:
- The task is well-defined and can be described in a prompt
- The user wants a "fire and forget" experience
- Simple features: add a form, create an API endpoint, etc.
- The user asks for something to be "spiked" or done "autonomously"
- Time is limited and the user doesn't want to SSH in

**Rule of thumb:** If you can describe the task completely in 1-2 sentences, use spike. If you need to ask clarifying questions or the task has many unknowns, use feature.

**Crafting better spike prompts:** When running `spike --dry-run`, the repo is automatically cloned/pulled to a local path (shown as "Local mirror" in the output). Browse the codebase — especially `docs/architecture.md`, `docs/patterns.md`, and relevant source files — to write more specific, context-aware prompts that reference actual file paths, component names, and patterns used in the project.

## Iterating on a Spike

When a user wants to make changes to an existing spike (e.g., "add phone number to that form"), you can continue the spike instead of starting fresh.

### Checking for Active Spikes

```bash
cd ~/.hatch-cli && pnpm dev list --json
```

Look for VMs with `spikeStatus: "completed"` matching the project. These are eligible for continuation.

### Continuing a Spike

```bash
cd ~/.hatch-cli && pnpm dev spike <feature> --project <project> --continue <vm-name> --prompt "additional changes"
```

**Options:**
- `--continue <vm-name>` - Continue an existing spike on the specified VM
- `--wait` - Block until iteration completes
- `--json` - Output result as JSON

### What Happens on Continuation

The agent will:
1. Load context of all previous prompts from `~/spike-context.json`
2. Make changes based on the new prompt
3. Add new commits to the existing branch
4. Push to update the existing PR (no new PR created)

### When to Ask About Continuation

When the user's request relates to a recently completed spike:
1. Check for active spikes in the same project
2. Ask: "You have an active spike 'feature-name' with PR at [url]. Continue that spike with additional changes, or start a new one?"
3. If continuing, use `--continue <vm-name>`

### Continuation Limitations

- Can only continue spikes with `spikeStatus: "completed"`
- Cannot continue while a spike is still running
- If VM is unreachable, user must clean and start fresh

## Monitoring Spike Progress

After starting a spike, monitor with these SSH commands:

```bash
# Human-readable log (best for following along)
ssh <vm>.exe.xyz 'tail -f ~/spike.log'

# Structured progress events (JSON lines)
ssh <vm>.exe.xyz 'tail -f ~/spike-progress.jsonl'

# Check if done
ssh <vm>.exe.xyz 'test -f ~/spike-done && echo "Done" || echo "Running"'

# Get final result (includes cost)
ssh <vm>.exe.xyz 'cat ~/spike-result.json'

# Get PR URL
ssh <vm>.exe.xyz 'cat ~/pr-url.txt'
```

## Spike Output Files

The spike writes these files to the VM home directory:

| File | Description |
|------|-------------|
| `~/spike.log` | Human-readable progress log |
| `~/spike-progress.jsonl` | Structured tool use events (JSON lines) |
| `~/spike-result.json` | Final status, session ID, token usage, USD cost |
| `~/spike-done` | Marker file indicating completion |
| `~/pr-url.txt` | The created PR URL |

## Observability (Structured Logs)

Generated projects include structured logging. In development, the server logger writes JSON log entries to `~/.harness/logs/app.jsonl` on the VM.

### Log Query Commands

Run these from the project root on the VM:

| Command | Description |
|---------|-------------|
| `pnpm harness:logs` | Last 50 log entries (human-readable) |
| `pnpm harness:logs:errors` | Error-level entries only |
| `pnpm harness:logs:slow` | Requests slower than 200ms |
| `pnpm harness:logs:summary` | Aggregate stats by route |
| `pnpm harness:logs:clear` | Truncate log file |

Additional flags: `--route <path>`, `--since <5m|1h|30s>`, `--limit <n>`, `--json`.

## Cost Tracking

Spikes track token usage and cost in `~/spike-result.json`:

```json
{
  "status": "completed",
  "sessionId": "session_abc123",
  "cost": {
    "inputTokens": 45000,
    "outputTokens": 12000,
    "totalUsd": 0.0234
  }
}
```

Report costs to the user when a spike completes.

## Environment Safety

**CRITICAL: Always confirm with the human before taking action.** Never run provisioning, destructive, or deployment commands without first showing the human exactly what you plan to do and getting explicit approval.

Commands that create, modify, or destroy resources (`new`, `spike`, `feature`, `clean`, `destroy`) enforce a two-step `--dry-run` → `--confirm <token>` flow. Agents should use this instead of relying on manual approval of interactive prompts.

### Pre-flight check (required before any provisioning command)

Before running `hatch new`, `hatch feature`, `hatch spike`, `hatch clean`, or `hatch add`:

1. **Gather context**: Run `hatch list --json` and `hatch config check --project <name> --json` to understand the current state.
2. **Show the human a summary** that includes:
   - The exact command you plan to run (with `--dry-run`)
   - Which project and config will be used (`~/.hatch/configs/<name>.json` or `~/.hatch.json`)
   - Which GitHub org, Vercel team, and Convex deployment will be affected
   - For spikes: the full prompt text
   - For clean: what resources will be deleted (VM name, Convex project)
3. **Run with `--dry-run`** to get a confirmation token, then **run with `--confirm <token>`** after human approval.

### General rules

- **Never assume config**: Always pass `--project <name>` explicitly. Do not rely on the global fallback when per-project configs exist.
- **Cross-check project names**: The `--project` value must match both the project name in `hatch list` and the config filename in `~/.hatch/configs/`. Mismatches mean wrong credentials.
- **NEVER run `hatch destroy`**: This command permanently deletes Convex and Vercel projects. Only a human operator should run destroy. If a user asks you to destroy a project, tell them to run the command manually.

## Workflows

Every workflow follows the same pattern: **gather context → show the human → get approval → execute**. For spikes specifically: **gather context → read local repo → draft prompt → iterate with human on prompt → execute**.

### Create new project
```bash
# 1. Check which config will be used
cd ~/.hatch-cli && pnpm dev config list --json

# 2. Show the human: "I'll create project 'my-app' using config ~/.hatch/configs/my-app.json
#    (GitHub org: X, Vercel team: Y). This provisions a temporary VM. Proceed?"

# 3. Dry run to get a confirmation token:
cd ~/.hatch-cli && pnpm dev new my-app --dry-run

# 4. After approval, confirm with the token:
cd ~/.hatch-cli && pnpm dev new my-app --confirm <token>
```
Share Vercel URL when complete.

### Manual feature development
```bash
# 1. Gather context
cd ~/.hatch-cli && pnpm dev list --json
cd ~/.hatch-cli && pnpm dev config check --project my-app --json

# 2. Show the human: "I'll create feature VM 'my-feature' for project 'my-app'
#    using config ~/.hatch/configs/my-app.json (GitHub: org/my-app, Convex: my-app).
#    This creates a VM, git branch, and isolated Convex project. Proceed?"

# 3. Dry run to get a confirmation token:
cd ~/.hatch-cli && pnpm dev feature my-feature --project my-app --dry-run

# 4. After approval, confirm with the token:
cd ~/.hatch-cli && pnpm dev feature my-feature --project my-app --confirm <token>
```
Share SSH host so user can connect with Claude Code.

### Autonomous spike
```bash
# 1. Gather context
cd ~/.hatch-cli && pnpm dev list --json
cd ~/.hatch-cli && pnpm dev config check --project my-app --json

# 2. Show the human: "I'll spike 'my-feature' on project 'my-app'
#    using config ~/.hatch/configs/my-app.json.
#    Prompt: 'Add contact form'
#    This creates a VM, runs Claude autonomously, and opens a PR. Proceed?"

# 3. Dry run to get a confirmation token (also clones/pulls repo locally):
cd ~/.hatch-cli && pnpm dev spike my-feature --project my-app --prompt "Add contact form" --dry-run
# The dry-run output includes a "Local mirror" path (e.g. ~/projects/my-app/repo/).
# Read the local repo to understand the codebase before crafting your spike prompt.
# Key files to check: docs/architecture.md, docs/patterns.md, CLAUDE.md, and relevant source files.

# 4. Draft a detailed prompt based on what you learned from the codebase, then
#    SHOW IT TO THE HUMAN for review before executing. Example:
#
#    "Here's the spike prompt I'd like to run:
#
#    Add a contact form to the /contact page. Use the existing FormCard component
#    from packages/ui/src/components/form-card.tsx. Fields: name, email, message.
#    Submit via a Convex mutation in convex/contact.ts. Add success toast using
#    the existing useToast hook.
#
#    Want me to adjust anything before I run it?"
#
# Iterate with the human until they approve the prompt.

# 5. After the human approves the prompt, confirm with the token:
cd ~/.hatch-cli && pnpm dev spike my-feature --project my-app --prompt "<final approved prompt>" --confirm <token>

# 6. Check status (VM liveness, spike progress, PR review/CI)
cd ~/.hatch-cli && pnpm dev status --project my-app --json

# 7. Optionally monitor progress
ssh <vm>.exe.xyz 'tail -f ~/spike.log'

# 8. Check for completion
ssh <vm>.exe.xyz 'test -f ~/spike-done && cat ~/spike-result.json'

# 9. Clean up after PR is merged (dry-run first, then confirm)
cd ~/.hatch-cli && pnpm dev clean my-feature --project my-app --dry-run
cd ~/.hatch-cli && pnpm dev clean my-feature --project my-app --confirm <token>
```
Share PR URL when complete.

### Blocking spike (wait for completion)
```bash
cd ~/.hatch-cli && pnpm dev spike my-feature --project my-app --prompt "Add contact form" --wait --json
```
Returns full result including PR URL and cost when done.

## Anthropic API Key

Spikes use the `anthropicApiKey` from the project config (`~/.hatch/configs/<name>.json`). The key is injected inline via the SSH command — it is not written to the VM environment, so interactive `claude` sessions on the VM use your own subscription. If the API key is invalid, update it in the config file and retry.

## Error Handling

If a spike fails:
1. Check `~/spike.log` for error details
2. The VM remains running for debugging
3. User can SSH in and fix issues manually
4. Or clean up with `cd ~/.hatch-cli && pnpm dev clean` and try again

If the spike command itself fails (before agent starts):
- The command automatically rolls back and deletes the VM
- Check error message for the cause (missing config, auth failure, network issues, etc.)
- For auth failures: update `anthropicApiKey` in the project config (`~/.hatch/configs/<name>.json`) and retry

## Important: Do Not Modify VM Code Directly

NEVER SSH into a spike VM to make code changes directly. This bypasses the Claude agent
running on the VM which has full project context, proper tooling, and tracks its work.

Instead:
- Use `hatch spike --continue <vm-name> --prompt "description of changes"` to iterate
- The agent on the VM has Claude Code with Bash, Read, Write, Edit, Glob, and Grep tools
- Direct SSH modifications will conflict with the agent's work and won't be tracked

You may SSH into VMs only for **read-only monitoring** (e.g., `tail -f ~/spike.log`).
