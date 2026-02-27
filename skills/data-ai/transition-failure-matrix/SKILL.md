---
name: "Transition Failure Matrix"
description: "Identify failure hotspots in multi-step agent workflows using state transition analysis. Use when debugging agent pipelines, analyzing LLM orchestration failures, or systematically identifying where multi-step processes break down. Creates a grid mapping source states to failure points for targeted debugging."
---

# Transition Failure Matrix

## What This Skill Does

Teaches systematic identification of **failure hotspots** in multi-step workflows using state transition analysis. When an agent has many steps (Parse, Search, Code, Execute, etc.), it's hard to know which step is failing most. A Transition Failure Matrix reveals exactly where failures cluster, enabling targeted debugging and reliability improvements.

**Core Technique:**
1. **Define states** - List all steps your agent can be in
2. **Create a matrix** - Grid where rows = "From State", columns = "To State"
3. **Count failures** - For each failure, record which transition was attempted
4. **Find hotspots** - High-count cells reveal where to focus effort

---

## Choose Your Implementation

This skill provides **two implementations** for different environments:

| Feature | Python | TypeScript |
|---------|--------|------------|
| **Runtime** | Backend/server-side, CLI tools | Browser-based apps, frontend |
| **Persistence** | Log files, filesystem | localStorage (in-browser) |
| **Analysis** | CLI tool (`tfm_analyze.py`) | Browser console functions |
| **Integration** | Decorators (`@track_state`) | Direct function calls |
| **Best For** | Server agents, data pipelines, log analysis | React/Vue apps, browser workflows |

**Quick Decision Guide:**
- **Is your code running in a browser?** → Use TypeScript
- **Do you have log files to analyze?** → Use Python
- **Building a CLI tool or backend service?** → Use Python
- **Building a React/Vue/Svelte app?** → Use TypeScript

---

## Prerequisites

- Multi-step workflow with identifiable states
- Basic understanding of state machines
- **For Python:** Python 3.8+, log file access
- **For TypeScript:** Browser environment (React/Vue/etc.)

---

## Quick Start

### Python Version (Backend/CLI)

#### 1. Define Your States

```python
STATES = [
    "ParseReq",      # Parse user request
    "IntentClass",   # Classify intent
    "DecideTool",    # Decide which tool to use
    "GenSQL",        # Generate SQL query
    "ExecSQL",       # Execute SQL query
    "FormatResp",    # Format response
]
```

#### 2. Add Decorator Logging

```python
from scripts.tfm_decorator import track_state, TransitionTracker

tracker = TransitionTracker()

@track_state("ParseReq")
async def parse_request(data):
    return parse(data)

@track_state("ExecSQL")
async def execute_sql(query):
    return db.execute(query)
```

#### 3. Generate the Matrix

```bash
# From log files
python scripts/tfm_analyze.py --log-file agent.log

# Or from the tracker
print(tracker.render_markdown())
```

### TypeScript Version (Browser)

#### 1. Copy Files to Your Project

```bash
cp typescript/transitionLogger.ts src/services/
cp typescript/analyzeTransitions.ts src/utils/
```

#### 2. Import and Use

```typescript
import { logTransition } from './services/transitionLogger';

// In your workflow functions
async function executeStep(from: string, to: string) {
  try {
    await doWork();
    logTransition({
      fromState: from,
      toState: to,
      status: 'SUCCESS',
      timestamp: Date.now(),
      framework: 'your-framework',
    });
  } catch (e) {
    logTransition({
      fromState: from,
      toState: to,
      status: 'FAILURE',
      error: String(e),
      timestamp: Date.now(),
      framework: 'your-framework',
    });
  }
}
```

#### 3. Analyze in Browser Console

```javascript
// Open DevTools console and run:
analyzeTransitions()
```

---

## Python Implementation Guide

### Step 1: Define Your States

**Goal:** Identify discrete, observable steps in your workflow.

