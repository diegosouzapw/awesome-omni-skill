---
name: investigation-codex-vscode-extension
description: This skill should be used when investigating why the Codex VS Code extension fails to spawn or connect to a local Codex agent, especially after updates or CLI changes.
---

# Investigation: Codex VS Code Extension
*[Created by Codex: 019be564-39d9-7b12-a4cf-fb7a7f1af223 2026-01-22]*

## Purpose
Provide a fast, repeatable workflow to diagnose why the Codex VS Code extension cannot spawn or connect to the local `codex` app-server.

## When to Use
Use when a user reports that Codex in VS Code will not start, errors on spawn, or fails after an extension or CLI update.

## Warning Flags (Check First)
- **Non-default CLI override is set**: If `"chatgpt.cliExecutable"` is configured, the extension will run that binary instead of the bundled 45MB `codex`.
- **Assume “custom build” risk**: An override often points at a locally built or modified Codex repo; the user must know exactly what it is and why it differs.
- **Always surface it explicitly**: State the resolved `codex` path and `codex --version` in the RCA, and ask the user to confirm the repo/build they intended to run.

## Incident Example (2026-01-22)
- Symptom: “Cannot spawn a Codex agent in VS Code” after updating the extension to `openai.chatgpt` `0.4.62`.
- Root cause: The extension started `codex app-server --analytics-default-enabled`, but the user had `"chatgpt.cliExecutable"` pinned to a `codex-cli 0.77.0` binary that did not recognize that flag, so the process exited with code `2`.
- Evidence: `~/Library/Application Support/Code/logs/.../window*/exthost/openai.chatgpt/Codex.log` contained `error: unexpected argument '--analytics-default-enabled' found`.
- Fix: Remove or update `"chatgpt.cliExecutable"` to a compatible `codex` binary (or let the extension use its bundled one), then reload VS Code.

## Workflow (Non-Invasive)
1. Identify the *actual* `codex` binary VS Code is running (this is frequently the root cause):
   - Check for `"chatgpt.cliExecutable"` in:
     - `~/Library/Application Support/Code/User/settings.json`
     - `<repo>/.vscode/settings.json`
   - If set, treat it as a **high-priority hypothesis**: VS Code is running a non-default `codex` (often a custom build).
   - If set, tell the user the resolved path and ask them to confirm which repo/build that binary came from.
2. Identify extension version and entrypoint:
   - `ls ~/.vscode/extensions | rg -i 'openai.chatgpt'`
   - `cat ~/.vscode/extensions/<ext>/package.json` (read `main`)
   - `rg -n -- 'spawn|CLI_EXECUTABLE|codex|app-server' ~/.vscode/extensions/<ext>/out/extension.js`
3. Determine which `codex` binary the extension uses:
   - If `"chatgpt.cliExecutable"` is set, it wins.
   - Otherwise, the fallback is typically: `~/.vscode/extensions/<ext>/bin/<platform>/codex`.
4. Pull extension logs (most authoritative):
   - `~/Library/Application Support/Code/logs/<latest>/window*/exthost/openai.chatgpt/Codex.log`
   - `.../exthost/exthost.log` for activation and stack traces.
   - Prioritize explicit CLI errors (unknown flags, `ENOENT`, `EACCES`, exit code).
5. Correlate failure with CLI arguments:
   - Compare args in `extension.js` with `codex app-server --help` for the selected binary.
   - If flags mismatch, conclude the extension is running an older CLI (often via `chatgpt.cliExecutable`).
6. Run sanity checks:
   - Compare `codex --version` for (a) terminal PATH (b) configured `chatgpt.cliExecutable` target (c) extension-bundled binary.
   - Verify executable permissions or quarantine status on macOS when `EACCES` appears.
7. Draft RCA and minimal fix:
   - Cite the exact log line and file path.
   - Provide the smallest fix: remove/update `chatgpt.cliExecutable`, point to newer binary, reload window.

## Common Root Causes
- `chatgpt.cliExecutable` pinned to an older CLI missing new flags.
- Extension update changed CLI arguments but local binary stayed old.
- Missing or non-executable bundled binary after update.
- macOS quarantine or code signing issues (`EACCES`).

## Output Expectations
- Provide a short RCA (1–3 bullets) and minimal fix steps.
- Avoid invasive code changes unless explicitly requested.
