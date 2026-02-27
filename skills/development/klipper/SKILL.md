---
name: klipper
description: Monitor and control Klipper/Moonraker 3D printers with safety confirmations.
---

# Klipper

Monitor and control Klipper-based 3D printers via the Moonraker API. This skill includes safety protocols to prevent accidental printer control.

## ⚠️ Safety Protocols

All control commands (Pause, Resume, Cancel) require a `--confirm` flag. The AI agent MUST request user confirmation before executing these actions.

## Configuration

Scripts default to `localhost:7125`. Pass `[host]` and `[port]` as arguments to target a specific printer.

## Commands

### Check Status
Get high-level printer state, temperatures, and progress.
```bash
./scripts/status.sh [host] [port]
```

### Webcam Snapshot
Capture a timestamped image from the configured webcam.
```bash
./scripts/snapshot.sh [host] [port]
```

### Pause Print
**Requires Confirmation.**
```bash
./scripts/pause.sh [host] [port] --confirm
```

### Resume Print
**Requires Confirmation.**
```bash
./scripts/resume.sh [host] [port] --confirm
```

### Cancel Print
**Requires Confirmation.**
```bash
./scripts/cancel.sh [host] [port] --confirm
```

### Emergency Stop
**IMMEDIATE ACTION.** No confirmation required for emergency shutdowns.
```bash
./scripts/emergency_stop.sh [host] [port]
```
