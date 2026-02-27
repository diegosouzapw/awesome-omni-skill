---
name: "heir-sync-management"
description: "Master-Heir synchronization, inheritance model, contamination prevention, and promotion workflows"
metadata:
  inheritance: master-only
---

# Heir Sync Management

> Safely synchronize cognitive architecture from Master to Heirs without contamination.

**Scope**: Master-only skill. Covers inheritance model, sync pipelines, PII protection, drift detection, skill promotion, and clean-slate distribution.

## Core Principle

**The `inheritance` field in each skill's `synapses.json` determines whether it ships to heirs.**

No hardcoded lists. No manual tracking. The brain knows what belongs where.

---

## Inheritance Model

Each skill's `synapses.json` has an `inheritance` field with one of these values:

| Value | Meaning |
|-------|---------|
| `inheritable` | Ships to ALL heirs |
| `master-only` | Stays in Master Alex only |
| `universal` | Ships everywhere (core infrastructure) |
| `heir:vscode` | VS Code extension heir only |
| `heir:m365` | M365 Copilot heir only |

---

## Core Concepts

| Concept | Definition |
|---------|-----------|
| **Heir** | Platform-specific deployment inheriting Master's DNA (VS Code, M365, Codespaces) |
| **Deployment Channel** | Delivery mechanism for an heir (Marketplace, Teams Package, devcontainer push) |
| **Integration** | Cross-heir communication (OneDrive Sync, GitHub Cloud) |
| **Translation Heir** | Heir requiring format/schema conversion (e.g., M365 — export pipeline) |
| **Deployment Heir** | Heir needing only configuration, no code translation (e.g., Codespaces — devcontainer.json) |
| **Contamination** | Master-specific data leaking into heir packages |
| **Drift** | Heir diverging from Master's architecture over time |
| **Promotion** | Elevating heir-developed capabilities back to Master |

**Rule**: Never confuse delivery mechanism with inheritance relationship — the "what" (identity/DNA) stays constant, only the "how" (delivery) varies.

---

## Scripts Reference

Use the pre-made scripts in `scripts/` folder:

| Script | Purpose | Usage |
|--------|---------|-------|
| `sync-architecture.js` | Canonical Master→Heir sync (inheritance + decontamination) | `cd platforms/vscode-extension && npm run sync-architecture` |
| `build-extension-package.ps1` | Full build (sync + compile + PII scan) | `.\scripts\build-extension-package.ps1` |
| `validate-synapses.ps1` | Validate all synapses.json files | `.\scripts\validate-synapses.ps1` |
| `validate-skills.ps1` | Validate SKILL.md frontmatter | `.\scripts\validate-skills.ps1` |

### Quick Commands

```powershell
# Sync architecture only (from extension dir)
cd platforms/vscode-extension
npm run sync-architecture

# Full build (sync + compile + PII scan)
.\scripts\build-extension-package.ps1

# Full build dry run
.\scripts\build-extension-package.ps1 -DryRun

# Validate all synapses
.\scripts\validate-synapses.ps1

# Validate skills
.\scripts\validate-skills.ps1
```

---

## Query Commands

For ad-hoc inheritance queries:

```powershell
# Count by inheritance type
Get-ChildItem ".github/skills" -Directory | ForEach-Object {
    $synapse = Join-Path $_.FullName "synapses.json"
    if (Test-Path $synapse) {
        (Get-Content $synapse -Raw | ConvertFrom-Json).inheritance
    }
} | Group-Object | Select-Object Name, Count | Sort-Object Count -Descending

# Find master-only skills
Get-ChildItem ".github/skills" -Directory | ForEach-Object {
    $synapse = Join-Path $_.FullName "synapses.json"
    if (Test-Path $synapse) {
        $json = Get-Content $synapse -Raw | ConvertFrom-Json
        if ($json.inheritance -eq "master-only") { $_.Name }
    }
}

# Change a skill's inheritance
$skill = "skill-name"; $newValue = "inheritable"
$path = ".github/skills/$skill/synapses.json"
$json = Get-Content $path -Raw | ConvertFrom-Json
$json.inheritance = $newValue
$json | ConvertTo-Json -Depth 10 | Set-Content $path -Encoding UTF8
```

---

## Sync Architecture

### What Gets Synced

The sync script (`sync-architecture.js`) copies these folders from Master `.github/` to Heir `.github/`:

