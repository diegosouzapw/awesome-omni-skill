---
name: quickstart
description: 'New user onboarding. Detect setup, explain what AgentOps does, give one next action. Under 30 seconds. Triggers: "quickstart", "get started", "onboarding", "how do I start".'
skill_api_version: 1
metadata:
  tier: session
  dependencies: []
---

# /quickstart

> **One job:** Tell a new user what AgentOps does and what to do first. Fast.

**YOU MUST EXECUTE THIS WORKFLOW. Do not just describe it.**

## Execution Steps

### Step 1: Detect setup

```bash
git rev-parse --is-inside-work-tree >/dev/null 2>&1 && echo "GIT=true" || echo "GIT=false"
command -v ao >/dev/null && echo "AO=true" || echo "AO=false"
command -v bd >/dev/null && echo "BD=true" || echo "BD=false"
[ -d .agents ] && echo "AGENTS=true" || echo "AGENTS=false"
```

### Step 2: Show what AgentOps does

Output exactly this (no additions, no diagrams):

```
AgentOps gives your coding agent three things it doesn't have by default:

  Memory    — sessions accumulate learnings in .agents/ and inject them back
  Judgment  — /council spawns independent judges to validate plans and code
  Workflow  — /rpi chains research → plan → implement → validate in one command

Key skills: /rpi  /research  /plan  /implement  /vibe  /council  /swarm  /status
Full reference: /quickstart --catalog
```

### Step 3: One next action

Match the first row that applies. Output only that message — nothing else.

| Condition | Message |
|-----------|---------|
| GIT=false | "⚠ Not in a git repo. Run `git init` first." |
| AO=false | "📦 Install ao CLI first:\n  brew tap boshu2/agentops https://github.com/boshu2/homebrew-agentops\n  brew install agentops\n  ao init --hooks && ao seed\nThen: `/rpi \"a small goal\"` to run your first cycle." |
| AGENTS=false | "🌱 ao is installed but not initialized here.\n  ao init --hooks && ao seed\nThen: `/rpi \"a small goal\"` to run your first cycle." |
| BD=false | "✅ Flywheel active. Start now:\n  `/rpi \"your goal\"` — full research → plan → implement pipeline\n  `/vibe recent` — validate recent changes\n  `/research <topic>` — explore the codebase\n  Want issue tracking? `brew install boshu2/agentops/beads && bd init --prefix <prefix>`" |
| BD=true | "✅ Full stack ready.\n  `bd ready` — see open work\n  `/rpi \"your goal\"` — start a new goal from scratch\n  `/status` — see current session state" |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Skills not installed | `bash <(curl -fsSL https://raw.githubusercontent.com/boshu2/agentops/main/scripts/install.sh)` |
| Flywheel count is 0 | First session — run `/rpi "a small goal"` to start it |
| Want the full skill catalog | Ask: "show me all the skills" or see `references/full-catalog.md` |

## Reference Documents

- [references/getting-started.md](references/getting-started.md)
- [references/troubleshooting.md](references/troubleshooting.md)
- [references/full-catalog.md](references/full-catalog.md)
