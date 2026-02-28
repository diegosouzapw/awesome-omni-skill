---
name: zed-editor
description: Configure, customize, and manage the Zed code editor. Use when the user wants to modify Zed keybindings, settings, tasks, or extensions — especially for chaining editor actions via workspace::SendKeystrokes, creating custom workflows, or opening projects in Zed. Also use when launching Zed from the CLI, setting up vim/helix mode bindings, or troubleshooting Zed configuration.
---

# Zed Editor Skill

Configure and customize the Zed code editor by reading and writing its JSON config files, managing keybindings (including `workspace::SendKeystrokes` command chaining), defining tasks, and launching Zed via CLI.

## Config File Locations

### macOS
```
~/Library/Application Support/Zed/settings.json    # Editor settings
~/Library/Application Support/Zed/keymap.json       # Keybindings
~/Library/Application Support/Zed/tasks.json        # Global tasks
```

### Linux
```
~/.config/zed/settings.json
~/.config/zed/keymap.json
~/.config/zed/tasks.json
```

### Per-project overrides
```
<project>/.zed/settings.json
<project>/.zed/tasks.json
```

**Detect OS first** — check `uname` to determine which paths to use.

---

## Core Workflow

1. **Detect OS** to resolve config paths
2. **Read existing config** before making any changes (preserve user's work)
3. **Validate JSON** — all Zed configs are JSON arrays or objects; malformed JSON will silently break the editor config
4. **Write changes** using careful JSON merging — don't clobber existing bindings/settings
5. **Verify** the file is valid JSON after writing

### Reading Config Safely

```bash
# macOS
CONFIG_DIR="$HOME/Library/Application Support/Zed"
# Linux
CONFIG_DIR="$HOME/.config/zed"

# Read keymap (may not exist yet)
cat "$CONFIG_DIR/keymap.json" 2>/dev/null || echo "[]"
# Read settings
cat "$CONFIG_DIR/settings.json" 2>/dev/null || echo "{}"
```

---

## Keybindings (keymap.json)

The keymap file is a JSON **array** of binding objects. Each object can have an optional `context` and a required `bindings` map.

### Structure

```json
[
  {
    "context": "Editor && vim_mode == normal",
    "bindings": {
      "key-combo": "action::Name",
      "key-combo": ["action::Name", { "param": "value" }]
    }
  }
]
```

### Key Syntax

Keys use modifier-key format separated by `-`:
- Modifiers: `cmd`, `ctrl`, `alt`, `shift`, `fn`
- Special keys: `up`, `down`, `left`, `right`, `enter`, `escape`, `tab`, `space`, `backspace`, `delete`, `home`, `end`, `pageup`, `pagedown`
- Letters/numbers: `a`-`z`, `0`-`9`
- Multi-key sequences: `"g e"` (space-separated, typed in sequence)

Examples: `"cmd-shift-p"`, `"ctrl-w h"`, `"alt-down"`, `"g e"`

### Context Expressions

Contexts scope bindings to specific UI states. Use `&&` for AND, `||` for OR, `!` for NOT, `>` for ancestor matching.

Common contexts:
- `Editor` — any editor pane
- `Terminal` — terminal panel
- `ProjectPanel` — file tree
- `Dock` — any dock panel
- `vim_mode == normal` / `insert` / `visual` — vim mode states
- `VimControl` — normal + visual modes combined
- `menu` — autocomplete/palette is open
- `EmptyPane` — no file open

Examples:
```json
"context": "Editor && vim_mode == normal && !menu"
"context": "Terminal"
"context": "VimControl && !menu"
```

---

## workspace::SendKeystrokes — Command Chaining

This is Zed's most powerful keybinding primitive. It dispatches a sequence of keystrokes synchronously, letting you chain multiple actions into a single keybinding **without writing an extension**.

### Syntax

```json
"key": ["workspace::SendKeystrokes", "keystroke1 keystroke2 keystroke3"]
```

Keystrokes are space-separated. Each keystroke uses the same modifier-key syntax as keybinding keys.

### Important Constraints

- **Synchronous only**: SendKeystrokes dispatches all keys before any async operation completes. You CANNOT rely on async results (opening files, language server responses, command palette searches) between keystrokes.
- **No cross-view chaining**: You cannot send keys to a view that opens as a result of a previous keystroke in the same sequence.
- Async operations include: opening command palette, language server communication, changing buffer language, network requests.

### Patterns

**Chain copy + deselect:**
```json
"alt-w": ["workspace::SendKeystrokes", "cmd-c escape"]
```

**Move down N lines:**
```json
"alt-down": ["workspace::SendKeystrokes", "down down down down"]
```

**Select syntax node + copy + undo selection:**
```json
"cmd-alt-c": [
  "workspace::SendKeystrokes",
  "ctrl-shift-right ctrl-shift-right ctrl-shift-right cmd-c ctrl-shift-left ctrl-shift-left ctrl-shift-left"
]
```

**Vim yank to end of line (neovim style):**
```json
{
  "context": "vim_mode == normal && !menu",
  "bindings": {
    "shift-y": ["workspace::SendKeystrokes", "y $"]
  }
}
```

**Vim insert mode escape via jk:**
```json
{
  "context": "Editor && vim_mode == insert",
  "bindings": {
    "j k": ["workspace::SendKeystrokes", "escape"]
  }
}
```

### Strategy for Building SendKeystrokes Chains

1. First identify the individual actions needed (use `zed: open default keymap` or the All Actions reference)
2. Find the existing keybindings for each action
3. Chain those key combos in a single SendKeystrokes string
4. Test that none of the intermediate actions are async

---

## Common Actions Reference

### Editor Actions
| Action | Description |
|--------|-------------|
| `editor::Copy` | Copy selection |
| `editor::Cut` | Cut selection |
| `editor::Paste` | Paste |
| `editor::Undo` | Undo |
| `editor::Redo` | Redo |
| `editor::SelectAll` | Select all |
| `editor::Format` | Format document |
| `editor::ToggleComments` | Toggle line comments |
| `editor::MoveLineUp` | Move current line up |
| `editor::MoveLineDown` | Move current line down |
| `editor::DuplicateLineDown` | Duplicate line |
| `editor::SelectLargerSyntaxNode` | Expand selection to syntax |
| `editor::SelectSmallerSyntaxNode` | Shrink selection |
| `editor::GoToDefinition` | Go to definition |
| `editor::GoToDeclaration` | Go to declaration |
| `editor::GoToImplementation` | Go to implementation |
| `editor::Rename` | Rename symbol |
| `editor::ToggleCodeActions` | Show code actions |
| `editor::Newline` | Insert newline |
| `editor::Tab` | Insert tab/indent |
| `editor::Outdent` | Outdent |
| `editor::FoldAll` | Fold all |
| `editor::UnfoldAll` | Unfold all |

### Workspace Actions
| Action | Description |
|--------|-------------|
| `workspace::NewFile` | New file |
| `workspace::Save` | Save |
| `workspace::SaveAll` | Save all |
| `workspace::Open` | Open file dialog |
| `workspace::ToggleLeftDock` | Toggle left dock |
| `workspace::ToggleRightDock` | Toggle right dock |
| `workspace::ToggleBottomDock` | Toggle bottom dock |
| `workspace::ActivateNextPane` | Focus next pane |
| `workspace::ActivatePreviousPane` | Focus previous pane |
| `workspace::SendKeystrokes` | Chain keystrokes |

### Navigation
| Action | Description |
|--------|-------------|
| `file_finder::Toggle` | Open file finder |
| `project_symbols::Toggle` | Open symbol search |
| `buffer_search::Deploy` | Find in file |
| `project_search::ToggleFocus` | Find in project |
| `outline::Toggle` | Open outline view |
| `go_to_line::Toggle` | Go to line number |
| `tab_switcher::Toggle` | Switch tabs |
| `diagnostics::Deploy` | Open diagnostics |

### Terminal
| Action | Description |
|--------|-------------|
| `terminal_panel::ToggleFocus` | Toggle/focus terminal |
| `workspace::NewTerminal` | New terminal tab |

### Git
| Action | Description |
|--------|-------------|
| `git_panel::ToggleFocus` | Toggle/focus git panel |
| `git::Diff` | Open project diff view |
| `git::OpenModifiedFiles` | Open all modified files |
| `git::StageAndNext` | Stage current hunk, advance to next |
| `git::UnstageAndNext` | Unstage current hunk, advance to next |
| `git::ToggleStaged` | Toggle staged state |
| `git::Commit` | Commit staged changes |
| `git::GenerateCommitMessage` | AI-generated commit message |
| `git::Fetch` | Fetch from remote |
| `git::Push` | Push to remote |
| `git::Pull` | Pull from remote |
| `git::Restore` | Restore/undo changes |
| `editor::GoToHunk` | Navigate to next diff hunk |
| `editor::GoToPreviousHunk` | Navigate to previous diff hunk |
| `editor::ExpandAllDiffHunks` | Expand all hunks inline |
| `editor::ToggleSelectedDiffHunks` | Toggle diff for selected hunks |

---

## Settings (settings.json)

Settings is a JSON **object**. Key settings to know about:

```json
{
  // Font
  "buffer_font_family": "Berkeley Mono",
  "buffer_font_size": 14,

  // Theme
  "theme": "One Dark",

  // Vim/Helix mode
  "vim_mode": true,
  "helix_mode": false,

  // Formatting
  "format_on_save": "on",
  "formatter": "auto",

  // Tab settings
  "tab_size": 2,
  "hard_tabs": false,

  // Soft wrap
  "soft_wrap": "editor_width",
  "preferred_line_length": 100,

  // Autosave
  "autosave": { "after_delay": { "milliseconds": 1000 } },

  // Per-language overrides
  "languages": {
    "Python": {
      "tab_size": 4,
      "formatter": {
        "external": {
          "command": "black",
          "arguments": ["-"]
        }
      }
    }
  },

  // Inlay hints
  "inlay_hints": {
    "enabled": true
  },

  // Telemetry
  "telemetry": {
    "diagnostics": false,
    "metrics": false
  },

  // AI / Assistant
  "assistant": {
    "default_model": {
      "provider": "anthropic",
      "model": "claude-sonnet-4-5-20250514"
    }
  }
}
```

### Git Settings

```json
{
  "git_panel": {
    "button": true,
    "dock": "left",
    "default_width": 360,
    "status_style": "icon",
    "sort_by_path": false,
    "tree_view": false
  },
  "git": {
    "git_gutter": "tracked_files",
    "inline_blame": {
      "enabled": true,
      "delay_ms": 600,
      "show_commit_summary": true
    },
    "hunk_style": "staged_hollow"
  }
}
```

---

## Tasks (tasks.json)

Tasks let you run shell commands from Zed with access to editor context variables.

### Structure

```json
[
  {
    "label": "Run current file",
    "command": "python $ZED_FILE",
    "use_new_terminal": false,
    "allow_concurrent_runs": false,
    "reveal": "always"
  }
]
```

### Available Environment Variables

| Variable | Value |
|----------|-------|
| `$ZED_FILE` | Absolute path of current file |
| `$ZED_FILENAME` | Filename only |
| `$ZED_DIRNAME` | Directory of current file |
| `$ZED_RELATIVE_FILE` | File path relative to project root |
| `$ZED_RELATIVE_DIR` | Directory relative to project root |
| `$ZED_STEM` | Filename without extension |
| `$ZED_ROW` | Current cursor row |
| `$ZED_COLUMN` | Current cursor column |
| `$ZED_SELECTED_TEXT` | Currently selected text |
| `$ZED_WORKTREE_ROOT` | Project root directory |

### Task Options

| Option | Values | Default |
|--------|--------|---------|
| `use_new_terminal` | `true`/`false` | `false` |
| `allow_concurrent_runs` | `true`/`false` | `false` |
| `reveal` | `"always"`, `"no_focus"`, `"never"` | `"always"` |
| `hide` | `"never"`, `"always"`, `"on_success"` | `"never"` |
| `cwd` | path string | project root |
| `env` | object of env vars | `{}` |
| `tags` | array of strings | `[]` |

---

## CLI Usage

```bash
# Open a file or directory
zed .
zed file.txt
zed project/ file.txt

# Open in new window
zed --new file.txt

# Diff two files
zed --diff old.rs new.rs

# Wait for file to close (for EDITOR usage)
zed --wait file.txt

# Open at specific line:column
zed file.txt:42
zed file.txt:42:10

# Set as default editor
export EDITOR="zed --wait"
export VISUAL="zed --wait"

# Open settings/keymap directly
zed zed://settings
zed zed://keymap

# macOS: choose release channel
zed --stable file.txt
zed --preview file.txt
zed --nightly file.txt
```

---

## Extensions

### Installed extensions location
- macOS: `~/Library/Application Support/Zed/extensions/installed/`
- Linux: `~/.local/share/zed/extensions/installed/`

Extensions are primarily managed through the Zed UI (`cmd-shift-x` / `ctrl-shift-x`), but you can inspect installed extensions via the filesystem.

---

## Workflow: Adding a Custom Keybinding

This is the most common task. Follow this procedure:

1. **Read the current keymap:**
   ```bash
   cat "$CONFIG_DIR/keymap.json" 2>/dev/null || echo "[]"
   ```

2. **Parse the JSON** and check for conflicting bindings on the same key in the same context.

3. **Determine the action(s) needed.** If a single action suffices, bind directly:
   ```json
   "cmd-shift-d": "editor::DuplicateLineDown"
   ```

4. **If multiple actions are needed**, use `workspace::SendKeystrokes`:
   - Look up existing keybindings for each desired action
   - Chain them: `["workspace::SendKeystrokes", "key1 key2 key3"]`
   - Verify none are async

5. **Add the binding** to the appropriate context block (or create a new one).

6. **Write the file** and validate JSON:
   ```bash
   python3 -c "import json; json.load(open('$CONFIG_DIR/keymap.json'))" && echo "Valid JSON"
   ```

---

## Workflow: Opening a Project in Zed

```bash
# Check if zed CLI is available
which zed || echo "Zed CLI not installed. Run 'Zed > Install CLI' from the app menu."

# Open project
zed /path/to/project

# Open project with specific file focused
zed /path/to/project /path/to/project/src/main.rs
```

---

## Workflow: Creating a Project Task Runner

1. Create `.zed/tasks.json` in the project root
2. Define tasks that use Zed environment variables
3. Run tasks from the command palette or bind them to keys

Example — Rust project:
```json
[
  {
    "label": "cargo run",
    "command": "cargo run",
    "cwd": "$ZED_WORKTREE_ROOT"
  },
  {
    "label": "cargo test current file",
    "command": "cargo test --lib $ZED_STEM",
    "cwd": "$ZED_WORKTREE_ROOT",
    "reveal": "always"
  },
  {
    "label": "cargo clippy",
    "command": "cargo clippy --all-targets",
    "cwd": "$ZED_WORKTREE_ROOT",
    "hide": "on_success"
  }
]
```

---

## Gotchas

- **JSON only** — Zed configs don't support comments (unlike VS Code's JSONC). Strip any `//` comments before writing.
- **Array vs Object** — `keymap.json` and `tasks.json` are arrays `[]`. `settings.json` is an object `{}`.
- **Later bindings win** — If two bindings match the same context+key, the one defined later takes precedence. User bindings always override defaults.
- **SendKeystrokes is synchronous** — Don't try to chain actions that depend on async results (file opens, palette searches, LSP responses).
- **Context matching** — Contexts match at specific tree levels. `vim_mode` is set at `Editor` level, so `"Workspace && vim_mode == normal"` will never match.
- **Zed auto-reloads config** — changes to keymap.json and settings.json take effect immediately without restarting.

