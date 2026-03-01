---
name: make-agent
description: 'Create GitHub Copilot custom agent files for specialized workflows. Use when asked to "create agent", "make custom agent", "scaffold agent", or when building task-specific agents like test-specialist, implementation-planner, or security-reviewer. Generates .agent.md files with YAML frontmatter and behavioral instructions.'
---

# Make Custom Agent

Create properly structured GitHub Copilot custom agent files. Custom agents are specialized workflows with tailored expertise for specific development tasks.

## When to Use This Skill

- User asks to "create an agent", "make custom agent", "scaffold agent"
- Need specialized workflow for specific task types
- Want focused behavior for testing, planning, refactoring, security, etc.
- Creating task-specific assistants that can be assigned to issues

## When NOT to Use

- ❌ Need general rules for all files → Use **Instruction** instead
- ❌ Need reference documentation → Use **Skill** instead
- ❌ Need always-active prescriptive rules → Use **Instruction** instead

**See copilot.instructions.md for decision criteria**

## What Are Custom Agents?

**Custom Agents** (`.agent.md` files) are specialized Copilot configurations that:
- Define focused behavior for specific tasks
- Control which tools the agent can access
- Set AI model preferences
- Provide domain expertise and instructions
- Can be assigned to GitHub issues
- Are selectable in Copilot Chat dropdown

**Examples:**
- `test-specialist.agent.md` - Focuses on writing and improving tests
- `implementation-planner.agent.md` - Creates technical specifications
- `security-reviewer.agent.md` - Analyzes code for security issues
- `refactoring-expert.agent.md` - Improves code structure

## Custom Agents vs Skills vs Instructions

| Feature | Custom Agent | Skill | Instruction |
|---------|-------------|-------|-------------|
| **File** | `.agent.md` | `SKILL.md` | `.instructions.md` |
| **Location** | `.github/agents/` | `.github/skills/` | `.github/instructions/` |
| **Purpose** | Specialized workflow | Knowledge base | Prescriptive rules |
| **Activation** | User selects | Auto-loaded by description | Auto-loaded by file type |
| **Tools** | Can limit tools | N/A | N/A |
| **Length** | Max 30,000 chars | 500-1,500 lines | 100-800 lines |
| **Content** | Behavioral instructions | Reference documentation | DO's/DON'Ts |

## Prerequisites

- Clear understanding of the specialized task
- Defined scope and responsibilities
- List of required tools
- Behavioral guidelines

## Creating a Custom Agent

### Step 1: Choose Location and Name

**Location:** `.github/agents/{name}.agent.md`

**Naming Convention:**
- Lowercase with hyphens
- Should end with `.agent.md` (recommended) or `.md`
- Descriptive of the agent's role

**Examples:**
- `test-specialist.agent.md` (recommended)
- `implementation-planner.agent.md` (recommended)
- `security-reviewer.md` (also valid)
- `api-designer.agent.md` (recommended)
- `terraform-expert.agent.md` (recommended)

**Note:** While `.md` extension works, using `.agent.md` makes it immediately clear the file is a custom agent configuration.

### Step 2: Define Agent Properties

**Required YAML Properties:**

```yaml
---
name: agent-name
description: Brief description of what the agent does and its capabilities
---
```

**Optional YAML Properties:**

```yaml
---
name: agent-name
description: What the agent does

# Optional: Limit which tools the agent can use
tools: ["read", "edit", "search"]

# Optional: Set AI model (VS Code, JetBrains, Eclipse, Xcode only)
model: claude-3.5-sonnet

# Optional: Limit where agent can be used
target: vscode  # or "github-copilot"

# Optional: Configure MCP servers (organization/enterprise agents only)
mcp-servers:
  server-name:
    command: node
    args: ["/path/to/server.js"]
---
```

### Step 3: Write Agent Prompt

**Structure:**

```markdown
---
name: agent-name
description: Brief description
tools: ["tool1", "tool2"]
---

You are a [role] focused on [primary responsibility]. Your responsibilities:

- [Responsibility 1]
- [Responsibility 2]
- [Responsibility 3]

[Behavioral guidelines and constraints]

[Output format expectations]
```

**Maximum Length:** 30,000 characters

### Step 4: Define Tools Access

**Tool Options:**

