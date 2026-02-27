---
name: code-archaeologist
description: Deep historical context analysis for code evolution, risk assessment, and pattern compliance. Use BEFORE modifying any code to detect reverts, hotspots, god objects, and required patterns. Prevents repeating past mistakes by surfacing what was tried before and why it failed.
allowed-tools:
  - Bash(python3:*)
  - Bash(git:*)
  - Read
  - Grep
  - Glob
user-invocable: true
---

# Code Archaeologist Skill

**CRITICAL**: This skill provides proactive intelligence BEFORE code changes to prevent bugs, maintain consistency, and learn from history.

## When to Use This Skill (Auto-Invocation Guide)

### 🚨 ALWAYS Before Modifying Code
**Trigger**: User asks to fix, modify, refactor, or change any existing code

**Commands to run**:
```bash
# 1. Smart analysis - CRITICAL for risk assessment
python3 .claude/skills/code_archaeologist/skill.py smart <folder> --question "$ARGUMENTS"

# 2. File analysis - Check if function is a hotspot
python3 .claude/skills/code_archaeologist/skill.py file <file_path> --function <function_name>

# 3. Pattern compliance - Ensure consistency
python3 .claude/skills/code_archaeologist/skill.py pattern-adoption <folder>
```

**What to look for in output**:
- ⚠️ **REVERTS DETECTED**: Previous attempts failed - escalate to HIGH risk
- ⚠️ **HOTSPOT**: Function modified >20 times - bug-prone, needs comprehensive tests
- ⚠️ **GOD OBJECT**: File >2000 lines or >50 dependencies - refactor first
- ⚠️ **PATTERN VIOLATIONS**: Missing patterns used in >50% of files - enforce

### 🎯 Before Adding New Features
**Trigger**: User wants to add new functionality to existing folder

**Commands to run**:
```bash
# 1. Pattern requirements
python3 .claude/skills/code_archaeologist/skill.py pattern-adoption <folder>

# 2. Architecture constraints
python3 .claude/skills/code_archaeologist/skill.py dep-graph <folder>

# 3. Historical context
python3 .claude/skills/code_archaeologist/skill.py smart <folder>

# 4. Optional: Generate design doc with historical intelligence
python3 .claude/skills/code_archaeologist/skill.py design <folder> \
  --feature "$FEATURE_NAME" \
  --description "$FEATURE_DESC"
```

### 🤔 When User Asks "Why" Questions
**Trigger**: User asks "why", "how did", "what is the history"

**Command to run**:
```bash
python3 .claude/skills/code_archaeologist/skill.py context <folder> --question "$ARGUMENTS"
```

**Provides**:
- Creation story and evolution
- Similar past issues (keyword search)
- Revert detection with failure reasons
- Architectural decisions from commit messages

## Available Commands

### 1. `smart` - Adaptive Risk Analysis (MOST IMPORTANT)
**When**: Before ANY code modification
**What it does**: Intelligent analysis with auto-calculated commit limits, risk scoring, revert detection

```bash
python3 skill.py smart src/auth --question "Fix OAuth bug"
```

**Output**:
- 🧠 Adaptive Intelligence (smart commit limit, reasoning, confidence)
- 📊 Risk Assessment (CRITICAL/HIGH/MEDIUM/LOW with score 0-100)
- ⚠️ Revert History (failed previous attempts)
- 🔍 Similar Issues (keyword-based commit search)
- 📅 Timeline (condensed or full depending on history size)
- ✅ Recommended Actions (based on risk level)

**Critical for**: Preventing repeat failures, understanding risk, knowing what was tried before

### 2. `file` - Function-Level Hotspot Detection
**When**: Before modifying specific functions
**What it does**: Identifies bug-prone functions, complexity growth, coupling

```bash
python3 skill.py file src/router.py --function handle_streaming
```

**Output**:
- 🔥 Hotspots (most frequently modified functions)
- 📈 Complexity Growth (functions that grew significantly)
- 🔗 Function Coupling (what changes together)
- 📅 Function Lifecycle (when added/modified/removed)

**Critical for**: Knowing if a function is high-risk before touching it

