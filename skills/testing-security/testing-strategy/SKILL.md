---
name: testing-strategy
description: Comprehensive guide for implementing AIDB tests following E2E-first philosophy,
  DebugInterface abstraction, and MCP response health standards
version: 1.0.0
tags:
  - testing
  - e2e
  - integration
  - unit
  - mcp
  - framework
  - debugging
---

# AIDB Testing Strategy

**Priority:** E2E → Integration → Unit (Highest ROI First)

______________________________________________________________________

## CRITICAL: Test Execution Command

**All tests MUST be run via `./dev-cli test run`:**

```bash
./dev-cli test run -s {suite} [-k 'pattern'] [-l {lang}]
```

- Multiple `-k` and `-l` flags supported
- **NEVER use `--local`** - suites know their natural execution environment; forcing local causes unexpected behavior
- Direct `pytest` invocation is NOT supported

______________________________________________________________________

This skill guides you through creating and modifying tests for the AIDB project. The test infrastructure is complete - your job is to implement tests using proven patterns.

## Related Skills

When implementing tests, you may also need:

- **adapter-development** - Tests validate adapter behavior across languages
- **mcp-tools-development** - MCP tools tested with DebugInterface abstraction
- **dap-protocol-guide** - Tests exercise DAP protocol flows end-to-end
- **ci-cd-workflows** - For testing CI/CD workflows themselves (not application code)

## Core Philosophy

**For complete test architecture**, see test infrastructure in `src/tests/`.

### 1. E2E First, Unit Last

**Why E2E First?**

- Validates real user workflows
- Exercises the full stack (catches integration bugs early)
- Most bugs discovered when testing actual components
- Higher ROI than unit tests initially

**Testing Priority:**

1. **E2E Tests** - Complete workflows, real programs, full integration
1. **Integration Tests** - Component interactions, adapter lifecycle, state management
1. **Unit Tests** - Edge cases, specific logic, error handling

### 2. Framework Tests: Keep It Simple

**Goal:** Verify we can launch and debug programs using various frameworks

**Pattern:**

1. Connect to framework app (Django, Express, Spring Boot, etc.)
1. Quick smoke test: set breakpoint → inspect → step
1. That's it. Don't test framework internals.

**We're testing that AIDB works WITH frameworks, not testing the frameworks themselves.**

### 3. MCP Responses: Structure + Content + Efficiency

**Don't just validate structure** - validate content accuracy and payload efficiency.

**Bad test:**

```python
assert "locals" in response["data"]  # Structure only
```

**Good test:**

```python
# Structure
assert "locals" in response["data"]

# Content accuracy
assert response["data"]["locals"]["x"]["value"] == 10
assert response["data"]["locals"]["x"]["line"] == 5

# Efficiency (no junk)
assert len(response["data"]) <= 3  # No bloated payloads
assert len(response["summary"]) <= 200  # Concise summaries
```

### 4. VS Launch: Critical for Agent Workflows

**Why critical:**

- Primary entry point for agents using framework debugging
- Enables use of existing workspace launch configs
- Complex variable substitution must work (`${workspaceFolder}`, etc.)

**Test thoroughly:**

- Core launch.json parsing (language-independent)
- Per-language config translation (Python/JavaScript/Java)
- Variable substitution and resolution
- Framework-specific launch configs

**Critical Breakpoint Timing:**

Set breakpoints when STARTING sessions (not after) to avoid race conditions with fast-executing programs:

```python
# ✅ CORRECT
await debug_interface.start_session(program=prog, breakpoints=[{"line": 10}])

# ❌ WRONG: Race condition
await debug_interface.start_session(program=prog)
await debug_interface.set_breakpoint(file=prog, line=10)  # May be too late!
```

Exception: Long-running processes (servers) where you attach. Reference: `src/tests/aidb_shared/e2e/test_complex_workflows.py`

## DebugInterface Abstraction

The cornerstone of our test strategy is the **DebugInterface abstraction** - a unified API that works with both MCP tools and the direct API.

**For implementation details**, see:

- [DebugInterface](resources/debug-interface.md) - Skill resource file
- `src/tests/_helpers/debug_interface/` - Debug interface source and docstrings

**Why?** One test validates both entry points.

**Hypothetical Example** (illustrates pattern, not a real test file):

```python
from tests._helpers.parametrization import parametrize_interfaces

class TestBreakpoints(BaseE2ETest):
    @parametrize_interfaces  # Runs twice: MCP and API
    @pytest.mark.asyncio
    async def test_set_breakpoint(self, debug_interface, simple_program):
        """Test works with BOTH MCP and API."""
        await debug_interface.start_session(program=simple_program)

        bp = await debug_interface.set_breakpoint(
            file=simple_program,
            line=5
        )

        self.verify_bp.verify_breakpoint_verified(bp)
        await debug_interface.stop_session()
```

**Key Points:**

- `@parametrize_interfaces` runs test with both MCP and API
- Same test logic validates both entry points
- No duplication, no drift
- See [DebugInterface](resources/debug-interface.md) for details

## The Shared Suite: Testing Debug Fundamentals

