---
name: incident-report-debugging
description: Create comprehensive incident reports with knowledge graphs. Use when debugging production issues where you need to trace root cause through multiple code entities. Documents debug process, entity relationships, reasoning→pattern→codebase chain, and prevention strategies.
---

# Incident Report Debugging Methodology

## What This Solves

How to systematically debug complex production issues and document the entire investigation process using a knowledge graph approach. Traces root cause by connecting entities (logs → components → functions → bug) using reasoning, pattern recognition, and codebase validation.

## When to Use This

- User reports vague symptoms ("app is broken", "blank screen", "not working")
- Need to trace through multiple layers of code to find root cause
- Want to document debugging process for future reference
- Need to create post-mortem / incident reports

## The Core Methodology

### Three Pillars of Investigation

```
REASONING → Provides DIRECTION
   ↓
PATTERN → Enables RECOGNITION
   ↓
CODEBASE → GROUND TRUTH (validates and points to next entity)
```

**Critical insight**: Reasoning and patterns give you hypotheses, but **only the codebase tells you where to look next**. Each entity discovered in code points to the next entity, forming a chain to root cause.

## Working Approach

### Phase 1: Observation & Infrastructure Check
```bash
# Verify processes are running
ps aux | grep '<app-name>'
lsof -i :<port> -P -n | grep LISTEN

# Check network/connectivity
curl http://localhost:<port>/health
```

**Output**: Establish baseline - is infrastructure healthy or broken?

### Phase 2: Locate Logs
**Don't guess where logs are** - read the code to find them.

```bash
# Find initialization/setup code
grep -r "log.*file\|LOG.*PATH" <entrypoint-files>
```

Common locations to check:
- Main process entry point (e.g., `main.js`, `server.py`)
- Environment variable definitions (`.env`, config files)
- Documentation (CLAUDE.md, README)

**Pattern**: Applications with structured logging usually write to:
- `~/centralized-logs/<app-name>/`
- `/var/log/<app-name>/`
- `<project>/.logs/`

### Phase 3: Analyze Logs for Entities
```bash
# Search for errors
grep -i 'error\|fail\|crash' <logfile> | tail -50

# Count error frequency (indicates severity)
grep -c "specific error message" <logfile>

# Find component/function names mentioned
grep -o "component.*name\|function.*name" <logfile>
```

**Extract**:
- Error messages (patterns)
- Component/module names (next entity to investigate)
- Timestamps (frequency/timing of issue)
- Stack traces (call chain)

### Phase 4: Trace Through Code Entities
For each entity discovered, find the next entity:

```bash
# Search for the entity in codebase
grep -rn "EntityName" <src-directory>

# Find dependencies/imports
grep -rn "import.*Entity\|from.*Entity" .

# Find where entity is used
grep -rn "Entity\.<method>\|<Entity>" .
```

**Build chain**: Entity₁ → Entity₂ → Entity₃ → Root Cause

### Phase 5: Root Cause Hypothesis
**Pattern Recognition Examples**:

| Pattern in Logs | Likely Root Cause | Next Investigation |
|-----------------|-------------------|-------------------|
| "Maximum update depth exceeded" | React infinite render loop | Find useEffect with changing dependencies |
| "ECONNREFUSED" | Service not running or wrong port | Check process list, verify port config |
| "Module not found" | Missing dependency | Check package.json vs imports |
| "Segmentation fault" | Memory corruption, unsafe code | Check pointer usage, array bounds |
| "TypeError: ... is not a function" | Wrong API version, missing method | Check package versions, API docs |

### Phase 6: Validate Hypothesis
```bash
# Apply targeted fix
<make code change>

# Verify fix (dev tools hot-reload, or restart)
# Check logs for absence of error
grep "error message" <logfile> | tail -10
```

## Incident Report Template

Create in: `./incident_reports/YYYY_MM_DD_<short-description>.md`

```markdown
# Incident Report: <Title>

## Metadata
- **Date**: YYYY-MM-DD
- **Severity**: Critical | High | Medium | Low
- **Component**: <service/app name>
- **Status**: Resolved | Investigating | Mitigated
- **Time to Resolution**: <duration>

## Summary
<3-4 sentences: what broke, root cause, fix>

## Debug Process

### Phase 1: Initial Observation
**Input**: User reports "<symptom>"

**Action**: <commands run>

**Findings**:
- ✅ What's working
- ❌ What's broken

**Reasoning**: <why this matters>

---

### Phase 2: Log Discovery
**Pattern**: "<known pattern that guided search>"

**Action**: <how you found log location>

**Ground Truth**: <actual log path from code>

**Findings**: <key log entries>

**Reasoning**: <what this tells you>

---

### Phase 3: Component Identification
**Pattern**: "<error pattern recognized>"

**Action**: <search commands>

**Ground Truth**: <file:line from logs>

**Next Entity**: <next code location to investigate>

---

### Phase 4: Entity Analysis
**Reasoning**: "<hypothesis about the issue>"

**Action**: <code inspection>

**Ground Truth**: <specific code that's suspicious>

**Next Entity**: <where this code came from / calls to>

---

### Phase 5: Root Cause Identification
**Action**: <final code inspection>

**Ground Truth**: <exact code causing issue>

**Reasoning**: <why this causes the problem>

**Pattern Recognition**: <what anti-pattern / known issue this matches>

---

### Phase 6: Hypothesis Validation
**Hypothesis**: <what you think will fix it>

**Solution Applied**: <code change>

**Result**: <outcome>

**Status**: ✅ RESOLVED | ⚠️ MITIGATED | ❌ FAILED

## Knowledge Graph

### Entities
```
E1: <Entity Name>
    - Type: Symptom | Infrastructure | Data Source | Component | Function | Variable
    - Location: <path:line or runtime location>
    - Data: <relevant details>

