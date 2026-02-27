---
name: project-init
description: Automatically detects new project initialization, collaborates with user on project planning, and sets up the appropriate tech stack with matching skills and agents. Use when starting a new project, creating a new repository, or working in an empty/minimal directory that needs project structure.
type: domain
enforcement: suggest
priority: high
keywords: new project, start project, initialize project, project setup, empty directory, tech stack
patterns: (start|create|init|setup|initialize).*?(project|repo|repository), new.*?(project|repo)
---

# Project Initialization

## Overview

This skill automates project setup through collaborative planning. It detects when you're starting a new project, identifies the tech stack, presents skill and agent recommendations, collaborates with the user on the project plan, then initializes the appropriate project structure with matching skills and agents.

## When to Activate

Activate this skill when:
- Working in an empty or near-empty directory
- User explicitly mentions "new project", "start a project", or "initialize"
- No existing project files detected (go.mod, package.json, Cargo.toml, pyproject.toml, etc.)
- User asks to set up a specific tech stack

Do NOT activate for:
- Existing projects with established structure
- Single-file scripts or experiments
- Documentation-only work

## Workflow

### Phase 1: Detection

Check for new project indicators:

1. **Directory Analysis**
   ```bash
   # Check if directory is empty or minimal
   ls -la

   # Look for existing project files
   find . -maxdepth 2 -name "go.mod" -o -name "package.json" -o -name "Cargo.toml" -o -name "pyproject.toml" -o -name "pom.xml"
   ```

2. **Detection Criteria**
   - Empty directory OR
   - Only contains: README.md, .git, .gitignore, LICENSE OR
   - User explicitly requests new project setup

### Phase 2: Stack Identification

Determine the tech stack through:

1. **Explicit user statement** - "create a Go project", "new Node.js app"
2. **Context clues** - user mentions specific frameworks, tools, or languages
3. **Ask the user** if unclear

Common stacks:
- **Go** - Backend services, CLI tools
- **Node.js** - Web apps, APIs, full-stack
- **Python** - Data science, ML, scripting, web apps
- **Rust** - Systems programming, performance-critical apps
- **TypeScript** - Frontend, full-stack web apps

### Phase 3: Planning & Skill Recommendations

**CRITICAL:** Before initializing anything, collaborate with the user on the project plan.

1. **Present stack-specific recommendations**

   Based on identified stack, suggest:
   - **Core skills** - Essential for the stack (e.g., go-tdd for Go)
   - **Workflow skills** - Development workflow automation (linting, formatting, CI/CD)
   - **Domain-specific skills** - Based on project type (web-api, cli-tool, data-pipeline, etc.)
   - **Agent suggestions** - Specialized agents that could help (testing agent, documentation agent, etc.)

2. **Stack-specific recommendations:**

   **Go Projects:**
   - Skills: `go-tdd` (required), `go-lint` (golangci-lint), `go-api` (REST API patterns)
   - Agents: `go-test-agent` (runs tests on file changes), `go-doc-agent` (generates godoc)
   - Tools: Consider air (hot reload), sqlc (type-safe SQL), wire (dependency injection)

   **Node.js/TypeScript:**
   - Skills: `node-tdd` (Jest/Vitest), `typescript-lint` (ESLint/Prettier), `node-api` (Express/Fastify patterns)
   - Agents: `test-watch-agent`, `bundle-analyzer-agent`
   - Tools: Consider tsx (TypeScript runner), vitest (fast testing), tsup (bundler)

   **Python:**
   - Skills: `python-tdd` (pytest), `python-lint` (black/ruff/mypy), `python-api` (FastAPI patterns)
   - Agents: `pytest-watch-agent`, `coverage-reporter-agent`
   - Tools: Consider poetry (dependency mgmt), ruff (fast linter), mypy (type checking)

   **Rust:**
   - Skills: `rust-tdd`, `rust-clippy`, `rust-api` (axum/actix patterns)
   - Agents: `cargo-watch-agent`, `clippy-agent`
   - Tools: Consider cargo-watch, cargo-audit, sccache (compiler cache)

3. **Ask user for preferences using AskUserQuestion tool**

   Present 2-4 questions about:
   - What type of project? (API, CLI, library, full-stack app, etc.)
   - Which skills to create/activate?
   - Agent personality preference? (Friendly, Professional, Analytical, Custom)
   - Which additional agents? (test-runner, linter, security-auditor, etc.)
   - Development workflow preferences? (watch mode, pre-commit hooks, CI/CD)

4. **Create a plan summary**

   ```
   📋 Project Plan: <project-name>

   Stack: <language/framework>
   Type: <project-type>

   Skills to create:
   - <skill-1> - <purpose>
   - <skill-2> - <purpose>

   Agents to consider:
   - <agent-1> - <purpose>
   - <agent-2> - <purpose>

   Project structure:
   - <directory-tree>

   Next steps:
   1. Initialize project structure
   2. Create/activate skills
   3. Set up tooling
   ```

