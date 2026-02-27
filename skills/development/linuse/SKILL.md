---
name: linuse
description: Control the Linux/GNOME/Wayland desktop — screenshots, mouse taps, keyboard input. Use when asked to interact with the desktop, automate GUI apps, or do computer-use on the local Linux machine.
metadata: {"emoji": "🐧"}
---

# LinUse — Linux Desktop Automation (Wayland/GNOME)

Control the local GNOME Wayland desktop: take screenshots, tap/click, type text, press keys. Uses evdev virtual touchscreen + keyboard to bypass Wayland's input restrictions.

## Prerequisites

- GNOME on Wayland (tested on GNOME 44–46, Ubuntu 22.04–24.04)
- `gnome-screenshot` installed (`sudo apt install gnome-screenshot`)
- `evdev` Python package (`pip install evdev`)
- `/dev/uinput` writable (`sudo chmod 666 /dev/uinput`)
- Screen lock disabled for automation:
  ```bash
  gsettings set org.gnome.desktop.screensaver lock-enabled false
  gsettings set org.gnome.desktop.session idle-delay 0
  gsettings set org.gnome.desktop.interface enable-hot-corners false
  ```

## Location

```
/path/to/linuse/
```

## Quick Reference

All commands via: `cd /path/to/linuse && python3 -m linuse <command>`

### Single Commands

```bash
# Display info
python3 -m linuse info

# Screenshot (returns file path)
python3 -m linuse screenshot
python3 -m linuse screenshot -o /tmp/screen.png

# Tap at coordinates
python3 -m linuse tap 640 400

# Double-tap
python3 -m linuse double-tap 640 400

# Long press
python3 -m linuse long-press 640 400 --duration 0.5

# Drag
python3 -m linuse drag 100 100 500 500 --duration 0.3

# Type text (character by character via evdev)
python3 -m linuse type "hello world"

# Press single key
python3 -m linuse key Return
python3 -m linuse key Tab
python3 -m linuse key Escape

# Key combo
python3 -m linuse combo ctrl c
python3 -m linuse combo alt F4
python3 -m linuse combo ctrl shift t
```

### Chained Commands (IMPORTANT)

For keyboard input to work reliably, **chain commands in a single invocation** using `,` as separator. This keeps the evdev devices alive across actions:

```bash
python3 -m linuse tap 640 60 , sleep 0.5 , type "example.com" , sleep 0.3 , key Return
python3 -m linuse combo ctrl l , sleep 0.5 , type "google.com" , key Return
python3 -m linuse combo alt F4
```

**Why?** Mutter registers/deregisters evdev devices when they're created/destroyed. Single commands create→use→destroy per invocation. Chaining keeps them alive.

## Computer-Use Workflow

```
1. Screenshot  →  python3 -m linuse screenshot -o /tmp/screen.png
                  → analyze with image tool
2. Decide      →  identify target coordinates or text to type
3. Act         →  python3 -m linuse tap X Y , sleep 0.3 , type "text" , key Return
4. Verify      →  python3 -m linuse screenshot -o /tmp/after.png
                  → analyze with image tool
5. Repeat
```

### Example: Open a URL in Firefox

```bash
# Focus address bar, type URL, navigate
python3 -m linuse combo ctrl l , sleep 0.5 , type "example.com" , sleep 0.3 , key Return
sleep 3
python3 -m linuse screenshot -o /tmp/result.png
```

### Example: Close current window

```bash
python3 -m linuse combo alt F4
```

### Example: Type in a text field after clicking it

```bash
python3 -m linuse tap 400 300 , sleep 0.3 , type "Hello from Clawdia"
```

## Coordinate Tips

- Screen resolution detected automatically (check with `info`)
- Coordinates are in logical pixels (1280x800 on this machine)
- Insert `sleep 0.3` between tap and type to let focus settle
- Insert `sleep 0.1` between screenshot and the previous action
- For dock icons: x≈27, y varies by icon position (top of dock ≈ 150-170)

## Limitations

- **Touch semantics only** — no hover state, no right-click (it's a touchscreen)
- **No clipboard paste** — type goes character by character (slow for long text)
- **No window enumeration** — can't list windows like winuse does
- **GNOME-specific** — won't work on KDE/Sway without changes
- **Needs uinput access** — requires chmod or udev rule

## Troubleshooting

### Keyboard input not reaching app
- Use **chained commands** (`,` separator) — single-shot keyboard commands may not work
- Make sure the target window has focus (tap it first, add sleep)

### Screenshot returns black
- Screen might be locked: `loginctl unlock-session`
- Display might be off: `gsettings set org.gnome.desktop.session idle-delay 0`

### "Permission denied" on /dev/uinput
```bash
sudo chmod 666 /dev/uinput
# Or persistent: echo 'KERNEL=="uinput", MODE="0666"' | sudo tee /etc/udev/rules.d/99-uinput.rules
```

### Taps land in wrong place
- Verify resolution with `python3 -m linuse info`
- Take a screenshot and estimate coordinates from the image
- Remember: coordinates are absolute screen pixels
