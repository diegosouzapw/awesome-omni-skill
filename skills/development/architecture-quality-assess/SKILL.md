---
name: architecture-quality-assess
description: "Converted Claude skill for architecture-quality-assess. Use when Codex should run the converted architecture-quality-assess workflow."
---

# Architecture Quality Assess

Converted Claude skill workflow for Codex/OpenAI use.

## Source

Converted from `skills/architecture-quality-assess/SKILL.md`.

## Bundled Resources

Supporting files copied from the Claude source:

- `assets/.architecture-assess-cache`
- `references/README.md`
- `references/USAGE_GUIDE.md`
- `references/architecture-assessment.md`
- `assets/examples`
- `assets/lib`
- `references/phases-4-5-6-complete.md`
- `scripts`
- `assets/tests`

## Converted Instructions

The content below was adapted from the Claude source. Rewrite tool and runtime assumptions as needed when they refer to Claude-only features.

# Architecture Quality Assessment Skill

**Version**: 1.0.0
**Status**: Active
**Category**: Code Analysis & Quality

---

## Overview

The Architecture Quality Assessment skill performs deep static analysis of codebases to detect architecture quality issues, measure technical debt, identify drift from intended patterns, and generate actionable refactoring recommendations.

### Purpose

This skill fills the gap between documentation analysis (handled by `document-hub-analyze`) and code architecture quality analysis. It provides automated assessment of:

- **Layer Separation**: Clean Architecture validation (Presentation/Business/Data layers)
- **SOLID Principles**: All 5 principles compliance checking
- **Design Patterns**: Repository, Factory, Strategy, Singleton detection
- **Dependency Management**: Coupling metrics (FAN-IN/FAN-OUT), circular dependencies
- **Code Organization**: File structure, naming conventions, module cohesion

### Key Features

✅ **Multi-Language Support** - Detects and analyzes Python, JavaScript/TypeScript, Next.js, React, Vue, Node.js projects
✅ **Framework Detection** - Identifies FastAPI, Django, Flask, Express, NestJS, and more
✅ **SOLID Analysis** - Checks all 5 principles with violation detection
✅ **Coupling Metrics** - Calculates FAN-IN/FAN-OUT scores for modules
✅ **Circular Dependencies** - Detects and reports dependency cycles
✅ **Actionable Output** - Generates markdown reports and refactoring task lists
✅ **Memory Bank Integration** - Reads systemPatterns.md to detect drift from documented architecture

---

## Installation

This skill is installed by default in the Claude CLI skills directory:

```bash
~/.codex/skills$architecture-quality-assess/
```

### Requirements

**Core Dependencies** (No installation required):
- Python 3.8+
- Standard library: `ast`, `pathlib`, `json`, `re`, `dataclasses`

**Optional Dependencies** (Recommended):
- `networkx` - Enhanced graph algorithms for circular dependency detection
- `tree-sitter` - Advanced multi-language parsing (fallback available)

**Install optional dependencies:**
```bash
pip install networkx tree-sitter
```

---

## Usage

### Basic Invocation

```bash
# Analyze current project
$architecture-quality-assess

# Analyze specific project
$architecture-quality-assess /path/to/project

# Analyze with verbose output
$architecture-quality-assess /path/to/project --verbose

# JSON output for CI/CD
$architecture-quality-assess /path/to/project --format json
```

### Command-Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `path` | Project root directory to analyze | Current directory |
| `--verbose` | Enable detailed progress output | Off |
| `--format` | Output format: `markdown` or `json` | `markdown` |
| `--output` | Output file path | `architecture-assessment.md` |
| `--incremental` | Only analyze changed files (git-based) | Off |
| `--cache` | Enable file parsing cache | On |
| `--severity` | Minimum severity to report: `critical`, `high`, `medium`, `low` | `low` |

### Example Workflows

**Basic Project Assessment:**
```bash
# Analyze current project and generate report
$architecture-quality-assess

# View report
cat architecture-assessment.md
```

**CI/CD Integration:**
```bash
# Run analysis and fail build if critical issues found
$architecture-quality-assess --format json --severity critical > assessment.json

# Check exit code
if [ $? -ne 0 ]; then
  echo "Critical architecture violations detected"
  exit 1
fi
```

**Incremental Analysis:**
```bash
# Only analyze files changed since last commit
$architecture-quality-assess --incremental

# Review violations in changed code only
```

---

## Features

### 1. Project Type Detection

Automatically detects project type and framework:

