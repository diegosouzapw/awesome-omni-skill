---
name: skill-creator
description: Guide for creating effective skills. Use when users want to create a new skill or update an existing skill that extends Claude's capabilities with specialized knowledge, workflows, or tool integrations.
license: Complete terms in LICENSE.txt
---

# Skill Creator

This skill provides guidance for creating effective skills that extend Claude's capabilities.

## About Skills

Skills are self-contained packages providing specialized knowledge, workflows, and tools. They transform Claude from general-purpose to domain-specific agent.

### Core Principles

**1. Concise is Key**: Context window is shared resource. Only add what Claude doesn't already know.

**2. Set Appropriate Freedom**:
- High freedom: Multiple valid approaches, context-dependent decisions
- Medium freedom: Preferred patterns with some variation
- Low freedom: Fragile operations requiring specific sequences

## Skill Structure

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description)
│   └── Markdown instructions
└── Optional Resources
    ├── scripts/      - Executable code
    ├── references/   - Documentation for context
    └── assets/       - Output templates/files
        └── templates/ - Template files (for skill-creator only)
```

## Creation Workflow

### 1. Initialize Skill

```bash
# Interactive mode (recommended)
uv run scripts/init_skill.py my-skill-name

# Non-interactive mode
uv run scripts/init_skill.py my-skill-name --non-interactive

# Custom path
uv run scripts/init_skill.py my-skill-name --path ./custom/location
```

### 2. Edit SKILL.md

**Frontmatter Requirements**:
- `name`: skill-name (hyphen-case, lowercase)
- `description`: What it does + when to use it (this triggers skill selection)

**Body Guidelines**:
- Keep under 500 lines
- Use imperative form
- Include essential workflows only
- Reference bundled resources clearly

### 3. Add Resources (Optional)

**Scripts** (`scripts/`): Executable code for deterministic tasks

Each generated skill includes dependency management:
- `pyproject.toml` - Defines required packages
- `scripts/check_dependencies.py` - Checks and installs dependencies

```python
#!/usr/bin/env python3
def process_file(input_path):
    # Implementation here
    pass
```

**Dependency Management Workflow:**
```bash
# First time or when dependencies change
uv run scripts/check_dependencies.py --install

# Before using any scripts
uv run scripts/check_dependencies.py

# Run scripts
uv run scripts/helper.py
```

**References** (`references/`): Detailed documentation loaded on-demand
```markdown
# API Reference
## Endpoints
- GET /users - List users
- POST /users - Create user
```

**Assets** (`assets/`): Templates, images, boilerplate for output
- PowerPoint templates
- HTML boilerplate directories
- Configuration files
- `templates/` (skill-creator only) - Template files for skill generation:
  - `skill.md.template` - Main SKILL.md template
  - `helper_script.py.template` - Python script template  
  - `reference.md.template` - Reference documentation template
  - `asset_readme.md.template` - Asset directory template
  - `pyproject.toml.template` - Python project configuration
  - `check_dependencies.py.template` - Dependency management script

### 4. Package Skill

```bash
# Package to current directory
uv run scripts/package_skill.py path/to/my-skill

# Package to specific output directory
uv run scripts/package_skill.py path/to/my-skill ./dist
```

## Best Practices

### Progressive Disclosure
1. **Metadata** (name/description) - Always loaded (~100 words)
2. **SKILL.md body** - When skill triggers (<5k words)  
3. **Bundled resources** - As needed by Claude

### Content Organization
- Keep core workflow in SKILL.md
- Move detailed examples to references/
- Split by domain for multi-domain skills:
  ```
  bigquery-skill/
  ├── SKILL.md (overview + navigation)
  └── references/
      ├── finance.md
      ├── sales.md
      └── product.md
  ```

### What NOT to Include
- README.md files
- Installation guides  
- Auxiliary documentation
- Only include files Claude needs to do the job

**Note**: 
- **Python skills**: Dependency management (pyproject.toml + check_dependencies.py) is automatically included for seamless setup
- **Other language skills**: Include appropriate config files (package.json, Gemfile, go.mod, etc.) and setup instructions in SKILL.md to guide Claude on dependency management

## Using Skills with Dependencies

When Claude encounters a skill with scripts in any programming language:

### General Dependency Management Principles

**Always check for dependency files first:**
- `pyproject.toml` (Python with uv/pip)
- `package.json` (Node.js/JavaScript)
- `Gemfile` (Ruby)
- `go.mod` (Go)
- `Cargo.toml` (Rust)  
- `requirements.txt` (Python legacy)
- `composer.json` (PHP)
- `pom.xml`/`build.gradle` (Java/JVM)

**Common dependency setup patterns:**
```bash
# Python (modern)
uv run scripts/check_dependencies.py --install
uv run scripts/script.py

# Node.js
npm install  # or yarn install, pnpm install
npm run script

# Ruby
bundle install
bundle exec ruby script.rb

# Go
go mod tidy
go run script.go

# Rust
cargo build
cargo run --bin script
```

### 1. Python Skills (Automated)
```bash
# Always run this before using Python skill scripts
uv run scripts/check_dependencies.py --install
```

### 2. Other Language Skills (Manual Check)
For non-Python skills, check the skill directory for:

1. **Look for dependency files** (package.json, Gemfile, go.mod, etc.)
2. **Check SKILL.md** for setup instructions
3. **Run standard dependency commands** for that language
4. **Follow any custom setup scripts** provided

### 3. Standard Skill Usage Pattern
```bash
# Step 1: Navigate to skill directory
cd path/to/skill

# Step 2: Check for and install dependencies
# (Language-specific command based on what files are present)

# Step 3: Use skill scripts/tools
# (Follow instructions in SKILL.md)
```

### 3. Troubleshooting Dependencies
If dependency issues occur:
```bash
# Clean and reinstall
rm -rf .venv
uv sync --reinstall

# Check specific package versions
uv tree
```

## Validation

```bash
# Validate skill structure
uv run scripts/quick_validate.py path/to/skill
```

Validates:
- YAML frontmatter format
- Required fields (name, description)
- Naming conventions
- File structure

## Common Patterns

**Task-Based**: Different operations
```markdown
## Quick Start
## Merge PDFs  
## Split PDFs
## Extract Text
```

**Workflow-Based**: Sequential processes
```markdown
## Overview
## Step 1: Preparation
## Step 2: Processing
## Step 3: Output
```

**Reference-Based**: Standards/specifications
```markdown
## Guidelines
## Color Specifications
## Typography Rules
## Usage Examples
```

## Examples

See existing skills for patterns:
- **python**: Development workflow with uv
- **slidev**: Presentation creation
- **code-styleguide**: Coding standards

Start simple, iterate based on actual usage.