---
name: kano-commit-convention-skill
description: Commit/change description convention (KCC) with Subsystem + Type + Ticket formatting, lint rules, and VCS-agnostic guidance.
metadata:
  short-description: Commit message convention (KCC)
---

# Kano Commit Convention (KCC)

## Scope

This skill enforces a structured commit message convention (KCC-STCC) across the repository. It helps maintain a machine-readable history for automated changelogs and semantic versioning.

Use this skill when:
- Setting up or troubleshooting commit message linting.
- Generating release notes or identifying breaking changes.
- Calculating the next semantic version.
- Guidance on compliant commit subject formatting.

## Non-negotiables

- **Format**: `[Subsystem][Type] Summary (Ticket)`
- **Tickets**: Always require an explicit ID (e.g., `APP-123`) or `(NO-TICKET)`.
- **Breaking Changes**: Must include `[Breaking]` in the subject or `BREAKING CHANGE` in the footer.
- **Backlog Sync**: Linter verifies ticket existence against `_kano/backlog/`.

## CLI (new, Typer)

- Entrypoint: `python skills/kano-commit-convention-skill/scripts/kano-commit <command>`
- Commands:
  - `lint message --message/--file [--json] [--language en|zh] [--strict-breaking]`
  - `hook install|uninstall [--repo-root <path>]`
  - `commit wizard`
  - `release changelog|bump|release` (changelog generation, version bump, release helper)
  - `doctor check` (lightweight env check)

Legacy分散腳本已整合進 Typer CLI；請改用 `kano-commit` 子命令。

## Configuration (kcc.json)

Customize KCCS behavior at the repo root:
```json
{
  "allowed_types": ["Feature", "BugFix", "Refactor"],
  "language": "en"
}
```
- `language`: `en` (default) or `zh` for localized linter messages.

## Usage Examples

### Installing Hooks
```bash
python scripts/vcs/install_hooks.py
```

### Validating a Message Manually
```bash
python scripts/vcs/linter.py "my commit message"
```

### Generating a P4 Changelog
```bash
python scripts/release/generate_changelog.py --vcs p4 --depot-path //depot/my-project/...
```

### Performing a Release Dry-Run
```bash
python scripts/release/release.py --dry-run
```

## Primary Reference
- [KCC Specification v0.0.1](references/Kano-commit-convention.md)
