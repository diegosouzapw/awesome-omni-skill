---
name: docs-sync
description: Sync AILANG documentation website with codebase reality. Use after releases, when features are implemented, or when website accuracy is questioned. Checks design docs vs website, validates examples, updates feature status.
---

# Documentation Sync Skill

Keep the AILANG website in sync with actual implementation by checking design docs, validating examples, and tracking feature status.

## Quick Start

```bash
# After a release - full sync check
# User: "sync docs for v0.5.6" or "run docs-sync"

# Check specific area
# User: "verify landing pages" or "check working examples"
```

## When to Use This Skill

Invoke this skill when:
- After any release (post-release should trigger this)
- User questions website accuracy
- Features move from planned → implemented
- Examples may be broken or outdated
- Version constants need updating

## Workflow

### 1. Pre-flight Checks

Run all diagnostic scripts to understand current state:

```bash
# Check design docs status
.claude/skills/docs-sync/scripts/audit_design_docs.sh

# Check version constants
.claude/skills/docs-sync/scripts/check_versions.sh

# Validate working examples
.claude/skills/docs-sync/scripts/check_examples.sh
```

### 2. Review Feature Themes

Features are grouped into themes (see [resources/feature_themes.md](resources/feature_themes.md)):

| Theme | Status Page | Description |
|-------|-------------|-------------|
| Core Language | `/reference/language-syntax` | Types, ADTs, pattern matching |
| Effect System | `/reference/effects` | Capabilities, IO, FS, Net |
| Module System | `/reference/modules` | Imports, exports, aliasing |
| Go Codegen | `/guides/go-codegen` | Compilation to Go |
| AI Integration | `/guides/ai-integration` | Prompts, benchmarks, agents |
| Testing | `/guides/testing` | Inline tests, property-based |
| Developer Experience | `/guides/development` | REPL, debugging, CLI |
| **Roadmap: Execution Profiles** | `/roadmap/execution-profiles` | v0.6.0 planned |
| **Roadmap: Shared Semantic State** | `/roadmap/shared-semantic-state` | v0.6.0 planned |
| **Roadmap: Deterministic Tooling** | `/roadmap/deterministic-tooling` | v0.7.0 planned |

### 3. Fix Priority Order

1. **Version Constants** - Update `docs/src/constants/version.js`
2. **Landing Pages** - intro.mdx, vision.mdx, why-ailang.mdx
3. **Working Examples** - Ensure examples use raw-loader, all pass
4. **Status Banners** - Add "PLANNED" banners to future features
5. **Feature Docs** - Create missing docs for implemented features
6. **Roadmap Section** - Move theoretical pages to roadmap

### 4. Update Checklist

After fixing, verify:

```bash
# Rebuild and check
cd docs && npm run build

# Verify no broken links
npm run serve  # Manual check

# Commit changes
git add docs/
git commit -m "docs: sync website with v0.X.X implementation"
```

## Scripts

| Script | Purpose |
|--------|---------|
| `audit_design_docs.sh` | Compare planned vs implemented design docs |
| `derive_roadmap_versions.sh` | **Derive target versions from design doc folders** |
| `check_versions.sh` | Verify version constants match releases |
| `check_examples.sh` | Validate example files compile/run |
| `generate_report.sh` | Generate sync status report |

### CLI Example Verification (External)

The Makefile provides CLI example verification that complements this skill:

```bash
# Verify all CLI examples documented in examples/cli_examples.txt
make verify-cli-examples

# Full verification: code examples + CLI examples
make verify-examples && make verify-cli-examples
```

**CLI Examples File Format** (`examples/cli_examples.txt`):
```
# Comment explaining the example
$ ailang run --caps IO --entry main examples/runnable/hello.ail
Hello, AILANG!
```

This ensures CLI syntax in documentation matches actual behavior.

### Version Derivation Script