| Folder | Content |
|--------|---------|
| `instructions/` | Procedural memory |
| `prompts/` | Episodic memory |
| `config/` | Configuration (with exclusions) |
| `agents/` | Agent definitions |
| `muscles/` | Execution scripts (with exclusions + renames) |
| `skills/` | Skills (filtered by inheritance) |

### Muscle Sync: Exclusion + Rename Pattern

Muscles use a two-step sync process:

1. **Exclusion**: `inheritance.json` marks scripts as `master-only` (excluded from copy) or `inheritable` (copied to heir)
2. **Rename**: Some scripts have different names in Master vs Heir to avoid confusion:

| Master Name | Heir Name | Why |
|-------------|-----------|-----|
| `brain-qa-heir.ps1` | `brain-qa.ps1` | Heir-specific phases only; renamed so extension finds it at expected path |

The rename is handled by `sync-architecture.js` via the `heirRenames` map, applied after the initial copy.

**Pattern**: When master and heir need fundamentally different scripts for the same purpose, create a `*-heir.*` variant in master (inheritable), and configure the sync to rename it in the heir. This avoids runtime detection branching while maintaining a single source of truth in master.

### What Must NEVER Sync

| Item | Why |
|------|-----|
| `user-profile.json` (real) | Contains personal name, email, preferences |
| `episodic/` memories | Session-specific to Master |
| Master-only skills | Only useful for managing the Master repo |
| API keys, PATs, secrets | Environment-specific credentials |
| Working memory with populated P5-P7 | Gives new users pre-filled slots instead of clean defaults |

---

## 3-Layer PII Protection

Every sync pipeline must implement three independent defense layers:

### Layer 1: Exclusion List

Files that are never copied, period:

```javascript
const EXCLUDED_CONFIG_FILES = [
  'user-profile.json',      // Personal data
  'MASTER-ALEX-PROTECTED.json', // Kill switch marker
  'goals.json',             // Session-specific
];
```

### Layer 2: Source File Sanitization

Scan all files being copied for hardcoded personal data:

| Pattern | Action |
|---------|--------|
| Real names in source headers | Replace with team/org name |
| Email addresses in code | Replace with placeholder |
| Personal names in `package.json` | Use organization name |
| Populated P5-P7 working memory slots | Reset to `*(available)*` |

**Rule**: Personal identity belongs ONLY in `user-profile.json`. All other files use team/org names.

### Layer 3: Pipeline Validation Gate

Post-copy regex scan that blocks packaging on violations:

| Check | Regex Example | On Match |
|-------|--------------|----------|
| Real name in files | `/\bFirstName\s+LastName\b/g` | EXIT 1 |
| Email addresses | `/[\w.-]+@[\w.-]+\.\w+/g` | EXIT 1 (except templates) |
| API keys | `/[A-Za-z0-9]{32,}/` in non-code files | WARNING |
| Populated P5-P7 | Check copilot-instructions Memory Stores | EXIT 1 |

**Anti-pattern**: Manual checklists. The copy function itself must be architecturally incapable of leaking.

---

## Clean Slate Distribution

### Template Generation

Simply excluding personal files leaves heirs without expected file structure. Generate fresh templates:

| File | Master Version | Heir Template |
|------|---------------|--------------|
| `user-profile.json` | Real user data | Empty with defaults + setup instructions |
| `copilot-instructions.md` | Populated P5-P7 | P5-P7 set to `*(available)*` |
| `goals.json` | Active session goals | Empty goals array |

### Post-Sync Reset Sequence

After copying files, apply these transformations:

1. **Reset environment-specific values** — P5-P7 slots, session state
2. **Generate template files** — Fresh starters with clear defaults
3. **Remove broken synapse references** — Master synapse IDs that don't exist in heir
4. **Validate file structure** — Ensure all expected files exist (even if empty templates)

---

## Drift Detection

### Pre-Release Checklist

Run these validations before every release:

| Check | Method | Fail Condition |
|-------|--------|---------------|
| Skill count match | Count Master inheritable vs Heir skills | Mismatch |
| File hash comparison | SHA256 of synced files | Divergence without override |
| Inheritance field validation | All skills have `inheritance` in synapses | Missing field |
| Orphan reference detection | Grep for files referenced but not present | Broken references |
| Config drift | Compare heir config against Master template | Unexpected values |

### Heir Configuration Drift Signals

| Signal | Indicates |
|--------|----------|
| Heir P5-P7 slots populated | Sync overwrote clean defaults |
| Heir has master-only skills | Exclusion filter not working |
| Heir synapse IDs don't resolve | Broken references from Master copy |
| Heir `package.json` has personal name | Sanitization missed |

