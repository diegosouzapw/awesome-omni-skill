---
name: inf
description: Generate hierarchical Inf diagram notes as YAML files in the inf-notes directory. First run creates root.yaml + Level 1. Subsequent runs scan all nodes across all levels and extend expandable ones by one level. Use --focus <file> to expand only a specific file.
---

# Inf Repository Notes Generator

Generate comprehensive visual documentation for the current repository using the Inf YAML format.

## Arguments

**--focus <file>** - Expand only a specific subgraph file and its children

```bash
/inf --focus api.yaml              # Only expand api.yaml
/inf --focus api__auth.yaml        # Only expand api__auth.yaml (nested file)
/inf --focus module-frontend.yaml  # Only expand module-frontend.yaml
```

**When to use `--focus`:**
- You want to deeply explore one specific area without touching others
- You're iterating on a particular module's documentation
- You need to regenerate/fix one subgraph without affecting the rest

**Behavior:**
- Scans only the specified file for nodes to expand
- Creates subgraph files only for that focused file's children (one level deep)
- Ignores all other YAML files in inf-notes/
- **One-level-at-a-time**: Allows controlled iteration and review before going deeper
- **Contrast**: Without `--focus`, scans ALL files at ALL levels for expandable nodes

**--yaml-convert** - Convert all YAML files to Inf JSON format

```bash
/inf --yaml-convert               # Convert inf-notes/*.yaml to JSON
```

**What it does:**
- Runs `python3 ~/.claude/skills/inf/scripts/yaml_convert.py --dir inf-notes/`
- Converts all YAML files in `inf-notes/` directory to Inf JSON format
- Computes layout positions using Graphviz
- Outputs JSON files alongside the YAML files

**When to use `--yaml-convert`:**
- After completing YAML creation and validation
- When you want to view diagrams in the Inf canvas application
- As a final step to generate viewable output

---

# YAML Format Specification

## Core Principle

- **Every YAML file MUST have a title and introduction:**
  - `title` node — The name or title of this graph (e.g., "Authentication Module")
  - `intro` node — A brief explanation of what this graph covers
- Provide a comprehensive overview (the full picture) at the root level (root.yaml).
- Place detailed explanations in separate YAML files, using file-based subgraphs.
- Use appropriate node types:
  - rectangle — concepts, components, modules
  - circle — entry / exit points, external systems
  - diamond — decisions, conditionals
  - text — details / annotations (no border)
  - code — commands, pseudocode, or source code snippets
  - table — data or comparisons (cells may contain subgraphs)
  - url — references / resources (use text/rectangle type with URL in content)
- Create meaningful connections:
    - Directed edges for flow or dependencies
    - Undirected edges for associations
- Use groups to organize related nodes, with clear visual boundaries and labels.
- Go as deep as needed — subgraphs support infinite nesting levels.
- Separate YAML files for each subgraph, with clear and descriptive names (e.g., module-authentication.yaml, concept-event-loop.yaml). Use relative paths only.
- It's fine to use a large text node or a code node to include details or pseudocode.
- **IMPORTANT**: Keep related content together in the same node. Use multiline text (with `\n`) to combine titles and content instead of creating separate nodes. The layout algorithm may place separate nodes far apart, breaking visual relationships. Example: Use `"Title\nContent here..."` instead of two separate nodes.
- **AVOID isolated nodes**: Don't create standalone title nodes or short text nodes without connections. Instead, use groups with descriptive names to organize related nodes. Groups provide better visual structure than isolated title nodes.


---

## Format

### Normal Node

```yaml
nodes:
  - text: "Node Name"
    type: [rectangle|circle|diamond|text|code|table]  # default: rectangle
    align: [left|center|right]                        # default: center
    attr: [title|intro]                               # optional: layout attribute
```

**Node Types:**
- `rectangle` - Concepts, process steps, components, modules (default)
- `circle` - Start/end points, states, actors, external systems
- `diamond` - Decision points, conditionals, gateways
- `text` - Titles, labels, details, annotations (no border)
- `code` - Code snippets, configuration, commands, scripts

**URLs/References:**
URLs are automatically detected and highlighted in any node type (clickable with Ctrl+Click):
```yaml
- text: "API Docs\nhttps://api.example.com/docs"
  type: text
  align: left
```

**Text Alignment:**
- `left` - For lists, code blocks
- `center` - For titles, labels (default)
- `right` - For dates, metadata

**Layout Attributes (`attr`) - REQUIRED:**

Every YAML file must start with a `title` and `intro` node:

