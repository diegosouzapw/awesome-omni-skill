---
name: adapter-development
description: Comprehensive guide for AIDB debug adapter development. Covers component-based
  architecture, language-specific patterns (Python/debugpy, JavaScript/vscode-js-debug,
  Java/java-debug), lifecycle hooks, process management, port management, launch orchestration,
  resource cleanup, child sessions, and common pitfalls. Essential for developing
  or maintaining AIDB debug adapters.
---

# AIDB Adapter Development Skill

## Executive Summary

AIDB debug adapters provide language-agnostic debugging capabilities through the Debug Adapter Protocol (DAP). The architecture is **component-based** - adapters delegate to specialized components rather than implementing everything in monolithic classes.

**For comprehensive architecture details**, see `docs/developer-guide/overview.md` for the complete system architecture and data flow diagrams.

### Core Architecture

```
DebugAdapter (base class)
├── ProcessManager       - Process lifecycle (launch, monitor, stop, cleanup)
├── PortManager          - Port acquisition, verification, release
├── LaunchOrchestrator   - Launch sequence coordination
├── TargetResolver       - Target type detection and normalization
├── SourcePathResolver   - Source path resolution for remote debugging
└── AdapterHooksMixin    - Lifecycle hooks for extension points
```

### Key Principles

1. **Component Delegation**: Adapters delegate to focused components (ProcessManager, PortManager, LaunchOrchestrator)
1. **Lifecycle Hooks**: Customize behavior via hooks rather than overriding entire methods
1. **Resource Management**: Centralized cleanup via ResourceManager
1. **Human-Cadence Debugging**: Operations happen at human speed, not API speed
1. **Language-Agnostic API**: Same Python API works across Python, JavaScript, Java

## Resource Files

This skill is organized into focused resource files for language-specific patterns:

- **[python-adapter-patterns.md](resources/python-adapter-patterns.md)** - debugpy configuration, module vs script patterns
- **[javascript-adapter-patterns.md](resources/javascript-adapter-patterns.md)** - vscode-js-debug, child sessions, Node.js patterns
- **[java-adapter-patterns.md](resources/java-adapter-patterns.md)** - java-debug + JDT LS integration, compilation

## Related Skills

When working on adapter development, you may also need:

- **dap-protocol-guide** - Adapters heavily rely on DAP protocol types and request/response patterns
- **testing-strategy** - Adapters must be tested using E2E patterns with DebugInterface abstraction
- **code-reuse-enforcement** - Always check for existing utilities before implementing adapter components

## Quick Start: Creating a New Adapter

```python
from aidb.adapters.base import DebugAdapter
from aidb.adapters.base.config import AdapterConfig

class MyLanguageAdapter(DebugAdapter):
    """My language debug adapter using component architecture."""

    def __init__(self, session, ctx=None, config=None, **kwargs):
        if config is None:
            config = MyLanguageConfig()

        super().__init__(
            session=session,
            ctx=ctx,
            config=config,
            **kwargs
        )

        # Register language-specific hooks
        self._register_my_language_hooks()

    def _register_my_language_hooks(self):
        """Register language-specific lifecycle hooks."""
        self.register_hook(
            LifecycleHook.PRE_LAUNCH,
            self._validate_environment,
            priority=90  # High priority = runs first
        )

    async def _validate_environment(self, context: HookContext):
        """Pre-launch hook to validate environment."""
        # Validation logic here
        pass

    async def _build_launch_command(self, target, adapter_host,
                                   adapter_port, args=None):
        """Build the command to launch the debug adapter."""
        return [
            "/path/to/debug/adapter",
            "--host", adapter_host,
            "--port", str(adapter_port),
            target
        ]

    def _add_adapter_specific_vars(self, env: dict) -> dict:
        """Add language-specific environment variables."""
        env["MY_LANG_DEBUG"] = "1"
        return env

    def _create_target_resolver(self) -> "TargetResolver":
        """Create language-specific target resolver."""
        return MyLanguageTargetResolver(adapter=self, ctx=self.ctx)

    def _create_source_path_resolver(self) -> "SourcePathResolver":
        """Create language-specific source path resolver for remote debugging."""
        return MyLanguageSourcePathResolver(adapter=self, ctx=self.ctx)

    def _get_process_name_pattern(self) -> str:
        """Get process name pattern for cleanup."""
        return "my-language-debug"
```

## Component Access Patterns

### ProcessManager Usage