**Guidelines:**
- Each state should represent a **complete** action, not partial progress
- States should be **observable** (you can tell when you're in one)
- Start with **5-10 states** - add granularity only if needed

**Common State Patterns:**

| Workflow Type | Example States |
|--------------|----------------|
| LLM Agent | ParseRequest, PlanSteps, SelectTool, ExecuteTool, FormatOutput |
| ETL Pipeline | Extract, Validate, Transform, Load, Verify |
| API Handler | ReceiveRequest, Authenticate, Authorize, Process, Respond |
| Multi-Tool Agent | Parse, Route, Tool_FileOps, Tool_API, Tool_DB, Aggregate |

**Anti-patterns to Avoid:**

```python
# Too coarse - hides where failures actually occur
STATES = ["Start", "Middle", "End"]

# Too fine - matrix becomes too sparse
STATES = ["ParseChar1", "ParseChar2", ..., "ParseCharN"]

# Overlapping - confuses transition tracking
STATES = ["Processing", "StillProcessing", "AlmostDone"]
```

### Step 2: Instrument Transition Logging

**Option A: Manual Logging**

Add transition logs at each state change:

```python
import logging
from datetime import datetime

logger = logging.getLogger("tfm")

def log_transition(from_state: str, to_state: str, success: bool, error: str = None):
    """Log a state transition for failure matrix analysis."""
    status = "SUCCESS" if success else "FAILURE"
    msg = f"TRANSITION: {from_state} -> {to_state} {status}"
    if error:
        msg += f" ERROR: {error}"
    logger.info(msg)

# Usage in your code
def execute_sql(query: str):
    try:
        result = db.execute(query)
        log_transition("GenSQL", "ExecSQL", success=True)
        return result
    except Exception as e:
        log_transition("GenSQL", "ExecSQL", success=False, error=str(e))
        raise
```

**Option B: Decorator-Based (Recommended)**

Use the provided decorator for automatic tracking:

```python
from scripts.tfm_decorator import track_state, TransitionTracker

tracker = TransitionTracker()

@track_state("ParseRequest")
async def parse_request(data):
    return parse(data)

@track_state("ClassifyIntent")
async def classify_intent(parsed):
    return classify(parsed)

@track_state("ExecuteSQL")
async def execute_sql(query):
    return db.execute(query)

# After running, get the matrix
print(tracker.get_hotspots())
```

### Step 3: Collect Failure Data

**From Log Files:**

```bash
# Extract transition logs
rg "TRANSITION:" agent.log > transitions.log

# Analyze with CLI tool
python scripts/tfm_analyze.py --log-file transitions.log
```

**From Python Tracker:**

```python
tracker = TransitionTracker.get_instance()

# Get matrix summary
summary = tracker.get_matrix_summary()
print(f"Total transitions: {summary['total_events']}")
print(f"Total failures: {summary['total_failures']}")

# Get ranked hotspots
for from_s, to_s, count in tracker.get_hotspots(min_count=2):
    print(f"  {from_s} -> {to_s}: {count} failures")
```

**Minimum Sample Size:**
- 50+ transitions to see patterns
- 100+ for reliable hotspot identification
- Consider time windowing for production systems

### Step 4: Generate the Matrix

**CLI Tool Usage:**

```bash
# Markdown output (default)
python scripts/tfm_analyze.py --log-file agent.log --output matrix.md

# ASCII for terminal viewing
python scripts/tfm_analyze.py --log-file agent.log --format ascii

# JSON for programmatic use
python scripts/tfm_analyze.py --log-file agent.log --format json

# Specify states explicitly
python scripts/tfm_analyze.py --states "Parse,Route,Execute,Format" --log-file agent.log
```

**Output Example:**

```markdown
# Transition Failure Matrix

Total Transitions: 847
Total Failures: 45
Failure Rate: 5.3%

| From \ To | ParseReq | IntentClass | DecideTool | GenSQL | ExecSQL |
|-----------|----------|-------------|------------|--------|---------|
| **START** | 2 | - | - | - | - |
| **ParseReq** | - | **3** | - | - | - |
| **IntentClass** | - | - | **4** | - | - |
| **DecideTool** | - | - | - | **6** | - |
| **GenSQL** | - | - | - | - | **12** |
| **ExecSQL** | - | - | - | - | 5 |

## Hotspots (failures >= 2)
- GenSQL -> ExecSQL: 12 failures
- DecideTool -> GenSQL: 6 failures
- ExecSQL -> ExecSQL: 5 failures (retry loop)
```

### Step 5: Analyze Hotspots

**Reading the Matrix:**

| Pattern | Interpretation |
|---------|---------------|
| High count in one cell | Single problematic transition - focus here |
| High counts in a row | Source state is unstable - everything after it fails |
| High counts in a column | Target state is hard to reach - upstream issues |
| Diagonal entries | Retry loops - may indicate infinite retry without fix |
| Sparse matrix | Good! Failures are isolated, not systemic |

**Common Hotspot Causes:**

| Transition Type | Common Causes |
|----------------|---------------|
| Parse -> Next | Invalid input format, encoding issues |
| Decide -> Execute | Wrong tool selected, missing parameters |
| Generate -> Execute | Syntax errors, permission issues |
| Execute -> Format | Timeout, resource exhaustion |
| Any -> Same (diagonal) | Retry loop without fixing root cause |

### Step 6: Take Action

**Priority Order:**
1. Fix highest-count hotspots first (biggest reliability impact)
2. Address diagonal entries (prevent infinite loops)
3. Look for patterns in error messages

**Actions Per Hotspot:**

```python
# 1. Create bug ticket (integrate with bugtracker-workflow)
mcp__bhp_server__add_bug(
    title="High failure rate: GenSQL -> ExecSQL (12 failures)",
    description="""
TRANSITION: GenSQL -> ExecSQL
FAILURE COUNT: 12 (highest in matrix)
PERIOD: Last 24 hours

LIKELY CAUSES:
- SQL syntax errors from LLM
- Permission denied on certain tables
- Query timeout on large datasets

INVESTIGATION:
1. Sample error messages from logs
2. Check SQL validation before execution
3. Review timeout settings
    """,
    priority="high",
    tags=["transition-failure", "GenSQL", "ExecSQL"]
)

# 2. Add targeted test
def test_sql_execution_failure_modes():
    """Test known failure modes for GenSQL -> ExecSQL transition."""
    # Test SQL syntax validation
    # Test permission handling
    # Test timeout behavior

# 3. Implement retry with backoff
@retry(max_attempts=3, backoff=exponential)
def execute_sql(query):
    ...
```

---

## CLI Tool Reference

### tfm_analyze.py

```bash
python scripts/tfm_analyze.py [OPTIONS]

Options:
  --log-file, -l PATH    Log file to analyze (or stdin if not specified)
  --states, -s TEXT      Comma-separated list of states (auto-detected if omitted)
  --format, -f FORMAT    Output format: markdown, ascii, json (default: markdown)
  --output, -o PATH      Output file (stdout if not specified)
  --min-failures INT     Minimum failures to show in hotspots (default: 1)
  --help                 Show help message

Examples:
  # Analyze log file, output markdown
  python scripts/tfm_analyze.py -l agent.log -o matrix.md

  # Pipe from grep, ASCII output
  rg "TRANSITION" app.log | python scripts/tfm_analyze.py -f ascii

  # Explicit states, JSON output
  python scripts/tfm_analyze.py -s "A,B,C,D" -l data.log -f json
```

---

## Integration with Other Skills

### bugtracker-workflow

Auto-create bugs for high-failure transitions:

```python
hotspots = matrix.get_hotspots(min_count=5)
for from_state, to_state, count in hotspots:
    # Threshold-based bug creation
    mcp__bhp_server__add_bug(
        title=f"Transition failure: {from_state} -> {to_state} ({count}x)",
        priority="high" if count > 10 else "medium",
        tags=["transition-failure", from_state, to_state]
    )
```

### python-debugging-pdb

Add conditional breakpoints at hotspot transitions:

```python
@track_state("ExecSQL")
def execute_sql(query):
    tracker = TransitionTracker.get_instance()

    # Break only at known hotspot transition
    if tracker.get_current_state() == "GenSQL":
        import ipdb; ipdb.set_trace()  # Debug the hotspot

    return db.execute(query)
```

### test-coverage-analysis

Ensure hotspot transitions have test coverage:

```bash
# Generate coverage for hotspot code paths
pytest --cov=src/sql_executor --cov-report=term-missing tests/

# Focus on the transition that fails most
pytest tests/test_sql_executor.py -v -k "test_execute"
```

---

## Troubleshooting

### Issue: States Too Coarse

**Symptoms:** All failures cluster in one or two cells

**Solution:** Add intermediate states to reveal where failures actually occur

```python
# Before (too coarse)
STATES = ["Input", "Process", "Output"]

# After (better granularity)
STATES = ["Input", "Validate", "Transform", "Execute", "Format", "Output"]
```

### Issue: States Too Fine

**Symptoms:** Matrix is very sparse, no clear patterns

**Solution:** Aggregate related states

```python
# Before (too fine)
STATES = ["Parse_JSON", "Parse_XML", "Parse_CSV", ...]

# After (aggregated)
STATES = ["Parse", "Validate", "Execute", ...]
```

### Issue: Missing Transitions

**Symptoms:** Matrix undercounts known failures

**Solution:** Check logging coverage - ensure every state change is logged

```bash
# Verify transition log format
rg "TRANSITION:" agent.log | head -10

# Should see: TRANSITION: StateA -> StateB SUCCESS|FAILURE
```

### Issue: Retry Loops Dominate

**Symptoms:** Diagonal entries (A -> A) have highest counts

**Solution:** This indicates retries without fixing root cause. Add retry limits and investigate underlying failures.

---

## Advanced Techniques

### Temporal Analysis

Track how the matrix changes over time:

```bash
# Generate matrix for each day
for day in 01 02 03 04 05; do
  python scripts/tfm_analyze.py \
    --log-file "logs/2025-01-$day.log" \
    --output "matrices/2025-01-$day.md"
done

# Compare: Did hotspots improve after a fix?
```

### Conditional Matrices

Generate separate matrices for different conditions:

```python
# Matrix per user type
for user_type in ["free", "premium", "enterprise"]:
    logs = filter_logs_by_user_type(user_type)
    matrix = analyze(logs)
    save(f"matrix_{user_type}.md")

# Reveals: Premium users fail at different transitions than free users
```

### Root Cause Chains

Extend analysis beyond single transitions:

```python
# Track failure chains: A -> B -> C -> FAIL
# Which 3-step sequences fail most often?
chains = analyze_failure_chains(logs, chain_length=3)
# Output: ["Parse", "Validate", "Execute"] fails 15 times
```

---

## Best Practices

1. **Start Simple** - Begin with 5-10 states, add granularity only when needed
2. **Consistent Logging** - Use the exact format: `TRANSITION: A -> B SUCCESS|FAILURE`
3. **Regular Analysis** - Run matrix analysis after each release or weekly
4. **Track Improvements** - Save historical matrices to measure progress
5. **Integrate Early** - Add instrumentation before problems arise
6. **Prioritize by Impact** - Focus on high-count, high-severity cells first
7. **Share Insights** - Include matrix in post-mortems and retrospectives

---

## Related Skills

- **bugtracker-workflow** - Create tickets for identified hotspots
- **python-debugging-pdb** - Debug hotspot transitions with breakpoints
- **test-coverage-analysis** - Ensure hotspots have test coverage
- **swarm-orchestration** - Debug parallel agent failures
- **tdd-red-green-refactor** - Write tests to prevent hotspot recurrence

---

## TypeScript Implementation Guide

### Browser-Based Analysis

The TypeScript implementation is designed for **browser-based applications** where you want to analyze failures without backend infrastructure.

**Key Differences from Python:**
- No decorators - direct function calls
- Data stored in localStorage, not log files
- Analysis via browser DevTools console
- Auto-exposes `window.analyzeTransitions()` for easy access

### Step 1: Install Files

Copy to your project:

```bash
cp typescript/transitionLogger.ts src/services/
cp typescript/analyzeTransitions.ts src/utils/
```

Import in your main entry point:

```typescript
// index.tsx or main.ts
import './utils/analyzeTransitions'; // Auto-binds to window
```

### Step 2: Instrument Your Workflow

```typescript
import { logTransition } from './services/transitionLogger';

// Wrap state transitions with logging
async function runWorkflowStep(from: string, to: string) {
  try {
    await performStep();
    logTransition({
      fromState: from,
      toState: to,
      status: 'SUCCESS',
      timestamp: Date.now(),
      framework: 'corporate-board', // your framework type
      metadata: { userId, sessionId }
    });
  } catch (e) {
    logTransition({
      fromState: from,
      toState: to,
      status: 'FAILURE',
      error: e instanceof Error ? e.message : String(e),
      timestamp: Date.now(),
      framework: 'corporate-board',
      metadata: { userId, sessionId }
    });
  }
}
```

### Step 3: Analyze in Console

After running your app, open DevTools and run:

```javascript
// Full analysis with tables
analyzeTransitions()

// Get raw data
getTransitionMatrix()

// Export as JSON
exportTransitions()

// Clear stored data
clearTransitions()
```

**Console Output Example:**

```
=== TRANSITION FAILURE MATRIX ANALYSIS ===

SUMMARY:
  Total Transitions: 45
  Successes: 38 (84.4%)
  Failures: 7 (15.6%)
  Overall Failure Rate: 15.6%

🔥 FAILURE HOTSPOTS (2+ failures):
┌─────────┬──────────────────────┬──────────┬──────────┬────────────────────────┐
│ (index) │     From → To        │ Failures │ Fail Rate│      Top Error          │
├─────────┼──────────────────────┼──────────┼──────────┼────────────────────────┤
│    0    │ 'API_CALL → API_...  │    4     │  '80%'   │  'HTTP 429: Rate limit' │
│    1    │ 'STAGE1_COMPLETE ...  │    2     │  '40%'   │  'No fallback model'   │
└─────────┴──────────────────────┴──────────┴──────────┴────────────────────────┘
```

### TypeScript vs Python Reference

| Task | Python | TypeScript |
|------|--------|------------|
| **Instrument code** | `@track_state("State")` decorator | `logTransition({ ... })` call |
| **Store data** | Log files / in-memory | localStorage |
| **Analyze** | `python scripts/tfm_analyze.py` | `analyzeTransitions()` in console |
| **Export data** | `--format json` | `exportTransitions()` |
| **Clear data** | Delete log file | `clearTransitions()` |

---

## Resources

### Python Implementation
- [Log Format Specification](resources/log-format.md)
- [Sample Matrix Example](resources/examples/sample-matrix.md)
- `scripts/tfm_analyze.py` - CLI analysis tool
- `scripts/tfm_decorator.py` - Decorator instrumentation

### TypeScript Implementation
- `typescript/transitionLogger.ts` - Core logging service
- `typescript/analyzeTransitions.ts` - Browser console analysis

### Inspiration
- [Bryan Bischof's Original Talk](https://bit.ly/failure-matrix) - Applied AI Evals

---

**Created**: 2025-01-20
**Category**: Debugging & Observability
**Difficulty**: Intermediate
**Best For**: Multi-step agent workflows, LLM pipelines, complex automation