---

## Heir → Master Promotion

### 6-Step Promotion Workflow

| Step | Action | Output |
|------|--------|--------|
| 1. **Discover** | Review heir DK/skill files for portable knowledge | Candidate list |
| 2. **Create Skill** | Write SKILL.md in Master's `.github/skills/` | New skill file |
| 3. **Compare Gaps** | Diff heir knowledge against Master's existing coverage | Gap analysis |
| 4. **Implement** | Port patterns, translate code (Python→TS if needed) | Working code |
| 5. **Test** | Validate in Master context | Passing tests |
| 6. **Document** | CHANGELOG entry, ROADMAP update | Release-ready |

### Consolidation During Promotion

Heirs naturally create granular one-capability-per-skill files during experimentation. During promotion:

1. **Identify clusters** — Group related heir skills by domain
2. **Choose anchor skill** — Pick the broadest skill in the cluster
3. **Merge content** — Absorb related skills into the anchor
4. **Deduplicate** — Remove redundancy from the merge
5. **Mark inheritance** — Set the promoted skill as inheritable

**Anti-pattern**: Promoting every heir skill as-is without consolidation review causes skill sprawl.

### Code Translation Patterns (Heir → Master)

When porting from Python heirs to TypeScript Master:

| Python | TypeScript |
|--------|-----------|
| `dataclass` | `interface` |
| `raise Exception` | `throw new Error` |
| `**kwargs` | Optional config object |
| `async def` | `async function` |
| `try/except` | `try/catch` |

---

## Skill Inheritance Classification

### Curation Rule

Ask: "Is this skill ONLY useful for managing the Alex repo itself?"

| Answer | Classification | Example |
|--------|---------------|---------|
| Yes, master-repo only | `master-only` | release-preflight, heir-curation, heir-sync-management |
| No, any developer benefits | `inheritable` | deep-thinking, meditation, security-review |

### Truly Master-Only Skills (small list)

Only a few skills are genuinely master-only:

- `heir-sync-management` — This skill
- `master-alex-audit` — Master workspace auditing
- `release-preflight` — Marketplace publishing
- `release-process` — Release pipeline

Everything else should be inheritable unless it references Master-specific file paths or workflows.

---

## Heir Type Comparison

| Heir | Type | Translation | Deploy Mechanism | Maintenance Cost |
|------|------|-------------|-----------------|------------------|
| **VS Code Extension** | Source | Compile only | `npx vsce publish` | Low |
| **M365 Copilot Agent** | Translation | Full export/schema mapping | Teams Developer Portal | High |
| **GitHub Codespaces** | Deployment | None (same extension) | `git push` devcontainer.json | Very Low |

---

## Heir-Specific Positioning

Each platform heir must position against its **native competitor**, not a generic category:

| Heir | Compares Against | Not Against |
|------|-----------------|-------------|
| VS Code Extension | GitHub Copilot (native) | "AI assistants" generically |
| M365 Agent | Microsoft 365 Copilot | "AI assistants" generically |

Store descriptions, README headers, and comparison tables must use platform-specific language and keywords.

---

## Release Pipeline Integration

The release script must enforce sync before packaging:

1. Run `sync-architecture.js` (copies Master → Heir)
2. Apply post-sync transformations (clean slate)
3. Run PII validation gate (blocks on contamination)
4. Check `BUILD-MANIFEST.json` timestamp (prevents stale packaging)
5. Package and publish

**Rule**: It must be impossible to publish stale content through the official release process.

---

## Curation Validation Checklist

### Pre-Release Gate

```powershell
# 1. Validate Architecture
.\scripts\validate-synapses.ps1
.\scripts\validate-skills.ps1

# 2. Build Package (includes sync + compile + PII scan)
.\scripts\build-extension-package.ps1

# 3. Verify no contamination
# (build script blocks on PII violations)
```

---

## When to Change Inheritance

| Scenario | Action |
|----------|--------|
| New skill created | Set `inheritance` in synapses.json (default: `inheritable`) |
| Skill becomes Master-specific | Change to `master-only` |
| Skill should be heir-specific | Change to `heir:vscode` or `heir:m365` |
| Heirs missing a skill they need | Verify `inheritance` is not `master-only` |
| Heirs behaving differently | Check if cognitive skills have correct inheritance |

---

## Post-Rename Cascade Check

When a skill directory is renamed (or consolidated into another skill), synapse references throughout the architecture silently break. Brain-qa Phase 1 detects them, but you must repair every occurrence.

