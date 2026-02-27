---
name: excalidraw-agent
description: Integrate and automate Excalidraw in React and Next.js apps, convert Mermaid to .excalidraw scenes, merge/normalize .excalidrawlib libraries, and troubleshoot Excalidraw runtime issues. Use when users mention Excalidraw, Mermaid diagrams, .excalidraw or .excalidrawlib files, Excalidraw API props/methods, export/import flows, or host-app collaboration wiring.
---

# Excalidraw Agent

## Overview

Use this skill to deliver end-to-end Excalidraw work in host apps: integration, API-driven scene updates, Mermaid conversion, library operations, export/import pipelines, collaboration patterns, and runtime diagnostics.

Prefer this workflow: triage the task, load only the relevant reference file, execute using deterministic scripts when possible, validate output, and report exact outcomes.

## Core Workflow

1. Triage the request type.
2. Load only the required reference file(s) from `references/`.
3. Run environment check when scripts are involved:
   - `scripts/check_env.sh`
4. Execute the task path.
5. Validate the result (script checks + structural checks + behavior checks).
6. Report what changed, what was verified, and remaining risks.

## Task Triage

- Installation or framework embedding: load `references/excalidraw-install-and-integration.md`
- Props and imperative API usage: load `references/excalidraw-props-and-api.md`
- Serialization, restore, and export/import: load `references/excalidraw-utils-restore-export.md`
- Editor composition (`MainMenu`, `Sidebar`, `WelcomeScreen`, `Footer`): load `references/excalidraw-ui-composition.md`
- Mermaid conversion workflows: load `references/excalidraw-mermaid-conversion.md`
- Collaboration architecture and remote update behavior: load `references/excalidraw-collaboration-pattern.md`
- Account-linked import/publish via authenticated browser session: load `references/excalidraw-account-linking.md`
- Client setup/portability across Codex, Claude Code, Cursor, and Zed: load `references/excalidraw-client-compatibility.md`
- Runtime or build failures: load `references/excalidraw-troubleshooting.md`
- Skills protocol constraints and metadata behavior: load `references/agent-skills-protocol-notes.md`

## Hard Integration Requirements

Apply these on every Excalidraw integration unless the user explicitly asks otherwise:

1. Import Excalidraw stylesheet:
   - `import "@excalidraw/excalidraw/index.css";`
2. Render in a container with non-zero height and width.
3. Use `excalidrawAPI` callback for imperative control; do not use removed legacy `ref` API patterns.
4. For Next.js, disable SSR for Excalidraw rendering via dynamic import.
5. If self-hosting fonts/assets, set `window.EXCALIDRAW_ASSET_PATH` correctly.

## Script Interfaces

Run scripts from this skill root or by absolute path.

- `scripts/check_env.sh`
- `scripts/mermaid_to_scene.mjs --input <mmd|-> --output <scene.excalidraw> --font-size <n> --regenerate-ids <true|false> --pretty <true|false>`
- `scripts/scene_lint.mjs --input <scene.excalidraw> --strict-diagram <true|false>`
- `scripts/library_merge.mjs --base <a.excalidrawlib> --other <b.excalidrawlib> --output <merged.excalidrawlib> --default-status <published|unpublished>`
- `scripts/import_to_excalidraw.sh --input <path> --destination <plus|excalidraw> --kind <auto|scene|library> --mode <headed|headless> --session <name> --output-dir <path> --timeout-sec <n> --dry-run <true|false> --close-on-complete <true|false> --pwcli <path>`
- `scripts/self_test.sh`

## Deterministic Execution Notes

- Scripts resolve Excalidraw dependencies from `process.cwd()` first, then from the skill directory.
- Recommended runtime is Node + pnpm.
- For conversions and merges, run from a project directory where `@excalidraw/excalidraw` and `@excalidraw/mermaid-to-excalidraw` are available, or install them in a temporary working directory first.
- If `@excalidraw/excalidraw` cannot be loaded in the active Node runtime, scripts emit a warning and apply deterministic fallback normalization so workflows continue.
- The account-link import script supports multiple agent environments by resolving Playwright in this order:
  1. explicit `--pwcli` or `PWCLI`
  2. known wrapper paths under `~/.codex`, `~/.claude`, `~/.cursor`, and `~/.config/zed`
  3. global `playwright-cli`
  4. `npx --yes --package @playwright/mcp playwright-cli`
- API-level account linking is not available in this skill. Account linking is implemented as authenticated UI import on `excalidraw.com` or `plus.excalidraw.com`.