The **shared suite** is AIDB's language-agnostic test foundation that validates core debugging capabilities across all supported languages using normalized, programmatically generated test programs.

**Key Innovation:** Semantic markers that map identical logic to language-specific line numbers.

**Location:** `src/tests/aidb_shared/` (integration/ + e2e/)

**What it tests:**

- Debug primitives (breakpoints, stepping, variables)
- Control flow across all 3 languages (Python, JavaScript, Java)
- Zero duplication: One test → 6 execution paths (2 interfaces × 3 languages)

**For complete details**, see [DebugInterface](resources/debug-interface.md).

### When to Use the Shared Suite

**Use shared suite when:**

- Testing core debug operations (breakpoint, step, inspect)
- Validating adapter behavior across languages
- Ensuring language parity (all adapters work identically)

**Use framework tests when:**

- Testing framework-specific debugging (Django ORM, Express middleware)
- Validating launch.json configurations
- Testing real-world application patterns

## Test Organization

```
src/tests/
├── aidb_shared/               # ⭐ Shared suite: language-agnostic debug fundamentals
│   ├── integration/          # Core debug operations (breakpoints, stepping, variables)
│   └── e2e/                  # Complex workflows, parallel sessions
├── aidb/                      # Core API tests - organized by component
│   ├── adapters/             # Adapter-specific tests
│   ├── audit/                # Audit logging tests
│   ├── common/               # Common utilities tests
│   ├── dap/                  # DAP client tests
│   ├── models/               # Model tests
│   ├── resources/            # Resource management tests
│   ├── service/              # Service layer tests
│   └── session/              # Session management tests
├── aidb_mcp/                  # MCP server tests - organized by component
├── frameworks/                # Framework integration tests
│   ├── python/               # Flask, FastAPI, pytest
│   ├── javascript/           # Express, Jest
│   └── java/                 # Spring Boot, JUnit
├── _helpers/                  # Test helpers and utilities
├── _fixtures/                 # Shared fixtures
│   └── unit/                 # ⭐ Unit test infrastructure (see below)
└── _assets/                   # Test programs and data
    ├── framework_apps/       # Framework test applications
    └── test_programs/        # Generated programs for shared suite
```

### Unit Test Infrastructure

The centralized unit test infrastructure at `src/tests/_fixtures/unit/` provides reusable mocks, builders, and fixtures:

```
_fixtures/unit/
├── builders/           # DAPRequestBuilder, DAPResponseBuilder, DAPEventBuilder
├── dap/               # Transport, events, receiver mocks
├── session/           # Registry, lifecycle, state, child_manager mocks
├── adapter/           # Port, process, launch_orchestrator mocks
├── mcp/               # DebugService, MCPSessionContext mocks
├── conftest.py        # Master fixture re-exports
├── context.py         # mock_ctx, null_ctx, tmp_storage
└── assertions.py      # UnitAssertions class
```

**Usage Pattern:**

```python
# In domain conftest.py (e.g., src/tests/aidb/dap/unit/conftest.py)
from tests._fixtures.unit.conftest import *  # noqa: F401, F403
from tests._fixtures.unit.builders import DAPEventBuilder, DAPResponseBuilder

# In test file
def test_something(mock_ctx, mock_transport):
    event = DAPEventBuilder.stopped_event(reason="breakpoint")
    # ...
```

**Key Components:**

- **Builders** - Fluent API for DAP protocol objects (requests, responses, events)
- **mock_ctx** - Standard logging context mock with debug/info/warning/error methods
- **UnitAssertions** - DAP-specific assertion helpers

### Test Execution Modes

Test suites run in different environments based on their requirements:

**Local-Only Suites** (no Docker):

- `cli` - CLI command tests
- `mcp` - MCP server unit/integration tests
- `core` - Core AIDB API tests
- `common` - Common utilities tests
- `logging` - Logging framework tests
- `ci_cd` - CI/CD workflow tests

**Docker Suites** (require containers):

- `shared` - Multi-language shared tests (parallel language containers)
- `frameworks` - Framework integration tests (parallel language containers)
- `launch` - Launch config tests (parallel language containers)

**Why the split?**

- **Local suites** test Python-only logic (handlers, validation, utils)
- **Docker suites** test multi-language scenarios
- Multi-language MCP functionality tested in `shared`/`frameworks`/`launch`

**Running tests:**

```bash
./dev-cli test run -s mcp      # Local execution
./dev-cli test run -s shared   # Docker execution
```

## Code Reuse: Don't Reinvent

**Always use existing infrastructure:**

- **Test Base Classes** - `BaseE2ETest`, `BaseIntegrationTest`, `FrameworkDebugTestBase`
- **Parametrization Decorators** - `@parametrize_interfaces`, `@parametrize_languages`
- **Helper Assertions** - `self.verify_bp`, `self.verify_exec`, `MCPAssertions`
- **Constants** - `StopReason`, `TestTimeouts`, `MCPTool`

**For complete details**, see [E2E Patterns](resources/e2e-patterns.md).

## Working Examples

**Study these real tests before writing new ones:**

### Framework Tests (E2E)

