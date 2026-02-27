---
name: screenshot
description: Capture screenshots of application windows
user-invocable: true
---

# Screenshot Skill

Capture screenshots of macOS application windows using ScreenCaptureKit.

## Usage

```bash
shotty <command> [options]
```

## Commands

| Command | Description |
|---------|-------------|
| `capture <app\|pid> [output.png]` | Screenshot an app's window(s) |
| `list` | List all capturable windows |
| `list-apps` | List running applications |

## Options

| Flag | Description |
|------|-------------|
| `--all` | Capture all windows (not just first) |
| `--json` | Output as JSON |
| `--no-frame` | Exclude window frame/shadow |

## When to Use

- User asks to "screenshot this conversation" or "capture this window"
- User wants visual verification of UI changes
- User needs to document current application state
- User asks to see what an app looks like

## Examples

Screenshot the current terminal:
```bash
shotty capture Ghostty /tmp/terminal.png
```

Screenshot a browser:
```bash
shotty capture Safari ~/Desktop/browser.png
```

Screenshot by PID:
```bash
shotty capture 12345 output.png
```

List available apps:
```bash
shotty list-apps
```

Capture all windows of an app:
```bash
shotty capture "VS Code" --all
```

## Workflow

1. If user doesn't specify an app, use `list-apps` to find the right target
2. Run `capture` with the app name or PID
3. Use the `Read` tool to view the resulting PNG and verify the capture
4. Report the file path to the user

## Notes

- Requires Screen Recording permission (System Settings → Privacy & Security → Screen Recording)
- App names are matched case-insensitively with substring support
- Default output is `screenshot.png` in current directory
- Use `--no-frame` for cleaner captures without window shadow