### 3. `pattern-adoption` - Consistency Enforcement
**When**: Before adding code to a folder
**What it does**: Shows mandatory patterns, identifies violations

```bash
python3 skill.py pattern-adoption src/auth
```

**Output**:
- 📊 Pattern adoption statistics (percentage for each pattern)
- ✅ Common patterns (>50% adoption = mandatory)
- ⚠️ Pattern violations (files missing common patterns)
- 💡 Recommendations for new code

**Detects 14 patterns**: Async/Await, Structured Error Handling, Context Manager, Input Validation, Logging, Decorator, Factory, Dependency Injection, Retry, Caching, Singleton, Observer, Rate Limiting, Circuit Breaker

**Critical for**: Maintaining codebase consistency

### 4. `dep-graph` - Circular Dependency Detection
**When**: Before refactoring or adding imports
**What it does**: Detects god objects, circular dependencies, fan-in/fan-out

```bash
python3 skill.py dep-graph src/auth
```

**Output**:
- 🎯 God Objects (files with >10 dependents)
- 🔁 Circular Dependencies (import cycles)
- 📊 Fan-in/Fan-out metrics
- 🏝️ Isolated Modules

**Critical for**: Understanding blast radius, avoiding circular imports

### 5. `context` - Historical Context
**When**: User asks "why" questions
**What it does**: Provides targeted historical information

```bash
python3 skill.py context src/auth \
  --question "Why was JWT authentication chosen?"
```

**Output**:
- ⚠️ Revert warnings
- 🔍 Similar issues (keyword search)
- 📖 Evolution summary
- 🏗️ Design decisions

### 6. `design` - Design Doc Generation
**When**: Planning new features
**What it does**: Generates design doc with historical intelligence

```bash
python3 skill.py design src/auth \
  --feature "OAuth 2.0 Authentication" \
  --description "Add OAuth 2.0 support for enterprise customers"
```

**Output**:
- Historical context (reverts, activity, risk)
- Pattern requirements (mandatory patterns)
- Architectural constraints (god objects to avoid)
- Similar features (prior art references)
- Risk assessment

**Critical for**: Avoiding past mistakes, learning from similar features

### 7. `analyze` - Standard Timeline
**When**: Understanding component evolution
**What it does**: Generates full timeline of folder changes

```bash
python3 skill.py analyze src/auth --format markdown
```

### 8. `dependencies` - Import Evolution
**When**: Understanding dependency history
**What it does**: Tracks when each import was added/removed

```bash
python3 skill.py dependencies src/main.py
```

### 9. `patterns` - Pattern Introduction Timeline
**When**: Understanding when patterns were adopted
**What it does**: Shows when each pattern first appeared

```bash
python3 skill.py patterns src/router.py
```

### 10. `discover` - Folder Discovery
**When**: Exploring unfamiliar codebase
**What it does**: Categorizes and recommends folders to analyze

```bash
python3 skill.py discover
python3 skill.py suggest  # Top recommendations
```

## Workflow Examples

### Example 1: Fixing a Bug

**User**: "Fix the streaming timeout in router.py"

**Claude automatically runs**:
```bash
# Step 1: Risk analysis
python3 skill.py smart src/router --question "Fix streaming timeout"

# Output: "⚠️ CRITICAL RISK:
# - Previous REVERT on 2025-06-14
# - router.py is GOD OBJECT (12 dependents)
# - Risk Score: 87/100"

# Step 2: Function analysis
python3 skill.py file src/router.py --function handle_streaming

# Output: "🔥 HOTSPOT: handle_streaming() modified 47 times"

# Step 3: Pattern requirements
python3 skill.py pattern-adoption src/router

# Output: "MUST use: async (100%), error handling (100%), logging (50%)"
```

**Claude then**:
1. Informs user of CRITICAL risk findings
2. Recommends: Feature flag + comprehensive tests + senior review
3. Implements fix following ALL discovered patterns
4. Writes tests based on hotspot analysis

### Example 2: Adding New Feature

**User**: "Add OAuth 2.0 authentication to proxy/auth"