## Multi-Agent Compatibility

- Core Agent Skills contract is `SKILL.md` + optional `references/`, `scripts/`, and `assets/`.
- `agents/openai.yaml` is optional metadata for Codex UX and can be ignored by other clients.
- This skill is designed to be portable: all operational instructions are rooted in `SKILL.md`, script paths are skill-root relative, and account-link import can run with any Playwright CLI provider via `--pwcli`.
- For client-specific setup and invocation patterns, load `references/excalidraw-client-compatibility.md`.

## Execution Paths

### 1) Embed Excalidraw in a Host App

1. Load `references/excalidraw-install-and-integration.md`.
2. Implement base integration for React/Next.js/Preact as needed.
3. Confirm CSS import and non-zero container dimensions.
4. Validate editor rendering and input behavior.

### 2) Implement API-Driven Scene or UI Behavior

1. Load `references/excalidraw-props-and-api.md`.
2. Use `excalidrawAPI` callback to capture API instance.
3. Use `updateScene` and `captureUpdate` semantics intentionally:
   - local undoable updates: `IMMEDIATELY`
   - async grouped updates: `EVENTUALLY`
   - remote/collab updates: `NEVER`
4. Verify behavior via event subscriptions (`onChange`, pointer handlers).

### 3) Convert Mermaid to `.excalidraw`

1. Load `references/excalidraw-mermaid-conversion.md`.
2. Run conversion:
   - `node scripts/mermaid_to_scene.mjs --input diagram.mmd --output diagram.excalidraw --font-size 16 --regenerate-ids true --pretty true`
3. Validate output:
   - `node scripts/scene_lint.mjs --input diagram.excalidraw --strict-diagram true`
4. If parser fallback behavior appears, report which nodes were downgraded.
5. If import layout is unreadable (for example tall single-column collapse), simplify labels/graph, reconvert, and expect a manual arrangement pass in Excalidraw.

### 4) Merge and Normalize Libraries

1. Load `references/excalidraw-utils-restore-export.md`.
2. Merge:
   - `node scripts/library_merge.mjs --base lib-a.excalidrawlib --other lib-b.excalidrawlib --output merged.excalidrawlib --default-status unpublished`
3. Confirm merged output contains normalized `libraryItems` and expected dedupe behavior.

### 5) Build Collaboration Glue

1. Load `references/excalidraw-collaboration-pattern.md`.
2. Keep transport and persistence in host app; keep editor as client state surface.
3. Apply remote updates with non-undo capture behavior.
4. Verify collaborator map rendering and local-vs-remote conflict handling.

### 6) Troubleshoot Runtime Failures

1. Load `references/excalidraw-troubleshooting.md`.
2. Diagnose by category: SSR/build, CSS/layout, browser quirks, asset path, env flags.
3. Validate fixed state with a minimal reproducible integration.

### 7) Link Generated Content to Excalidraw Account

1. Load `references/excalidraw-account-linking.md`.
2. Run import script in headed mode for interactive login:
   - `bash scripts/import_to_excalidraw.sh --input diagram.excalidraw --destination plus --mode headed`
3. Complete manual login/MFA at checkpoint.
4. Let script run deterministic import strategy sequence and UI assertions.
5. Confirm persistence checkpoint for `plus` destination.
6. Collect screenshot proof path and `RESULT_JSON` output.

### 8) Configure Skill for a Specific Agent Client

1. Load `references/excalidraw-client-compatibility.md`.
2. Place or symlink the skill directory into the client's skill root.
3. Validate script execution from that environment:
   - `bash scripts/check_env.sh`
   - `bash scripts/import_to_excalidraw.sh --input assets/examples/scene-minimal.excalidraw --dry-run true`
4. If Playwright is not discovered automatically, set `--pwcli` explicitly.
5. Confirm the client can load `SKILL.md` and access `references/` and `scripts/`.

## Validation Checklist

- Skill metadata and structure validate cleanly.
- Script CLIs run with documented flags.
- Converted scenes parse and lint successfully.
- Merged libraries are normalized and readable by Excalidraw tooling.
- Recommendations reference the correct framework constraints (React/Next/Preact).
- Cross-client setup guidance is present and does not assume a single agent runtime.

## Skill Maintenance

- Keep `SKILL.md` concise and procedural.
- Keep deep details in `references/`.
- Keep references one hop from `SKILL.md`.
- Re-run validation after edits:
  - `python <path-to-skill-creator>/scripts/quick_validate.py <path-to-skill-dir>`
  - `skills-ref validate <path-to-skill-dir>`
