---
name: protext
description: Dynamic context management for AI agents. Invoke /protext at session
  start to load token-efficient project orientation. Use when (1) Starting a session
  and need quick orientation, (2) Handing off between sessions, (3) Managing context
  for multi-scope projects (dev/ops/security), (4) Linking related projects,
  (5) Reducing token usage while maintaining awareness. Primary command is /protext
  to load context. Other commands include protext init, status, scope, handoff,
  extract, link.
---

# Protext: Dynamic Context Management

Protext provides a three-layer context hierarchy for token-efficient AI agent orientation.

## Core Concept

```
PROTEXT.md (Layer 0)  →  .protext/index.yaml (Layer 1)  →  Deep Context (Layer 2)
     ~500 tokens              Signposts only                 Full docs/memory
     Always loaded            On-demand hints                Explicit extraction
```

**Separation of Concerns:**
- Behavior file = Agent behavior (how to act) — stable, rarely changes
  - `CLAUDE.md` (Claude Code) · `GEMINI.md` (Gemini CLI) · `AGENTS.md` (Codex/OpenCode)
- `PROTEXT.md` = Project state (what's happening) — dynamic, session-aware

## File Structure

```
project-root/
├── PROTEXT.md                  # Layer 0: Orientation (~500 tokens)
├── CLAUDE.md / GEMINI.md / AGENTS.md   # Existing: Behavior instructions (platform-specific)
└── .protext/
    ├── index.yaml              # Layer 1: Extraction signposts
    ├── handoff.md              # Session continuity
    ├── scopes/
    │   ├── ops.md              # Operations context
    │   ├── dev.md              # Development context
    │   └── security.md         # Security context
    └── config.yaml             # Protext settings
```

## PROTEXT.md Format

```markdown
# Protext: [Project Name]
> Generated: YYYY-MM-DD | Scope: [active-scope] | Tokens: ~XXX

## Identity
[1-2 sentences: What is this project/system?]

## Current State
Active: [current work] | Blocked: [blockers] | Recent: [last completed]

## Hot Context
- [Critical point 1]
- [Critical point 2]
- [Critical point 3]

## Scope Signals
- `@ops` → .protext/scopes/ops.md
- `@security` → .protext/scopes/security.md
- `@deep:[name]` → Extract from index

## Links
- `[path]` → [rel-type] | [note]

## Handoff
Last: [summary] | Next: [suggested] | Caution: [warnings]
```

## Commands

### `/protext` (Primary - Session Start)

**Load project orientation.** This is the main entry point - invoke at session start.

```text
/protext              # Load PROTEXT.md + active scope + handoff status
/protext @security    # Load with security scope
/protext --full       # Include available extractions list
```

**What it does:**
1. Reads and displays PROTEXT.md (orientation layer)
2. Shows handoff status (FRESH/AGING/STALE)
3. Loads active scope context
4. Lists available deep extractions (with `--full`)

**Cross-platform:** Works as slash command on Claude Code, or natural language on other platforms:
- "Load protext"
- "Show me the project context"
- "What's the current state?"

**If no PROTEXT.md exists:** Inform the user and offer to run `protext init` to bootstrap one. Do not fail silently.

---

### `protext init`

Initialize protext in a project. Reads existing CLAUDE.md to bootstrap.

```text
# Standard mode
"Initialize protext for this project"
"Set up protext here"

# Parent mode (aggregates children)
"protext init --parent"
```

Creates: PROTEXT.md, .protext/ directory with full structure.

**Parent mode:** Scans for child `.protext/` directories, aggregates their status into `## Child Projects` section. One-level hierarchy only.

### `protext status`

Display current protext state.

```text
"Show protext status"
"What's the current context state?"
```

Shows: Tier, active scope, handoff age, token budget, available extractions.

### `protext scope [name]`

Switch active scope context.

```text
"Switch to ops scope"
"Focus on security context"
"@security"  # Shorthand
```

### `protext handoff`

Capture or display session handoff. **User-initiated only** — no auto-capture, no TTL enforcement.

```text
"Capture handoff: stopped mid-refactor, next step is testing"
"What was the last handoff?"
```

Handoff is an optional scratchpad. Only created when user explicitly captures.

### `protext extract [name]`

Pull deep context from index.

```text
"Extract network context"
"@deep:services"  # Shorthand
```

Agent receives extraction suggestion; confirm to load.

### `protext link [path]`

Add a cross-project link. Guided flow — agent asks for relationship type and note.

```text
"Link this project to ../skills-validator"
"protext link ../homelab"
"What projects are linked?"
```

Relationship types with spatial validation:
- `child` — Subdirectory (./name) — aggregates status
- `parent` — Ancestor (../) — no aggregation
- `sibling` — Adjacent (../name) — no aggregation
- `peer` — Any path (relative preferred) — no aggregation

**Path preference:** Use relative paths (`../sibling`, `./child`) wherever possible — they are portable across nodes and OS mount points. Absolute paths may break on machines with different prefixes (e.g., `/mnt/ops` on Linux vs `/Volumes/ops` on macOS).

Validates path patterns with regex enforcement (ERROR if spatial rules violated). Max 5 lateral links. See **Spatial Validation Rules** in `references/commands.md` for pattern specs, validation flow, and edge cases.

### `protext refresh --children`

Re-aggregate child status in parent protext. **User-initiated only** — no auto-refresh.

```text
"protext refresh --children"
"Update children status"
```

Scans child `.protext/` directories, extracts status from markers (or headings fallback), updates parent `## Child Projects` section.

## Parent Protext

Meta-projects that aggregate multiple child protexts. Created with `protext init --parent`.

**Key features:**
- `## Child Projects` section lists all children with status (active/idle/stale)
- Child status based on PROTEXT.md modification time (< 7 days = active)
- No extraction index (children have their own)
- `protext refresh --children` re-aggregates (explicit user command only)
- One-level hierarchy: parent → children only

**Marker extraction:** Parent uses `<!-- marker:identity -->`, `<!-- marker:state -->` from children. Falls back to heading-based parsing if markers absent (backward compatible).

**Zero auto-execution:** Parent never auto-refreshes on load. Always explicit.

## Extraction Modes

Configure in `.protext/config.yaml`:

```yaml
extraction_mode: suggest  # suggest | auto | confirm
token_budget: 2000        # Max extraction tokens per session
```

- **suggest** (default): Agent sees "context available: X" but doesn't auto-load
- **auto**: Keyword triggers load (opt-in, risky for token budget)
- **confirm**: Agent proposes extraction, user must approve

## Index Schema

`.protext/index.yaml`:

```yaml
extractions:
  network:
    source: docs/NETWORK.md
    triggers: [dns, ip, tailscale, mesh, routing]
    summary: "IPs, DNS config, mesh nodes"
    tokens: ~600

  services:
    source: docs/SERVICES.md
    triggers: [docker, container, service, port]
    summary: "Docker services, ports, domains"
    tokens: ~800

  secrets:
    source: docs/SECRETS.md
    triggers: [secret, credential, infisical, auth]
    summary: "Secrets management, auth patterns"
    tokens: ~400
```

**Limits:** Max 20 extractions per project.

## Scopes

Scopes provide domain-specific orientation. Max 5 per project.

Default scopes:
- `ops` — Infrastructure, deployment, monitoring
- `dev` — Development workflow, code patterns
- `security` — Auth, secrets, vulnerabilities
- `project` — Project-specific context (custom)

Scope file format (`.protext/scopes/ops.md`):

```markdown
# Scope: Operations

## Focus
Infrastructure management, service health, deployment.

## Key Resources
- [Service config paths]
- [Key infrastructure locations]

## Current Priorities
1. [Priority 1]
2. [Priority 2]

## Cautions
- [Ops-specific warnings]
```

## Handoff Protocol

**User-initiated only** — handoff is an optional scratchpad for session continuity.

`.protext/handoff.md` format:

```markdown
# Session Handoff
> Updated: YYYY-MM-DDTHH:MM

## Last Session
**Completed:**
- [Task 1]
- [Task 2]

**In Progress:**
- [Task] (stopped at: [point])

**Deferred:**
- [Task] (blocked by: [reason])

## Cautions
- [Warning 1]
- [Warning 2]

## Agent Notes
[Observations that might help next session]
```

Behavior:
- No auto-capture at session end
- No TTL warnings based on age
- Created only when user runs `protext handoff capture`
- Agents do not automatically update handoff

## Progressive Tiers

### Beginner
- PROTEXT.md only
- No scopes, extractions, or handoff
- Direct editing

### Intermediate
- PROTEXT.md + handoff.md
- Session continuity
- No extraction index

### Advanced (Default)
- Full feature set
- Scopes, extractions, token budgets
- Config-driven behavior

## Constraints

| Limit | Value | Rationale |
|-------|-------|-----------|
| Max scopes | 5 | Prevent fragmentation |
| Max lateral links | 5 | Keep peer/sibling/reference list concise |
| Max child links | Unlimited | Structure dictates count |
| Max extractions | 20 | Index stays scannable |
| Token budget | 2000 | Default per-session limit |
| Handoff TTL | 48h | Legacy, no longer enforced |
| PROTEXT.md size | ~500 tokens | Quick orientation |
| Hierarchy depth | 1 level | Parent → children only |

## Integration Patterns

### With Behavior Files

The platform behavior file (`CLAUDE.md`, `GEMINI.md`, `AGENTS.md`) provides behavioral instructions (stable):
```markdown
# CLAUDE.md  (or GEMINI.md / AGENTS.md)
## How to Work
- Validate Caddy config before reloading
- Never hardcode secrets
```

PROTEXT.md provides state orientation (dynamic):
```markdown
# PROTEXT.md
## Current State
Active: Caddy refactor | Recent: Pi-hole v6 migration
```

### With Memory Systems

Protext complements but doesn't replace:
- **memory/** — Long-term learnings, patterns
- **PROTEXT.md** — Current session orientation

## Scripts

Requires **Python 3.8+**. No external packages needed (yaml parsed with fallback).

- `scripts/init_protext.py` — Bootstrap protext in a project
- `scripts/protext_status.py` — Display current state

## Reference Files

Consult these only when deeper detail is needed:

- `references/formats.md` — Read when creating or editing PROTEXT.md, index.yaml, handoff.md, or scope files. Contains complete templates and field guidelines.
- `references/commands.md` — Read when implementing command handling or troubleshooting. Contains full syntax, flags, error messages, and natural language alternatives.
