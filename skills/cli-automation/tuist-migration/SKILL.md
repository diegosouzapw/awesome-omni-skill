---
name: tuist-migration
description: Integrates Tuist into an existing iOS project that uses SPM local packages. Use when migrating a project from a manually maintained .xcodeproj to a Tuist-generated project, adding Tuist as an orchestration layer on top of SPM. Covers 8 sequential phases — audit, base structure, helpers, generation, schemes, CI, cache, and validation. Includes automation scripts and AI prompt templates for each phase.
---

# Tuist Migration

Integrate Tuist into an existing iOS project with SPM local packages, without modifying the modules' `Package.swift` files. Tuist acts as an orchestration layer that generates the `.xcodeproj`/`.xcworkspace` from declarative Swift manifests.

## Fundamental Principle

**Tuist does not touch modules, it only orchestrates the project that consumes them.** The `Package.swift` files remain intact. If Tuist is removed later, the modules are still functional SPM packages.

## Reference Project

Use the current project's Tuist configuration (`Tuist/ProjectDescriptionHelpers/`, `Project.swift`, `Workspace.swift`, `Tuist.swift`, `Tuist/Package.swift`) as the **primary reference** for API syntax and helper patterns. **Never copy** configurations, bundle IDs, modules, or dependencies — adapt patterns to the target project using audit data.

## Migration Phases

8 sequential phases. Each has a script and/or prompt template in the bundled resources.

| # | Phase | Script | Prompt | Output |
|---|-------|--------|--------|--------|
| 1 | Audit | `scripts/audit.sh` | `references/prompts/01-audit.md` | `AUDIT_REPORT.md` |
| 2 | Base structure | — | `references/prompts/02-base-structure.md` | `Tuist.swift`, `Tuist/Package.swift`, `.gitignore` |
| 3 | Helpers | — | `references/prompts/03-helpers.md` | `Tuist/ProjectDescriptionHelpers/*.swift` |
| 4 | Generation | `scripts/generate.sh` | `references/prompts/04-generation.md` | `Project.swift`, `Workspace.swift`, `generate.sh` |
| 5 | Schemes | `scripts/compare_schemes.sh` | `references/prompts/05-schemes.md` | Verified `AppScheme.swift` |
| 6 | CI | — | `references/prompts/06-ci.md` | Updated CI pipelines |
| 7 | Cache | `scripts/extract_cache_data.sh`, `scripts/warm_cache.sh` | `references/prompts/07-cache.md` | Optimized `productTypes` |
| 8 | Validation | `scripts/compare_settings.sh`, `scripts/validate_migration.sh` | `references/prompts/08-validation.md` | PASS/FAIL report |


## Workflow

Read `references/prompts/00-context.md` for the full orchestration context. For each phase:

1. **Read the phase prompt** — Load `references/prompts/0X-*.md` for detailed instructions.
2. **Run the script** (if applicable) — Execute the phase script to collect data.
3. **Execute the phase** — Generate/modify files following the prompt instructions.
4. **Present results** — Summarize files created, decisions made, warnings found.
5. **Get feedback** — Ask user before proceeding to next phase.
6. **Validate** — From phase 4 onward, verify `tuist generate` succeeds and project builds.

**Never skip phases or execute multiple at once without explicit user approval.**

## Scripts Reference

All scripts use `set -euo pipefail` and require `mise` for tool versioning. **All scripts have placeholder values marked with `← Adjust` comments** — update them before running.

| Script | Phase | Purpose |
|--------|-------|---------|
| `scripts/setup.sh` | Pre-migration | Install brew, mise, and tools from `.mise.toml` |
| `scripts/audit.sh` | 1 | Collect all project data into `/tmp/tuist-audit/` |
| `scripts/generate.sh` | 4 | `tuist install` + `tuist generate` with optional `--clean` |
| `scripts/compare_schemes.sh` | 5 | Compare schemes between original and Tuist projects |
| `scripts/extract_cache_data.sh` | 7 | Extract dependency graph for cache optimization |
| `scripts/warm_cache.sh` | 7 | Pre-compile external dependencies |
| `scripts/compare_settings.sh` | 8 | Compare build settings between both projects |
| `scripts/validate_migration.sh` | 8 | End-to-end validation with 7 checks and PASS/FAIL |

## Tuist File Structure

```
Project root (new files)
├── .mise.toml                          ← Tool versions (tuist, swiftlint)
├── Project.swift                       ← Minimal: `let project = App.project`
├── Workspace.swift                     ← Root project + module paths + schemes
├── Tuist.swift                         ← Xcode/Swift version constraints
├── Tuist/Package.swift                 ← External deps + targetSettings per module
└── Tuist/ProjectDescriptionHelpers/
    ├── Config.swift                    ← App name, destinations, base settings
    ├── BuildConfiguration.swift        ← Debug/Staging/Release configurations
    ├── Environment.swift               ← Dev/Staging/Prod: API URLs, bundle IDs
    ├── Module.swift                    ← Struct wrapping SPM local package refs
    ├── Modules.swift                   ← Central registry of all modules
    ├── App.swift                       ← App target + UI tests target
    ├── AppScheme.swift                 ← Scheme factory per environment
    └── BuildScripts.swift              ← Run Script build phases (SwiftLint, etc.)
```

## Key Rules

1. **All values from audit** — Build settings, bundle IDs, signing, dependencies come from `AUDIT_REPORT.md`, never from reference projects.
2. **`-Tuist` suffix during migration** — Name the project `AppName-Tuist` so both projects coexist. Remove suffix after validation.
3. **`targetSettings` synchronizes, not overrides** — Ensures Xcode build settings match what each `Package.swift` declares.
4. **Modules with `nonisolated` default** — Need a separate `SettingsDictionary` in `targetSettings`.
5. **`productTypes` for cache** — External dependencies declared as `.framework` get cached as pre-compiled binaries.
6. **Incremental validation** — From phase 4, run `tuist generate` after each change.

## Audit to Helpers Mapping

| Tuist Helper | Built From (Audit Section) |
|---|---|
| `Config.swift` | Build settings common to all targets |
| `BuildConfiguration.swift` | Build configurations (Debug/Release/Staging) |
| `Environment.swift` | Schemes + environment-specific values |
| `Module.swift` | Module list + Package.swift files |
| `Modules.swift` | Complete module list from targets |
| `App.swift` | Info.plist + resources + build phases + signing |
| `AppScheme.swift` | Schemes (run config, test targets, coverage) |
| `BuildScripts.swift` | Run Script build phases |
| `Tuist/Package.swift` | `swiftSettings` per module + external deps |

## Validation Criteria

Migration is valid when ALL pass:
1. `tuist migration check-empty-settings` reports no orphaned settings
2. Build settings diff shows no functional differences
3. `xcdiff` shows no missing source files or resources
4. Dependency graph matches
5. Full test suite passes (unit + snapshot + UI)
6. `validate_migration.sh` exits with code 0

## Common Failure Patterns

- `.intentdefinition` must go in sources, not resources
- `.xcstrings` collides with `.strings`/`.stringsdict` globs
- ObjC categories in static frameworks need `-ObjC` in `OTHER_LDFLAGS`
- SPM resource bundles require `.process("Resources")` and `Bundle.module`
- Types not found → source files accidentally excluded
- Undefined symbols → missing SDK frameworks or dependency products
- Launch crashes → incorrect bundle IDs, entitlements, or resources
