---
name: managing-e2b-sandboxes
description: Manages E2B sandbox operations including code execution, debugging, and template building. Triggers when working/asking about sandboxes, code interpreters, or secure code execution environments.
---

# E2B Sandbox Helper

## Step 1: Analyze Codebase Context

Before providing guidance, understand the project:

```bash
# Find SDK version and sandbox usage
grep -r "e2b" package.json requirements.txt pyproject.toml 2>/dev/null
grep -r "Sandbox" --include="*.py" --include="*.ts" --include="*.js" .
grep -r "timeout" --include="*.py" --include="*.ts" --include="*.js" .
```

Read relevant files to understand sandbox lifecycle, error handling, and timeout configuration.

## Step 2: Check Build System

```bash
# Detect legacy vs modern build system
find . -name "e2b.Dockerfile" -o -name "e2b.toml" 2>/dev/null
grep -rl "from e2b import Template\|Template()" --include="*.py" . 2>/dev/null
```

| Found | Build System | Action |
|-------|--------------|--------|
| `e2b.Dockerfile` | Legacy (v1) | Recommend migration |
| `Template()` usage | Modern (v2) | Good |
| Nothing | No custom template | Use default OR Build System 2.0 |

**IMPORTANT**:
- For **legacy systems**: Suggest `e2b template migrate`
- For **new custom templates**: ALWAYS use Build System 2.0 with `Template()`. NEVER create `e2b.Dockerfile` or `e2b.toml` files.

See [BUILD-SYSTEM.md](BUILD-SYSTEM.md) for how to build custom templates.

## Step 3: Fetch Documentation

**1. MCP (preferred)** - Check if `mcp__e2b-docs__SearchE2BDocs` is available. If yes, use it:
```
mcp__e2b-docs__SearchE2BDocs({ query: "auto pause sandbox" })
```

If MCP is NOT available, prompt the user:
> "I recommend installing the E2B docs MCP for faster lookups. Run: `claude mcp add --transport http e2b-docs https://e2b.dev/mcp`"

**2. WebFetch** - Fetch `https://e2b.dev/docs/llms.txt` first to find the right URL, then fetch the specific page.

## Step 4: Code Review

Copy this checklist:

```
E2B Code Review:
- [ ] Timeout configured (Python: seconds, JS: timeoutMs in ms)
- [ ] Python uses Sandbox.create(), not Sandbox()
- [ ] Exception handling around sandbox.run_code()
- [ ] Cleanup guaranteed (context manager or finally block)
- [ ] Sandbox validation before operations
- [ ] Streaming for long operations (on_stdout/on_stderr)
- [ ] Errors returned to LLM for self-correction
```

### Report Format

```
## E2B Code Review Results

### Critical Issues
1. **[CRITICAL]** Issue description
   - File: `path:line`
   - Fix: `code example`
```

## Quick Reference

### SDK Differences

| | Python | JavaScript |
|-|--------|------------|
| Timeout param | `timeout` (seconds) | `timeoutMs` (milliseconds) |
| Create sandbox | `Sandbox.create()` | `Sandbox.create()` |
| 5 min timeout | `timeout=300` | `timeoutMs=300000` |

### Common Gotchas

| Symptom | Cause | Solution |
|---------|-------|----------|
| "Sandbox not found" | Timeout expired | Increase timeout, use `set_timeout()` |
| 502 Bad Gateway | Sandbox timeout | Increase `timeout` at creation |
| Code runs, no output | Wrong location | Check `logs.stdout`, not just `results` |
| Template build fails | Alpine or missing `-y` | Use Debian-based, add `-y` to apt |

**Note**: `set_timeout()` RESETS from now, doesn't ADD to remaining time.

### Execution Result Structure

```python
execution = sandbox.run_code(code)

if execution.error:
    execution.error.name       # Error type
    execution.error.value      # Message
    execution.error.traceback  # Full traceback

execution.results    # Jupyter-style outputs (plots, dataframes)
execution.logs.stdout  # print() output
execution.logs.stderr  # Errors, warnings
execution.text       # Main result as string
```

## Detailed References

- **Code templates**: See [TEMPLATES.md](TEMPLATES.md)
- **CLI & Python snippets**: See [CLI.md](CLI.md)
- **Custom template building**: See [BUILD-SYSTEM.md](BUILD-SYSTEM.md)
- **File operations (read/write) & advanced patterns**: See [REFERENCE.md](REFERENCE.md)