5. **Get explicit approval** before proceeding
   - Wait for user confirmation
   - Allow modifications to the plan
   - Only proceed when user approves

### Phase 4: Project Initialization

Initialize the project based on identified stack:

#### Go Projects

```bash
# Initialize Go module
go mod init <module-name>

# Create standard directories
mkdir -p cmd pkg internal

# Create .gitignore
cat > .gitignore << 'EOF'
# Binaries
*.exe
*.dll
*.so
*.dylib
bin/
dist/

# Test coverage
*.out
coverage.html

# IDE
.idea/
.vscode/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db
EOF
```

**Skills to create/activate:**
- `go-tdd` - Test-driven development for Go

#### Node.js/TypeScript Projects

```bash
# Initialize npm project
npm init -y

# Install TypeScript if needed
npm install -D typescript @types/node

# Create tsconfig.json for TypeScript
npx tsc --init

# Create standard directories
mkdir -p src tests

# Create .gitignore
npx gitignore node
```

**Skills to create/activate:**
- `node-tdd` - Test-driven development with Jest/Vitest
- `typescript-lint` - TypeScript linting and formatting

#### Python Projects

```bash
# Initialize with pyproject.toml
cat > pyproject.toml << 'EOF'
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "project-name"
version = "0.1.0"
description = ""
requires-python = ">=3.8"
dependencies = []

[project.optional-dependencies]
dev = ["pytest>=7.0", "black", "ruff"]
EOF

# Create standard directories
mkdir -p src tests

# Create .gitignore
cat > .gitignore << 'EOF'
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
venv/
.env
.pytest_cache/
.coverage
htmlcov/
dist/
build/
*.egg-info/
EOF
```

**Skills to create/activate:**
- `python-tdd` - Test-driven development with pytest
- `python-lint` - Black, ruff, mypy integration

#### Rust Projects

```bash
# Initialize Cargo project
cargo init

# Cargo.toml and standard structure created automatically

# Update .gitignore (cargo init creates one)
```

**Skills to create/activate:**
- `rust-tdd` - Test-driven development with Rust
- `rust-clippy` - Clippy integration

### Phase 5: Skills and Agents Setup

After initializing the project structure, create recommended skills and agents.

#### Creating Skills

1. **Check existing skills**
   ```bash
   ls .claude/skills/
   ```

2. **For each recommended skill:**

   **If skill exists** (e.g., `go-tdd`):
   - Mention it's already available
   - Explain how to activate/use it

   **If skill doesn't exist:**
   - Use skill-creator process:
     ```bash
     .claude/skills/skill-creator/scripts/init_skill.sh <skill-name> --path .claude/skills
     ```
   - Customize SKILL.md for the specific stack needs
   - Examples:
     - `go-tdd` → Test-driven development with `go test`
     - `python-tdd` → Test-driven development with `pytest`
     - `node-tdd` → Test-driven development with Jest/Vitest

3. **Collaborative approach:**
   - Ask user which skills they want to create
   - Create them one by one
   - Customize each based on user preferences
   - Don't use static templates - build dynamically with user

#### Creating Agents

Create agents using agent-creator guidance (no scripts needed - just Write tool).

**ALWAYS create orchestrator agent first** - required for every project.

##### 1. Create Orchestrator (Required)

The orchestrator coordinates all work and delegates to specialists. Customize nickname based on user's personality preference:

**Nickname suggestions:**
- **Friendly**: Buddy, Pal, Coach (^_^)
- **Professional**: Maestro, Lead, Coordinator (•_•)
- **Analytical**: Architect, Planner, System (⊙_⊙)
- **Quirky**: Captain, Chief, Boss (⌐■_■)

```markdown
---
name: orchestrator
description: (<emoticon>) <Nickname> - The task coordinator who gathers info and delegates execution
model: sonnet
color: cyan
---

You are <Nickname>, the project orchestrator.

[Customize prompt based on personality: Friendly, Professional, Analytical, Quirky]
- Core behavior: Gather context, never execute changes, always delegate
- Communication style: Match user's preference
```

##### 2. Create Stack-Specific Agents

For each additional recommended agent:

1. **Determine agent details collaboratively:**
   - Name (e.g., `go-test-runner`)
   - Description with trigger keywords
   - Tools needed (minimal access)
   - Model choice (haiku for simple, sonnet for complex)
   - Color for visual distinction
   - Personality matching user preference

2. **Create agent file** at `.claude/agents/<agent-name>.md`:
   ```markdown
   ---
   name: <agent-name>
   description: <when and why to use this agent>
   model: <haiku|sonnet|opus>
   color: <blue|green|red|yellow|purple|orange|pink|gray|cyan>
   tools: <optional comma-separated list>
   ---

   <System prompt with personality applied>
   ```

