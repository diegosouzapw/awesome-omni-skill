---
name: skill-creator-from-docs
description: Create comprehensive PAI skills from tool/CLI/API documentation. Analyzes docs (markdown or URLs via crawl4ai), deduces workflows, generates helper scripts/templates/guardrails, and builds complete skill packages. USE WHEN user says 'create skill from docs', 'build skill for [tool]', 'turn documentation into skill', or provides documentation to transform into a skill.
---

# Skill Creator from Documentation

Transform documentation for tools, CLIs, APIs, or libraries into comprehensive, production-ready PAI skills with helper scripts, templates, and guardrails.

## When to Activate This Skill

- "Create a skill from [tool] documentation"
- "Build a skill for [CLI tool]"
- "Turn this documentation into a skill"
- User provides documentation (markdown or URLs) and wants a skill
- "Generate a skill that helps me use [API/library]"
- "Make [tool] easier to use with a skill"

## Core Workflow

### Phase 1: Documentation Gathering

**Input Methods:**

1. **Markdown Documentation (Direct)**
   ```
   User provides markdown content directly
   ```

2. **Documentation URLs (Extract)**
   Use crawl4ai-cli skill to extract documentation:
   ```bash
   # For single page
   crwl https://docs.example.com/tool -o markdown > docs.md

   # For multiple pages
   crwl https://docs.example.com/guide/page1 -o markdown > part1.md
   crwl https://docs.example.com/guide/page2 -o markdown > part2.md
   ```

3. **Mixed Approach**
   Combine crawled content with user-provided context

**Deliverable:** Complete documentation in markdown format

### Phase 2: Documentation Analysis

**Analyze the documentation to extract:**

1. **Tool Overview**
   - What the tool does
   - Primary use cases
   - Key capabilities

2. **Command/API Patterns**
   - Common commands or API calls
   - Flag patterns and options
   - Configuration approaches

3. **Workflows**
   - Standard workflows from docs
   - Multi-step processes
   - Common task sequences

4. **Pitfalls & Gotchas**
   - Explicitly mentioned warnings
   - Error-prone areas
   - Common mistakes
   - Prerequisites or setup requirements

5. **Best Practices**
   - Recommended approaches
   - Performance tips
   - Security considerations

**Deliverable:** Structured analysis document

### Phase 3: Skill Design Brainstorming

**Design the skill components:**

1. **Identify Helper Scripts Needed**
   - Setup/installation scripts
   - Validation scripts
   - Generator scripts (for configs, schemas, etc.)
   - Inspection/debugging scripts

2. **Design Config Templates**
   - Common configuration presets
   - Template files for different scenarios
   - Example configurations

3. **Create Guardrails**
   - Mandatory workflows (⚠️ MANDATORY sections)
   - Validation steps
   - Common mistake prevention
   - Required checks

4. **Define Checklists**
   - First-time setup checklist
   - Pre-execution checklist
   - Troubleshooting checklist

5. **Build Reference Structure**
   - What goes in SKILL.md (quick reference)
   - What goes in references/ (detailed guides)
   - What needs examples

**Deliverable:** Skill design document with component list

### Phase 4: Artifact Creation

**Create all skill components using templates:**

1. **Directory Structure**
   ```bash
   ~/.claude/skills/[tool-name]/
   ├── SKILL.md                    # Main skill file
   ├── references/                 # Detailed references
   │   ├── cli-reference.md        # Complete CLI docs
   │   ├── config-reference.md     # Configuration options
   │   ├── patterns.md             # Proven workflows
   │   └── troubleshooting.md      # Common issues
   ├── scripts/                    # Helper scripts
   │   ├── setup.sh                # First-time setup
   │   ├── validate_*.py           # Validation scripts
   │   ├── generate_*.py           # Generator scripts
   │   └── README.md               # Scripts documentation
   └── templates/                  # Config templates
       ├── basic_config.yml
       ├── advanced_config.yml
       └── [other templates]
   ```

2. **Generate Helper Scripts**
   - Use `templates/helper-script.py.template`
   - Use `templates/helper-script.sh.template`
   - Customize for specific tool needs
   - Add tool-specific logic
   - Include validation and error handling

3. **Create Config Templates**
   - Use `templates/config-template.yml.template`
   - Create presets (basic, advanced, dynamic, etc.)
   - Add comments explaining each option
   - Include usage examples

4. **Write Reference Docs**
   - CLI reference (complete flag documentation)
   - Config reference (all options explained)
   - Patterns (proven workflows)
   - Troubleshooting (common issues + solutions)

**Deliverable:** Complete skill directory with all artifacts

### Phase 5: SKILL.md Creation

**Write the main SKILL.md file:**

1. **YAML Frontmatter**
   ```yaml
   ---
   name: tool-name
   description: Clear description. USE WHEN triggers. Keywords.
   ---
   ```

