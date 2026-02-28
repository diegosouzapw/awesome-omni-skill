---
name: vscode-copilot-instructions
description: Expert guidance for creating VSCode Copilot custom instructions, prompt files, and AGENTS.md files. Use when user wants to create, modify, or understand .github/copilot-instructions.md, .instructions.md, AGENTS.md, or .prompt.md files. Includes YAML frontmatter structure, variable syntax, tool references, and best practices for each file type.
license: MIT
metadata:
  author: FeatBit
  version: 1.0.0
  category: vscode-copilot-customization
---

# VSCode Copilot Custom Instructions & Prompt Files

Expert guidance for creating and managing VSCode Copilot customization files including custom instructions, prompt files, and agent configurations.

## When to Use This Skill

Activate this skill when:
- Creating `.github/copilot-instructions.md` files
- Building `.instructions.md` files with applyTo patterns
- Setting up `AGENTS.md` configurations
- Creating `.prompt.md` reusable prompts
- Configuring YAML frontmatter for any of these file types
- Understanding variable syntax (`${variable}`)
- Referencing tools (`#tool:toolName`) or files in instructions
- Troubleshooting instruction file activation issues
- Sharing custom instructions across teams or repositories

## Quick Overview: File Types

### 1. `.github/copilot-instructions.md`
**Always-on workspace instructions** — Applied automatically to all chat requests in the workspace.

**When to use**: General coding guidelines, team conventions, project-specific rules that apply throughout the project.

**Location**: `.github/copilot-instructions.md` at workspace root

**Example**:
```markdown
# Project Guidelines

- Use TypeScript for all new code
- Follow functional programming patterns
- Write tests using Jest
- API responses must use our standard error format
```

📄 **Complete Guide**: [references/copilot-instructions-guide.md](references/copilot-instructions-guide.md)

---

### 2. `.instructions.md` Files
**Conditional, targeted instructions** — Apply to specific file types, locations, or tasks using `applyTo` glob patterns.

**When to use**: Language-specific guidelines, framework-specific patterns, file-type-specific rules.

**Location**: `.github/instructions/` folder or user profile

**Example**:
```markdown
---
name: python-coding-standards
description: Python coding standards and best practices
applyTo: "**/*.py"
---

# Python Standards

- Follow PEP 8 style guide
- Use type hints for all functions
- Write docstrings using Google style
```

📄 **Complete Guide**: [references/instructions-files-guide.md](references/instructions-files-guide.md)

---

### 3. `AGENTS.md` File
**Multi-agent workspace instructions** — Useful when working with multiple AI agents, applied automatically like copilot-instructions.md.

**When to use**: When you work with multiple AI coding assistants and want unified instructions.

**Location**: Workspace root, or subfolders (experimental)

**Example**:
```markdown
# Agent Guidelines

All agents working in this codebase must:

- Never modify files in the `legacy/` directory
- Always run tests before committing
- Use the project's error handling utilities
```

📄 **Complete Guide**: [references/agents-md-guide.md](references/agents-md-guide.md)

---

### 4. `.prompt.md` Files
**Reusable, on-demand prompts** — Triggered via `/command` in chat for specific development tasks.

**When to use**: Standardized workflows, code generation templates, review checklists, common development tasks.

**Location**: `.github/prompts/` folder or user profile

**Example**:
```markdown
---
name: create-react-component
description: Generate a React component with TypeScript and tests
argument-hint: component-name
agent: edit
---

# Create React Component

Create a new React component named ${input:componentName} with:

1. TypeScript interface for props
2. Function component implementation
3. Jest test file with basic tests
4. Storybook story file

Follow our component patterns in [docs/component-guidelines.md](docs/component-guidelines.md).
```

📄 **Complete Guide**: [references/prompt-files-guide.md](references/prompt-files-guide.md)

---

## Decision Tree: Which File Type?

```
Do you want instructions that apply automatically?
├─ YES → Always-on for entire workspace?
│   ├─ YES → Use .github/copilot-instructions.md
│   └─ NO → Apply to specific files/patterns?
│       └─ Use .instructions.md with applyTo pattern
│
└─ NO → Want to trigger on-demand?
    └─ Use .prompt.md file (invoked via /command)
```

**Also consider**: Use `AGENTS.md` if working with multiple AI agents simultaneously.

---

## Common Elements Across All File Types

### YAML Frontmatter (Optional but Recommended)

All file types support YAML frontmatter for metadata:

```yaml
---
name: file-name-slug
description: What this file does
applyTo: "**/*.py"           # .instructions.md only
agent: edit                  # .prompt.md only
tools: [githubRepo, files]   # .prompt.md only
---
```

### Variable Syntax

Reference dynamic values in file bodies:

```markdown
- `${workspaceFolder}` — Workspace root path
- `${file}` — Currently open file path
- `${selection}` — Selected text
- `${input:variableName}` — User input from chat
- `${input:variableName:placeholder}` — With placeholder hint
```

