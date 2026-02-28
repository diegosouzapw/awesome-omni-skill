---
name: sumo-core
description: "Core SUMO simulation workflows and CLI usage: build/import networks (netgenerate/netconvert/OSM/netedit), generate demand and routes (randomTrips/od2trips/duarouter), run sumo/sumo-gui with .sumocfg and additional files. Use for standard SUMO operations when MCP automation, RL tooling, or output analysis are not the primary focus."
---

# Sumo Core

## Overview
Use this skill to plan and specify standard SUMO workflows and command lines for network creation, demand generation, routing, and running simulations.

## Skill Routing
- Use sumo-env for installation, SUMO_HOME/PATH, or tool availability issues.
- Use sumo-output for output configuration, parsing, and post-processing.
- Use sumo-rl for reinforcement learning traffic signal control tasks.
- Use sumo-mcp when you must run multi-step workflows, live TraCI control, or automated tool execution.

## Core Workflow (CLI)
1. Build or import a network (netgenerate, netconvert, netedit, OSM import).
2. Generate trips or flows (randomTrips.py, od2trips).
3. Compute routes (duarouter) if trips are used.
4. Run the simulation (sumo or sumo-gui) with a .sumocfg file.
5. Add extra definitions via additional files (traffic lights, detectors, outputs).

## MCP Decision
- Stay in this skill for single-step commands or config edits that do not require executing tools.
- Switch to sumo-mcp when the request needs automated execution, multi-step pipelines, or TraCI control.

## References
- Command line and config structure: references/command-line-basics.md
- Network creation/import: references/network-build.md
- Demand and routing: references/demand-routing.md
- Running simulations: references/simulation-run.md
- TraCI basics (handoff to MCP for control): references/traci-basics.md
