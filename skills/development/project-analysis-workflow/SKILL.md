---
name: analyze-project
description: "Use when starting work on an unfamiliar project or needing to understand a codebase - performs comprehensive analysis discovering architecture, patterns, dependencies, testing coverage, and improvement opportunities. Do NOT use on projects you already know well or for targeted questions about specific files - use direct exploration instead for focused queries."
inputs:
  - from: any
    field: focus
    required: false
outputs:
  - field: analysis_report
    type: file_path
  - field: analysis_json
    type: file_path
next_skills:
  - pop-project-setup
  - pop-writing-plans
workflow:
  id: analyze-project
  name: Project Analysis Workflow
  version: 1
  description: Comprehensive codebase analysis with progressive disclosure
  steps:
    - id: detect_project
      description: Detect project type and basic structure
      type: agent
      agent: code-explorer
      next: depth_decision
    - id: depth_decision
      description: Choose analysis depth
      type: user_decision
      question: "What level of analysis do you need?"
      header: "Depth"
      options:
        - id: quick
          label: "Quick"
          description: "5-10 line summary, ~30 seconds"
          next: quick_analysis
        - id: standard
          label: "Standard"
          description: "Full analysis with recommendations"
          next: focus_decision
        - id: deep
          label: "Deep dive"
          description: "Exhaustive analysis with metrics"
          next: focus_decision
      next_map:
        quick: quick_analysis
        standard: focus_decision
        deep: focus_decision
    - id: quick_analysis
      description: Generate quick summary
      type: skill
      skill: pop-auto-docs
      next: complete
    - id: focus_decision
      description: Choose analysis focus area
      type: user_decision
      question: "Which area should we focus on?"
      header: "Focus"
      options:
        - id: architecture
          label: "Architecture"
          description: "Structure, patterns, entry points"
          next: run_analysis
        - id: quality
          label: "Quality"
          description: "Tests, linting, TypeScript"
          next: run_analysis
        - id: dependencies
          label: "Dependencies"
          description: "Packages, vulnerabilities"
          next: run_analysis
        - id: all
          label: "All areas"
          description: "Complete analysis"
          next: run_analysis
      next_map:
        architecture: run_analysis
        quality: run_analysis
        dependencies: run_analysis
        all: run_analysis
    - id: run_analysis
      description: Execute analysis based on selections
      type: spawn_agents
      agents:
        - type: code-explorer
          task: "Map project structure and architecture patterns"
        - type: performance-optimizer
          task: "Identify performance issues and bottlenecks"
      wait_for: all
      next: output_decision
    - id: output_decision
      description: Choose output format
      type: user_decision
      question: "How should we present the analysis?"
      header: "Output"
      options:
        - id: markdown
          label: "Markdown"
          description: "Human-readable report"
          next: generate_report
        - id: json
          label: "JSON"
          description: "Machine-readable for tooling"
          next: generate_json
        - id: both
          label: "Both"
          description: "Markdown + JSON files"
          next: generate_both
      next_map:
        markdown: generate_report
        json: generate_json
        both: generate_both
    - id: generate_report
      description: Generate markdown analysis report
      type: skill
      skill: pop-auto-docs
      next: next_step_decision
    - id: generate_json
      description: Generate JSON analysis output
      type: skill
      skill: pop-auto-docs
      next: next_step_decision
    - id: generate_both
      description: Generate both output formats
      type: skill
      skill: pop-auto-docs
      next: next_step_decision
    - id: next_step_decision
      description: Decide what to do after analysis
      type: user_decision
      question: "Analysis complete. What's next?"
      header: "Next Step"
      options:
        - id: generate
          label: "Generate MCP"
          description: "Create project-specific tools"
          next: generate_mcp
        - id: plan
          label: "Create plan"
          description: "Plan improvements"
          next: create_plan
        - id: done
          label: "Done"
          description: "Stop here"
          next: complete
      next_map:
        generate: generate_mcp
        plan: create_plan
        done: complete
    - id: generate_mcp
      description: Generate project-specific MCP server
      type: skill
      skill: pop-generate-mcp
      next: complete
    - id: create_plan
      description: Create implementation plan for improvements
      type: skill
      skill: pop-writing-plans
      next: complete
    - id: complete
      description: Analysis workflow finished
      type: terminal
