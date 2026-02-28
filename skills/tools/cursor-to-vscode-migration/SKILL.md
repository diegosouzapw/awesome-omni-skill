---
name: cursor-to-vscode-migration
description: Migrate editor setup from Cursor to VS Code. Cleans old VS Code data, copies settings/keybindings from Cursor, removes Cursor-specific config keys, and installs all extensions fresh. Use when switching from Cursor to VS Code.
---

# Cursor to VS Code Migration

## Overview

Automate the full migration from Cursor to VS Code on macOS: clean stale VS Code data, transfer settings/keybindings, filter out Cursor-specific config, and install extensions.

## Workflow

### 1) Pre-flight checks

Before doing anything, verify the environment:

- Confirm Cursor config exists at `~/Library/Application Support/Cursor/User/` and `~/.cursor/extensions/`.
- Confirm VS Code is installed (`code --version`).
- Warn the user if VS Code is currently running -- it should be closed during migration.

If either Cursor config or VS Code is missing, stop and inform the user.

### 2) Audit old VS Code data

Check what exists and report sizes:

```shell
du -sh ~/.vscode/ 2>/dev/null
du -sh ~/Library/Application\ Support/Code/ 2>/dev/null
ls ~/.vscode/extensions/ 2>/dev/null | wc -l
```

Show the user what will be cleaned and **confirm before proceeding**.

### 3) Clean old VS Code data

Only after user confirmation, clean in this order:

**a) Remove old extensions:**

```shell
rm -rf ~/.vscode/extensions/
```

**b) Remove old settings:**

```shell
rm -f ~/.vscode/settings.json
rm -f ~/Library/Application\ Support/Code/User/settings.json
rm -f ~/Library/Application\ Support/Code/User/keybindings.json
```

**c) Clear caches and storage (optional, confirm separately):**

Run `scripts/clean_vscode_caches.sh` to remove all cache/storage directories under `~/Library/Application Support/Code/` while preserving `User/` and `machineid`.

### 4) Copy settings and keybindings from Cursor

```shell
cp ~/Library/Application\ Support/Cursor/User/settings.json \
   ~/Library/Application\ Support/Code/User/settings.json

cp ~/Library/Application\ Support/Cursor/User/keybindings.json \
   ~/Library/Application\ Support/Code/User/keybindings.json
```

If keybindings.json doesn't exist in Cursor, skip it silently.

### 5) Remove Cursor-specific settings

After copying, strip keys that are Cursor-specific and meaningless in VS Code. Run:

```shell
node ~/.codex/skills/cursor-to-vscode-migration/scripts/strip_cursor_keys.cjs \
  ~/Library/Application\ Support/Code/User/settings.json
```

This removes any keys matching `cursor.*` and `cursor.cpp.*` patterns from the JSON. Report which keys were removed.

### 6) Copy snippets (if they exist)

```shell
if [ -d ~/Library/Application\ Support/Cursor/User/snippets ]; then
  cp -r ~/Library/Application\ Support/Cursor/User/snippets/ \
        ~/Library/Application\ Support/Code/User/snippets/
fi
```

### 7) Install extensions

Do NOT copy extension binaries from `~/.cursor/extensions/`. Install fresh from the VS Code marketplace.

**a) Extract extension IDs from Cursor:**

```shell
node ~/.codex/skills/cursor-to-vscode-migration/scripts/list_cursor_extensions.cjs
```

This reads `~/.cursor/extensions/`, extracts publisher.name IDs, and filters out known Cursor-only extensions (see skip list below).

**b) Install into VS Code:**

```shell
cat /tmp/extensions_to_install.txt | xargs -L 1 code --install-extension --force
```

**Skip list** -- extensions that only work in Cursor or are no longer needed:

- `openai.chatgpt`

If an extension fails to install, log it and continue. Report all failures at the end.

### 8) Post-migration checklist

After installation completes, report:

- Number of extensions installed successfully
- Any extensions that failed to install
- Any Cursor-specific settings keys that were removed
- Remind the user to:
  - Open VS Code and verify everything looks correct
  - Enable Settings Sync (Settings > Turn on Settings Sync) to back up the new config
  - Close and reopen VS Code if any extensions show update errors

## Error handling

- If a step fails, do not proceed to the next step. Report the error and ask the user how to proceed.
- Never force-delete anything without user confirmation.
- If VS Code CLI (`code`) is not available, instruct the user to install it: Command Palette > "Shell Command: Install 'code' command in PATH".

## Platform note

This skill targets macOS. Key path differences for Linux:

| macOS | Linux |
|-------|-------|
| `~/Library/Application Support/Cursor/` | `~/.config/Cursor/` |
| `~/Library/Application Support/Code/` | `~/.config/Code/` |
