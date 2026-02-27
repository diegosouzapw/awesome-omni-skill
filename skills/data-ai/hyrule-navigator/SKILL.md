---
name: hyrule-navigator
description: Autonomous navigation agent for Oracle of Secrets. Uses static map data and dynamic RAM state to localize Link and route him to destinations.
---

# Hyrule Navigator

## Scope
- Localize Link in the game world (Dungeon Room or Overworld Screen).
- Plan paths between rooms (Macro-Pathing) using `world_graph.json`.
- Steer Link to specific coordinates (Micro-Pathing) within a room.
- Handle transitions (Doors, Stairs, Entrances).

## Core Capabilities

### 1. Localization
Identify where Link is using RAM:
- **Dungeon:** RoomID (`$7E00A0`), X (`$7E0022`), Y (`$7E0020`).
- **Overworld:** ScreenID (Calc from X/Y), Global X/Y.
- **Context:** `Indoors` flag (`$7E001B`).

### 2. Macro-Pathfinding
Route from current location to a target room/screen.
- Uses `world_graph.json` (compiled from `z3ed` data).
- Returns a sequence of actions: `[Go West Door, Take Stairs Up, Enter Cave]`.

### 3. Micro-Steering
Move Link to a specific pixel/tile coordinate.
- **Input:** Target X, Y.
- **Control:** PID-like loop reading RAM and pressing D-Pad.
- **Safety:** Checks for collisions (future) and stuck states.

## Workflow

1.  **Where am I?**
    - `navigator locate` -> "Room 0x1B (Dungeon), Tile (10, 20)".
2.  **Route:**
    - `navigator route --to 0x1A` -> "Path: West Door -> Room 0x1A".
3.  **Execute:**
    - `navigator drive --path ...` -> Autonomously moves Link.

## Dependencies
- **Data:** `~/src/hobby/yaze/world_graph.json` (Must be up-to-date with ROM).
- **Tools:** `z3ed` (for map data), Mesen2 socket CLI (for RAM/Input).
- **Scripts:** `~/src/hobby/yaze/scripts/ai/navigator.py`.

## Commands
- `navigator locate`: Print current localization info.
- `navigator route <target_id>`: Plan path to target.
- `navigator goto <target_id>`: Plan and execute movement.

## Example Prompts
- "Where am I currently located in the game?"
- "Navigate Link to Room 0x1A from the current position."
- "Find a path to the nearest Dungeon Exit."
- "What are the neighbors of the current room?"

## Troubleshooting
- **"Neighbors: []"**: Check `world_graph.json` connectivity. Run `map_compiler.py` again.
- **Localization Failed**: Ensure Mesen2 socket control is running (`/tmp/mesen2-*.sock`). Verify RAM addresses in `navigator.py` match the active ROM version.
- **Stuck Walking**: Micro-pathing collision avoidance is primitive. Manually guide Link if stuck on complex geometry.
