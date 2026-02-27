---
name: end-session
description: Session closeout — canonicalize, verify, commit, produce handoff artifacts
user-invocable: true
---

# /end-session

You are closing a working session on **The Long Walk**. This skill is **write-heavy and procedural**. Execute all steps in order. Do not skip steps. Only ask questions if execution is impossible without clarification.

---

## Step 1: Canonicalize state

Reconcile everything from this session into durable artifacts.

### 1a. Update MEMORY.md
- Update `~/.claude/projects/-Volumes-Queen-Amara-The-Long-Walk/memory/MEMORY.md`
- Add any new patterns, lessons learned, or architecture changes discovered this session
- Remove or correct any entries that are now outdated
- Keep it under 200 lines

### 1b. Update CHANGELOG.md
- If significant features were added, add a new version entry at the top of `docs/CHANGELOG.md`
- Follow semver: MAJOR (breaking), MINOR (new feature), PATCH (fix)

### 1c. Mark deferred items explicitly
If anything was discussed but intentionally not done, record it. "We chose not to do X" is valuable.

---

## Step 2: Verify builds

Both stacks must pass before committing.

### Client (Vite + TypeScript)
```bash
cd "/Volumes/Queen Amara/The Long Walk" && npx tsc --noEmit
```
**Gate:** If this fails, fix the type errors before proceeding.

### Server (if server/ exists)
```bash
cd "/Volumes/Queen Amara/The Long Walk" && npx tsc --noEmit -p server/tsconfig.json
```
**Gate:** Fix errors before proceeding.

### Build check
```bash
cd "/Volumes/Queen Amara/The Long Walk" && npx vite build
```
**Gate:** Must succeed.

If any gate fails and you cannot fix it, **stop and tell the user what's broken**.

---

## Step 3: Commit and push

### 3a. Stage changes
Review all modified files with `git status` and `git diff`. Stage files intentionally.

**Never stage:**
- `.env`, `.env.local`, credentials files
- `node_modules/`, `dist/`

### 3b. Commit
Use atomic commits — one logical change per commit.

Commit message format: `type(scope): description`
- Types: `feat`, `fix`, `docs`, `refactor`, `chore`, `test`
- Scope: `client`, `server`, `viz`, `agents`, `docs`, `skills`

```bash
git commit -m "$(cat <<'EOF'
type(scope): description

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

### 3c. Push
```bash
git push origin main
```

---

## Step 4: Generate session ledger (YAML)

Save to `docs/sessions/SESSION_LEDGER_{date}_{slug}.yaml`

```yaml
session_id: "{date}-{slug}"
date: "{YYYY-MM-DD}"
branch: "main"
version: "{from CHANGELOG}"
prior_session: "{session_id from the last ledger}"

build_verification:
  client_typescript: "pass | fail"
  server_typescript: "pass | fail | n/a"
  vite_build: "pass | fail"

commits:
  - hash: "{short hash}"
    message: "{commit message}"

artifacts_updated:
  - path: "{file path}"
    change: "{what changed and why}"

decisions_made:
  - decision: "{what was decided}"
    rationale: "{why}"

open_questions:
  - "{question that needs future resolution}"

known_risks:
  - "{risk that exists}"

explicitly_deferred:
  - "{thing we chose not to do and why}"

next_actions:
  - "{ordered, executable step 1}"
  - "{ordered, executable step 2}"
  - "{ordered, executable step 3}"
```

---

## Step 5: Write human-readable handoff

Save to `docs/sessions/HANDOFF_{date}_{slug}.md`

This file must be **self-contained** — sufficient to resume work without prior conversation history.

### Required sections:

```markdown
# Session Handoff: {Session Title}

**Date:** {YYYY-MM-DD}
**Session ID:** {from ledger}
**Version:** {from ledger}
**Branch:** main

---

## Where We Are
{2-3 paragraphs: Current state of the project. What works. What the user sees.}

## What Changed This Session
{Bullet list of every meaningful change, organized by area:}
### Client
- {change}
### Server
- {change}
### Infrastructure
- {change}

## Why These Changes Were Made
{Brief context — what prompted this work, what problem it solves}

## What Must Happen Next
{Ordered list of immediate next steps}

## Decisions Made (Do Not Re-Debate)
{List of decisions with rationale}

## Explicitly Deferred
{Things we discussed but chose not to do, and why}

## Known Risks
{Anything that could bite us}

## Key Files Modified
{Table of files touched and what changed}
```

---

## Step 6: Final integrity check

Run through this checklist:

- [ ] All durable knowledge persisted (MEMORY.md, CHANGELOG updated as needed)
- [ ] Working tree clean (`git status` shows nothing)
- [ ] Pushed to remote (`git log origin/main..HEAD` shows nothing)
- [ ] Session ledger saved to `docs/sessions/`
- [ ] Session handoff saved to `docs/sessions/`
- [ ] MEMORY.md is under 200 lines

Report the checklist results. If anything is blocked, state what, why, and what input is needed.

---

## Rules

1. Execute steps in order. Do not skip steps.
2. Do not ask exploratory or preference questions. Only ask if execution is impossible without clarification.
3. If a step can't be completed, explicitly state what's blocked and why.
4. Atomic commits — one logical change per commit, clear messages.
5. Never commit secrets, credentials, or `.env` files.
6. The handoff must be self-contained — no "see our conversation" references.
7. If nothing meaningful changed this session (just research/conversation), produce a minimal ledger noting "research-only session, no code changes" and skip build/commit steps.
