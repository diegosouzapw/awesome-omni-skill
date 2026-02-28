---
name: amplifier-cli-skill
description: Skill for building CLI applications on the Amplifier platform. Teaches amplifier-foundation patterns as the source of truth for composable AI application development. Use when building CLI tools, understanding bundle composition, implementing sessions, or extending the Amplifier ecosystem.
compatibility: Requires Python 3.10+, uv package manager. Works with amplifier-foundation, amplifier-core, and related ecosystem packages.
license: MIT
metadata:
  author: Michael Jabbour
  version: "2.0"
  repository: https://github.com/michaeljabbour/amplifier-cli-skill
---

# Building CLI Applications on Amplifier

[Amplifier](https://github.com/microsoft/amplifier) is Microsoft's composable AI agent framework that enables:
- **Bundle composition** - Mix session orchestrators, hooks, tools, and providers
- **Module ecosystem** - 36+ modules available ([Module Catalog](https://github.com/microsoft/amplifier/blob/main/docs/MODULES.md))
- **Provider flexibility** - Claude, GPT, Gemini, local models, and more
- **Production-ready** - Logging, observability, and hooks built-in

This skill teaches how to build CLI applications using **amplifier-foundation** patterns - the source of truth for Amplifier application development.

## Amplifier Ecosystem

This skill is part of the broader Amplifier ecosystem:

### Core Repositories

- **[amplifier](https://github.com/microsoft/amplifier)** - Main framework repository
- **[amplifier-core](https://github.com/microsoft/amplifier-core)** - Core library (execution engine)
- **[amplifier-foundation](https://github.com/microsoft/amplifier-foundation)** - Foundation library (configuration/composition layer)
- **[Amplifier Modules](https://github.com/microsoft/amplifier/blob/main/docs/MODULES.md)** - Module catalog (36+ available modules)

### Reference Implementations

- **[amplifier-simplecli](https://github.com/michaeljabbour/amplifier-simplecli)** - Production terminal UI implementation

## Reference Implementation

**See [amplifier-simplecli](https://github.com/michaeljabbour/amplifier-simplecli)** for a complete working example demonstrating these patterns in production. This terminal UI implementation showcases:
- Full bundle composition workflow
- Memory system integration (tool-memory, hooks-memory-capture, context-memory)
- 14 pre-configured modules
- Streaming UI with hooks-event-broadcast
- Complete documentation structure

**Key Resources:**
- 📊 [ARCHITECTURE_FLOWS.md](https://github.com/michaeljabbour/amplifier-simplecli/blob/main/docs/ARCHITECTURE_FLOWS.md) - Visual diagrams showing Foundation (setup) → Core (runtime) separation
- 🗺️ [ROADMAP.md](https://github.com/michaeljabbour/amplifier-simplecli/blob/main/docs/ROADMAP.md) - Feature roadmap with implementation details
- 🔧 [MODULE_GAP_ANALYSIS.md](https://github.com/michaeljabbour/amplifier-simplecli/blob/main/docs/MODULE_GAP_ANALYSIS.md) - Module coverage and requirements

Use it as a reference when building your own CLI applications.

## Core Workflow

Every Amplifier application follows this fundamental pattern:

```python
from amplifier_foundation import load_bundle

async def main():
    # 1. Load bundles
    foundation = await load_bundle("git+https://github.com/microsoft/amplifier-foundation@main")
    provider = await load_bundle("./providers/anthropic.yaml")  # Your provider config

    # 2. Compose bundles (later overrides earlier)
    composed = foundation.compose(provider)

    # 3. Prepare (resolves and downloads modules)
    prepared = await composed.prepare()

    # 4. Create session and execute
    async with await prepared.create_session() as session:
        response = await session.execute("Hello!")
        print(response)
```

**This is the pattern.** Everything else builds on this foundation.

## Quick Start

```bash
# Install amplifier-foundation
pip install amplifier-foundation

# Or with uv
uv add amplifier-foundation
```

## Example Bundles

Complete, production-ready bundle examples are available in [references/EXAMPLES.md](references/EXAMPLES.md):

- **Base Bundle** - Session orchestrator, memory system, hooks, and tools
- **Provider Configurations** - Claude Opus 4.5, Sonnet 4.5, Haiku 4
- **Agent Bundle** - Specialized bug-hunter agent example

These examples show real bundle structures you can copy and adapt. In your CLI app, organize them as:
```
your-cli/
├── bundles/base.md
├── providers/opus.yaml
└── agents/bug-hunter.md
```

## What is a Bundle?

A **Bundle** is a composable configuration unit that produces a **mount plan** for sessions.

```
Bundle → compose() → prepare() → create_session() → execute()
```

### Bundle Sections

| Section | Purpose |
|---------|---------|
| `bundle` | Metadata (name, version) |
| `session` | Orchestrator and context manager |
| `providers` | LLM backends |
| `tools` | Agent capabilities |
| `hooks` | Observability and control (see [references/HOOKS.md](references/HOOKS.md)) |
| `agents` | Named agent configurations |
| `context` | Context files to include |
| `instruction` | System instruction (markdown body) |

### Bundle Format

Bundles are markdown files with YAML frontmatter:

```markdown
---
bundle:
  name: my-app
  version: 1.0.0

session:
  orchestrator: {module: loop-streaming}
  context: {module: context-simple}

providers:
  - module: provider-anthropic
    source: git+https://github.com/microsoft/amplifier-module-provider-anthropic@main
    config:
      default_model: claude-sonnet-4-5

tools:
  - module: tool-filesystem
    source: git+https://github.com/microsoft/amplifier-module-tool-filesystem@main
---

You are a helpful assistant for software development.
```

## Composition Pattern

Bundles compose with later overriding earlier:

```python
result = base.compose(overlay)  # overlay wins on conflicts
```

### Merge Rules

| Section | Rule |
|---------|------|
| `session` | Deep merge (nested dicts merged) |
| `providers` | Merge by module ID |
| `tools` | Merge by module ID |
| `hooks` | Merge by module ID |
| `instruction` | Replace (later wins) |

### Common Composition Patterns

**Base + Environment:**
```python
import os

base = await load_bundle("./bundles/base.md")
dev_overlay = await load_bundle("./bundles/dev.md")
prod_overlay = await load_bundle("./bundles/prod.md")

env = os.getenv("ENV", "dev")
overlay = dev_overlay if env == "dev" else prod_overlay
bundle = base.compose(overlay)
```

**Feature Bundles:**
```python
filesystem = Bundle(name="fs", tools=[{"module": "tool-filesystem", "source": "..."}])
web = Bundle(name="web", tools=[{"module": "tool-web", "source": "..."}])

# Compose what you need
full = base.compose(filesystem).compose(web)
```

**Includes Chain (Declarative):**
```yaml
# dev.md
bundle:
  name: dev
includes:
  - ./base.md
providers:
  - module: provider-anthropic
    config: {debug: true}
```

## CLI Application Architecture

Use this blueprint for building CLI applications:

```python
import asyncio
import logging
from dataclasses import dataclass
from pathlib import Path

from amplifier_foundation import Bundle, load_bundle


@dataclass
class AppConfig:
    """Application configuration."""
    provider_bundle: str = "anthropic-sonnet.yaml"
    api_key: str | None = None
    log_level: str = "INFO"

    @classmethod
    def from_env(cls) -> "AppConfig":
        return cls(
            provider_bundle=os.getenv("PROVIDER", "anthropic-sonnet.yaml"),
            api_key=os.getenv("ANTHROPIC_API_KEY"),
            log_level=os.getenv("LOG_LEVEL", "INFO"),
        )

    def validate(self) -> None:
        if not self.api_key:
            raise ValueError("ANTHROPIC_API_KEY not set")


class AmplifierApp:
    """Amplifier CLI application pattern."""

    def __init__(self, config: AppConfig):
        self.config = config
        self.session = None
        self.logger = logging.getLogger("amplifier_app")

    async def initialize(self) -> None:
        """Initialize: load bundles, compose, prepare, create session."""
        # Load foundation
        foundation = await load_bundle("git+https://github.com/microsoft/amplifier-foundation@main")

        # Load provider
        provider = await load_bundle(f"./providers/{self.config.provider_bundle}")

        # Add tools
        tools = Bundle(
            name="app-tools",
            tools=[
                {"module": "tool-filesystem", "source": "git+..."},
                {"module": "tool-bash", "source": "git+..."},
            ],
        )

        # Compose all bundles
        composed = foundation.compose(provider).compose(tools)

        # Prepare (downloads modules)
        prepared = await composed.prepare()

        # Create session
        self.session = await prepared.create_session()

    async def execute(self, prompt: str) -> str:
        """Execute a prompt."""
        if not self.session:
            raise RuntimeError("Session not initialized")
        return await self.session.execute(prompt)

    async def shutdown(self) -> None:
        """Graceful shutdown."""
        if self.session:
            await self.session.cleanup()

    async def __aenter__(self):
        await self.initialize()
        return self

    async def __aexit__(self, *args):
        await self.shutdown()


async def main():
    config = AppConfig.from_env()
    config.validate()

    async with AmplifierApp(config) as app:
        response = await app.execute("Hello!")
        print(response)


if __name__ == "__main__":
    asyncio.run(main())
```

### Key Architectural Principles

1. **Configuration as Dataclass**: Use `@dataclass` for config, load from env/files
2. **Initialize/Execute/Shutdown Lifecycle**: Clean lifecycle management
3. **Context Manager Pattern**: Use `async with` for automatic cleanup
4. **Composition Over Inheritance**: Compose bundles, don't subclass

## Session Patterns

### Basic Session

```python
bundle = await load_bundle("/path/to/bundle.md")
prepared = await bundle.prepare()

async with await prepared.create_session() as session:
    response = await session.execute("Hello!")
```

### Multi-Turn Conversation

Session maintains context automatically:

```python
async with await prepared.create_session() as session:
    await session.execute("My name is Alice")
    response = await session.execute("What's my name?")
    # Response knows about Alice
```

### Resuming Sessions

```python
# First session
async with await prepared.create_session() as session:
    await session.execute("Remember: X=42")
    session_id = session.session_id

# Later: resume
async with await prepared.create_session(session_id=session_id) as session:
    response = await session.execute("What is X?")
    # Knows X=42
```

## Agent Delegation

### Defining Agents

Example agent bundle structure (see [references/EXAMPLES.md](references/EXAMPLES.md) for complete example):

```markdown
---
bundle:
  name: bug-hunter
  version: 1.0.0
  description: Finds and fixes bugs

providers:
  - module: provider-anthropic
    config:
      default_model: claude-sonnet-4-5
      temperature: 0.3
---

You are an expert bug hunter. Find bugs systematically.
```

### Spawning Agents

```python
# Load agent as bundle
agent_bundle = await load_bundle("./agents/bug-hunter.md")

# Spawn sub-session
result = await prepared.spawn(
    child_bundle=agent_bundle,
    instruction="Find the bug in auth.py",
    compose=True,            # Compose with parent bundle
    parent_session=session,  # Inherit UX from parent
)

print(result["output"])
```

## @Mention System

Reference context files using `@namespace:path` syntax:

```markdown
See @foundation:context/guidelines.md for guidelines.
```

How it works:
1. During composition, each bundle's `base_path` is tracked by namespace
2. PreparedBundle resolves `@namespace:path` references
3. Content is loaded and included inline

## Memory System (Custom Extension)

**⚠️ Note:** The memory system is a custom community extension, NOT part of official amplifier-foundation.

Amplifier can be extended with persistent memory capabilities using three custom modules:

| Module | Purpose |
|--------|---------|
| **tool-memory** | SQLite storage with full-text search (FTS5) |
| **hooks-memory-capture** | Automatic observation capture from tool outputs |
| **context-memory** | Progressive disclosure context injection at session start |

### What Memory Enables

- **Persistent observations** across sessions (bugfix, feature, discovery, decision)
- **Session tracking** with structured summaries and next_steps
- **Automatic learning capture** from tool outputs (no manual add_memory calls)
- **Full-text search** across all memory content
- **Progressive disclosure** for cost-effective context injection

### Quick Setup

Add to your bundle:

```yaml
session:
  memory_context:
    module: context-memory
    source: git+https://github.com/michaeljabbour/amplifier-module-context-memory@main

hooks:
  - module: hooks-memory-capture
    source: git+https://github.com/michaeljabbour/amplifier-module-hooks-memory-capture@main
    config:
      min_content_length: 50
      auto_summarize_interval: 10

tools:
  - module: tool-memory
    source: git+https://github.com/michaeljabbour/amplifier-module-tool-memory@main
    config:
      storage_path: ~/.amplifier/memories.db
      max_memories: 1000
      enable_fts: true
      enable_sessions: true
```

For complete setup, configuration options, and usage patterns, see [references/MEMORY.md](references/MEMORY.md).

## Philosophy

### Mechanism, Not Policy

**Foundation provides mechanism** for bundle composition. It doesn't decide which bundles to use - those are **policy decisions** for your application.

```
Foundation (mechanism):          App (policy):
- load_bundle()                  - Search path order
- compose()                      - Well-known bundles
- prepare()                      - @user:, @project: shortcuts
- create_session()               - Environment-specific config
```

### Text-First

- Bundles are markdown (human-readable)
- Configuration is YAML (diffable)
- No binary formats

### Composable

Small bundles compose into larger configurations. Prefer composition over complexity.

## Deep Dive References

For detailed information on specific topics, see the `references/` directory:

### Architecture & Core Concepts
- **[ARCHITECTURE.md](references/ARCHITECTURE.md)** - Foundation architecture patterns and philosophy
- **[HOOKS.md](references/HOOKS.md)** - Complete hooks system documentation

### Building Applications
- **[BUILD_PATTERNS.md](references/BUILD_PATTERNS.md)** - Comprehensive CLI build patterns and implementation details

### Extensions & Customization
- **[MEMORY.md](references/MEMORY.md)** - Custom memory system extension guide (community contribution)
- **[CUSTOM_EXTENSIONS.md](references/CUSTOM_EXTENSIONS.md)** - Information about community extensions

### Project Resources
- **[CONTRIBUTING.md](references/CONTRIBUTING.md)** - Repository structure and contribution guidelines
- **[TROUBLESHOOTING.md](references/TROUBLESHOOTING.md)** - Production patterns, bug fixes, and troubleshooting guide
- **[REFERENCE_IMPLEMENTATION_HISTORY.md](references/REFERENCE_IMPLEMENTATION_HISTORY.md)** - Evolution of amplifier-simplecli reference implementation

These references provide comprehensive coverage beyond the quick start patterns in this document. Start here for overview, explore references for depth.

## Reference: amplifier-app-cli

The `amplifier-app-cli` repository is a reference implementation. Learn from it, but understand it implements **app-layer policy** on top of foundation mechanisms:

| App Policy (amplifier-app-cli) | Foundation Mechanism |
|--------------------------------|---------------------|
| SessionStore (persistence) | Session lifecycle |
| session_spawner (delegation) | spawn() API |
| paths.py (search paths) | load_bundle() |
| CLI commands | Direct API usage |

When building your own CLI, start from the foundation patterns above, not from copying amplifier-app-cli internals.

## Production Configuration Choices

When building production CLIs, leverage **amplifier-foundation's native capabilities**. The example bundles in [references/EXAMPLES.md](references/EXAMPLES.md) demonstrate production-ready choices.

### Orchestrator: loop-streaming

**Why loop-streaming over loop-events:**

```yaml
session:
  orchestrator:
    module: loop-streaming
    config:
      extended_thinking: true
      max_iterations: 25
```

**Key benefits:**
- **Parallel tool execution** via `asyncio.gather()` prevents race conditions
- Tool results are guaranteed complete before being added to context
- Streaming support for better user experience
- Production-tested in amplifier-app-cli

**Foundation provides:**
- `loop-basic` - Sequential execution (simple, deterministic)
- `loop-events` - Event-driven with hooks (vulnerable to race conditions)
- `loop-streaming` - Parallel execution with determinism (production choice)

### Context Manager: context-simple

**Why context-simple over context-persistent:**

```yaml
session:
  context:
    module: context-simple
    config:
      max_tokens: 200000
      compact_threshold: 0.8
      auto_compact: true
```

**Key benefits:**
- In-memory context management (fast startup)
- Automatic compaction when approaching token limits
- Same compaction logic as context-persistent
- Perfect for stateless CLI sessions

**Foundation provides:**
- `context-simple` - In-memory with auto-compaction (CLI default)
- `context-persistent` - File-based memory loading (for long-term memory)
- `context-memory` - Advanced memory patterns

**When to use persistent:** If your CLI needs to load previous conversation history at startup (rare for CLI tools).

### Safety Hooks from Foundation Ecosystem

The example bundle includes production safety hooks from amplifier-foundation:

```yaml
hooks:
  # Approval for dangerous operations
  - module: hooks-approval
    source: git+https://github.com/microsoft/amplifier-module-hooks-approval@main
    config:
      auto_approve: false
      dangerous_patterns: ["rm -rf", "sudo", "DROP TABLE", ...]

  # Automatic file backups
  - module: hooks-backup
    source: git+https://github.com/microsoft/amplifier-module-hooks-backup@main

  # Git and datetime awareness
  - module: hooks-status-context
    source: git+https://github.com/microsoft/amplifier-module-hooks-status-context@main
    config:
      include_datetime: true
      include_git_status: true

```

**Note:** Cost tracking hooks are not currently available in amplifier-foundation. Monitor token usage through the hooks-streaming-ui module's token display feature.

**Validation:** amplifier-app-cli uses the same foundation modules, proving production viability.

## Common Development Tasks

### Adding a New Provider

1. Create `providers/my-provider.yaml`:
   ```yaml
   bundle:
     name: my-provider
   providers:
     - module: provider-openai
       source: git+https://github.com/microsoft/amplifier-module-provider-openai@main
       config:
         default_model: gpt-4o
   ```

2. Load and compose:
   ```python
   provider = await load_bundle("./providers/my-provider.yaml")
   composed = foundation.compose(provider)
   ```

### Adding Tools

Compose a tools bundle:

```python
tools = Bundle(
    name="my-tools",
    tools=[
        {"module": "tool-filesystem", "source": "git+..."},
        {"module": "tool-bash", "source": "git+...", "config": {"allowed_commands": ["ls", "cat"]}},
    ],
)
composed = base.compose(tools)
```

### Testing with Mock Provider

```python
test_bundle = Bundle(
    name="test",
    providers=[{
        "module": "provider-mock",
        "source": "git+https://github.com/microsoft/amplifier-module-provider-mock@main",
        "config": {"responses": ["Hello!", "How can I help?"]},
    }],
)
```

### Validation

```python
from amplifier_foundation import load_bundle

bundle = await load_bundle(path)
prepared = await bundle.prepare()  # Activates modules (may download/install)
```

## Error Handling

```python
from amplifier_foundation import (
    load_bundle,
    BundleNotFoundError,
    BundleLoadError,
    BundleValidationError,
)

try:
    bundle = await load_bundle(path)
except BundleNotFoundError:
    print(f"Bundle not found: {path}")
except BundleLoadError as e:
    print(f"Failed to load: {e}")
except BundleValidationError as e:
    print(f"Invalid bundle: {e}")
```

## Building a Complete CLI Application

> **Detailed guide available:** For comprehensive build patterns, implementation details, and best practices, see [references/BUILD_PATTERNS.md](references/BUILD_PATTERNS.md).

This section demonstrates how to build a full-featured CLI using foundation patterns. Following these patterns results in significantly less code (~85% reduction) compared to implementing session management, bundle loading, and configuration merging yourself.

### Example Architecture

```
your-cli/
├── your_cli/
│   ├── app.py                  Core app class encapsulating foundation workflow
│   ├── config.py               Configuration dataclass
│   ├── session_manager.py      Session metadata storage (not state!)
│   ├── project.py              Project detection logic
│   ├── main.py                 CLI entry point (Typer/Click/argparse)
│   └── commands/               Command implementations
├── providers/                   Provider bundles
├── bundles/                     Base bundles
└── agents/                      Agent bundles
```

### Core Pattern: AmplifierApp Class

The `AmplifierApp` class encapsulates the foundation workflow:

```python
class AmplifierApp:
    """Foundation-based CLI application."""

    def __init__(self, config: Config):
        self.config = config
        self.session: Optional[Session] = None
        self.prepared: Optional[PreparedBundle] = None

    async def initialize(self) -> None:
        """Initialize: load → compose → prepare → session."""
        bundles = []

        # 1. Load foundation
        foundation = await load_bundle(
            "git+https://github.com/microsoft/amplifier-foundation@main"
        )
        bundles.append(foundation)

        # 2. Load provider
        provider = await load_bundle(self.config.provider)
        bundles.append(provider)

        # 3. Load config files (configs ARE bundles!)
        global_config = Path.home() / ".amplifier" / "config.yaml"
        if global_config.exists():
            bundles.append(await load_bundle(str(global_config)))

        # 4. Load project config
        if self.config.project_root:
            project_config = find_project_config(self.config.project_root)
            if project_config:
                bundles.append(await load_bundle(str(project_config)))

        # 5. Compose all (later wins)
        composed = bundles[0]
        for bundle in bundles[1:]:
            composed = composed.compose(bundle)

        # 6. Prepare and create session
        self.prepared = await composed.prepare()
        self.session = await self.prepared.create_session()

    async def execute(self, prompt: str) -> str:
        """Execute prompt against session."""
        return await self.session.execute(prompt)

    async def spawn_agent(self, agent_path: str, instruction: str) -> dict:
        """Spawn agent sub-session."""
        agent_bundle = await load_bundle(agent_path)
        return await self.prepared.spawn(
            child_bundle=agent_bundle,
            instruction=instruction,
            compose=True,
            parent_session=self.session,
        )
```

**Key Insights:**
- Direct foundation API usage (no wrappers)
- Config files ARE bundles (reuse mechanism)
- Composition handles all merging
- Foundation manages session state
- App layer just orchestrates composition order

### Configuration System

Simple dataclass with environment variable support:

```python
@dataclass
class Config:
    """Application configuration."""

    provider: str = field(
        default_factory=lambda: os.getenv(
            "AMPLIFIER_PROVIDER",
            "./providers/anthropic.yaml"
        )
    )
    bundle: Optional[str] = field(
        default_factory=lambda: os.getenv("AMPLIFIER_BUNDLE")
    )
    bundles: list[str] = field(default_factory=list)
    project_root: Optional[Path] = None

    @classmethod
    def from_env_and_project(cls) -> "Config":
        """Load from env + project detection."""
        config = cls()
        manager = ProjectManager()
        config.project_root = manager.get_project_root()
        return config
```

**No custom config merging needed** - bundle composition handles it!

### Project Detection

Lightweight project management (~50 lines vs app-cli's complex multi-project system):

```python
class ProjectManager:
    def get_project_root(self) -> Path:
        """Find project root (.git or .amplifier directory)."""
        cwd = Path.cwd()
        for parent in [cwd, *cwd.parents]:
            if (parent / ".git").exists() or (parent / ".amplifier").exists():
                return parent
            if parent == Path.home():
                return cwd
        return cwd

    def get_project_id(self, project_root: Path) -> str:
        """Generate stable project ID from path."""
        return hashlib.md5(str(project_root.resolve()).encode()).hexdigest()[:8]

    def get_project_storage(self, project_root: Path) -> Path:
        """Get project-specific storage directory."""
        project_id = self.get_project_id(project_root)
        storage = Path.home() / ".amplifier" / "projects" / project_id
        storage.mkdir(parents=True, exist_ok=True)
        return storage
```

### Session Metadata (Not State!)

**Critical distinction:** We store metadata ONLY (for user reference), not session state.

```python
class SessionManager:
    """Lightweight session metadata storage."""

    def save_session_metadata(self, session_id: str, metadata: dict):
        """Save session metadata."""
        path = self.storage / f"{session_id}.json"
        metadata.update({
            "session_id": session_id,
            "created_at": datetime.now().isoformat(),
        })
        path.write_text(json.dumps(metadata, indent=2))

    def list_sessions(self) -> list[dict]:
        """List all sessions."""
        sessions = []
        for path in self.storage.glob("*.json"):
            sessions.append(json.loads(path.read_text()))
        return sorted(sessions, key=lambda s: s["created_at"], reverse=True)
```

**50 lines vs app-cli's SessionStore: 479 lines** (90% reduction)

Why so much smaller?
- Foundation handles session state during lifetime
- We only store metadata for user to remember context
- No atomic operations, backup files, corruption recovery
- "Resumption" = showing user past context, not restoring state

### CLI Commands with Typer

Modern CLI framework with auto-completion and rich output:

```python
import typer
from rich.console import Console

app = typer.Typer()
console = Console()

@app.command()
def run_prompt(
    prompt: str = typer.Argument(...),
    provider: Optional[str] = typer.Option(None, "--provider", "-p"),
    bundle: Optional[list[str]] = typer.Option(None, "--bundle", "-b"),
):
    """Execute a single prompt."""
    asyncio.run(_execute_prompt(prompt, provider, bundle))

async def _execute_prompt(prompt: str, provider: Optional[str], bundles: Optional[list[str]]):
    config = Config.from_env_and_project()
    if provider:
        config.provider = provider
    if bundles:
        config.bundles = list(bundles)

    async with AmplifierApp(config) as app:
        response = await app.execute(prompt)
        console.print(Markdown(response))
```

### REPL Mode

Interactive mode using prompt_toolkit:

```python
async def repl_mode():
    """Interactive REPL mode."""
    config = Config.from_env_and_project()
    config.validate()

    async with AmplifierApp(config) as app:
        history_file = Path.home() / ".amplifier" / "repl_history"
        prompt_session = PromptSession(
            history=FileHistory(str(history_file)),
            multiline=True,
        )

        while True:
            try:
                user_input = await prompt_session.prompt_async(">>> ")
                if user_input.lower() in ("exit", "quit"):
                    break

                response = await app.execute(user_input)
                console.print(Markdown(response))

            except KeyboardInterrupt:
                continue
            except EOFError:
                break
```

Foundation maintains context across all REPL inputs automatically!

### Agent Commands

Direct spawn() usage (no wrapper needed):

```python
@app.command(name="run")
def run_agent(
    agent: str = typer.Argument(...),
    instruction: str = typer.Argument(...),
    resume: Optional[str] = typer.Option(None, "--resume"),
):
    """Spawn an agent."""
    asyncio.run(_spawn_agent(agent, instruction, resume))

async def _spawn_agent(agent: str, instruction: str, resume_id: Optional[str]):
    config = Config.from_env_and_project()
    async with AmplifierApp(config) as app:
        agent_path = _resolve_agent_path(agent)
        result = await app.spawn_agent(
            agent_path,
            instruction,
            session_id=resume_id  # None = new, or ID to resume
        )
        console.print(Markdown(result["output"]))
```

Foundation handles agent resumption natively!

### Bundle Commands

Inspect bundles using foundation's Bundle object:

```python
@app.command()
def inspect(bundle_path: str):
    """Inspect bundle contents."""
    asyncio.run(_inspect_bundle(bundle_path))

async def _inspect_bundle(bundle_path: str):
    bundle = await load_bundle(bundle_path)

    console.print(f"{bundle.name} v{bundle.version}")
    if bundle.providers:
        console.print("\nProviders:")
        for p in bundle.providers:
            console.print(f"  • {p['module']}")

    if bundle.tools:
        console.print("\nTools:")
        for t in bundle.tools:
            console.print(f"  • {t['module']}")
```

Bundle object is self-documenting - no custom introspection needed!

### Code Reduction Breakdown

| Component | Without Foundation | With Foundation | Reduction |
|-----------|-------------------|-----------------|-----------|
| Session management | ~800+ lines | ~50 lines (metadata only) | ~94% |
| Bundle loading | ~700+ lines | 0 (foundation) | 100% |
| Mention loading | ~450+ lines | 0 (foundation) | 100% |
| Config system | ~700+ lines | ~100 lines (simple dataclass) | ~86% |
| Commands | ~6,000+ lines | ~800 lines | ~87% |
| **Total** | **~10,000+** | **~1,500** | **~85%** |

### Key Takeaways

**Do:**
- Use foundation APIs directly
- Let config files BE bundles
- Store only session metadata, not state
- Use bundle composition for all merging
- Leverage foundation features (mentions, caching, hooks)

**Don't:**
- Wrap foundation APIs (spawner, session store)
- Create custom config merging
- Try to persist session state
- Reimplement foundation features
- Copy app-cli internals

**Pattern Summary:**
```
Your CLI → AmplifierApp → foundation workflow
            ↓
         load → compose → prepare → session → execute
            ↑
      All heavy lifting done by foundation
```

The app layer is truly thin (typically ~1,500 LOC) - just CLI commands, config search, and metadata tracking.

## File Reference

See [references/ARCHITECTURE.md](references/ARCHITECTURE.md) for foundation architecture.

## External Documentation

- **Foundation Concepts**: https://github.com/microsoft/amplifier-foundation/blob/main/docs/CONCEPTS.md
- **Common Patterns**: https://github.com/microsoft/amplifier-foundation/blob/main/docs/PATTERNS.md
- **CLI Blueprint**: https://github.com/microsoft/amplifier-foundation/blob/main/examples/08_cli_application.py
- **Profile Authoring**: https://github.com/microsoft/amplifier-profiles/blob/main/docs/PROFILE_AUTHORING.md