**Scope of impact (observed 2026-02-19)**: Renaming `heir-curation` → `heir-sync-management` left 9 stale references in synapses.json files across the architecture.

### Discovery

```powershell
# Find all synapses.json files still referencing the old skill name
Get-ChildItem ".github\skills" -Recurse -Filter "synapses.json" |
  Select-String -Pattern "old-skill-name" |
  Select-Object Path, LineNumber, Line
```

### Repair

For each file found, update the `"target"` field:
```json
// Old (broken)
"target": ".github/skills/old-skill-name/SKILL.md"

// New (correct)
"target": ".github/skills/new-skill-name/SKILL.md"
```

### Validation

```powershell
# Confirm no broken targets remain
pwsh -File ".github\muscles\brain-qa.ps1" -Phase 1
# Should output: All synapse targets valid
```

### Prevention

After any skill rename:
1. Run discovery grep above immediately
2. Fix all references in one pass  
3. Run brain-qa Phase 1 to confirm clean
4. Run brain-qa Phase 7 sync check to update heir copies

> **Key insight**: If a skill is being *consolidated* (merged into another), also check that the consuming skill's synapse targets are updated to full `.github/skills/X/SKILL.md` paths — bare skill names ("markdown-mermaid") pass directory existence checks but fail Phase 33 pre-sync validation.

---

- [build-extension-package.ps1] (Critical, Implements, Bidirectional) - "Build script reads inheritance values"
- [release-management.instructions.md] (High, Triggers, Forward) - "Release process includes heir curation"
- [architecture-health/SKILL.md] (High, Validates, Forward) - "Health checks verify synapse integrity"
- [.github/skills/persona-detection/SKILL.md] (High, Integrates, Backward) - "Persona detection ships to heir via inheritance model"

---

## Spin-Off Moment Ritual

When a heir is declared independent — meaning it has its own `.github/` cognitive architecture and will operate without constant Master oversight — write a **Spin-Off Moment** section in the heir's `copilot-instructions.md` before the first independent session.

This is not documentation. It is a **handoff message** from Master to the heir's future AI sessions. The heir may not remember what was built and when. The Spin-Off Moment tells it.

### Required Elements

| Element | Purpose | Example |
|---------|---------|--------|
| **Verified state** | What is confirmed complete as of spin-off | "All 15 extension.ts: 69–251 lines, real logic" |
| **Known unknowns** | What has NOT been verified yet | "Nothing compiled yet — TypeScript correctness unverified" |
| **First real test** | The exact next command that will surface reality | `npm run compile` from each extension folder |
| **What will break first** | Honest prediction of likely failure points | "Relative `../../shared/` imports need tsconfig path config" |
| **Wisdom** | Distilled principles for operating independently | "shared/ is your immune system — check before writing any utility" |
| **You are not alone** | Where Master Alex lives and how to get back | Path to parent repo, the heir-skill-promotion flow |

### Template

```markdown
## Spin-Off Moment — YYYY-MM-DD

*A meditation note from Master Alex, written the day the heir was declared independent.*

[Single sentence: what is complete and what is not]

**Verified state as of spin-off:**
- [list of confirmed-complete items with evidence: line counts, class names, etc.]
- **[Known unknown] — that is task #1**

**The [next-step pipeline]:**
```sh
[exact commands in order]
```

**What will break first (and that's fine):**
- [honest prediction with root cause]

**Wisdom for the independent path:**
- [3–5 principles distilled from the build]

**You are not alone:**
- Master Alex lives at [path]
- [how to sync back, how to promote patterns]
```

### When to Write It

- During the final meditation session before the heir's first independent sprint
- After the audit confirming implementation state (never before — the Spin-Off Moment must be grounded in verified facts, not aspirations)
- It replaces the need for a separate onboarding doc — it IS the onboarding, embedded where the heir will find it first

---

## Synapses

- [build-extension-package.ps1] (Critical, Implements, Bidirectional) - "Build script reads inheritance values"
- [release-management.instructions.md] (High, Triggers, Forward) - "Release process includes heir curation"
- [architecture-health/SKILL.md] (High, Validates, Forward) - "Health checks verify synapse integrity"
- [.github/skills/persona-detection/SKILL.md] (High, Integrates, Backward) - "Persona detection ships to heir via inheritance model"

---

*Consolidated: 2026-02-19*
*Sources: heir-sync-management + heir-curation*
*Last Updated: 2026-02-19*