```yaml
nodes:
  # REQUIRED: Title - the name of this graph
  - text: "Authentication Module"
    attr: title

  # REQUIRED: Introduction - explain what this graph covers
  - text: "This module handles user login, session management, and access control."
    attr: intro
    align: left

  # Other nodes follow...
  - text: "Login Service"
```

- `title` - The name/heading of this graph, rendered as full-width text at the top
- `intro` - Brief explanation of the graph's purpose, rendered as full-width text below title

Layout order (top to bottom):
```
┌─────────────────────────────┐
│         title               │  ← Full width, graph name
├─────────────────────────────┤
│         intro               │  ← Full width, explanation
├─────────────────────────────┤
│  other nodes...             │
│  (normal layout)            │
└─────────────────────────────┘
```

**Multiline text:** Use `\n` for line breaks:
```yaml
- text: "Title\nSubtitle\nDetails"
  type: text
  align: center
```

**Keep related content together:**
```yaml
# ✓ GOOD: Title and content in one node
- text: "Configuration\n\nAPI key: xxx\nTimeout: 30s\nRetries: 3"
  type: text
  align: left

# ✗ BAD: Separate nodes may be placed far apart by layout
- text: "Configuration"
  type: text
- text: "API key: xxx\nTimeout: 30s\nRetries: 3"
  type: text
```

### Subgraph

Link to another YAML file for hierarchical depth:

```yaml
nodes:
  - text: "Authentication"
    type: rectangle
    subgraph: "module-auth.yaml"

  - text: "Frontend"
    type: rectangle
    subgraph: "frontend/frontend.yaml"
```

**File-based subgraphs:**
- Always include `.yaml` extension for clarity
- Use relative paths only
- Referenced file must exist (or will be created in next level)

**Naming conventions:**
- **Top-level**: `api.yaml`, `database.yaml`, `frontend.yaml`
- **Nested (level 2+)**: `parent__child.yaml` (double underscore)
- **Prefixes**: `api-`, `module-`, `concept-`, `flow-`
- **Kebab-case**: `module-authentication.yaml`, `api-endpoints.yaml`

**Examples:**
```yaml
# Level 1: root.yaml references
- text: "API Layer"
  subgraph: "api.yaml"

# Level 2: api.yaml references
- text: "Authentication"
  subgraph: "api__authentication.yaml"

# Level 3: api__authentication.yaml references
- text: "OAuth Flow"
  subgraph: "api__authentication__oauth.yaml"
```

### Table

Use Markdown table syntax:

```yaml
nodes:
  - text: "Table Name"
    type: table
    table: |
      | Header 1 | Header 2 |
      |----------|----------|
      | Value 1  | Value 2  |
      | Value 3  | Value 4  |
```

**Table alignment:**
- `:---` = left-aligned column
- `:---:` = center-aligned column
- `---:` = right-aligned column

**Example with alignment:**
```yaml
- text: "System Config"
  type: table
  table: |
    | Component  | Status | Version |
    |:-----------|:------:|--------:|
    | API        | Active | 2.1.0   |
    | Database   | Active | 14.5    |
```

## Connections

```yaml
connections:
  - from: "Source"
    to: "Target"
    directed: [true|false]  # default: true
```

**Connection Types:**
- `directed: true` - Arrow showing flow/dependency (default)
- `directed: false` - Line without arrow for associations

**Important:** Node text must match exactly (case-sensitive, including `\n` line breaks)

## Groups

Organize related nodes with visual boundaries:

```yaml
groups:
  - name: "Group Name"
    nodes: ["Node 1", "Node 2"]
```

**Note:** Node text must match exactly (case-sensitive)

**Best Practice - Use groups instead of isolated title nodes:**

```yaml
# ✓ GOOD: Group with descriptive name organizes related nodes
nodes:
  - text: "Authentication Service"
  - text: "Token Validation"
  - text: "User Session"

groups:
  - name: "Auth System"
    nodes: ["Authentication Service", "Token Validation", "User Session"]

# ✗ BAD: Isolated title node with no connections
nodes:
  - text: "Auth System"  # Isolated title node - avoid this!
  - text: "Authentication Service"
  - text: "Token Validation"
```

## Layout

```yaml
layout:
  engine: [dot|neato|fdp|circo|twopi]  # default: dot
  rankdir: [TB|LR|BT|RL]                # default: TB
  ranksep: 1.5                          # default: 1.0
  nodesep: 1.0                          # default: 0.5
```

**Layout Engines:**
- `dot` - Hierarchical (flowcharts, org charts)
- `neato` - Force-directed (networks)
- `fdp` - Force-directed with clustering
- `circo` - Circular layout
- `twopi` - Radial layout