```bash
# List all planned features with derived target versions
.claude/skills/docs-sync/scripts/derive_roadmap_versions.sh

# Full lifecycle: planned + implemented features
.claude/skills/docs-sync/scripts/derive_roadmap_versions.sh --full

# Check website consistency (exits 1 if mismatches)
.claude/skills/docs-sync/scripts/derive_roadmap_versions.sh --check

# JSON output for automation
.claude/skills/docs-sync/scripts/derive_roadmap_versions.sh --json --full

# Full validation: all features + website check
.claude/skills/docs-sync/scripts/derive_roadmap_versions.sh --full --check
```

## Resources

| Resource | Content |
|----------|---------|
| `feature_themes.md` | Feature groupings and expected pages |
| `landing_page_checklist.md` | Requirements for main pages |

## Integration with Post-Release

The `post-release` skill should invoke docs-sync automatically:

```bash
# In post-release workflow after eval baselines
# Run docs-sync to update website
```

## Key Principles

1. **Theoretical is OK** - Future features can be documented, but must:
   - Link to design docs on GitHub
   - Have clear status banner (e.g., "PLANNED FOR v0.6.0")
   - Be in roadmap section, not current features

2. **Examples Must Work** - Every code example should:
   - Use raw-loader to import from `examples/`
   - Be tested with `ailang run` or `ailang test`
   - Never embed code directly in MDX
   - **CLI commands must be added to `examples/cli_examples.txt`** and verified with `./tools/verify_cli_examples.sh`
   - Default entrypoint is `main` - don't use `--entry main` unless showing non-main entry
   - Test all commands before documenting: `./bin/ailang run --caps IO examples/runnable/hello.ail`

   **Exception for Reference Documentation:**
   - Reference pages (e.g., `language-syntax.md`, `effects.md`) may use inline syntax snippets
   - These are small patterns showing language constructs, not complete runnable programs
   - Tutorial pages (`examples.mdx`, `getting-started.mdx`, `guides/`) must always import from files
   - Rule: If the code snippet is a complete runnable program, it MUST be imported

3. **Design Docs = Ultimate Source of Truth** - The folder structure tracks complete feature lifecycle:

   **Planned features:**
   - `design_docs/planned/v0_6_0/foo.md` → Feature targets v0.6.0
   - Website pages must match: `PLANNED FOR v0.6.0` banner
   - Website MUST link to design doc on GitHub as ultimate reference
   - OVERDUE = design doc folder version ≤ current release but not implemented

   **Implemented features:**
   - `design_docs/implemented/v0_5_6/foo.md` → Feature shipped in v0.5.6
   - Website reference pages can link to implemented design docs
   - Moving `planned/` → `implemented/` = feature is done

   **Validation:**
   - Run `./scripts/derive_roadmap_versions.sh --check` to validate website
   - Run `./scripts/derive_roadmap_versions.sh --full` to see complete lifecycle
   - Script checks for GitHub links in roadmap pages

4. **One Source of Truth** - Version comes from:
   - `git describe --tags` → actual version
   - `prompts/versions.json` → latest prompt
   - These feed into `docs/src/constants/version.js`

5. **Themes Over Changelog** - Group features by theme, not by version. Users care about "how do effects work?" not "what changed in v0.5.3?"

6. **Evolving Themes** - Themes should evolve as the language grows:
   - When a feature doesn't fit existing themes, consider a new theme
   - New themes warrant new landing pages
   - Update `resources/feature_themes.md` when adding themes
   - Current themes: Core Language, Type System, Effect System, Module System, Go Codegen, Arrays, Testing, AI Integration, Developer Experience
   - Roadmap themes: Execution Profiles, Deterministic Tooling, Shared Semantic State

## Theme Evolution Guidelines

When reviewing new features, ask:

1. **Does this fit an existing theme?** → Add to that theme's page
2. **Is this a major new capability?** → Consider a new theme
3. **Is this cross-cutting?** → May need multiple mentions but one authoritative page

### Signals for a New Theme

- 3+ related features with no natural home
- A new design doc folder (e.g., `v0_7_0/`) with a coherent focus
- User questions consistently asking "how do I do X?" where X isn't covered
- A planned feature graduating to implemented that's substantial

### Creating a New Theme

1. Add entry to `resources/feature_themes.md`
2. Create website page at appropriate location
3. Update sidebar in `docs/sidebars.js`
4. Cross-link from related themes
5. Update this skill's theme table above