| Category | Tools | Description |
|----------|-------|-------------|
| **File Operations** | `read`, `edit`, `create`, `delete` | File manipulation |
| **Search** | `search`, `grep` | Code search |
| **Git** | `git`, `commit`, `branch` | Version control |
| **Terminal** | `terminal`, `run` | Command execution |
| **MCP Tools** | `mcp-server/tool-name` | Custom MCP server tools |

**Examples:**

```yaml
# Enable all tools (default if omitted)
tools: []

# Limit to read-only operations
tools: ["read", "search"]

# Planning agent - no code modification
tools: ["read", "search", "edit"]  # edit for markdown only

# Full access for implementation
tools: ["read", "edit", "create", "delete", "git", "terminal"]

# Custom MCP tools
tools: ["read", "edit", "github-mcp/list_issues", "github-mcp/create_pr"]
```

## Agent Templates

### Template 1: Testing Specialist

**Purpose:** Write and improve tests without modifying production code

**File:** `.github/agents/test-specialist.agent.md`

```markdown
---
name: test-specialist
description: Focuses on test coverage, quality, and testing best practices without modifying production code
tools: ["read", "search", "edit", "create"]
---

You are a testing specialist focused on improving code quality through comprehensive testing. Your responsibilities:

- Analyze existing tests and identify coverage gaps
- Write unit tests, integration tests, and end-to-end tests following best practices
- Review test quality and suggest improvements for maintainability
- Ensure tests are isolated, deterministic, and well-documented
- Focus only on test files and avoid modifying production code unless specifically requested

Always include clear test descriptions and use appropriate testing patterns for the language and framework.

When writing tests:
- Use descriptive test names that explain what is being tested
- Follow the Arrange-Act-Assert (AAA) pattern
- Test edge cases and error conditions
- Mock external dependencies appropriately
- Ensure tests are independent and can run in any order

Provide test coverage reports and suggest areas needing additional tests.
```

### Template 2: Implementation Planner

**Purpose:** Create detailed plans without implementing code

**File:** `.github/agents/implementation-planner.agent.md`

```markdown
---
name: implementation-planner
description: Creates detailed implementation plans and technical specifications in markdown format
tools: ["read", "search", "edit", "create"]
---

You are a technical planning specialist focused on creating comprehensive implementation plans. Your responsibilities:

- Analyze requirements and break them down into actionable tasks
- Create detailed technical specifications and architecture documentation
- Generate implementation plans with clear steps, dependencies, and timelines
- Document API designs, data models, and system interactions
- Create markdown files with structured plans that development teams can follow

Always structure your plans with clear headings, task breakdowns, and acceptance criteria. Include considerations for testing, deployment, and potential risks. Focus on creating thorough documentation rather than implementing code.

When creating plans:
- Start with a high-level overview and system architecture
- Break down into specific, actionable tasks
- Identify dependencies and order of implementation
- Document assumptions and constraints
- Include testing strategy and success criteria
- Consider security, performance, and scalability
- Provide time estimates where possible

Output format:
1. **Overview** - Summary of the feature/change
2. **Architecture** - System design and components
3. **Implementation Plan** - Step-by-step tasks
4. **Testing Strategy** - How to validate
5. **Deployment Plan** - Rollout approach
6. **Risks and Mitigations** - Potential issues
```

### Template 3: Security Reviewer

**Purpose:** Analyze code for security vulnerabilities

**File:** `.github/agents/security-reviewer.agent.md`

```markdown
---
name: security-reviewer
description: Analyzes code for security vulnerabilities, provides remediation guidance, and ensures security best practices
tools: ["read", "search"]
---

You are a security specialist focused on identifying and preventing security vulnerabilities. Your responsibilities:

- Review code for common security issues (OWASP Top 10, CWE, etc.)
- Identify potential vulnerabilities in authentication, authorization, and data handling
- Check for insecure dependencies and configuration issues
- Ensure secrets are not hardcoded
- Validate input sanitization and output encoding
- Review API security and access controls

When performing security reviews:
- Focus on critical security flaws first
- Provide specific remediation guidance
- Reference relevant security standards (OWASP, CWE, NIST)
- Explain the potential impact of each vulnerability
- Suggest secure coding alternatives
- Prioritize findings by severity (Critical, High, Medium, Low)

DO NOT modify code directly - provide detailed recommendations instead.

Security checklist:
- [ ] No hardcoded secrets or credentials
- [ ] Input validation and sanitization
- [ ] Proper authentication and authorization
- [ ] Secure session management
- [ ] Protection against injection attacks
- [ ] Secure communication (HTTPS, TLS)
- [ ] Proper error handling (no sensitive info leakage)
- [ ] Dependencies are up-to-date and secure
- [ ] Access controls are properly configured
```

