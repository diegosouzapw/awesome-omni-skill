---
name: opencode-orchestrator
description: Delegate complex coding, refactoring, research, or multi-step development tasks to OpenCode, an autonomous AI coding agent running in the terminal.
---

# OpenCode Orchestrator

<purpose>
This skill enables Antigravity to delegate complex multi-step coding tasks to OpenCode—a terminal-based autonomous AI coding agent. OpenCode operates independently with its own context, tools, and model access, making it ideal for tasks requiring deep focus, extensive file operations, or specialized model capabilities.
</purpose>

## Trigger Workflow

Use the `/use-opencode` slash command to invoke this skill:
```
/use-opencode
```
This will guide you through constructing an optimal handoff to OpenCode.

## When to Use OpenCode

| Use OpenCode When | Stay in Antigravity When |
|-------------------|--------------------------|
| Multi-file refactoring across 15+ files | Quick single-file edits |
| Long-running code generation tasks | Answering questions about code |
| Tasks requiring Plan Mode review | Browser-based testing |
| Deep codebase exploration | Tasks needing current conversation context |
| Specialized model requirements | Interactive debugging with user |

---

## 1. Capabilities

<capabilities>
### Core Features
- **Autonomous Execution**: Runs independently, editing files, executing commands, and managing its own context
- **Plan Mode**: Reviews and explains proposed changes before execution
- **Multi-Provider Support**: Access to OpenAI, Anthropic, Google Gemini, AWS Bedrock, Groq, Azure OpenAI
- **Session Management**: Persistent conversations with save/restore functionality
- **Auto-Compact**: Automatic context summarization when approaching token limits
- **LSP Integration**: Language Server Protocol support for code intelligence
- **Project Awareness**: Reads `AGENTS.md` for project-specific conventions

### Available Tools (within OpenCode)
- File read/write/search operations
- Shell command execution  
- Code analysis and refactoring
- Git operations
- Web search and URL reading
</capabilities>

---

## 2. Handoff Protocol

<handoff_instructions>
### Critical: Context Transfer

OpenCode does **NOT** share Antigravity's conversation context. You must explicitly provide:

1. **File Paths**: Absolute paths to all relevant files
2. **Task Context**: Background information and requirements
3. **Constraints**: Any specific approaches to use or avoid
4. **Success Criteria**: Clear definition of done

### Command Structure

```bash
opencode run "YOUR_DETAILED_PROMPT"
```

### Optional Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-m MODEL` | Specify model | `-m github-copilot/claude-sonnet-4` |
| `-a AGENT` | Use custom agent | `-a code-reviewer` |
| `--plan` | Force Plan Mode | Review before execution |

</handoff_instructions>

---

## 3. Prompt Construction

<prompt_template>
Structure prompts with clear sections for best results:

```
<context>
Project: [Project name and type]
Working Directory: [Absolute path]
Related Files: [List of relevant file paths]
</context>

<background>
[Any necessary context not obvious from files]
</background>

<task>
[Clear, specific task description]
</task>

<requirements>
- [Specific requirement 1]
- [Specific requirement 2]
</requirements>

<constraints>
- [What to avoid or preserve]
</constraints>

<success_criteria>
- [How to know the task is complete]
</success_criteria>
```
</prompt_template>

---

## 4. Model Selection

<model_guidance>
Choose models based on task characteristics:

### For Code-Heavy Tasks
```bash
opencode run -m github-copilot/claude-sonnet-4 "..."
# or
opencode run -m github-copilot/gemini-3-pro "..."
```

### For Research/Documentation
```bash
opencode run -m github-copilot/gpt-4o "..."
```

### Check Available Models
```bash
opencode models
```

### Default Behavior
If no `-m` flag is provided, OpenCode uses the model configured in:
- `~/.opencode.json`
- `$XDG_CONFIG_HOME/opencode/.opencode.json`
- `./.opencode.json` (project-local)
</model_guidance>

---

## 5. Example Workflows