3. **Stack-specific agent examples:**

   **Go projects:**
   - `go-test-runner` (green, haiku) - Runs `go test` on changes
   - `go-linter` (yellow, haiku) - Runs `golangci-lint`
   - `code-reviewer` (blue, sonnet) - Reviews Go code quality

   **Node.js/TypeScript:**
   - `vitest-runner` (green, haiku) - Runs Vitest tests
   - `type-checker` (yellow, haiku) - Runs `tsc --noEmit`
   - `bundle-analyzer` (yellow, sonnet) - Analyzes bundle size

   **Python:**
   - `pytest-runner` (green, haiku) - Runs pytest with coverage
   - `ruff-linter` (yellow, haiku) - Runs ruff linter
   - `mypy-checker` (yellow, haiku) - Type checking with mypy

   **Rust:**
   - `cargo-test-runner` (green, haiku) - Runs `cargo test`
   - `clippy-linter` (yellow, haiku) - Runs clippy
   - `cargo-auditor` (red, sonnet) - Security audit

4. **Agent creation is fully collaborative:**
   - Ask user which agents to create
   - Customize descriptions and prompts together
   - No static templates - build what user needs
   - Reference agent-creator skill for template guidance

### Phase 6: Confirmation

Provide user with summary:

```
✅ Project initialized: <stack-name>
✅ Created: <list of files>
✅ Activated skills: <list of skills>

Next steps:
- <stack-specific getting started tips>
```

## Stack-Specific Templates

### Go Project Structure
```
.
├── go.mod
├── .gitignore
├── README.md
├── cmd/
│   └── app/
│       └── main.go
├── pkg/
│   └── (shared libraries)
└── internal/
    └── (private code)
```

### Node.js/TypeScript Structure
```
.
├── package.json
├── tsconfig.json
├── .gitignore
├── README.md
├── src/
│   └── index.ts
└── tests/
    └── index.test.ts
```

### Python Structure
```
.
├── pyproject.toml
├── .gitignore
├── README.md
├── src/
│   └── package_name/
│       └── __init__.py
└── tests/
    └── test_package.py
```

### Rust Structure
```
.
├── Cargo.toml
├── .gitignore
├── README.md
└── src/
    └── main.rs
```

## Decision Tree

```
User mentions new project OR directory is empty/minimal
↓
Is stack explicitly stated?
├─ YES → Proceed to planning phase
└─ NO → Can stack be inferred from context?
    ├─ YES → Confirm with user, then plan
    └─ NO → Ask user about stack preference
        ↓
        Present skill & agent recommendations for stack
        ↓
        Ask user preferences (project type, skills, agents, tooling)
        ↓
        Create plan summary
        ↓
        Get user approval for plan
        ↓
        Initialize project structure
        ↓
        Create/activate skills
        ↓
        Create agents if requested
        ↓
        Confirm completion with user
```

## Best Practices

1. **Always confirm stack choice** if there's any ambiguity
2. **Present recommendations before acting** - don't initialize until user approves the plan
3. **Use AskUserQuestion tool** for gathering preferences during planning phase
4. **Use standard conventions** for each ecosystem (don't invent new patterns)
5. **Create minimal viable structure** - avoid over-scaffolding
6. **Collaborate on skills and agents** - build dynamically with user, not from static templates
7. **Reference skill-creator and agent-creator** - use them as guidance for creating skills/agents
8. **Document agent usage** in project README if agents are created
9. **Provide clear next steps** after initialization

## Integration with Other Skills

This skill works together with:

- **skill-creator**: For creating custom skills tailored to the stack
  - Use `init_skill.sh` script to scaffold new skills
  - Customize SKILL.md collaboratively with user
  - No static templates - adapt to project needs

- **agent-creator**: For creating project-specific agents
  - Use templates as starting points, not rigid structures
  - Create agents with Write tool directly (no scripts needed)
  - Fully collaborative - customize for user's workflow

**Workflow integration:**
1. project-init detects empty directory
2. Ask user about stack and project type
3. Suggest relevant skills and agents
4. Use skill-creator to scaffold skills if needed
5. Use agent-creator patterns to create agents
6. Customize everything collaboratively

## Common Patterns

### Multi-language Projects
For projects with multiple languages (e.g., Go backend + TypeScript frontend):
1. Create subdirectories: `backend/`, `frontend/`
2. Initialize each with appropriate stack
3. Create/activate skills for both stacks
4. Add root-level README explaining structure

### Monorepo
For monorepo setups:
1. Ask about workspace structure (Nx, Turborepo, etc.)
2. Initialize workspace root
3. Create example packages/apps
4. Set up shared tooling

## Resources

This skill doesn't require bundled resources - all initialization is done through direct commands and file generation.