See `ProcessManager` in `src/aidb/adapters/base/components/process_manager.py` for full implementation.

```python
from aidb.resources.process_tags import ProcessType

# Launch subprocess with session tagging
proc = await self._process_manager.launch_subprocess(
    cmd=cmd,
    env=env,
    session_id=self.session.id,
    language=self.config.language,
    process_type=ProcessType.ADAPTER,  # String constant: "adapter", "debuggee", or "lsp_server"
    kwargs={}
)

# Wait for adapter readiness
await self._process_manager.wait_for_adapter_ready(port, start_time)

# Get process status
if self._process_manager.is_alive:
    pid = self._process_manager.pid
```

### PortManager Usage

See `PortManager` in `src/aidb/adapters/base/components/port_manager.py` for full implementation. Key methods:

- `acquire(requested_port=None)` - Acquire and verify port availability
- `release()` - Release the port
- `port` property - Get current port

### LaunchOrchestrator Usage

See `LaunchOrchestrator` in `src/aidb/adapters/base/components/launch_orchestrator.py` for full implementation. Key methods:

- `launch(target, port, args)` - Simple launch with target file
- `launch_with_config(launch_config, port, workspace_root)` - Launch with VS Code launch.json configuration

### SourcePathResolver Usage

See `SourcePathResolver` in `src/aidb/adapters/base/source_path_resolver.py` for base implementation. Each language adapter implements its own resolver:

- **Python**: `src/aidb/adapters/lang/python/source_path_resolver.py` - Handles site-packages, venv, egg paths
- **JavaScript**: `src/aidb/adapters/lang/javascript/source_path_resolver.py` - Handles node_modules, webpack paths
- **Java**: `src/aidb/adapters/lang/java/source_path_resolver.py` - Handles JAR notation, Maven layouts

Key methods:

- `extract_relative_path(file_path)` - Extract language-specific relative path from adapter-returned path
- `resolve(file_path, source_paths)` - Resolve remote path to local source file

## Lifecycle Hooks Reference

Hooks execute in priority order (lower number = runs first):

### Hook Priorities

- **90-100**: Critical validation (environment, target file)
- **70-80**: Setup/preparation (workspaces, configurations)
- **50**: Default priority
- **20-30**: Post-operation delays/waits
- **10**: Cleanup operations

### Common Hooks

```python
# Pre-launch validation
self.register_hook(
    LifecycleHook.PRE_LAUNCH,
    self._validate_environment,
    priority=90
)

# Post-launch setup
self.register_hook(
    LifecycleHook.POST_LAUNCH,
    self._wait_for_ready,
    priority=20
)

# Post-stop cleanup
self.register_hook(
    LifecycleHook.POST_STOP,
    self._cleanup_resources,
    priority=10
)
```

## Code Reuse: Existing Utilities

Before implementing new functionality, check these shared resources:

### Base Classes

- **DebugAdapter** (`aidb/adapters/base/adapter.py`) - Base adapter with component architecture
- **AdapterConfig** (`aidb/adapters/base/config.py`) - Configuration base class
- **BaseLaunchConfig** (`aidb/adapters/base/launch.py`) - VS Code launch.json support

### Components

- **ProcessManager** (`aidb/adapters/base/components/process_manager.py`)
- **PortManager** (`aidb/adapters/base/components/port_manager.py`)
- **LaunchOrchestrator** (`aidb/adapters/base/components/launch_orchestrator.py`)

### Utilities

- **AdapterBinaryLocator** (`aidb/adapters/utils/binary_locator.py`) - Find adapter binaries
- **AdapterOutputCapture** (`aidb/adapters/utils/output_capture.py`) - Capture stdout/stderr
- **AdapterTraceLogManager** (`aidb/adapters/utils/trace_log.py`) - Trace log management

### Common Patterns (`aidb_common/`)

See source code docstrings in `src/aidb_common/` for detailed API documentation.

- **Obj** (`aidb_common/patterns/`) - Base class with context support
- **normalize_path()** (`aidb_common/path.py`) - Path normalization
- **config** (`aidb_common/config/`) - Environment variable reading
- **Language** enum (`aidb_common/constants.py`) - Supported language constants

## Launch Configuration Patterns

Adapters can support VS Code launch.json configurations:

```python
def get_launch_configuration(self) -> dict[str, Any]:
    """Get launch configuration for DAP Launch request."""
    config = {
        "type": "mylang",
        "request": "launch",
        "name": "Debug MyLang",
        "program": self._target_file,
        "args": self._target_args,
        "cwd": self._target_cwd,
        "env": self._target_env,
    }
    return config
```

