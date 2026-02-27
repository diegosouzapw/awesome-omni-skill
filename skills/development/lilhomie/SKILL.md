---
name: lilhomie
description: Control HomeKit devices via REST API. Use when controlling lights, switches, scenes, or checking device status in the user's home.
homepage: https://github.com/ghostmfr/lilhomie
metadata: {"clawdbot":{"emoji":"🏠","os":["darwin"],"requires":{"bins":["curl"]}}}
---

# lilhomie - HomeKit REST API

Control HomeKit devices from your AI assistant. lilhomie is a macOS app that exposes HomeKit via a local REST API.

## When to use (trigger phrases)

- "turn on/off the lights"
- "toggle the [device name]"
- "control my home"
- "what devices are on?"
- "trigger [scene name] scene"
- "turn on/off the [room]"

## Prerequisites

- macOS 13.0+ with lilhomie.app installed and running
- HomeKit devices configured in Apple Home
- lilhomie server running on `localhost:8420`

## API Basics

Base URL: `http://localhost:8420`

**Important:** Use underscores for spaces in device/room names: `Desk_Lamp`, `Guest_Bedroom`

## Common Commands

### List all devices
```bash
curl localhost:8420/devices
```

### Toggle a device
```bash
curl -X POST localhost:8420/device/Desk_Lamp/toggle
```

### Get device status
```bash
curl localhost:8420/device/Desk_Lamp
```

### Room-scoped toggle (for duplicate names)
```bash
curl -X POST localhost:8420/room/Office/device/Lamp/toggle
```

### Turn room on/off
```bash
curl -X POST localhost:8420/room/Office/on
curl -X POST localhost:8420/room/Office/off
```

### List scenes
```bash
curl localhost:8420/scenes
```

### Trigger a scene
```bash
curl -X POST localhost:8420/scene/Good_Night/trigger
```

## Endpoints Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/devices` | List all devices |
| GET | `/device/{name}` | Get device details |
| POST | `/device/{name}/toggle` | Toggle device |
| POST | `/device/{name}/set` | Set brightness/state |
| GET | `/rooms` | List all rooms |
| GET | `/room/{name}` | Get devices in room |
| POST | `/room/{name}/on` | Turn all room devices on |
| POST | `/room/{name}/off` | Turn all room devices off |
| POST | `/room/{r}/device/{d}/toggle` | Room-scoped toggle |
| GET | `/scenes` | List all scenes |
| POST | `/scene/{name}/trigger` | Trigger a scene |

## Known Issues

- Toggle response `isOn` state may be inverted (#1)
- Use room-scoped endpoints when device names are duplicated across rooms

## Tips

1. Always use room-scoped endpoints for devices with common names like "Light" or "Lamp"
2. Don't trust the `isOn` field in responses until issue #1 is fixed
3. If unsure of device names, query `/devices` first