<examples>
### Multi-File Refactoring
```bash
opencode run -m github-copilot/claude-sonnet-4 "
<context>
Working Directory: /home/user/project
Files: src/api/*.ts, src/services/*.ts
</context>

<task>
Refactor all API handlers to use async/await instead of Promise chains.
Add proper error handling with custom error classes.
</task>

<requirements>
- Preserve existing functionality
- Add JSDoc comments to all public functions
- Update corresponding test files
</requirements>

<success_criteria>
- All Promise.then() chains converted to async/await
- Error handling uses ApiError class from src/errors.ts
- All tests pass
</success_criteria>
"
```

### Code Review Delegation
```bash
opencode run "
<context>
Repository: /home/user/project
Branch: feature/new-auth
Compare against: main
</context>

<task>
Review the changes in the current branch compared to main.
Focus on security, performance, and code quality.
</task>

<requirements>
- Check for SQL injection vulnerabilities
- Identify N+1 query patterns
- Note any breaking API changes
</requirements>

<success_criteria>
- Written review summary in REVIEW.md
- Critical issues clearly flagged
- Suggested improvements provided
</success_criteria>
"
```

### Research and Documentation
```bash
opencode run -m github-copilot/gpt-4o "
<context>
Project: /home/user/project
Tech Stack: Node.js, PostgreSQL, Redis
</context>

<task>
Research caching strategies for our API endpoints and create an implementation plan.
</task>

<requirements>
- Consider Redis caching patterns
- Evaluate cache invalidation strategies
- Account for our microservices architecture
</requirements>

<success_criteria>
- CACHING_PLAN.md created with recommendations
- Code examples for key patterns
- Migration steps outlined
</success_criteria>
"
```
</examples>

---

## 6. Integration with Antigravity Workflow

<antigravity_integration>
### Coordinated Handoffs

When delegating from Antigravity to OpenCode:

1. **Prepare Context**: Gather all relevant file paths and requirements
2. **Construct Prompt**: Use the structured template above
3. **Execute Handoff**: Run via `run_command` tool
4. **Monitor Output**: Read OpenCode's logs and final result
5. **Verify Results**: Check file changes and run tests
6. **Continue Work**: Resume in Antigravity with updated codebase

### Artifact Coordination

OpenCode may create its own files (plans, documentation). After handoff:
- Check for new files in the working directory
- Review any generated documentation
- Incorporate findings into Antigravity artifacts if relevant

### When to Bring Work Back

Return work to Antigravity when:
- Browser testing is needed
- User interaction is required
- Results need integration with current conversation
- Visual verification is needed
</antigravity_integration>

---

## 7. Troubleshooting

<troubleshooting>
### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| `ProviderModelNotFoundError` | Invalid model identifier | Run `opencode models` to list available models |
| Context too short | Auto-compact triggered | Task is self-contained; continue monitoring |
| Permission denied | File access issues | OpenCode runs with user permissions; check file ownership |
| Timeout | Long-running task | Increase `WaitMsBeforeAsync` or send to background |

### Model-Specific Notes

- **GitHub Copilot models**: Require active Copilot subscription
- **Anthropic models**: Best for complex reasoning and code generation
- **Gemini models**: Strong at multi-file understanding
- **OpenAI models**: Effective for research and documentation

### Debugging Commands
```bash
# Check OpenCode configuration
cat ~/.opencode.json

# List available models  
opencode models

# List configured agents
opencode agent list

# Check authentication status
opencode auth status
```
</troubleshooting>

---

## 8. Advanced: Custom Agents

<custom_agents>
For recurring specialized tasks, create custom OpenCode agents:

```bash
opencode agent create
```

This launches an interactive wizard to define:
- Agent name and description
- Custom system prompt
- Tool permissions
- Default model

Then invoke with:
```bash
opencode run -a my-custom-agent "Task description"
```

### Example Use Cases
- **Code Reviewer**: Specialized prompts for security and performance review
- **Documentation Writer**: Focused on JSDoc/TSDoc generation
- **Test Generator**: Configured for your testing framework
</custom_agents>

---

## Quick Reference

```bash
# Basic execution
opencode run "Your task description"

# With specific model
opencode run -m github-copilot/claude-sonnet-4 "Task"

# With custom agent
opencode run -a custom-agent "Task"

# Check available models
opencode models

# Check available agents
opencode agent list
```