**Direction:**
- `TB` - Top to bottom (default)
- `LR` - Left to right (timelines)
- `BT` - Bottom to top
- `RL` - Right to left

## Special Characters Warning

**CRITICAL: Avoid special characters in node text that may confuse Graphviz:**

❌ **Do NOT use these characters in node names:**
- Backslashes: `\` (except in `\n` for newlines)
- Quotes inside text: `"` or `'` (use descriptive words instead)
- Angle brackets: `<` `>`
- Curly braces in isolation: `{` `}`
- Periods and commas in isolation: `..` `,`

✅ **Use descriptive words instead:**
```yaml
# BAD - Special characters cause Graphviz errors
- text: "Check Path\n.., /, \\"

# GOOD - Descriptive words
- text: "Check Path\n(dots, slashes)"

# BAD - Quotes in text
- text: "Validate \"name\" field"

# GOOD - Descriptive alternative
- text: "Validate name field"
```

**Why this matters:** Graphviz processes node text as DOT syntax, and special characters can cause parsing errors during layout computation.

---

# Sequential Creation Workflow

**Output Location**: `./inf-notes/`

## Repository Analysis

Explore the codebase to understand its architecture:

- **Entry points**: main.js, app.py, index.html
- **Structure**: src/, lib/, components/, tests/
- **Config**: package.json, requirements.txt, Makefile
- **Docs**: README, docs/
- **Build system**: scripts, Dockerfile

Identify key patterns: frontend/backend separation, modules, data flow, dependencies, algorithms.

---

## Workflow

**IMPORTANT: Sequential Processing Only**

DO NOT use agents in parallel. Process all files sequentially to avoid context limits. Create and validate files one at a time.

### Check if `inf-notes/` exists

**If `inf-notes/` does NOT exist (new repository):**

Generate **Level 0 (root) + Level 1 (direct children) only**:

1. Create `inf-notes/root.yaml` with system overview (5-10 major components)
2. **Mark ALL major components as extensible** by adding `subgraph: "filename.yaml"` reference (only skip truly atomic nodes)
3. Create all Level 1 subgraph files (direct children of root)
4. Validate all YAML files:
   ```bash
   python3 ~/.claude/skills/inf/scripts/yaml_convert.py inf-notes/root.yaml --validate
   python3 ~/.claude/skills/inf/scripts/yaml_convert.py inf-notes/api.yaml --validate
   # ... validate each Level 1 file
   ```
5. Fix errors until all files pass validation
6. **Stop here** - do not generate Level 2+ subgraphs yet

**If `inf-notes/` already EXISTS (deepening existing notes):**

Scan **all nodes across all levels** and extend expandable nodes by one level:

**A. Check for `--focus` argument:**

If `--focus <file>` is provided (e.g., `/inf --focus api.yaml`):
1. **Target file**: Read only `inf-notes/<file>` (e.g., `inf-notes/api.yaml`)
2. **Verify file exists**: If file doesn't exist, report error and stop
3. **Identify nodes for expansion**: For each node in the focused file, **default to marking as extensible** unless it's truly atomic (see "When to Stop" below)
   - **Bias toward expansion**: When uncertain, mark the node as extensible with `subgraph: "parent__child.yaml"`
   - Use double underscore naming: If focusing on `api.yaml`, children are `api__*.yaml`
   - If focusing on `api__auth.yaml`, children are `api__auth__*.yaml`
4. **Create all subgraph files** for the focused file's children only
5. **Validate all new files**:
   ```bash
   python3 ~/.claude/skills/inf/scripts/yaml_convert.py inf-notes/<new-file>.yaml --validate
   ```
6. Fix errors until all files pass validation
7. **Stop here** - user will run `/inf --focus <file>` again to go deeper, or `/inf` to expand other areas

**B. If no `--focus` argument (scan all levels, expand by one):**

**Breadth-first incremental expansion** - Scan all nodes across all existing files and extend by one level:

1. **Scan ALL YAML files** in `inf-notes/` (at every level, not just the deepest)
2. **For each file, identify nodes without subgraphs**: Check if the node could be extended
   - **Default to marking as extensible** unless it's truly atomic (see "Extensibility Exceptions" below)
   - **Bias toward expansion**: When uncertain, add `subgraph: "parent__child.yaml"` to the node
   - Use double underscore naming for nested levels
   - Only skip nodes that are clearly atomic (single functions, simple concepts, no sub-components)
3. **Update existing files** to add subgraph references to expandable nodes
4. **Create all new subgraph files** for the newly added references (one level of children)
5. **Validate all modified and new files**:
   ```bash
   python3 ~/.claude/skills/inf/scripts/yaml_convert.py inf-notes/<file>.yaml --validate
   ```
