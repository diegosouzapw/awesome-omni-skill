---
name: claude-player
description: An AI-powered Game Boy emulator agent that uses Claude's vision and reasoning to autonomously play Game Boy games.
---

# ClaudePlayer

An AI agent that plays Game Boy games autonomously using Claude's vision capabilities and the PyBoy emulator.

## Overview

ClaudePlayer connects Claude to a Game Boy emulator. Each turn, Claude receives a screenshot of the current game frame, reasons about what to do, and sends button inputs back to the emulator. It maintains a structured memory system to track game progress (items, NPCs, locations, quests, stats).

## Prerequisites

- Python 3.10+
- Pipenv
- An Anthropic API key
- A Game Boy ROM file (`.gb`)

## Setup

```bash
# Install dependencies
pipenv install

# Create .env with your API key
echo "ANTHROPIC_API_KEY=your_key_here" > .env

# Place a Game Boy ROM in the project directory
```

## Configuration

Edit `config.json`:

| Key | Description | Default |
|-----|-------------|---------|
| `ROM_PATH` | Path to Game Boy ROM | Required |
| `STATE_PATH` | Saved emulator state file | `null` |
| `EMULATION_MODE` | `"turn_based"` or `"continuous"` | `"turn_based"` |
| `MODEL_DEFAULTS.MODEL` | Claude model to use | `"claude-sonnet-4-5-20250929"` |
| `MODEL_DEFAULTS.THINKING` | Enable extended thinking | `true` |
| `RATE_LIMITS.RPM_THRESHOLD` | Requests per minute limit | Configured in file |
| `RATE_LIMITS.TPM_THRESHOLD` | Tokens per minute limit | Configured in file |
| `REDIS_LOGS` | Optional Redis logging config | `null` |

## Running

```bash
pipenv shell

# Default config
python play.py

# Custom config
python play.py --config my_config.json

# Create a saved emulator state
python emu_setup.py
```

## Project Structure

| Path | Purpose |
|------|---------|
| `play.py` | Launcher script |
| `claude_player/main.py` | CLI entry point with arg parsing |
| `claude_player/agent/game_agent.py` | Main orchestrator: emulator init, game loop, coordination |
| `claude_player/interface/claude_interface.py` | Claude API communication and rate limiting |
| `claude_player/state/game_state.py` | Game state tracking (memory, goals, history) |
| `claude_player/tools/tool_setup.py` | Tool definitions (send_inputs, memory ops, etc.) |
| `claude_player/tools/tool_registry.py` | Tool registry system |
| `claude_player/config/config_loader.py` | Config file parsing |
| `claude_player/config/config_class.py` | Configuration data class |
| `claude_player/utils/game_utils.py` | Button input parsing, screenshot capture |
| `claude_player/utils/memory_reader.py` | Game memory/stats reader |
| `claude_player/agent/summary_generator.py` | Periodic game progress summarization |
| `config.json` | Runtime configuration |

## Available Tools (for the agent during gameplay)

| Tool | Description |
|------|-------------|
| `send_inputs` | Send button sequences to the emulator |
| `set_game` | Identify the current game |
| `set_current_goal` | Update the gameplay objective |
| `add_to_memory` | Store items, NPCs, locations, quests, mechanics, stats |
| `remove_from_memory` | Remove a memory entry |
| `update_memory_item` | Update an existing memory entry |
| `toggle_thinking` | Toggle extended thinking mode on/off |

## How It Works

1. PyBoy emulator loads the ROM and renders a frame
2. The frame is captured as a screenshot and sent to Claude
3. Claude analyzes the screen, reasons about the game state, and calls tools
4. `send_inputs` presses buttons on the emulator (A, B, Up, Down, Left, Right, Start, Select)
5. In turn-based mode, the emulator only advances when the agent acts
6. The agent maintains structured memory to track long-term game progress
7. Periodic summaries compress the conversation history to stay within context limits