📄 **Complete Reference**: [references/variables-and-references.md](references/variables-and-references.md)

### Tool References

Reference agent tools using special syntax:

```markdown
Use #tool:githubRepo to search the repository.
Use #tool:files to read workspace files.
```

### File References

Link to other workspace files using relative Markdown links:

```markdown
Follow the patterns in [docs/api-guide.md](docs/api-guide.md).
See [scripts/validate.py](scripts/validate.py) for examples.
```

---

## Reference Documentation

All detailed guides are in the `references/` folder:

- [**Copilot Instructions Guide**](references/copilot-instructions-guide.md) — `.github/copilot-instructions.md` complete reference
- [**Instructions Files Guide**](references/instructions-files-guide.md) — `.instructions.md` with applyTo patterns
- [**AGENTS.md Guide**](references/agents-md-guide.md) — Multi-agent workspace configuration
- [**Prompt Files Guide**](references/prompt-files-guide.md) — `.prompt.md` reusable prompts
- [**Variables & References**](references/variables-and-references.md) — All variable syntax and file referencing
- [**Best Practices**](references/best-practices.md) — Writing effective instructions
- [**Troubleshooting**](references/troubleshooting.md) — Common issues and solutions

---

## Template Files

Ready-to-use templates in `assets/templates/`:

- [copilot-instructions.template.md](assets/templates/copilot-instructions.template.md)
- [instructions-file.template.md](assets/templates/instructions-file.template.md)
- [agents.template.md](assets/templates/agents.template.md)
- [prompt-file.template.md](assets/templates/prompt-file.template.md)

---

## Quick Start Workflows

### Creating Always-On Instructions

```bash
# 1. Create .github directory if needed
mkdir -p .github

# 2. Create copilot-instructions.md
code .github/copilot-instructions.md

# 3. Add your guidelines
# 4. Enable in settings: github.copilot.chat.codeGeneration.useInstructionFiles
```

### Creating Conditional Instructions

```bash
# 1. Create instructions directory
mkdir -p .github/instructions

# 2. Create instruction file
code .github/instructions/my-rules.instructions.md

# 3. Add YAML frontmatter with applyTo pattern
# 4. Enable: chat.includeApplyingInstructions
```

### Creating Reusable Prompts

```bash
# 1. Create prompts directory
mkdir -p .github/prompts

# 2. Create prompt file
code .github/prompts/my-task.prompt.md

# 3. Use in chat: /my-task
```

---

## Settings to Enable

Ensure these VS Code settings are enabled:

```json
{
  // Enable copilot-instructions.md
  "github.copilot.chat.codeGeneration.useInstructionFiles": true,
  
  // Enable .instructions.md with applyTo patterns
  "chat.includeApplyingInstructions": true,
  
  // Enable AGENTS.md
  "chat.useAgentsMdFile": true,
  
  // Enable nested AGENTS.md in subfolders (experimental)
  "chat.useNestedAgentsMdFiles": false,
  
  // Show prompts as recommendations
  "chat.promptFilesRecommendations": true
}
```

---

## Key Best Practices

✅ **Keep Instructions Concise**: Each instruction should be a single, clear statement  
✅ **Use Specific Examples**: Show code examples of what you want  
✅ **Reference, Don't Duplicate**: Use Markdown links to existing docs  
✅ **Test Activation**: Verify instructions trigger correctly  
✅ **Layer Appropriately**: Use the right file type for the right scope

❌ **Avoid Vague Guidance**: "Write good code" → Instead: "Use functional components with TypeScript interfaces"  
❌ **Don't Overload**: Too many instructions reduce effectiveness  
❌ **Don't Forget Settings**: Instructions won't work without proper VS Code settings

📄 **Full Best Practices**: [references/best-practices.md](references/best-practices.md)

---

## Troubleshooting

**Instructions not applying?**

1. Check the file is in the correct location
2. Verify the relevant setting is enabled
3. Check YAML frontmatter syntax
4. For `.instructions.md`: verify `applyTo` glob pattern matches
5. View diagnostics: Right-click in Chat → Diagnostics

📄 **Full Troubleshooting Guide**: [references/troubleshooting.md](references/troubleshooting.md)

---

## Example Use Cases

### Use Case 1: Team Coding Standards
**Solution**: `.github/copilot-instructions.md` with always-on guidelines

### Use Case 2: Python-Specific Rules
**Solution**: `.instructions.md` with `applyTo: "**/*.py"`

### Use Case 3: Generate API Endpoint
**Solution**: `.prompt.md` with `/create-endpoint` command

### Use Case 4: Multiple AI Assistants
**Solution**: `AGENTS.md` at workspace root

---

## Community Resources

- [Official VSCode Docs](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Prompt Files Docs](https://code.visualstudio.com/docs/copilot/customization/prompt-files)
- [Awesome Copilot Repository](https://github.com/github/awesome-copilot/tree/main) — Community examples

---

**Remember**: Instruction files are most effective when they're specific, actionable, and well-organized. Start simple and iterate based on what works for your team.