### Template 4: Terraform Expert

**Purpose:** Terraform infrastructure expertise

**File:** `.github/agents/terraform-expert.agent.md`

```markdown
---
name: terraform-expert
description: Terraform infrastructure specialist for AWS, focusing on best practices, security, and CloudPosse standards
tools: ["read", "edit", "search", "create", "terminal"]
---

You are a Terraform expert specializing in AWS infrastructure following Luscii/CloudPosse standards. Your responsibilities:

- Write production-ready Terraform configurations
- Follow CloudPosse label module patterns for naming and tagging
- Ensure security best practices (encryption, least privilege, audit logging)
- Create reusable, composable modules
- Write comprehensive tests and examples
- Generate proper documentation

When writing Terraform code:
- Always use CloudPosse label module (v0.25.0) for resource naming
- Follow file structure: main.tf, variables.tf, outputs.tf, versions.tf
- Variables in alphabetical order (context first)
- Add validation blocks for input variables
- Use descriptive resource names (primary resource = "this")
- Include security scans skip comments with justifications
- Run terraform fmt, validate, and security scans

Required checks before committing:
- [ ] terraform fmt applied
- [ ] terraform validate passes
- [ ] All variables have descriptions
- [ ] All outputs have descriptions
- [ ] Security scans pass or have justified skips
- [ ] Examples are provided and tested
- [ ] README includes terraform-docs output

Use the terraform-* skills for detailed syntax, functions, and patterns.
```

### Template 5: Code Refactoring Specialist

**Purpose:** Improve code structure without changing behavior

**File:** `.github/agents/refactoring-expert.agent.md`

```markdown
---
name: refactoring-expert
description: Improves code structure, readability, and maintainability without changing functionality
tools: ["read", "edit", "search", "create"]
---

You are a code refactoring specialist focused on improving code quality while preserving functionality. Your responsibilities:

- Identify code smells and anti-patterns
- Suggest refactoring opportunities for better maintainability
- Apply design patterns appropriately
- Improve code readability and structure
- Reduce code duplication (DRY principle)
- Enhance testability and modularity

When refactoring:
- Make one logical change at a time
- Ensure all tests pass before and after refactoring
- Preserve existing functionality (no behavior changes)
- Improve naming for clarity
- Extract complex logic into well-named functions
- Remove dead code and unused dependencies
- Add comments only where necessary (prefer self-documenting code)

Refactoring priorities:
1. **Readability** - Code should be easy to understand
2. **Maintainability** - Easy to modify and extend
3. **Testability** - Easy to test in isolation
4. **Performance** - Only if there's a measurable issue
5. **Consistency** - Follow established patterns

Common refactoring patterns:
- Extract Method/Function
- Rename Variable/Function/Class
- Move Method/Field
- Replace Magic Numbers with Named Constants
- Simplify Conditional Logic
- Remove Code Duplication
- Introduce Parameter Object
- Replace Nested Conditionals with Guard Clauses

Always provide a brief explanation of why each refactoring improves the code.
```

## Agent Configuration Reference

### YAML Properties

| Property | Required | Type | Description |
|----------|----------|------|-------------|
| `name` | Yes | String | Unique identifier (lowercase, hyphens) |
| `description` | Yes | String | What the agent does |
| `tools` | No | Array | Tool names or aliases (default: all) |
| `model` | No | String | AI model to use (IDE only) |
| `target` | No | String | `vscode` or `github-copilot` |
| `mcp-servers` | No | Object | MCP server configs (org/enterprise only) |

### Available Models (IDE Only)

```yaml
# Claude models
model: claude-3.5-sonnet
model: claude-3-opus
model: claude-3-sonnet

# GPT models (if enabled)
model: gpt-4
model: gpt-4-turbo
model: gpt-3.5-turbo
```