## Environment Variable Patterns

See `src/aidb_common/env/` for environment handling utilities (reader.py, resolver.py).

Use the template method pattern for environment preparation:

```python
def _prepare_environment(self) -> dict[str, str]:
    """Prepare environment (base implementation)."""
    env = self._load_base_environment()
    env = self._add_trace_configuration(env)
    return self._add_adapter_specific_vars(env)

def _add_adapter_specific_vars(self, env: dict) -> dict:
    """Add language-specific variables (override this)."""
    env["MY_LANG_HOME"] = "/path/to/lang"
    return env
```

## Resource Cleanup Patterns

Always implement proper cleanup in hooks:

```python
async def stop(self) -> None:
    """Stop the debug adapter and clean up."""
    context = await self.execute_hook(LifecycleHook.PRE_STOP)

    if not context.cancelled:
        # Stop process manager
        await self._process_manager.stop()

        # Release port
        if self.port:
            self._port_manager.release()

        # Cleanup auxiliary components
        if self._trace_manager:
            self._trace_manager.cleanup()

    await self.execute_hook(LifecycleHook.POST_STOP)
```

## Process Tagging for Orphan Detection

All AIDB-spawned processes are tagged with environment variables for safe cleanup:

```python
from aidb.resources.process_tags import ProcessType

# Automatically handled by ProcessManager
proc = await self.process_manager.launch_subprocess(
    cmd=cmd,
    env=env,
    session_id=self.session.id,        # Tags process with session ID
    language=self.config.language,      # Tags with language
    process_type=ProcessType.ADAPTER,   # String constant: "adapter", "debuggee", or "lsp_server"
    kwargs={}
)
```

Tags enable:

- Safe orphan detection across sessions
- Cleanup of only AIDB-owned processes
- Session-to-process mapping

## DAP Protocol Reference

The authoritative DAP protocol reference is in `src/aidb/dap/protocol/` - fully typed and always up-to-date.

```python
from aidb.dap.protocol.types import (
    Capabilities,
    InitializeRequest,
    LaunchRequest,
    SetBreakpointsRequest,
)
```

## Testing Your Adapter

Use the shared test infrastructure:

```python
# Test with API directly
session = await client.start_session(
    target="/path/to/file",
    language="mylang",
    breakpoints=[{"line": 10}]
)

# Test with launch.json
session = await client.start_session(
    target="/path/to/file",
    language="mylang",
    launch_config_name="Debug MyLang",
    workspace_root="/path/to/project"
)
```

## Navigation to Resource Files

For detailed language-specific patterns and examples:

1. **Python Adapter Patterns** → `resources/python-adapter-patterns.md`
   - debugpy configuration, module vs script patterns, trace logging
1. **JavaScript Adapter Patterns** → `resources/javascript-adapter-patterns.md`
   - vscode-js-debug, child sessions, Node.js debugging, breakpoint transfer
1. **Java Adapter Patterns** → `resources/java-adapter-patterns.md`
   - java-debug + JDT LS integration, compilation, pooling patterns

## Important Reminders

1. **Don't override entire methods** - Use lifecycle hooks for customization
1. **Don't manage processes directly** - Use ProcessManager component
1. **Don't handle ports manually** - Use PortManager component
1. **Don't forget cleanup** - Register POST_STOP hooks
1. **Don't block async operations** - Use await for I/O operations
1. **Check existing code first** - Look for reusable utilities before implementing

## File Paths Reference

All file paths mentioned in this skill are relative to the repo root:

- Base adapter: `src/aidb/adapters/base/adapter.py`
- Components: `src/aidb/adapters/base/components/`
- SourcePathResolver base: `src/aidb/adapters/base/source_path_resolver.py`
- Python adapter: `src/aidb/adapters/lang/python/python.py`
- Python source resolver: `src/aidb/adapters/lang/python/source_path_resolver.py`
- JavaScript adapter: `src/aidb/adapters/lang/javascript/javascript.py`
- JavaScript source resolver: `src/aidb/adapters/lang/javascript/source_path_resolver.py`
- Java adapter: `src/aidb/adapters/lang/java/java.py`
- Java source resolver: `src/aidb/adapters/lang/java/source_path_resolver.py`
- DAP protocol: `src/aidb/dap/protocol/` (types.py, requests.py, responses.py, events.py, bodies.py, base.py)
