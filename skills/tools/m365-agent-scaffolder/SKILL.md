---
name: m365-agent-scaffolder
description: Quickly scaffolds new Microsoft 365 Copilot declarative agent projects using ATK CLI. Collects basic project information and creates the initial project structure. Use only when creating a new empty M365 Copilot agent project from scratch.
compatibility: Designed for Microsoft 365 Copilot agents development on developer platforms supporting Agent Skills.
metadata:
  authors:
    - sebastienlevert
  version: 1.0.0
---

# M365 Agent Scaffolder

⚠️ **QUICK PROJECT CREATION ONLY** ⚠️

This skill does ONE thing: creates new M365 Copilot agent project structures using ATK CLI. It collects minimal required information and scaffolds the project. All architecture, planning, implementation, and deployment is handled by other skills.

---

## When to Use This Skill

Use this skill ONLY when:
- Creating a brand new empty M365 Copilot agent project from scratch
- The user explicitly asks to create a new project, agent, or workspace
- Starting a new agent development initiative that needs initial project structure

Do NOT use this skill when:
- Working with existing projects (use m365-agent-developer)
- Implementing features or capabilities (use m365-agent-developer)
- Deploying or managing agents (use m365-agent-developer)
- Troubleshooting issues (use m365-agent-developer)
- Designing architecture or planning (use m365-agent-developer)

## Instructions

Follow these exact steps when creating a new M365 Copilot agent project:

### Step 1: Understand the Request

**Action:** Verify the user wants to create a NEW project.

**Check for:**
- Keywords: "new project", "create agent", "scaffold", "start from scratch"
- Confirmation this is NOT an existing project

**If existing project:** Stop and recommend using m365-agent-developer skill.

### Step 2: Collect Required Information

**Action:** Ask the user for two pieces of information:

1. **Project name** (required)
   - What should the project be named?
   - Must be valid for directory names (no spaces, special characters)
   - Example: "customer-support-agent", "sales-assistant"

2. **Location preference** (optional, default: new subdirectory)
   - Create in current directory? (Will create project files directly here)
   - Create in new subdirectory? (Recommended - will create a folder)

**Example questions:**
```
What would you like to name your M365 Copilot agent project?

Should I create this in the current directory or a new subdirectory? 
(Default: new subdirectory)
```

### Step 3: Run ATK CLI Command

**Action:** Execute the scaffolding command with the collected information.

**Command:**
```bash
npx -p @microsoft/m365agentstoolkit-cli@latest atk new -n <project-name> -c declarative-agent -with-plugin type-spec -i false
```

**Parameters:**
- `-n <project-name>`: The project name provided by the user
- `-c declarative-agent`: Create a declarative agent
- `-with-plugin type-spec`: Include TypeSpec plugin scaffolding
- `-i false`: Non-interactive mode (no prompts)

**Why it's important:** This creates the complete project structure with all necessary files and configuration.

### Step 4: Confirm Creation and Stop

**Action:** Provide a simple confirmation message with next steps.

**Message template:**
```
✅ Project created at: <absolute-project-path>

Your empty M365 Copilot agent project structure is ready.

🎯 NEXT STEPS:

Use the m365-agent-developer skill to design, implement, and deploy your agent:
"Design and implement a [describe your agent] using the m365-agent-developer skill"
```

**STOP HERE.**

---


## Examples

### Example 1: Basic Project Scaffolding

**User:** "Create a new M365 Copilot agent project"

**Interaction:**
```
Assistant: What would you like to name your M365 Copilot agent project?

User: sales-assistant
Assistant: Should I create this in the current directory or a new subdirectory?

User: New subdirectory
```

**Script executed:**
```bash
npx -p @microsoft/m365agentstoolkit-cli@latest atk new -n sales-assistant -c declarative-agent -with-plugin type-spec -i false
```

**Confirmation:**
```
✅ Project created at: /home/user/projects/sales-assistant

Your new M365 Copilot agent project structure is ready.

🎯 NEXT STEPS:
Use the m365-agent-developer skill to design, implement, and deploy your agent.
```

### Example 2: Project with Name Provided


**Interaction:**
```
User: Scaffold a new project called customer-support-agent
```

**Script executed:**
```bash
npx -p @microsoft/m365agentstoolkit-cli@latest atk new -n customer-support-agent -c declarative-agent -with-plugin type-spec -i false
```

**Confirmation:**
```
✅ Project created at: /home/user/projects/customer-support-agent

Your new M365 Copilot agent project structure is ready.

🎯 NEXT STEPS:
Use the m365-agent-developer skill to design, implement, and deploy your agent.
```

### Example 3: Creating in Current Directory

**Interaction:**
```
User: Create a document-finder agent in this directory
```
**Script executed:**
```bash
npx -p @microsoft/m365agentstoolkit-cli@latest atk new -n document-finder -c declarative-agent -with-plugin type-spec -i false
```

**Confirmation:**
```
✅ Project created at: /home/user/projects/document-finder

Your new M365 Copilot agent project structure is ready.

🎯 NEXT STEPS:
Use the m365-agent-developer skill to design, implement, and deploy your agent.
```

### Example 4: Quick Succession

**Interaction:**
```
User 1: expense-tracker
User 2: inventory-manager
User 3: meeting-scheduler
```

**Scripts executed:**
```bash
npx -p @microsoft/m365agentstoolkit-cli@latest atk new -n expense-tracker -c declarative-agent -with-plugin type-spec -i false
npx -p @microsoft/m365agentstoolkit-cli@latest atk new -n inventory-manager -c declarative-agent -with-plugin type-spec -i false
npx -p @microsoft/m365agentstoolkit-cli@latest atk new -n meeting-scheduler -c declarative-agent -with-plugin type-spec -i false
```