E2: <Entity Name>
    ...

E<N>: <Root Cause Entity>
    - Type: Bug | Config Error | Missing Dependency
    - Location: <file:line>
```

### Relationships
```
E1 (User Symptom)
    → observed_by → User
    → symptom_of → E<N> (Root Cause)

E2 (Log File)
    → contains → E3 (Error Message)
    → revealed_by → Reading E<X> code
    → points_to → E4 (Component)

E3 (Error Message)
    → identifies_component → E4
    → describes_pattern → "<pattern name>"
    → triggered_by → E<N> (Root Cause)

...continue for all entities...
```

### How Dots Were Connected

#### Chain: From Symptom to Root Cause
```
[User Report: <symptom>]
        ↓
    REASONING: "<why this suggests X>"
        ↓
    PATTERN: "<known pattern>"
        ↓
    CODEBASE (<file:line>): <what code revealed>
        ↓
    GROUND TRUTH: <discovered entity>
        ↓
[Next Entity]
        ↓
    REASONING: "<why investigate this next>"
        ↓
    PATTERN: "<recognized anti-pattern>"
        ↓
    CODEBASE (<file:line>): <what code revealed>
        ↓
    GROUND TRUTH: <next entity>
        ↓
[Root Cause Discovered]
```

**Key Insight**: <describe critical connection that broke the case>

## Technical Deep Dive
<Explain WHY the bug happened at technical level>

### Why Did This Cause <Problem>?
<Step-by-step technical explanation>

### Why <Solution> Fixes It
<Explain mechanism of fix>

## Prevention Strategies

### Lint Rules
```json
{
  "rules": {
    "<rule-name>": "error"
  }
}
```

### Code Review Checklist
- [ ] <check item 1>
- [ ] <check item 2>

### Testing
```<language>
<test code that would have caught this>
```

### Monitoring
<Metrics/alerts that would have detected this>

## Resolution

**Changed Files**:
- `<file 1>`: <change description>
- `<file 2>`: <change description>

**Verification**:
- <how you verified fix works>

**Status**: ✅ RESOLVED
```

## Failed Attempts

⚠️ **Common mistakes when debugging production issues**:

| Attempt | Why It Failed | How to Avoid |
|---------|---------------|--------------|
| Guessing log locations without reading code | Wasted time checking wrong paths | Always read entry point code to find log paths |
| Jumping straight to hypothesis without entity tracing | Fixes wrong issue or misses root cause | Build entity chain systematically |
| Only documenting what worked | Loses valuable "what not to do" knowledge | Document ALL attempts, especially failures |
| Skipping the knowledge graph | Can't explain how you found the bug | Map entities and relationships explicitly |
| Relying only on reasoning/patterns | Leads to wrong conclusions | Always validate with codebase (ground truth) |
| Stopping at first suspicious code | Might not be actual root cause | Trace through entire chain until confirmed |

## Key Configurations

### Log File Discovery
- **Entry points to check**: `main.js`, `main.cjs`, `server.py`, `index.ts`, `__init__.py`
- **Search patterns**: `log.*file`, `LOG.*PATH`, `console.log`, `createWriteStream`
- **Env vars to check**: `LOG_DIR`, `LOG_FILE`, `<APP>_LOG_PATH`

### Error Pattern Recognition
Maintain personal database of patterns:
```
Pattern → Likely Cause → Investigation Path
```

### Incident Report Location
```bash
./incident_reports/YYYY_MM_DD_<short-description>.md
```

## Example: React Infinite Loop

**Symptom**: Blank screen, app running but not rendering

**Log Pattern**: "Maximum update depth exceeded" (113K times)

**Entity Chain**:
```
Blank Screen
  → Log File (from electron/main.cjs code)
  → "Maximum update depth" error
  → "AmsAgentsPage component" (from log)
  → useEffect hook (from AmsAgentsPage.tsx)
  → agentsByPid dependency
  → useAgentTopSse.ts (source of agentsByPid)
  → new Map() on every render (ROOT CAUSE)
```

**Pattern**: React infinite loop = useEffect + changing dependency

**Root Cause**: Map created with `new Map()` without memoization
```typescript
// ❌ Creates new object identity every render
const agentsByPid = new Map();

// ✅ Stable identity between renders
const agentsByPid = useMemo(() => new Map(), [data]);
```

**Fix**: Wrap in `useMemo()` → stable object identity → no false triggers

## Environment & Dependencies

- **Log Analysis Tools**: `grep`, `jq`, `tail`, `less`
- **Code Search**: `grep -rn`, `rg` (ripgrep), IDE search
- **Process Inspection**: `ps`, `lsof`, `netstat`
- **Language**: Any (methodology is language-agnostic)

## Next Steps / Future Work

- [ ] Create template generator script for incident reports
- [ ] Build error pattern database with common root causes
- [ ] Automate log analysis (frequency counting, pattern matching)
- [ ] Integrate with observability tools (Datadog, Sentry, etc.)

## Session Reference

- **Date**: 2025-01-22
- **Agent**: 9ccbed57-7337-4086-8bbf-876e198d8d1b
- **Original task**: "Electron app is blank. what's the problem here?"
- **Resolution**: Found infinite React render loop via systematic entity tracing
- **Key insight**: "Reasoning provides direction, Pattern enables recognition, Codebase is ground truth"