**Supported Project Types:**
- **Next.js** - App Router and Pages Router
- **Python** - FastAPI, Django, Flask, general Python
- **Node.js** - Express, NestJS, general Node
- **React** - Vite, Create React App
- **Vue** - Nuxt, Vue CLI
- **Angular** - Angular CLI

**Detection Strategy:**
1. Analyzes package.json, requirements.txt, pyproject.toml
2. Checks for framework-specific patterns (next.config.js, django settings)
3. Examines directory structure (app/, pages/, src/)
4. Validates configuration files

**Example Output:**
```markdown
## Project Detection

**Project Type**: Next.js (App Router)
**Framework Version**: 14.0.3
**Architecture Pattern**: Three-tier (Route Handlers → Service Layer → Data Layer)
```

---

### 2. Layer Separation Analysis

Validates Clean Architecture layer separation:

**Three-Tier Architecture:**
1. **Presentation Layer** - Routes, controllers, UI components
2. **Business Layer** - Services, domain logic, use cases
3. **Data Layer** - Database access, external APIs, repositories

**Violation Detection:**
- SQL queries in presentation layer (route handlers)
- Direct database access from UI components
- Business logic in data access objects
- Cross-layer tight coupling

**Example Violations:**
```markdown
### Layer Separation Violations (3)

**CRITICAL**: SQL in API Route
- File: `src/app/api/users/route.ts`
- Line: 12
- Issue: Direct SQL query in route handler
- Recommendation: Move to service layer

**HIGH**: Business Logic in Database Layer
- File: `src/lib/db/user-repository.ts`
- Line: 45
- Issue: User validation logic in repository
- Recommendation: Move to service layer
```

---

### 3. SOLID Principles Analysis

Checks compliance with all 5 SOLID principles:

#### Single Responsibility Principle (SRP)
Detects classes/modules with multiple responsibilities.

**Violations:**
- Classes with > 500 LOC
- Modules with > 10 public methods
- Files doing both business logic + data access

#### Open/Closed Principle (OCP)
Detects hardcoded conditional logic that should use polymorphism.

**Violations:**
- Large if/else chains (> 5 branches)
- Switch statements on type fields
- Repeated instanceof checks

#### Liskov Substitution Principle (LSP)
Detects inheritance violations.

**Violations:**
- Subclasses throwing NotImplementedError
- Overridden methods changing behavior contracts
- Subclasses requiring more preconditions

#### Interface Segregation Principle (ISP)
Detects overly large interfaces.

**Violations:**
- Interfaces with > 10 methods
- Implementations with empty stub methods
- Fat interfaces forcing unnecessary dependencies

#### Dependency Inversion Principle (DIP)
Detects direct dependencies on concrete implementations.

**Violations:**
- Direct imports of database clients in business logic
- Hardcoded external API URLs
- Tight coupling to specific libraries

**Example Output:**
```markdown
### SOLID Principles Compliance

**Overall Score**: 72/100 (Medium)

**Single Responsibility**: 65/100 (4 violations)
**Open/Closed**: 80/100 (2 violations)
**Liskov Substitution**: 90/100 (1 violation)
**Interface Segregation**: 75/100 (3 violations)
**Dependency Inversion**: 50/100 (6 violations) ⚠️
```

---

### 4. Design Pattern Detection

Identifies common design patterns and anti-patterns:

**Recognized Patterns:**
- **Repository Pattern** - Data access abstraction
- **Factory Pattern** - Object creation delegation
- **Strategy Pattern** - Algorithm encapsulation
- **Singleton Pattern** - Single instance management
- **Observer Pattern** - Event-driven architecture
- **Dependency Injection** - Inversion of Control

**Anti-Patterns Detected:**
- **God Object** - Classes with excessive responsibilities
- **Spaghetti Code** - Unstructured control flow
- **Tight Coupling** - Excessive inter-module dependencies
- **Magic Numbers** - Hardcoded values without constants