---

## Claude Code Integration: Change Review Workflow

This workflow bridges Claude Code (running in Warp, Ghostty, or any external terminal) with Zed for seamless change review.

### How It Works

1. A PostToolUse hook in Claude Code records every file edit to a session manifest
2. A Zed task reads the manifest and opens all changed files at the right lines
3. Git review keybindings let you cycle through diffs, stage/revert per hunk

### Setup

**Step 1: Install the hook**

Copy the hook script to a permanent location:

```bash
mkdir -p ~/.claude/hooks
cp hooks/post-tool-use.sh ~/.claude/hooks/post-tool-use.sh
chmod +x ~/.claude/hooks/post-tool-use.sh
```

**Step 2: Configure Claude Code**

Add to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/post-tool-use.sh"
          }
        ]
      }
    ]
  }
}
```

**Step 3: Add the Zed tasks**

Merge the contents of `examples/tasks.json` into your global Zed tasks file:
- macOS: `~/Library/Application Support/Zed/tasks.json`
- Linux: `~/.config/zed/tasks.json`

**Step 4: Add review keybindings (optional)**

Merge `examples/keymap.json` into your Zed keymap:
- macOS: `~/Library/Application Support/Zed/keymap.json`
- Linux: `~/.config/zed/keymap.json`

**Step 5: Configure git panel (optional)**

Merge `examples/settings.json` into your Zed settings to dock the git panel on the right for a review-friendly layout.

### Usage

1. Run Claude Code in your terminal -- it edits files as usual
2. Switch to Zed
3. Run the **"Review Claude Changes"** task from the command palette (`cmd-shift-p` > "task: spawn" > "Review Claude Changes")
4. All files Claude edited open at the relevant lines
5. Use `alt-]` / `alt-[` to jump between diff hunks
6. Use `alt-\` to expand all diffs inline
7. Use the git panel to stage/revert changes
8. When done reviewing, run **"Clear Claude Change Manifest"** to reset

### Change Manifest

Changes accumulate per Claude Code session at `~/.claude/changes/<session-id>.json`. A symlink at `~/.claude/changes/latest.json` always points to the current session.

Format:
```json
{
  "session_id": "abc123",
  "started": "2026-02-14T10:30:00Z",
  "changes": [
    {"file": "/path/to/file.rs", "line": 42, "tool": "Edit", "timestamp": "..."},
    {"file": "/path/to/new.rs", "line": 1, "tool": "Write", "timestamp": "..."}
  ]
}
```