### Example 5: What NOT to Do

**User:** "Create a customer support agent with SharePoint and Teams capabilities"

**WRONG - Do NOT do this:**
```
❌ Creating TODO.md with architecture plans
❌ Discussing which capabilities to use
❌ Writing TypeSpec code
❌ Implementing agent instructions
❌ Opening workspace in VS Code
```

**CORRECT - Do this:**
```
✅ Ask for project name
✅ Run: npx -p @microsoft/m365agentstoolkit-cli@latest atk new -n customer-support-agent -c declarative-agent -with-plugin type-spec -i false
✅ Confirm creation
✅ Direct to m365-agent-developer skill
✅ STOP
```

## Best Practices

Follow these best practices when scaffolding new M365 Copilot agent projects:

### Project Naming

- **Use Descriptive Names:** Choose names that clearly indicate the agent's purpose
  - ✅ Good: `customer-support-agent`, `sales-assistant`, `document-finder`
  - ❌ Bad: `agent1`, `test`, `myproject`
- **Use Kebab-Case:** Use lowercase with hyphens for maximum compatibility
  - ✅ Good: `expense-tracker-agent`
  - ❌ Bad: `ExpenseTrackerAgent`, `expense_tracker_agent`, `Expense Tracker`
- **Avoid Special Characters:** Stick to alphanumeric characters and hyphens only
  - No spaces, underscores, or special symbols
- **Keep It Concise:** 2-4 words maximum for readability
  - ✅ Good: `sales-dashboard`
  - ❌ Bad: `comprehensive-sales-and-marketing-analytics-dashboard-agent`

### Directory Management

- **Default to New Subdirectory:** Unless user explicitly requests current directory
  - Prevents cluttering existing directories
  - Creates clean, isolated project structure
- **Verify Current Location:** Always check and confirm the working directory
  - Use absolute paths in all confirmations
  - Prevent accidental creation in wrong location
- **Check for Conflicts:** Avoid creating projects in directories that already contain projects
  - Check for existing `package.json` or project files
  - Warn if directory is not empty
- **Use Absolute Paths:** Always provide full absolute paths in confirmation messages
  - Helps users locate the project easily
  - Prevents confusion about project location

### Command Execution

- **Always Use Full ATK CLI Command:** Never abbreviate or use shortcuts
  - Full command: `npx -p @microsoft/m365agentstoolkit-cli@latest atk new`
  - Ensures latest version is used
  - Prevents version compatibility issues
- **Non-Interactive Mode:** Always use `-i false` parameter
  - Prevents unexpected prompts
  - Ensures consistent, predictable behavior
- **Verify Exit Code:** Check command success before confirming to user
  - Handle errors gracefully
  - Provide clear error messages if scaffolding fails
- **Capture Output:** Monitor command output for errors or warnings
  - Log any issues encountered
  - Provide diagnostic information if needed

### Communication Best Practices

- **Be Brief and Direct:** Keep all messages short and actionable
  - No lengthy explanations or architectural discussions
  - Focus only on scaffolding confirmation
- **Clear Handoff:** Explicitly direct users to m365-agent-developer skill
  - Provide specific next-step guidance
  - Make it clear this skill's job is complete
- **No Extra Work:** Do NOT:
  - Create TODO.md files
  - Open workspaces in VS Code
  - Provide implementation guidance
  - Discuss architecture or design
  - Run additional commands beyond scaffolding
- **Set Clear Expectations:** Make it obvious this skill only scaffolds
  - "Your empty project structure is ready"
  - Emphasize that implementation happens in another skill

### Scope Boundaries

- **Stay in Scope:** Only handle project creation, nothing more
  - Ask for name
  - Ask for location
  - Run command
  - Confirm
  - Stop
- **Quick Exit:** Complete the task and stop immediately
  - No lingering to answer follow-up questions about implementation
  - Direct all follow-ups to m365-agent-developer skill
- **No Planning:** Don't discuss:
  - Architecture decisions
  - Capability selection
  - API plugin design
  - Security considerations
  - Deployment strategies
- **No Implementation:** Don't write or suggest:
  - TypeSpec code
  - Agent decorators
  - Capability configurations
  - API endpoints
  - Instruction text

### Error Handling

- **Validate Input:** Check project name before running command
  - Ensure valid characters
  - Warn about potential issues
  - Suggest corrections if needed
- **Handle Command Failures:** If scaffolding fails:
  - Capture and report error message
  - Suggest common fixes (permissions, disk space, network)
  - Don't proceed with confirmation
- **Provide Context:** When errors occur, give users actionable information
  - What failed
  - Why it might have failed
  - What they can try next
- **Don't Retry Automatically:** Let users diagnose and fix issues
  - Don't run the command multiple times
  - Let users verify prerequisites

### Quality Checks

- **Verify Prerequisites:** Before running command, ensure:
  - Node.js and npm are available
  - User has write permissions
  - Network connectivity for npm packages
- **Confirm Success:** After command completes:
  - Verify project directory exists
  - Check for key files (package.json, src/ directory)
  - Confirm structure was created properly
- **Provide Accurate Paths:** Double-check all paths in messages
  - Use `pwd` or similar to get absolute paths
  - Verify path format for user's OS

### Integration with m365-agent-developer

- **Seamless Transition:** Make it easy for users to move to next skill
  - Provide clear instruction on what to say next
  - Reference m365-agent-developer explicitly by name
- **Set Context:** In handoff message, briefly mention what comes next
  - "Use the m365-agent-developer skill to design, implement, and deploy"
  - Helps users understand the workflow
- **Don't Duplicate:** Trust m365-agent-developer skill for all post-scaffolding work
  - No architectural guidance
  - No implementation hints
  - Pure handoff