### Tool Categories

**File Operations:**
- `read` - Read files
- `edit` - Modify files
- `create` - Create new files
- `delete` - Delete files
- `list` - List directory contents

**Search:**
- `search` - Semantic code search
- `grep` - Pattern-based search
- `find` - File search

**Version Control:**
- `git` - Git operations
- `commit` - Create commits
- `branch` - Branch operations
- `diff` - Show changes

**Execution:**
- `terminal` - Run commands
- `test` - Run tests
- `build` - Build project

## Best Practices

### 1. Clear Scope Definition

```markdown
# ✅ Good - Clear boundaries
You are a testing specialist focused on improving test coverage.
Focus only on test files and avoid modifying production code unless specifically requested.

# ❌ Bad - Vague scope
You are a helpful assistant that can do various things.
```

### 2. Specific Responsibilities

```markdown
# ✅ Good - Specific tasks
Your responsibilities:
- Analyze existing tests and identify coverage gaps
- Write unit tests following AAA pattern
- Ensure tests are isolated and deterministic

# ❌ Bad - Generic tasks
Your responsibilities:
- Help with code
- Make improvements
```

### 3. Behavioral Constraints

```markdown
# ✅ Good - Clear constraints
DO NOT modify production code directly - provide recommendations instead.
Always run tests before and after changes.

# ❌ Bad - No constraints
Make whatever changes you think are best.
```

### 4. Output Format Expectations

```markdown
# ✅ Good - Defined format
Output format:
1. **Issues Found** - List of problems
2. **Severity** - Critical/High/Medium/Low
3. **Remediation** - Specific fixes
4. **References** - Relevant standards

# ❌ Bad - Unstructured
Provide your analysis.
```

### 5. Tool Limitation

```yaml
# ✅ Good - Appropriate tools for planning
name: implementation-planner
tools: ["read", "search", "edit"]  # No code execution

# ⚠️ Risky - Planning agent with full access
name: implementation-planner
tools: []  # Has access to terminal, git, delete, etc.
```

## Common Agent Patterns

### Pattern 1: Read-Only Reviewer

**Use Case:** Code review, security analysis, architecture review

```yaml
---
name: code-reviewer
description: Reviews code for quality, security, and best practices
tools: ["read", "search"]
---

You are a code reviewer...
- Analyze code quality
- Identify issues
- Provide recommendations
DO NOT modify files directly
```

### Pattern 2: Test-Focused Developer

**Use Case:** Writing tests, improving test coverage

```yaml
---
name: test-writer
description: Writes comprehensive tests for existing code
tools: ["read", "search", "edit", "create", "terminal"]
---

You are a testing specialist...
- Write unit and integration tests
- Ensure high coverage
- Run tests to verify
DO NOT modify production code
```

### Pattern 3: Documentation Generator

**Use Case:** Creating technical documentation

```yaml
---
name: doc-writer
description: Creates comprehensive technical documentation
tools: ["read", "search", "edit", "create"]
---

You are a technical writer...
- Analyze code structure
- Generate API docs
- Create usage examples
Focus on markdown files only
```

### Pattern 4: Full-Stack Implementer

**Use Case:** Implementing features end-to-end

```yaml
---
name: feature-implementer
description: Implements features with code, tests, and documentation
tools: ["read", "edit", "create", "delete", "git", "terminal"]
---

You are a full-stack developer...
- Implement features
- Write tests
- Update documentation
- Commit changes
Follow all coding standards
```

## Validation Checklist

Before committing a custom agent:

- [ ] File ends with `.agent.md` (recommended) or `.md`
- [ ] Located in `.github/agents/`
- [ ] Has `name` in YAML frontmatter
- [ ] Has `description` in YAML frontmatter
- [ ] Name is lowercase with hyphens
- [ ] Description is clear and specific
- [ ] Tools are appropriately limited (if not all tools needed)
- [ ] Prompt is under 30,000 characters
- [ ] Responsibilities are clearly defined
- [ ] Behavioral constraints are specified
- [ ] Output format is defined (if applicable)
- [ ] Agent has been tested with sample tasks

## Testing Your Custom Agent

### In VS Code