6. Fix errors until all files pass validation
7. **Stop here** - user will run `/inf` again to scan and extend the next level

---

## When to Stop (Extensibility Exceptions)

**Default: Mark nodes as extensible.** Only stop creating subgraphs when the node is truly atomic:
- Node represents a single function, constant, or variable (truly atomic)
- Further detail would be raw implementation code (use code node with snippet instead)
- Node is a leaf concept with no possible sub-components (e.g., "Port 3000", "UTF-8 encoding")

**When in doubt, mark as extensible** - it's better to create expansion opportunities than to miss important details.

**Note on behavior:** Both with and without `--focus`, expansion is incremental (one level at a time). The difference is scope: `--focus` targets one file, while no-focus scans all files at all levels.

---

# Quality Standards

## Root Level
- **Comprehensive overview** of entire system
- **Groups** for related areas (e.g., "Client Layer", "Services", "Data Layer")
- **Nearly all nodes have subgraphs** for maximum depth - only skip truly atomic nodes (single files, constants, etc.)

## Subgraphs
- **Focused scope**: One topic per file
- **Clear connections** showing relationships
- **Entry point nodes** (circles) showing context from parent
- **Groups** to organize subsections
- **Mark components as extensible**: Most nodes should have subgraph references unless truly atomic

## Naming Conventions
- **Kebab-case**: `module-authentication.yaml`
- **Prefixes**: `api-`, `module-`, `concept-`, `flow-`
- **Nested**: `parent__child.yaml` (double underscore)
- **Descriptive**: Filename should clearly indicate content

## Connection Design
- **Directed**: Flow, dependencies, sequence, cause-effect
- **Undirected**: Associations, relationships, mutual dependencies
- **Semantic accuracy**: Choose based on real relationships

---

# Usage Examples

## Initial Generation

```bash
/inf                    # Create root.yaml + Level 1 files
```

## Deepening All Areas (Breadth-First)

```bash
/inf                    # Scans ALL nodes in ALL files
                        # Identifies nodes without subgraphs that could be extended
                        # Adds subgraph references and creates one level of children
                        # Run multiple times to progressively deepen all areas
```

**Behavior:** Without `--focus`, the skill scans all existing files at every level, identifies nodes that could be extended but don't have subgraphs yet, adds subgraph references, and creates one level of children. This is breadth-first expansion - progressively adding depth across all areas simultaneously.

## Focused Expansion (One Level at a Time)

```bash
# Expand only the API documentation (one level)
/inf --focus api.yaml              # Creates api__*.yaml children only

# Go deeper into API authentication (one level)
/inf --focus api__auth.yaml        # Creates api__auth__*.yaml children only

# Continue deepening the auth flow (one level)
/inf --focus api__auth__oauth.yaml # Creates api__auth__oauth__*.yaml children only
```

**Behavior:** With `--focus`, expansion is limited to **one level** for controlled, iterative deepening. This allows you to carefully review and refine each level before proceeding deeper.

## Important: Sequential Processing Only

**DO NOT use agents in parallel to avoid context limits.** Process files sequentially, one at a time. This ensures:
- Stable context management
- Predictable memory usage
- Reliable validation and error handling

```bash
# CORRECT: Sequential expansion
/inf --focus api.yaml              # Wait for completion
/inf --focus frontend.yaml         # Then expand next area
/inf --focus database.yaml         # Continue sequentially

# WRONG: Do NOT spawn multiple agents in parallel
```

## Iterative Refinement

```bash
# Initial pass
/inf                              # Create root + Level 1

# Focus on one area to refine
/inf --focus api.yaml             # Expand API details (creates api__*.yaml)
/inf --focus api__auth.yaml       # Go deeper into auth (creates api__auth__*.yaml)

# Or switch focus to another area
/inf --focus database.yaml        # Now work on database
```

## Final Conversion

```bash
/inf --yaml-convert               # Convert all YAML to JSON for viewing
```

**When ready to view:** After completing YAML generation and validation, run `--yaml-convert` to produce JSON files that can be opened in the Inf canvas application.

---

**Ready to generate notes!**

- **First run**: Creates root.yaml + Level 1 subgraphs only
- **Subsequent runs (no --focus)**: Scans ALL nodes across ALL levels, extends expandable ones by one level
- **With `--focus <file>`**: Extends only nodes in the specified file by one level
- **Breadth-first by default**: `/inf` progressively deepens all areas simultaneously
- **Focused iteration**: Use `--focus` to target specific areas while leaving others unchanged
- **With `--yaml-convert`**: Converts all YAML files to JSON format for viewing in Inf canvas