2. **Structure** (use template: `templates/tool-skill-SKILL.md.template`)
   - Tool overview
   - Prerequisites (if needed)
   - Quick start
   - Core tasks
   - Command reference
   - Configuration files
   - Proven patterns
   - Troubleshooting
   - Helper scripts
   - Workflow requirements
   - Common workflows
   - Tips

3. **Key Elements**
   - ⚠️ MANDATORY sections for critical workflows
   - Quick reference tables
   - Code examples for every section
   - Links to detailed references
   - Helper script usage

**Deliverable:** Complete SKILL.md file

### Phase 6: Integration & Testing

1. **Add to Global Context**
   Update `~/.claude/global/KAI.md`:
   ```xml
   <skill>
   <name>tool-name</name>
   <description>Your skill description</description>
   <location>user</location>
   </skill>
   ```

2. **Test Activation**
   - Try natural language triggers
   - Verify skill loads correctly
   - Check all file references work

3. **Test Workflows**
   - Run setup scripts
   - Try common commands
   - Test validation scripts
   - Verify templates work

4. **Iterate**
   - Refine based on testing
   - Add missing components
   - Improve documentation

**Deliverable:** Production-ready skill integrated into PAI

## Templates Available

Located in `~/.claude/skills/skill-creator-from-docs/templates/`:

- `tool-skill-SKILL.md.template` - Main skill structure
- `helper-script.py.template` - Python helper script
- `helper-script.sh.template` - Bash helper script
- `config-template.yml.template` - Configuration file
- `README-scripts.md.template` - Scripts directory README

## Key Principles

1. **Template-Driven**: Use templates for consistency
2. **Helper Scripts Over Manual Steps**: Automate common tasks
3. **Guardrails Prevent Mistakes**: Add ⚠️ MANDATORY workflows
4. **Progressive Disclosure**: SKILL.md = quick ref, references/ = details
5. **Validation Built-In**: Create validation scripts for common errors
6. **Examples Everywhere**: Every section needs working examples
7. **Prerequisites Automated**: Setup scripts handle first-time setup
8. **Troubleshooting Proactive**: Anticipate and document common issues

## Example Pattern: CLI Tool Skill

For a CLI tool, typically create:

**Helper Scripts:**
- `setup.sh` - First-time installation and verification
- `validate_config.py` - Validate configuration files
- `generate_config.py` - Interactive config generator

**Templates:**
- `basic_config.yml` - Simple use case preset
- `advanced_config.yml` - Full-featured preset
- `[specific]_config.yml` - Task-specific presets

**References:**
- `cli-reference.md` - Complete flag documentation
- `config-reference.md` - All configuration options
- `patterns.md` - Proven multi-step workflows
- `troubleshooting.md` - Common issues and solutions

**SKILL.md Sections:**
- Prerequisites with automated setup
- Quick start with minimal example
- Core tasks (common operations)
- Proven patterns with complete examples
- Troubleshooting quick reference table
- Workflow requirements (mandatory steps)

## Supplementary Resources

For comprehensive methodology and detailed examples:
`read ~/.claude/skills/skill-creator-from-docs/CLAUDE.md`

For template usage:
`ls ~/.claude/skills/skill-creator-from-docs/templates/`

## Common Workflows

### Workflow 1: Create Skill from URLs

```
1. User provides documentation URLs
2. Use crawl4ai-cli to extract docs to markdown
3. Analyze extracted documentation
4. Brainstorm components (scripts, templates, guardrails)
5. Create directory structure
6. Generate artifacts from templates
7. Write SKILL.md
8. Test and iterate
```

### Workflow 2: Create Skill from Markdown

```
1. User provides markdown documentation
2. Analyze documentation directly
3. Brainstorm components
4. Create directory structure
5. Generate artifacts from templates
6. Write SKILL.md
7. Test and iterate
```

### Workflow 3: Enhance Existing Skill

```
1. Analyze current skill structure
2. Identify missing components (scripts, templates, refs)
3. Generate missing pieces from templates
4. Update SKILL.md with new components
5. Test enhancements
```

## Tips

**Documentation Analysis:**
- Look for "common mistakes" or "gotchas" sections
- Extract command patterns and flag combinations
- Identify multi-step workflows
- Note prerequisites and dependencies
- Find configuration patterns

**Script Creation:**
- Start with templates
- Focus on automating error-prone steps
- Add validation and clear error messages
- Include usage examples in script help

**Template Design:**
- Create presets for common scenarios
- Add explanatory comments
- Include usage instructions
- Show example output or results

**Guardrails:**
- Mark critical workflows as ⚠️ MANDATORY
- Create validation scripts for common mistakes
- Add prerequisite checks to scripts
- Include "why this prevents failures" sections

**Testing:**
- Test every helper script
- Verify all templates work
- Check all file references
- Try natural language activation
- Validate examples in SKILL.md