- **Python:** `test_flask_debugging.py`, `test_fastapi_debugging.py`, `test_pytest_debugging.py`
- **JavaScript:** `test_express_debugging.py`, `test_jest_debugging.py`
- **Java:** `test_springboot_debugging.py`, `test_junit_debugging.py`

### Core API Tests

- **Launch Variable Resolution:** `test_launch_variable_resolution.py`
- **Session Target Handling:** `test_session_target_handling.py`

**For complete file paths and patterns**, see [E2E Patterns](resources/e2e-patterns.md).

## Common Patterns

**For hypothetical examples illustrating common patterns**, see [E2E Patterns](resources/e2e-patterns.md).

**Key patterns covered:**

1. Basic E2E Test
1. Breakpoint Test with Markers
1. Dual-Launch Equivalence Test
1. MCP Response Validation

## When Creating New Tests

### Step 1: Choose Test Type

- **E2E?** Full workflow, real program, complete integration
- **Integration?** Component interactions, lifecycle management
- **Unit?** Specific function, edge case, error handling

### Step 2: Find Similar Test

Look at [E2E Patterns](resources/e2e-patterns.md):

- Django/Express for framework tests
- Existing tests in the same directory
- Similar test scenarios in other languages

### Step 3: Copy Pattern, Adapt

Don't start from scratch:

1. Copy a working test
1. Adapt to your scenario
1. Use same helpers and assertions
1. Follow same structure

### Step 4: Use Existing Infrastructure

**Don't create:**

- New assertion helpers (use existing)
- New fixtures (check `conftest.py` files first)
- New constants (use `constants.py`)
- New base classes (inherit from existing)

**Do create:**

- Tests using existing patterns
- Scenario-specific test data
- Framework-specific fixtures (if needed)

## Performance Testing

**Current State:** No performance baselines exist yet

**Phase 1:** Establish baselines

- Analyze existing metrics/instrumentation
- Determine "healthy" latencies
- Document target times

**Phase 2:** Regression testing

- Monitor key operations (breakpoint set, variable inspect, step)
- Alert on degradation

**For now:** Focus on functional correctness, not performance.

## Success Criteria

### Test Quality Checklist

- [ ] Test uses `@parametrize_interfaces` for MCP/API coverage
- [ ] Test inherits from appropriate base class
- [ ] Test uses helper assertions, not custom assertions
- [ ] Test validates content accuracy, not just structure
- [ ] MCP tests check efficiency (no bloated payloads)
- [ ] Test has clear docstring explaining what it validates
- [ ] Test follows working examples
- [ ] Test is in correct directory (e2e/integration/unit)

### Framework Test Checklist

- [ ] Inherits from `FrameworkDebugTestBase`
- [ ] Implements `test_launch_via_api()`
- [ ] Implements `test_launch_via_vscode_config()`
- [ ] Implements `test_dual_launch_equivalence()`
- [ ] Sets `framework_name` attribute
- [ ] Uses simple smoke tests (no deep framework testing)

## Investigating Test Failures

**CRITICAL:** When tests fail, check logs BEFORE attempting fixes.

**See:** **[Debugging Failures](resources/debugging-failures.md)** for log locations, investigation workflow, and common patterns.

**For CI test failures**, use the **ci-cd-workflows** skill's troubleshooting guide.

## Resources

| Resource                                              | Content                                              |
| ----------------------------------------------------- | ---------------------------------------------------- |
| [E2E Patterns](resources/e2e-patterns.md)             | Test patterns, markers, code reuse, working examples |
| [Framework Tests](resources/framework-tests.md)       | Dual-launch pattern, Flask/Express examples          |
| [DebugInterface](resources/debug-interface.md)        | Unified API abstraction, shared suite architecture   |
| [Debugging Failures](resources/debugging-failures.md) | Log locations, investigation workflow, common issues |

**Test Infrastructure:** `src/tests/` (see \_fixtures/, \_helpers/ for core components)

## Getting Started

1. **Read CONTEXT.md:** `wip/test-implementation-backlog/CONTEXT.md`
1. **Study working examples:** Flask (`test_flask_debugging.py`) and Express (`test_express_debugging.py`)
1. **Choose a test to implement:** Start with E2E (highest ROI)
1. **Copy a working test:** Don't start from scratch
1. **Adapt to your scenario:** Use same patterns, different data
1. **Validate:** Run test, ensure it passes with both MCP and API

## Questions?

**Internal Documentation**:

- `src/tests/` - Test infrastructure (see \_fixtures/, \_helpers/)
- `docs/developer-guide/overview.md` - System architecture

**Code References**:

- **DAP Protocol:** See `src/aidb/dap/protocol/` (fully typed, types.py + requests.py + responses.py + events.py)
- **Test Infrastructure:** See `src/tests/_helpers/` and `src/tests/_fixtures/`
- **Working Examples:** See Flask/Express framework tests

______________________________________________________________________

**Remember:**

- E2E first, validate content accuracy
- Use shared suite for debug fundamentals, framework tests for integration
- Keep framework tests simple (no framework internals)
- Always use the DebugInterface abstraction (zero duplication)