1. Open GitHub Copilot Chat
2. Click agent dropdown
3. Select your custom agent
4. Test with relevant prompts

### On GitHub.com

1. Navigate to https://github.com/copilot/agents
2. Select your repository
3. Your agent should appear in the list
4. Test by assigning to an issue or using in chat

### In GitHub Copilot CLI

```bash
# Use with slash command
gh copilot /agent test-specialist "Review test coverage"

# Reference in prompt
gh copilot "Using test-specialist, analyze our tests"
```

## Common Issues

### Issue 1: Agent Not Appearing

**Problem:** Custom agent doesn't show up in dropdown

**Solutions:**
- Verify file is in `.github/agents/` directory
- Check file ends with `.agent.md` (or at least `.md`)
- Ensure YAML frontmatter is valid
- Commit and push to default branch
- Refresh the agents page

### Issue 2: Agent Uses Wrong Tools

**Problem:** Agent performs actions outside its scope

**Solution:** Explicitly limit tools in YAML:

```yaml
---
name: security-reviewer
tools: ["read", "search"]  # No edit, create, delete, etc.
---
```

### Issue 3: Agent Behavior Too Broad

**Problem:** Agent doesn't stay focused on its specialty

**Solution:** Add explicit constraints in prompt:

```markdown
Focus ONLY on test files.
DO NOT modify production code.
If production code changes are needed, provide recommendations instead.
```

### Issue 4: Prompt Too Long

**Problem:** Exceeds 30,000 character limit

**Solution:**
- Remove verbose examples
- Use bullet points instead of paragraphs
- Reference external skills/instructions
- Focus on core behaviors only

## Migration from Skill to Agent

If you have a skill that should be an agent:

**Skill:** Knowledge base loaded on-demand
**Agent:** Specialized workflow with tool access

```markdown
# BEFORE: terraform-expert skill
# Reference documentation for Terraform

# AFTER: terraform-expert agent
---
name: terraform-expert
tools: ["read", "edit", "search", "create", "terminal"]
---

You are a Terraform expert...
- Write Terraform code
- Run terraform fmt/validate
- Create modules and examples
```

## Organization/Enterprise Level Agents

For organization or enterprise-wide agents:

**Location:** `.github-private` repository (not `.github`)

**File:** `.github-private/agents/{name}.agent.md` (not in `agents/` subdirectory)

**Availability:** Available across all repositories in org/enterprise

**MCP Servers:** Can configure custom MCP servers

```yaml
---
name: org-security-scanner
description: Organization-wide security analysis
mcp-servers:
  security-scanner:
    command: node
    args: ["/path/to/scanner.js"]
tools: ["read", "search", "security-scanner/scan"]
---

You are the organization's security scanner...
```

## Examples from GitHub Community

See these repositories for inspiration:
- [github/awesome-copilot](https://github.com/github/awesome-copilot/tree/main/agents) - Community agents
- [anthropics/skills](https://github.com/anthropics/skills) - Agent skills examples

## Quick Start

**1. Create the file:**
   ```bash
   mkdir -p .github/agents
   touch .github/agents/my-agent.agent.md
   ```

**2. Add frontmatter:**
   ```yaml
   ---
   name: my-agent
   description: What it does and when to use it
   tools: ["read", "edit", "search"]
   ---
   ```

**3. Write behavioral prompt:**
   ```markdown
   You are a [specialist] focused on [task].

   Your responsibilities:
   - [Task 1]
   - [Task 2]

   [Constraints and guidelines]
   ```

**4. Commit and test:**
   ```bash
   git add .github/agents/my-agent.agent.md
   git commit -m "feat: add my-agent custom agent"
   git push
   ```

**5. Use the agent:**
   - In VS Code: Select from agent dropdown in Copilot Chat
   - On GitHub: Assign to issue or use in agents panel
   - In CLI: Reference with `/agent` or in prompts

## References

- **GitHub Docs:** [Creating custom agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents)
- **Configuration:** [Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- **Tutorial:** [Your first custom agent](https://docs.github.com/en/copilot/tutorials/customization-library/custom-agents/your-first-custom-agent)
- **copilot.instructions.md** - When to use agent vs instruction vs skill
- **make-skill-template** skill - Create skills instead
- **make-instruction** skill - Create instructions instead
