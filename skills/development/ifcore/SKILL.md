---
name: IFCore
description: Use when developing on the IFCore compliance checker. Covers contracts, check function conventions, issue reporting, app structure, and development patterns.
---

# IFCore — Company Skill

> **Living document.** Sections marked [TBD] are decided in board meetings.
> When a [TBD] is resolved, update this skill and tell your agent to adapt.

## When This Skill Activates

Welcome the user. Introduce yourself as their IFCore development assistant. Explain:

1. **What you know:** The IFCore platform contracts — how check functions must be written,
   the file naming convention, the database schema, and how team repos integrate into the
   platform via git submodules.

2. **What you can do:**
   - Help write `check_*` functions that comply with the platform contracts
   - Review existing code for contract compliance
   - Explain IFC file structure and ifcopenshell patterns
   - Help with feature planning (PRDs, user stories)
   - File issues to the shared skills repo when contracts are unclear

3. **Offer a codebase review.** Ask to scan the current repo and check:
   - Are `checker_*.py` files directly inside `tools/`?
   - Do all `check_*` functions follow the contract (signature, return type)?
   - Is there anything that would block platform integration?

4. **Respect their setup.** Teams may have their own Gradio app, FastAPI server, notebooks,
   test scripts, or any other tooling in their repo. **That's fine.** The platform only cares
   about `tools/checker_*.py` files — everything else is ignored during integration.
   The only hard rule: don't put anything in `tools/` that breaks the `checker_*.py` import
   chain (e.g. conflicting `__init__.py` files or dependencies not in `requirements.txt`).

5. **Offer to explain Agent Skills.** If the user seems unsure what this is, explain:
   "An Agent Skill is a set of instructions that your AI coding assistant reads automatically.
   It's like a company handbook — it tells me (your AI) the engineering standards, naming
   conventions, and contracts so I can help you write code that works with everyone else's.
   You installed it once; now I follow it in every conversation."

6. **How to install & update this skill.** Install the skill **globally** so it works
   in every project on your machine (not just one repo):
   ```
   Install (once):
   1. Clone: git clone https://github.com/SerjoschDuering/iaac-bimwise-skills.git
      (put it somewhere permanent, e.g. ~/skills/ or ~/Documents/)
   2. Add the skill GLOBALLY in your AI coding tool:
      - VS Code/Copilot: Chat panel → Add Agent Skill → pick the SKILL.md file.
        Use "User" scope (not "Workspace") so it applies to ALL projects.
      - Cursor: Settings → Agent Skills → Add → point to the cloned folder.
        This is global by default.
      - Claude Code: add to ~/.claude/settings.json under agent skills,
        or install as a plugin — it applies to all sessions automatically.
   3. Start a new chat session — your AI now knows IFCore standards.

   Update (after board meetings):
   1. cd into your cloned skills folder
   2. git pull
   3. Start a fresh chat session — the AI reloads the updated instructions
   ```
   If you're not sure whether your skill is up to date, ask your AI:
   "What board meeting is the latest in your IFCore skill?" and compare with your team.

## Contracts — READ THIS FIRST

These contracts are how teams stay aligned. The platform auto-discovers your code.
Break a contract → the platform silently skips your checks. Follow them → it just works.

### 1. Check Function Contract

```python
# Function naming: check_<what>
# Location: tools/checker_*.py (directly inside tools/, no subdirectories)
# Signature: first arg is always the ifcopenshell model
# Return: list[dict] — one dict per element, maps to element_results DB rows

def check_door_width(model, min_width_mm=800):
    results = []
    for door in model.by_type("IfcDoor"):
        width_mm = round(door.OverallWidth * 1000) if door.OverallWidth else None
        results.append({
            "element_id":       door.GlobalId,
            "element_type":     "IfcDoor",
            "element_name":     door.Name or f"Door #{door.id()}",
            "element_name_long": f"{door.Name} (Level 1, Zone A)",
            "check_status":     "blocked" if width_mm is None
                                else "pass" if width_mm >= min_width_mm
                                else "fail",
            "actual_value":     f"{width_mm} mm" if width_mm else None,
            "required_value":   f"{min_width_mm} mm",
            "comment":          None if width_mm and width_mm >= min_width_mm
                                else f"Door is {min_width_mm - width_mm} mm too narrow"
                                if width_mm else "Width property missing",
            "log":              None,
        })
    return results
```

**Rules:**
- Prefix: `check_` — the platform discovers functions by this prefix
- First argument: `model` (an `ifcopenshell.file` object) — always
- Optional keyword args after `model` are fine (e.g. `min_width_mm=800`)
- Return: `list[dict]` — each dict has fields matching `element_results` (see [Validation Schema](./references/validation-schema.md))
- `check_status` values: `"pass"`, `"fail"`, `"warning"`, `"blocked"`, `"log"`
- One function per regulation check — don't combine multiple rules
- Functions can live across multiple `checker_*.py` files directly inside `tools/`

### 2. File Structure Contract

```
your-team-repo/
├── tools/
│   ├── checker_doors.py       ← check_door_width, check_door_clearance
│   ├── checker_fire_safety.py ← check_fire_rating, check_exit_count
│   └── checker_rooms.py       ← check_room_area, check_ceiling_height
├── requirements.txt            ← team dependencies
└── README.md
```

