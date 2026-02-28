---
name: xctrace-skill
version: 1.0.0
description: Profile iOS/macOS app performance with Instruments (xctrace). Use when investigating CPU hotspots, memory leaks, animation hitches, or comparing performance before/after changes.
---

# xctrace Skill

Profile and analyze app performance using Apple's Instruments toolchain via `xctrace`.

## Quick Start

```bash
# 1. List available profiling templates
python scripts/trace_templates.py

# 2. Record a 10-second Time Profiler trace of your app
python scripts/trace_record.py --template "Time Profiler" --attach MyApp --time-limit 10s

# 3. Export trace data to JSON for analysis
python scripts/trace_export.py --input recording.trace --toc

# 4. Analyze a trace and get a summary
python scripts/trace_analyze.py --input recording.trace
```

All scripts support `--help` for detailed options and `--json` for machine-readable output.

## Scripts

### Profiling

1. **trace_templates.py** - List available Instruments templates
   - Shows all templates: Time Profiler, Allocations, Leaks, SwiftUI, etc.
   - Options: `--json`

2. **trace_record.py** - Record a performance trace
   - Profile by attaching to running app or launching
   - Set time limits, output paths
   - Options: `--template`, `--attach`, `--launch`, `--time-limit`, `--output`, `--all-processes`, `--json`

3. **trace_attach.py** - Profile an already-running process
   - Convenience wrapper for trace_record with --attach
   - Options: `--pid`, `--name`, `--template`, `--time-limit`, `--output`, `--json`

### Analysis

4. **trace_export.py** - Export trace data
   - Table of contents (--toc) or XPath queries
   - Options: `--input`, `--output`, `--toc`, `--xpath`, `--json`

5. **trace_analyze.py** - Generate performance summary
   - Extracts key metrics from trace
   - Identifies hotspots, allocations, issues
   - Options: `--input`, `--verbose`, `--json`

6. **trace_compare.py** - Compare two traces
   - Before/after performance comparison
   - Options: `--baseline`, `--current`, `--json`

### Binary Inspection

7. **binary_inspect.py** - Inspect Mach-O binaries
   - View linked libraries, symbols, headers
   - Useful for understanding crashes, dependencies
   - Options: `--headers`, `--libraries`, `--symbols`, `--swift-symbols`, `--json`

## Common Templates

| Template | Use Case |
|----------|----------|
| Time Profiler | CPU usage, find slow functions |
| Allocations | Memory usage, object lifetimes |
| Leaks | Memory leaks |
| SwiftUI | SwiftUI view body evaluations, identity changes |
| Animation Hitches | UI jank, dropped frames |
| App Launch | Startup time analysis |
| Network | HTTP requests, latency |
| Swift Concurrency | Actor isolation, task scheduling |

## Typical Workflows

### "This screen is slow"
```bash
# Record while reproducing the issue
python scripts/trace_record.py --template "Time Profiler" --attach MyApp --time-limit 15s

# Analyze
python scripts/trace_analyze.py --input recording.trace
```

### "App is using too much memory"
```bash
python scripts/trace_record.py --template "Allocations" --attach MyApp --time-limit 30s
python scripts/trace_analyze.py --input recording.trace
```

### "Is my optimization working?"
```bash
# Record baseline
python scripts/trace_record.py --template "Time Profiler" --attach MyApp --output baseline.trace --time-limit 10s

# Make changes, rebuild, then record again
python scripts/trace_record.py --template "Time Profiler" --attach MyApp --output optimized.trace --time-limit 10s

# Compare
python scripts/trace_compare.py --baseline baseline.trace --current optimized.trace
```

### "What's in this binary?"
```bash
python scripts/binary_inspect.py --libraries /path/to/MyApp.app/MyApp
python scripts/binary_inspect.py --swift-symbols /path/to/MyApp.app/MyApp
```

## Requirements

- macOS 12+
- Xcode Command Line Tools
- Python 3

## Notes

- Traces can be large (100MB+). Use `--time-limit` to keep them manageable.
- Some templates require running on device (not simulator).
- For GUI analysis, open `.trace` files in Instruments.app.
