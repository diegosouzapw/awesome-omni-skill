---
name: aidb-architecture
description: Comprehensive architectural reference for AIDB core and MCP integration.
  Covers 6-layer architecture (MCP, Service, Session, Adapter, DAP Client, Protocol),
  component organization, data flow patterns, and design decisions. Use when explaining
  overall system design or understanding how layers interact.
version: 1.0.0
tags:
  - architecture
  - session
  - service
  - dap
  - mcp
  - design-patterns
  - components
  - data-flow
---

# AIDB Architecture Skill

## Overview

This skill provides a comprehensive architectural reference for understanding AIDB's multi-layered architecture, focusing on `aidb` core library and `aidb_mcp` MCP integration.

**Purpose:** Enable developers to navigate the codebase confidently, understand component responsibilities, trace data flows, and make correct architectural decisions.

**Scope:**

- **PRIMARY:** `aidb/` (core debugging library), `aidb_mcp/` (MCP server integration)
- **SECONDARY:** `aidb_common/`, `aidb_logging/` (supporting utilities)
- **EXCLUDED:** `aidb_cli/` (covered by `dev-cli-development` skill)

______________________________________________________________________

## When to Use This Skill

**Use this skill when:**

- Understanding overall system architecture
- Tracing data flow across layers (e.g., MCP tool call → DAP adapter)
- Identifying component responsibilities
- Making architectural decisions (which layer to modify)
- Understanding design patterns and rationale
- Debugging cross-layer issues

**Do NOT use this skill for:**

- Deep adapter implementation patterns → Use `adapter-development` skill
- DAP protocol details → Use `dap-protocol-guide` skill
- MCP tool development → Use `mcp-tools-development` skill

______________________________________________________________________

## 6-Layer Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 1: MCP Layer (aidb_mcp/)                             │
│  ├── 12 debugging tools for AI agents                       │
│  ├── Handler dispatch, response optimization                │
│  └── Session management integration                         │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Service Layer (aidb/service/)                     │
│  ├── DebugService - Main entry point                        │
│  ├── SessionManager, SessionBuilder (in aidb/session/)      │
│  └── .execution / .stepping / .breakpoints / .variables     │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: Session Layer (aidb/session/)                     │
│  ├── Session - Infrastructure hub                           │
│  ├── SessionState, SessionConnector                         │
│  ├── SessionRegistry, ResourceManager                       │
│  └── Parent-child session support (JavaScript)              │
├─────────────────────────────────────────────────────────────┤
│  Layer 4: Adapter Layer (aidb/adapters/)                    │
│  ├── DebugAdapter - Component delegation base               │
│  ├── ProcessManager, PortManager, LaunchOrchestrator        │
│  └── Language Adapters - Python, JavaScript, Java           │
├─────────────────────────────────────────────────────────────┤
│  Layer 5: DAP Client Layer (aidb/dap/client/)               │
│  ├── DAPClient - Single request path                        │
│  ├── Transport, RequestHandler, EventProcessor              │
│  └── MessageRouter, ConnectionManager                       │
├─────────────────────────────────────────────────────────────┤
│  Layer 6: Protocol Layer (aidb/dap/protocol/)               │
│  └── Fully-typed DAP specification (see dap-protocol-guide) │
└─────────────────────────────────────────────────────────────┘
```

______________________________________________________________________

## Quick Navigation

**"I want to understand..."**

| Topic                    | Resource                                                         | Contents                                                                        |
| ------------------------ | ---------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **MCP & Service Layers** | [api-mcp-layer.md](resources/api-mcp-layer.md)                   | 12 tools, handler pattern, response system, DebugService, execution/stepping    |
| **Session Layer**        | [session-layer.md](resources/session-layer.md)                   | Infrastructure hub, SessionState, SessionConnector, parent-child sessions       |
| **Adapter Layer**        | [adapter-architecture.md](resources/adapter-architecture.md)     | DebugAdapter base, ProcessManager, PortManager, lifecycle hooks, Python/JS/Java |
| **DAP Client**           | [dap-client.md](resources/dap-client.md)                         | Single request path, Future-based async, event handling, design decisions       |
| **Patterns & Resources** | [patterns-and-resources.md](resources/patterns-and-resources.md) | Architectural principles, three-tier cleanup, resource management, data flows   |

______________________________________________________________________

## Key Architectural Principles

1. **Component Delegation** - Focused components vs monolithic classes
1. **Language-Agnostic Design** - Pluggable adapter architecture
1. **Human-Cadence Debugging** - Breakpoints before execution, one step at a time
1. **Resource Lifecycle Management** - Multi-tier cleanup with defense-in-depth
1. **Parent-Child Session Support** - JavaScript subprocess debugging
1. **Single Request Path** - No circular dependencies in DAP client
1. **Three-Tier Cleanup** - DAP disconnect → process termination → port release

For detailed explanations, see [patterns-and-resources.md](resources/patterns-and-resources.md).

______________________________________________________________________

## Resource Management Summary

**Three-Tier Cleanup Strategy:**

1. **Tier 1:** DAP disconnect (graceful adapter shutdown)
1. **Tier 2:** Process termination (SIGTERM → SIGKILL escalation)
1. **Tier 3:** Port release (registry updates)

**Why Order Matters:** Prevents port conflicts and orphaned processes.

**Key Components:**

- Process Registry (`aidb/resources/pids.py`)
- Port Registry (`aidb/resources/ports.py`)
- Orphan Cleanup (`aidb/resources/orphan_cleanup.py`)
- ResourceManager (`aidb/session/resource.py`)

______________________________________________________________________

## Related Skills

| Skill                   | Use For                                           |
| ----------------------- | ------------------------------------------------- |
| `adapter-development`   | Language-specific adapter implementation patterns |
| `dap-protocol-guide`    | DAP protocol specification and usage              |
| `mcp-tools-development` | MCP tool creation and agent optimization          |

______________________________________________________________________

## Resources

| Resource                                                         | Content                                                                     |
| ---------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [api-mcp-layer.md](resources/api-mcp-layer.md)                   | MCP server, 12 tools, handler pattern, Service layer, execution/stepping    |
| [session-layer.md](resources/session-layer.md)                   | Session architecture, infrastructure hub, state management, parent-child    |
| [adapter-architecture.md](resources/adapter-architecture.md)     | Adapter base class, components, lifecycle hooks, language-specific patterns |
| [dap-client.md](resources/dap-client.md)                         | DAP client design, single request path, Future-based async, events          |
| [patterns-and-resources.md](resources/patterns-and-resources.md) | Architectural principles, resource management, cleanup, data flows          |

**Documentation:**

- Architecture overview → `docs/developer-guide/overview.md`
- Component source → `src/aidb/`, `src/aidb_mcp/`, `src/aidb_cli/`, `src/aidb_common/`, `src/aidb_logging/`

______________________________________________________________________

## Quick Reference

**6 Layers:** MCP → Service → Session → Adapter → DAP Client → Protocol

**Key Patterns:** Component delegation, language-agnostic, human-cadence debugging, resource lifecycle, parent-child sessions, single request path

**5 Resource Files:** api-mcp-layer, session-layer, adapter-architecture, dap-client, patterns-and-resources