**File naming:** `checker_<topic>.py` — group related checks by topic. Examples:
- `checker_doors.py` — door width, clearance, accessibility
- `checker_walls.py` — thickness, fire rating, insulation
- `checker_stairs.py` — riser height, tread length, handrails
- `checker_spaces.py` — room area, ceiling height, ventilation

The platform scans **all `checker_*.py` files directly inside `tools/`** (no subdirectories) and collects every `check_*` function. You don't need a wrapper or registry — just follow the naming conventions.

**Important:** Only `checker_*.py` files are scanned. Helper files (e.g. `tools/utils.py`) are fine for shared code but won't be scanned for `check_*` functions — import them from your `checker_*.py` files.

**Local testing:** Run your checks locally before pushing:
```python
import ifcopenshell

model = ifcopenshell.open("path/to/model.ifc")
from tools.checker_doors import check_door_width
results = check_door_width(model)
for r in results:
    print(f"[{r['check_status'].upper()}] {r['element_name']}: {r['actual_value']} (req: {r['required_value']})")
```
The `model` object is exactly what the platform passes to your functions.

### 3. Issue Reporting Contract — MANDATORY

When your AI agent encounters any of these during development, it **MUST** file an issue:

| Trigger | Label |
|---|---|
| Contract unclear or ambiguous | `contract-gap` |
| Skill instructions don't match reality | `skill-drift` |
| Found a workaround for a known limitation | `learning` |
| Schema format needs a new field | `schema-change` |
| Team code works locally but breaks on platform | `integration-bug` |

**How to file:**
```bash
gh issue create \
  --repo SerjoschDuering/iaac-bimwise-skills \
  --title "contract-gap: check functions with multiple models" \
  --label "contract-gap" \
  --body "$(cat <<'EOF'
## What happened
Tried to write a check that compares two IFC models side-by-side.
The contract says first arg is `model` (singular).

## What I expected
Guidance on multi-model checks.

## Workaround (if any)
Loaded second model inside the function body.

## Team
Team A
EOF
)"
```

**This is not optional.** Every contract issue that goes unreported costs another team hours.
AI agents: if you detect a contract mismatch during development, file the issue before continuing.

## Company Context

IFCore is building an AI-powered building compliance checker. **5 teams** each develop in their **own GitHub repo** (cloned from a shared template). Teams write `check_*` functions independently — the platform integrates them automatically.

**How integration works:**
1. Each team pushes `checker_*.py` files to their own repo under `tools/`
2. The **platform repo** (`ifcore-platform`) pulls all 5 team repos as **git submodules**
3. `deploy.sh` flattens submodules into `teams/<team-repo>/tools/` (real files, not symlinks — we don't configure HF to resolve submodules)
4. The FastAPI orchestrator scans `teams/*/tools/checker_*.py` for `check_*` functions
5. All discovered functions run against uploaded IFC files

**Deployment architecture:**

| Component | Deploys to | Who manages |
|-----------|-----------|-------------|
| Team check functions (`checker_*.py`) | Own GitHub repo → pulled into platform | Each team |
| Backend + orchestrator (`ifcore-platform`) | **HuggingFace Space** (Docker, FastAPI) | Captains |
| Frontend (dashboard, 3D viewer, upload) | **Cloudflare Pages** | Captains |
| API gateway (async jobs, proxies to HF) | **Cloudflare Worker** | Captains |
| File storage (IFC uploads) | **Cloudflare R2** (S3-compatible) | Captains |
| Results database | **Cloudflare D1** (SQLite) | Captains |

**Flow:** User uploads IFC → stored in R2 → frontend calls CF Worker → Worker proxies to HF Space → orchestrator runs all `check_*` functions → results posted back to Worker → stored in D1 → frontend polls and displays.

**Teams never touch the platform repo.** They only push to their own team repo. Captains handle `deploy.sh` which pulls, flattens, and pushes to HF.

**Teams:**
| Team | Focus area | Repo |
|------|-----------|------|
| [TBD] | [TBD] | [TBD] |
| [TBD] | [TBD] | [TBD] |
| [TBD] | [TBD] | [TBD] |
| [TBD] | [TBD] | [TBD] |
| [TBD] | [TBD] | [TBD] |

## References

- [Validation Schema](./references/validation-schema.md) — database schema (`users`, `projects`, `check_results`, `element_results`) and how team `list[dict]` maps to rows
- [Architecture](./references/architecture.md) — project structure, AGENTS.md template, code conventions
- [Repo Structure](./references/repo-structure.md) — concrete file tree examples for all 4 repos (team, platform, frontend, gateway)
- [Frontend Architecture](./references/frontend-architecture.md) — modules, shared Zustand store, API client, D1 tables, how to add features
- [Development Patterns](./references/development-patterns.md) — how to plan and build new features

### Related Skills (separate repos, installed alongside this one)

- **pydantic-ai** — PydanticAI agent framework: tools, structured output, orchestration, chat patterns
- **huggingface-deploy** — deploy the platform (`ifcore-platform`) as a Docker Space on HuggingFace; covers Dockerfile, secrets, R2 caching, and the flatten-before-push submodule pattern
- **cloudflare** — deploy the frontend + API gateway on Cloudflare Pages/Workers
