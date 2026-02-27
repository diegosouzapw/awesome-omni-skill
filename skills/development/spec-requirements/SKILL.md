---
name: spec-requirements
description: Generate comprehensive requirements definition documents with technology selection and improvement suggestions
---

# Spec Requirements

Generate comprehensive requirements definition documents with technology selection, improvement suggestions, and EARS format acceptance criteria.

## Usage

```bash
/spec:requirements <system_name> [options]
```

## Options

| Option | Short | Description | Example |
|--------|-------|-------------|---------|
| `--mode` | `-m` | Mode (new-creation/reverse-engineering) | `-m reverse-engineering` |
| `--backlog` | `-b` | Enable product backlog format | `--backlog` |
| `--epic` | `-e` | Epic name for backlog structure | `-e "User Management Epic"` |
| `--story-points` | | Include story point estimation | `--story-points` |
| `--app` | `-a` | Application name | `-a "Web Store"` |
| `--function` | `-f` | Function/feature name | `-f "Authentication"` |
| `--output` | `-o` | Output file path | `-o specs.md` |
| `--tech` | `-t` | Technology stack | `-t "react,nodejs,postgresql"` |
| `--priority` | `-p` | Priority level (low/medium/high/critical) | `-p high` |
| `--scope` | `-s` | Scope type (mvp/full/enterprise) | `-s mvp` |
| `--suggest` | | Include improvement suggestions | `--suggest` |
| `--examples` | | Include implementation examples | `--examples` |
| `--template` | | Template type | `--template agile` |
| `--hearing` | | Interactive clarification mode | `--hearing` |

## Templates

| Template | Description |
|----------|-------------|
| `standard` | General purpose with EARS format |
| `agile` | User story format with acceptance criteria |
| `waterfall` | Detailed specification document |
| `product-backlog` | Full backlog with epics, features, stories |

## Tool Priorities

**ALWAYS prioritize mcp__serena__ tools:**

### File Operations (Serena MCP First)
- `mcp__serena__find_file` → `Read` (fallback)
- `mcp__serena__search_for_pattern` → `Grep` (fallback)
- `mcp__serena__list_dir` → `LS` (fallback)

### Code Analysis (Serena MCP)
- `mcp__serena__get_symbols_overview`
- `mcp__serena__find_referencing_symbols`

### Context7 Integration
For technology documentation:
- Frontend: React, Vue, Angular
- Backend: Node.js, Express, FastAPI
- Database: PostgreSQL, MongoDB, Redis

## Modes

### New Creation Mode (Default)
Generate requirements from specifications and user input.

### Reverse Engineering Mode
Analyze existing codebase to extract requirements:
1. Component Discovery (controllers, services, models)
2. Feature Extraction (user-facing functionality)
3. User Story Generation (features to stories)

## EARS Format

Acceptance criteria use EARS (Easy Approach to Requirements Syntax):
- WHEN [event] THEN [system] SHALL [response]
- IF [precondition] THEN [system] SHALL [response]
- WHILE [condition] THE SYSTEM SHALL [behavior]
- WHERE [context] THE SYSTEM SHALL [behavior]

## Examples

```bash
# Basic requirements
/spec:requirements "E-commerce Platform" -a "Web Store" -t "react,nodejs"

# MVP with suggestions
/spec:requirements "Social Media App" -s mvp --suggest

# Interactive hearing mode
/spec:requirements "Mobile App" --hearing

# Reverse engineering with backlog
/spec:requirements "User Management" -m reverse-engineering -b -e "Auth Epic"

# Full enterprise with examples
/spec:requirements "CRM System" -s enterprise -t "react,nodejs,postgresql" --examples
```

## Output Structure

1. **Header Information**: Timestamp, system name, priority, scope
2. **Functional Requirements**: Core features, user scenarios (EARS format)
3. **Non-Functional Requirements**: Performance, security, availability
4. **Technical Specifications**: Architecture, tech stack, integrations
5. **Additional Sections**: Examples, suggestions (if enabled)
