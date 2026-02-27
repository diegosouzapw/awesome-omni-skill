---
name: winuse
description: Control a Windows desktop remotely — screenshots, mouse, keyboard, window management. Use when the user asks to interact with a Windows machine, automate Windows apps, or do computer-use on Windows. Requires WinUse server running on the target Windows host.
homepage: https://github.com/nedos/winuse
metadata: {"moltbot":{"emoji":"🪟","requires":{"bins":["curl"]}}}
---

# WinUse — Windows Desktop Automation

Control a Windows machine via its REST API: take screenshots, move/click mouse, type text, manage windows. The Windows equivalent of xdotool + scrot, but over HTTP.

## Prerequisites

- **WinUse server** running on the target Windows machine (tray app + FastAPI)
- Network access from this machine to the Windows host (LAN or localhost if WSL)
- No API key needed (v1)

## Configuration

Set the Windows host (default `http://127.0.0.1:8080`):
```bash
export WINUSE_HOST="http://<windows-ip>:8080"
```

Store the host in `TOOLS.md` so future sessions know:
```markdown
### WinUse
- Windows host: http://192.168.1.100:8080
```

## Quick Reference

All endpoints return `{"success": true, "data": {...}, "error": null}`.

### Health Check

```bash
curl -s "$WINUSE_HOST/health" | python3 -m json.tool
```

### Screenshots

```bash
# Full screen
curl -s -X POST "$WINUSE_HOST/screenshot" -o /tmp/win_screen.png

# Specific window by HWND
curl -s -X POST "$WINUSE_HOST/screenshot" \
  -H "Content-Type: application/json" \
  -d '{"hwnd": 2293790}' -o /tmp/win_window.png
```

Then analyze with the `image` tool:
```
image prompt="Describe what's on screen" image=/tmp/win_screen.png
```

### Window Management

```bash
# List all windows
curl -s "$WINUSE_HOST/windows" | python3 -m json.tool

# Get active window
curl -s "$WINUSE_HOST/windows/active" | python3 -m json.tool

# Focus / minimize / maximize / restore
curl -s -X POST "$WINUSE_HOST/windows/<hwnd>/focus"
curl -s -X POST "$WINUSE_HOST/windows/<hwnd>/minimize"
curl -s -X POST "$WINUSE_HOST/windows/<hwnd>/maximize"
curl -s -X POST "$WINUSE_HOST/windows/<hwnd>/restore"
```

### Mouse

```bash
# Move to coordinates (absolute)
curl -s -X POST "$WINUSE_HOST/mouse/move" \
  -H "Content-Type: application/json" \
  -d '{"x": 500, "y": 300}'

# Move with duration (smooth)
curl -s -X POST "$WINUSE_HOST/mouse/move" \
  -H "Content-Type: application/json" \
  -d '{"x": 500, "y": 300, "duration": 0.2}'

# Click (left/right/middle, single or multi)
curl -s -X POST "$WINUSE_HOST/mouse/click" \
  -H "Content-Type: application/json" \
  -d '{"x": 500, "y": 300, "button": "left", "clicks": 1}'

# Double click
curl -s -X POST "$WINUSE_HOST/mouse/click" \
  -H "Content-Type: application/json" \
  -d '{"x": 500, "y": 300, "button": "left", "clicks": 2}'

# Right click
curl -s -X POST "$WINUSE_HOST/mouse/click" \
  -H "Content-Type: application/json" \
  -d '{"x": 500, "y": 300, "button": "right", "clicks": 1}'
```

### Keyboard

```bash
# Type text (character by character)
curl -s -X POST "$WINUSE_HOST/keyboard/type" \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello World", "mode": "type"}'

# Paste text (clipboard — better for UTF-8 and speed)
curl -s -X POST "$WINUSE_HOST/keyboard/paste" \
  -H "Content-Type: application/json" \
  -d '{"text": "Привет мир 🌍"}'

# Type via paste mode (same as /paste)
curl -s -X POST "$WINUSE_HOST/keyboard/type" \
  -H "Content-Type: application/json" \
  -d '{"text": "fast text", "mode": "paste"}'

# Press keys (hotkeys, special keys)
curl -s -X POST "$WINUSE_HOST/keyboard/press" \
  -H "Content-Type: application/json" \
  -d '{"keys": ["ctrl", "v"]}'

# Single key
curl -s -X POST "$WINUSE_HOST/keyboard/press" \
  -H "Content-Type: application/json" \
  -d '{"keys": ["enter"]}'

# Alt+Tab
curl -s -X POST "$WINUSE_HOST/keyboard/press" \
  -H "Content-Type: application/json" \
  -d '{"keys": ["alt", "tab"]}'

# Win+R (Run dialog)
curl -s -X POST "$WINUSE_HOST/keyboard/press" \
  -H "Content-Type: application/json" \
  -d '{"keys": ["win", "r"]}'
```