---

# Analyze Project

## Overview

Perform deep analysis of a codebase to understand its architecture, patterns, dependencies, and opportunities for improvement.

**Core principle:** Understand before changing. Map before navigating.

**Trigger:** `/popkit:project analyze` command or when starting work on unfamiliar project

## Arguments

| Flag             | Description                                                                 |
| ---------------- | --------------------------------------------------------------------------- |
| `--json`         | Output structured JSON instead of markdown, save to `.claude/analysis.json` |
| `--quick`        | Quick summary only (5-10 lines)                                             |
| `--focus <area>` | Focus analysis: `arch`, `deps`, `quality`, `patterns`                       |

## JSON Output Mode

When `--json` flag is provided:

1. **Output Format**: Structured JSON matching `output-styles/schemas/project-analysis.schema.json`
2. **Save Location**: `.claude/analysis.json`
3. **Purpose**: Machine-readable output for skill generators and MCP generators

### JSON Output Process

```python
import sys
import json
from datetime import datetime
from pathlib import Path

# Add pattern detector to path
# No longer needed - install popkit-shared instead
from pattern_detector import analyze_project, detect_frameworks

# Detect patterns
project_dir = Path.cwd()
patterns = analyze_project(project_dir)
frameworks = detect_frameworks(project_dir)

# Build output
output = {
    "project_name": project_dir.name,
    "project_type": frameworks[0].name if frameworks else "unknown",
    "analyzed_at": datetime.now().isoformat(),
    "frameworks": [
        {"name": p.name, "confidence": p.confidence, "version": ""}
        for p in frameworks
    ],
    "patterns": [
        {
            "name": p.name,
            "category": p.category,
            "confidence": p.confidence,
            "examples": p.examples,
            "description": p.description
        }
        for p in patterns if p.category != "framework"
    ],
    "recommended_skills": [],  # Populated based on patterns
    "recommended_agents": [],  # Populated based on analysis
    "commands": {},  # Extracted from package.json
    "quality_metrics": {}  # TypeScript, linting, etc.
}

# Save to .claude/analysis.json
claude_dir = project_dir / ".claude"
claude_dir.mkdir(exist_ok=True)
(claude_dir / "analysis.json").write_text(json.dumps(output, indent=2))
print(json.dumps(output, indent=2))
```

### Skill/Agent Recommendation Logic

Based on detected patterns, recommend:

| Pattern                | Recommended Skill        | Priority |
| ---------------------- | ------------------------ | -------- |
| nextjs + vercel-config | `project:deploy`         | high     |
| prisma OR drizzle      | `project:db-migrate`     | high     |
| supabase               | `project:supabase-sync`  | medium   |
| docker-compose         | `project:docker-dev`     | medium   |
| feature-flags          | `project:feature-toggle` | low      |

| Pattern                     | Recommended Agent        |
| --------------------------- | ------------------------ |
| Large codebase (>100 files) | `performance-optimizer`  |
| React/Vue components        | `accessibility-guardian` |
| API routes                  | `api-designer`           |
| Security-sensitive          | `security-auditor`       |
| Low test coverage           | `test-writer-fixer`      |

## Analysis Areas

### 1. Project Structure

```bash
# Map directory structure
find . -type d -name "node_modules" -prune -o -type d -print | head -50

# Find main entry points
ls index.* main.* app.* src/index.* src/main.* 2>/dev/null

# Count files by type
find . -name "node_modules" -prune -o -type f -print | \
  sed 's/.*\.//' | sort | uniq -c | sort -rn | head -20
```

### 2. Technology Stack

**Detect package managers:**

```bash
ls package.json yarn.lock pnpm-lock.yaml Cargo.toml pyproject.toml go.mod 2>/dev/null
```

**Detect frameworks:**

- Next.js: `next.config.*`, `app/` or `pages/`
- React: `react` in dependencies
- Vue: `vue.config.*`
- Express: `express` in dependencies
- FastAPI: `fastapi` in dependencies
- Rust: `Cargo.toml`

**Detect databases:**

- Supabase: `@supabase/supabase-js`
- Prisma: `prisma/schema.prisma`
- MongoDB: `mongoose`
- PostgreSQL: `pg` or `postgres`