**Example Output:**
```markdown
### Design Patterns

**Detected Patterns (5)**:
✅ Repository Pattern - `lib/repositories/*`
✅ Factory Pattern - `lib/factories/user-factory.ts`
✅ Strategy Pattern - `lib/strategies/auth-strategy.ts`
✅ Dependency Injection - Constructor-based DI throughout

**Anti-Patterns (3)**:
❌ God Object - `src/lib/user-manager.ts` (1,200 LOC, 25 methods)
❌ Tight Coupling - `src/api/orders.ts` → 15 direct dependencies
❌ Magic Numbers - `src/lib/pricing.ts` (8 hardcoded constants)
```

---

### 5. Dependency Management

Analyzes module dependencies and coupling:

**Coupling Metrics:**
- **FAN-IN** - Number of modules depending on this module (higher = more central)
- **FAN-OUT** - Number of modules this module depends on (higher = more coupled)
- **Instability** - FAN-OUT / (FAN-IN + FAN-OUT) (0 = stable, 1 = unstable)

**Circular Dependencies:**
Uses graph algorithms to detect dependency cycles.

**Example Output:**
```markdown
### Coupling Metrics

**Most Coupled Modules** (FAN-OUT > 10):
1. `src/lib/auth-service.ts` - FAN-OUT: 18 (❌ too high)
2. `src/lib/user-service.ts` - FAN-OUT: 15 (⚠️ high)
3. `src/lib/order-service.ts` - FAN-OUT: 12 (⚠️ high)

**Circular Dependencies (2)**:
1. `src/lib/user-service.ts` ↔️ `src/lib/auth-service.ts`
2. `src/lib/order-service.ts` → `src/lib/product-service.ts` → `src/lib/inventory-service.ts` → `src/lib/order-service.ts`

**Recommendation**: Break cycles using interface abstractions or event-driven patterns.
```

---

### 6. Code Organization

Validates file structure and naming conventions:

**Checks:**
- File naming consistency (kebab-case, camelCase, PascalCase)
- Directory structure alignment with architecture patterns
- Module size (files > 500 LOC flagged)
- Unused imports/exports
- Public API surface area

**Example Output:**
```markdown
### Code Organization

**File Structure**: ✅ Follows Next.js App Router conventions
**Naming Consistency**: ⚠️ Mixed (kebab-case and camelCase)
**Module Sizes**: ⚠️ 3 files > 500 LOC

**Recommendations**:
- Standardize on kebab-case for file names
- Split large modules:
  - `src/lib/user-service.ts` (842 LOC)
  - `src/lib/order-service.ts` (654 LOC)
  - `src/lib/product-service.ts` (521 LOC)
```

---

### 7. Drift Detection (Memory Bank Integration)

Compares actual architecture vs documented architecture:

**Reads from Memory Bank:**
- `memory-bank/systemPatterns.md` - Documented architecture patterns
- `memory-bank/systemArchitecture.md` - System design decisions

**Detects:**
- Undocumented components (code exists, not in docs)
- Deviation from documented patterns (docs say X, code does Y)
- Deprecated patterns still in use
- New patterns not yet documented

**Example Output:**
```markdown
### Drift from Documented Architecture

**Drift Score**: 23/100 (Low drift = good)

**New Components (Undocumented)**:
- `src/lib/notification-service.ts` (added 2 weeks ago)
- `src/api/webhooks/` (new feature)

**Pattern Deviations**:
- Documentation specifies Repository Pattern
- Found: 8 files with direct database access (bypassing repositories)

**Recommendations**:
1. Document new notification service in systemPatterns.md
2. Update 8 files to use repository pattern
3. Archive deprecated patterns from documentation
```

---

## Report Output

### Markdown Report Structure

```markdown
# Architecture Quality Assessment Report

**Generated**: 2026-02-07 15:45:32
**Project**: my-nextjs-app
**Path**: /home/user/projects/my-nextjs-app

---

## Executive Summary

**Overall Score**: 76/100 (Good)
**Critical Issues**: 2
**High Priority**: 8
**Medium Priority**: 15
**Low Priority**: 23

---

## 1. Project Overview
[Project type, framework, version]

## 2. Layer Separation Analysis
[Violations, recommendations]

## 3. SOLID Principles
[Per-principle scores, violations]

## 4. Design Patterns
[Patterns found, anti-patterns]

## 5. Dependency Management
[Coupling metrics, circular dependencies]

## 6. Code Organization
[File structure, naming, module sizes]

## 7. Drift Detection
[Comparison with documented architecture]

## 8. Recommended Actions
[Prioritized refactoring task list]

---

## Appendix: Detailed Violations
[Full list with file paths, line numbers]
```

### JSON Report Structure

```json
{
  "metadata": {
    "generated_at": "2026-02-07T15:45:32Z",
    "project_name": "my-nextjs-app",
    "project_path": "/home/user/projects/my-nextjs-app",
    "analysis_duration_seconds": 42.3
  },
  "summary": {
    "overall_score": 76,
    "critical_count": 2,
    "high_count": 8,
    "medium_count": 15,
    "low_count": 23
  },
  "project_detection": {
    "type": "nextjs",
    "framework_version": "14.0.3",
    "architecture_pattern": "three-tier"
  },
  "violations": [
    {
      "id": "LSV-001",
      "category": "layer_separation",
      "severity": "critical",
      "title": "SQL in API Route",
      "file": "src/app/api/users/route.ts",
      "line": 12,
      "description": "Direct SQL query in route handler",
      "recommendation": "Move database access to service layer",
      "code_snippet": "const users = await db.query('SELECT * FROM users');"
    }
  ],
  "metrics": {
    "solid_compliance": {
      "overall": 72,
      "srp": 65,
      "ocp": 80,
      "lsp": 90,
      "isp": 75,
      "dip": 50
    },
    "coupling": {
      "highest_fan_out": {
        "module": "src/lib/auth-service.ts",
        "fan_out": 18
      },
      "circular_dependencies_count": 2
    }
  },
  "recommended_actions": [
    {
      "priority": "P0",
      "category": "layer_separation",
      "title": "Move SQL to Service Layer",
      "files": ["src/app/api/users/route.ts"],
      "estimated_effort": "1 hour"
    }
  ]
}
```

---

## Task List Generation

Automatically generates refactoring task list compatible with `$start-phase-execute`:

**Output File**: `architecture-refactoring-tasks.md`

**Structure:**
```markdown
# Architecture Refactoring Tasks

## Phase 1: Critical Fixes (Priority P0)

### Task 1: Move SQL to Service Layer
**File**: src/app/api/users/route.ts
**Issue**: Direct database access in route handler
**Action**: Create UserService with getUserList() method
**Verification**: Route handler only calls service method
**Estimated Time**: 1 hour

### Task 2: Break Circular Dependency
**Files**: src/lib/user-service.ts ↔️ src/lib/auth-service.ts
**Issue**: Circular dependency prevents clean testing
**Action**: Extract shared interface to src/types/auth-types.ts
**Verification**: No circular imports remain
**Estimated Time**: 2 hours

## Phase 2: High Priority Refactoring (Priority P1)

[...]

## Phase 3: Medium Priority Improvements (Priority P2)

[...]
```

**Integration:**
```bash
# Generate assessment and task list
$architecture-quality-assess

# Execute refactoring tasks
/start-phase execute architecture-refactoring-tasks.md
```

---

## Integration with Other Skills

### memory-bank Integration

**Reads:**
- `memory-bank/systemPatterns.md` - Expected architecture patterns
- `memory-bank/systemArchitecture.md` - System design

**Writes (optional):**
- Updates activeContext.md with discovered issues
- Flags outdated patterns in systemPatterns.md

**Workflow:**
```bash
# Run assessment
$architecture-quality-assess

# Sync findings to Memory Bank
$memorybank-sync
```

---

### document-hub Integration

**Complementary Analysis:**
- `document-hub-analyze` - Documentation ↔️ Code drift
- `architecture-quality-assess` - Code quality & architecture

**Combined Workflow:**
```bash
# Check documentation alignment
$document-hub-analyze

# Check architecture quality
$architecture-quality-assess

# Fix both documentation AND code quality issues
```

---

### pm-db Integration

**Tracking:**
- Assessment runs tracked as PM-DB jobs
- Task list tasks linked to PM-DB phases
- Refactoring progress monitored

**Workflow:**
```bash
# Run assessment and track in PM-DB
$architecture-quality-assess

# Import generated task list
$pm-db import architecture-refactoring-tasks.md

# Execute with tracking
/start-phase execute architecture-refactoring-tasks.md
```

---

## Performance

### Analysis Speed

**Small Project** (<100 files):
- Analysis time: <10 seconds
- Memory usage: ~50 MB

**Medium Project** (100-1000 files):
- Analysis time: 30-120 seconds
- Memory usage: ~200 MB

**Large Project** (1000-5000 files):
- Analysis time: 2-10 minutes
- Memory usage: ~500 MB

### Optimization Strategies

**Caching:**
- File parsing results cached (80% speedup on re-runs)
- AST analysis cached per file hash
- Dependency graph cached

**Incremental Analysis:**
- Git integration detects changed files
- Only re-analyzes modified modules
- 70%+ faster than full analysis

**Parallel Processing:**
- Multiple files parsed concurrently
- CPU-bound analysis distributed
- Scales with available cores

---

## Configuration

### Configuration File

**Location**: `.architecture-assess.json` (project root)

**Example:**
```json
{
  "exclude_paths": [
    "node_modules/",
    "dist/",
    "build/",
    ".next/",
    "__pycache__/",
    "*.test.ts",
    "*.spec.js"
  ],
  "severity_thresholds": {
    "critical": 0,
    "high": 5,
    "medium": 20
  },
  "rules": {
    "max_fan_out": 15,
    "max_file_loc": 500,
    "max_method_count": 10,
    "allow_sql_in_routes": false,
    "require_repository_pattern": true
  },
  "output": {
    "format": "markdown",
    "path": "docs/architecture-assessment.md",
    "generate_task_list": true
  },
  "integrations": {
    "memory_bank": {
      "enabled": true,
      "check_drift": true
    },
    "ci_cd": {
      "fail_on_critical": true,
      "comment_on_pr": true
    }
  }
}
```

---

## CI/CD Integration

### GitHub Actions

```yaml
name: Architecture Quality Gate

on: [pull_request]

jobs:
  architecture-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Install Claude CLI
        run: |
          curl -sS https://claude.ai/cli/install.sh | bash
          claude auth ${{ secrets.CLAUDE_API_KEY }}

      - name: Run Architecture Assessment
        run: |
          claude $architecture-quality-assess --format json --severity critical > assessment.json

      - name: Check Results
        run: |
          CRITICAL_COUNT=$(jq '.summary.critical_count' assessment.json)
          if [ "$CRITICAL_COUNT" -gt 0 ]; then
            echo "Critical architecture violations detected"
            exit 1
          fi

      - name: Upload Report
        uses: actions/upload-artifact@v2
        with:
          name: architecture-assessment
          path: assessment.json
```

---

## Troubleshooting

### Common Issues

**1. "No project type detected"**
- **Cause**: Missing package.json or requirements.txt
- **Fix**: Ensure project root has language-specific manifest file

**2. "Parser error on file X"**
- **Cause**: Syntax error in source file
- **Fix**: File skipped automatically, check --verbose for details

**3. "Analysis too slow"**
- **Cause**: Large project without caching
- **Fix**: Enable --cache flag, exclude test files

**4. "Memory Bank integration failed"**
- **Cause**: Memory Bank not initialized
- **Fix**: Run /memorybank init first

---

## Best Practices

### When to Run Analysis

**✅ Recommended:**
- Before major refactoring efforts
- After adding new major features
- Monthly architecture health checks
- As pre-merge quality gate in CI/CD

**❌ Not Recommended:**
- Every single commit (use incremental mode)
- On generated/build files
- On external libraries (exclude them)

### Interpreting Results

**Score Interpretation:**
- **90-100**: Excellent architecture
- **75-89**: Good, minor improvements needed
- **60-74**: Fair, moderate refactoring recommended
- **Below 60**: Poor, significant refactoring required

**Priority Guidance:**
- **Critical**: Fix immediately (blocks development)
- **High**: Fix within 1 sprint
- **Medium**: Plan for next quarter
- **Low**: Nice-to-have improvements

---

## Frequently Asked Questions

**Q: Does this modify my code?**
A: No, this is analysis-only. It generates reports and task lists, but never edits code.

**Q: What languages are supported?**
A: Python, JavaScript, TypeScript, JSX, TSX. More languages planned.

**Q: How accurate is the SOLID analysis?**
A: ~85% accuracy. Manual review recommended for borderline cases.

**Q: Can I customize rules?**
A: Yes, via `.architecture-assess.json` configuration file.

**Q: Does it work with monorepos?**
A: Yes, run analysis per sub-project or use --path to target specific packages.

**Q: How does it compare to SonarQube?**
A: Complements SonarQube. SonarQube focuses on bugs/security, this focuses on architecture patterns.

---

## Changelog

### Version 1.0.0 (2026-02-07)
- Initial release
- Multi-language project detection
- SOLID principles analysis
- Layer separation validation
- Coupling metrics calculation
- Circular dependency detection
- Markdown and JSON report output
- Task list generation
- Memory Bank integration

---

## License

MIT License - Part of Claude CLI Skills

---

## Support

**Documentation**: ~/.codex/skills$architecture-quality-assess/README.md
**Issues**: Report via `/help` in Claude CLI
**Contributing**: This is a built-in skill, customization via configuration files recommended

---

**Last Updated**: 2026-02-07
**Maintainer**: Codex Team