## Computer-Use Workflow

The standard agent loop for Windows automation:

```
1. Screenshot  →  POST /screenshot → save to /tmp/ → analyze with vision
2. Decide      →  Identify target coordinates or text to type
3. Act         →  POST /mouse/click or /keyboard/type or /keyboard/press
4. Verify      →  POST /screenshot → confirm action succeeded
5. Repeat
```

### Example: Open Notepad and Type

```bash
WINUSE_HOST="http://192.168.1.100:8080"

# 1. Open Run dialog
curl -s -X POST "$WINUSE_HOST/keyboard/press" \
  -H "Content-Type: application/json" \
  -d '{"keys": ["win", "r"]}'
sleep 1

# 2. Type "notepad" and press Enter
curl -s -X POST "$WINUSE_HOST/keyboard/paste" \
  -H "Content-Type: application/json" \
  -d '{"text": "notepad"}'
sleep 0.3
curl -s -X POST "$WINUSE_HOST/keyboard/press" \
  -H "Content-Type: application/json" \
  -d '{"keys": ["enter"]}'
sleep 2

# 3. Screenshot to verify Notepad opened
curl -s -X POST "$WINUSE_HOST/screenshot" -o /tmp/notepad.png

# 4. Type some text
curl -s -X POST "$WINUSE_HOST/keyboard/paste" \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello from the agent! 🤖"}'

# 5. Screenshot to verify
curl -s -X POST "$WINUSE_HOST/screenshot" -o /tmp/notepad_typed.png
```

### Example: Click a Button at Known Coordinates

```bash
# 1. Screenshot and analyze to find button position
curl -s -X POST "$WINUSE_HOST/screenshot" -o /tmp/screen.png
# → vision model says "OK button is at approximately (450, 320)"

# 2. Click it
curl -s -X POST "$WINUSE_HOST/mouse/click" \
  -H "Content-Type: application/json" \
  -d '{"x": 450, "y": 320, "button": "left", "clicks": 1}'

# 3. Verify
curl -s -X POST "$WINUSE_HOST/screenshot" -o /tmp/after_click.png
```

## Tips

- **Always prefer `paste` over `type` for text input** — faster, handles UTF-8/emoji correctly
- **Focus the window first** before typing — `POST /windows/<hwnd>/focus`
- **Add small delays** (`sleep 0.5`) between actions to let Windows catch up
- **Use HWND screenshots** when you only need one window — smaller, faster, no desktop clutter
- **List windows** to find the right HWND — match by window title
- **Store the WINUSE_HOST** in TOOLS.md so you don't have to ask every session

## Comparison with Linux CU

| Feature | WinUse (Windows) | CU Desktop API (Linux/Xvfb) |
|---------|-------------------|-------------------------------|
| Screenshot | `POST /screenshot` | `GET /screenshot` |
| Mouse move | `POST /mouse/move` | `POST /move` |
| Mouse click | `POST /mouse/click` | `POST /click` |
| Keyboard type | `POST /keyboard/type` | `POST /type` |
| Key press | `POST /keyboard/press` | `POST /key` |
| Window mgmt | `GET /windows`, focus/minimize/etc | N/A (single window) |
| Per-window capture | Yes (HWND) | No (full screen only) |
| Network | LAN / remote | Localhost (container) |
| Auth | None (v1) | None |

## Troubleshooting

### Can't connect

```bash
curl -s "$WINUSE_HOST/health"
```
- Check WinUse tray app is running on Windows
- Check firewall allows the port (default 8080)
- If on WSL: use the Windows host IP, not localhost

### Screenshots return empty/black

- Some apps (DRM-protected content, certain games) block screen capture
- Try window-specific screenshot with `--hwnd`

### Typing doesn't work

- Make sure the target window is focused: `POST /windows/<hwnd>/focus`
- Use `paste` mode for reliability
- Some apps intercept keyboard input differently — try `press` for hotkeys

### Mouse clicks miss

- Verify coordinates by taking a screenshot first
- Windows DPI scaling may affect coordinates — check display settings
- Some apps need a small delay after focus before accepting clicks