**Claude automatically runs**:
```bash
# Step 1: Pattern requirements
python3 skill.py pattern-adoption src/auth

# Output: "Mandatory patterns: Input Validation (83%), Error Handling (100%), Logging (67%)"

# Step 2: Architecture check
python3 skill.py dep-graph src/auth

# Output: "⚠️ WARNING: auth/__init__.py is GOD OBJECT (15 dependents)"

# Step 3: Generate design doc
python3 skill.py design src/auth \
  --feature "OAuth 2.0" \
  --description "Add OAuth 2.0 authentication"

# Output: Complete design doc with historical context, risks,
# and local prior art references
```

**Claude then**:
1. Shows user the design doc
2. Explains architectural constraints (avoid adding to god object)
3. Implements following mandatory patterns
4. Creates new module to avoid god object growth

### Example 3: Answering "Why" Question

**User**: "Why is authentication implemented with JWT instead of sessions?"

**Claude automatically runs**:
```bash
python3 skill.py context src/auth \
  --question "Why JWT instead of sessions"

# Output:
# - Historical context from commit messages
# - Design decision from 2024-03: "Use JWT for stateless auth"
# - Similar issues mentioning "session", "jwt", "stateless"
```

## Critical Warnings

When the skill outputs these warnings, STOP and inform the user:

### ⚠️ REVERTS DETECTED
**Meaning**: Previous attempts to modify this code failed and were reverted
**Action**:
- Escalate risk to HIGH/CRITICAL
- Require tests BEFORE fix
- Recommend feature flag
- Get senior engineer review
- Ask user if they want to proceed with caution

### ⚠️ GOD OBJECT
**Meaning**: File >2000 lines OR >10 files depend on it
**Action**:
- Warn about blast radius
- Test all dependents after changes
- Consider refactoring before adding features
- Recommend creating new module instead

### ⚠️ HOTSPOT
**Meaning**: Function modified >20 times in history
**Action**:
- Mark as bug-prone
- Require comprehensive tests
- Consider refactoring/simplification
- Extra scrutiny in code review

### ⚠️ PATTERN VIOLATIONS
**Meaning**: File missing patterns used in >50% of similar files
**Action**:
- Enforce pattern compliance in new code
- List mandatory patterns to user
- Reject code that violates established conventions

## Output Interpretation

### Risk Levels

| Score | Level | Actions Required |
|-------|-------|------------------|
| 0-25 | LOW | Standard testing, normal review |
| 26-50 | MEDIUM | Standard testing, careful review |
| 51-75 | HIGH | Feature flag, extra tests, senior review |
| 76-100 | CRITICAL | Feature flag, comprehensive tests, senior + principal review, gradual rollout |

### Pattern Adoption Thresholds

| Adoption | Meaning | Action |
|----------|---------|--------|
| >80% | Strong consensus | MUST use in all new code |
| 50-80% | Established pattern | SHOULD use in new code |
| 20-50% | Emerging pattern | Consider using |
| <20% | Rare/experimental | Optional |

## Supporting Documentation

- **[README.md](README.md)**: Complete feature documentation
- **[FINAL_SYSTEM_OVERVIEW.md](FINAL_SYSTEM_OVERVIEW.md)**: Architecture and value proposition
- **[CLAUDE_CODE_INTEGRATION.md](CLAUDE_CODE_INTEGRATION.md)**: Integration guide for Claude Code
- **[FILE_LEVEL_ANALYSIS.md](FILE_LEVEL_ANALYSIS.md)**: Function evolution details
- **[COUNTERFACTUAL_VALUE.md](COUNTERFACTUAL_VALUE.md)**: Proof of value methodology

## Performance

- Small folders (<100 commits): <1 second
- Medium folders (100-500 commits): 2-5 seconds
- Large folders (>500 commits): 5-10 seconds
- Full repo discovery: 10-30 seconds

All commands default to analyzing last 500 commits for performance.

## Dependencies

- Python 3.8+
- Git (for history analysis)
- Repository must be a git repository

## Troubleshooting

**No commits found**: Ensure you're in a git repository and folder path is correct

**Slow performance**: Use `--format compact` or reduce commit depth

**Function not found**: Function detection uses regex, may not find all functions in complex syntax

---

**Remember**: This skill's value is PROACTIVE use BEFORE changes, not reactive use AFTER problems occur. Always run smart analysis before modifying code!