### 3. Architecture Patterns

**Frontend:**

- Component structure (atomic design, feature-based, etc.)
- State management (Redux, Zustand, Context)
- Routing patterns

**Backend:**

- API design (REST, GraphQL, tRPC)
- Service layer organization
- Database access patterns

**Common:**

- Error handling patterns
- Logging approach
- Configuration management

### 4. Code Quality

```bash
# Check for linting config
ls .eslintrc* eslint.config.* .prettierrc* biome.json 2>/dev/null

# Check TypeScript config
ls tsconfig.json 2>/dev/null && grep "strict" tsconfig.json

# Find TODO/FIXME comments
grep -r "TODO\|FIXME" --include="*.ts" --include="*.tsx" --include="*.py" . | wc -l
```

### 5. Testing Coverage

```bash
# Find test files
find . -name "*.test.*" -o -name "*.spec.*" -o -name "test_*" 2>/dev/null | wc -l

# Check test config
ls jest.config.* vitest.config.* pytest.ini 2>/dev/null

# Find coverage reports
ls coverage/ .coverage htmlcov/ 2>/dev/null
```

### 6. Dependencies

```bash
# Count dependencies
jq '.dependencies | length' package.json 2>/dev/null
jq '.devDependencies | length' package.json 2>/dev/null

# Check for outdated
npm outdated 2>/dev/null | head -10

# Check for vulnerabilities
npm audit --json 2>/dev/null | jq '.metadata.vulnerabilities'
```

### 7. CI/CD and DevOps

```bash
# Find CI config
ls .github/workflows/*.yml .gitlab-ci.yml Jenkinsfile .circleci/config.yml 2>/dev/null

# Find Docker
ls Dockerfile docker-compose.yml 2>/dev/null

# Find deployment config
ls vercel.json netlify.toml fly.toml 2>/dev/null
```

## Output Format

```markdown
# [Project Name] Analysis Report

## Summary

- **Type**: [Web App / API / CLI / Library]
- **Stack**: [Primary technologies]
- **Size**: [Files, Lines of code]
- **Health**: [Good / Needs attention / Critical issues]

## Technology Stack

### Frontend

- Framework: [Next.js 14 / React / Vue / etc.]
- Styling: [Tailwind / styled-components / etc.]
- State: [Redux / Zustand / Context]

### Backend

- Runtime: [Node.js / Python / Rust / Go]
- Framework: [Express / FastAPI / Actix]
- Database: [PostgreSQL / MongoDB / etc.]

### DevOps

- CI/CD: [GitHub Actions / GitLab CI / etc.]
- Deployment: [Vercel / AWS / etc.]
- Container: [Docker / etc.]

## Architecture

### Directory Structure

\`\`\`
[Tree output of main directories]
\`\`\`

### Key Patterns

- [Pattern 1]: [Where used]
- [Pattern 2]: [Where used]

### Entry Points

- Main: `[path]`
- API: `[path]`
- Tests: `[path]`

## Code Quality

| Metric            | Value                | Status       |
| ----------------- | -------------------- | ------------ |
| Linting           | [Configured/Missing] | [OK/Warning] |
| TypeScript Strict | [Yes/No]             | [OK/Warning] |
| Test Coverage     | [X%]                 | [OK/Warning] |
| TODO Comments     | [N]                  | [OK/Warning] |

## Dependencies

### Production ([N] packages)

Top 5:

- [package]: [version]

### Security

- Vulnerabilities: [Low: X, Medium: Y, High: Z]

## Recommendations

### Critical

1. [Issue requiring immediate attention]

### High Priority

1. [Important improvement]

### Nice to Have

1. [Enhancement suggestion]

## Agent Opportunities

Based on analysis, these agents would be valuable:

- [agent-name]: [why]
- [agent-name]: [why]

## Next Steps

1. Run `/generate-mcp` to create project-specific tools
2. Run `/generate-skills` to capture discovered patterns
3. Run `/setup-precommit` to configure quality gates
```

## Integration

**Called by:**

- Manual `/analyze-project` command
- After `/init-project`

**Informs:**

- **/generate-mcp** - What tools to create
- **/generate-skills** - What patterns to capture
- **/setup-precommit** - What checks to configure
