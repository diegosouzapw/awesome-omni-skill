---
name: dev-swarm-install-ai-code-agent
description: Install AI code agent CLI tools including claude-code, gemini-cli, codex, and github copilot-cli. Use when setting up AI coding assistants or when the user asks to install an AI code agent.
---

# AI Builder - Install AI Code Agent

This skill helps you install AI code agent CLI tools on various platforms. It supports installing claude-code, gemini-cli, codex, github copilot-cli, and can search for other AI code agents.

## When to Use This Skill

- User wants to set up claude-code, gemini-cli, codex, copilot-cli, or others

## Your Roles in This Skill

- **DevOps Engineer**: Handle platform detection, execute installation commands, verify installation success, and troubleshoot installation issues.
- **Tech Manager**: Coordinate the installation process, gather requirements, present installation options, and ensure proper verification.

## Role Communication

As an expert in your assigned roles, you must announce your actions before performing them using the following format:

- As a DevOps Engineer, I will {action description}
- As a Tech Manager, I will {action description}

**Note:** Combine multiple roles when performing related tasks. For example: "As a Tech Manager and DevOps Engineer, I will detect your platform and provide installation options."

This communication pattern ensures transparency and allows for human-in-the-loop oversight at key decision points.

## Instructions

Follow these steps to install an AI code agent:

### Step 1: Identify the AI Code Agent

Ask the user which AI code agent they want to install if not already specified. Route to the appropriate reference file:

- If installing **claude-code**, refer to `references/claude-code.md`
- If installing **gemini-cli**, refer to `references/gemini-cli.md`
- If installing **codex**, refer to `references/codex.md`
- If installing **copilot-cli**, refer to `references/copilot-cli.md`
- If installing **other AI code agents**, perform a web search for installation instructions

### Step 2: Detect Platform and Available Tools

As a DevOps Engineer, detect the user's platform and check which installation tools are available:

1. Check if npm is available (requires Node.js 18+)
2. Check if curl is available
3. Check if brew (Homebrew) is available

Priority order: **npm → curl → brew**

### Step 3: Present Installation Options

Read the appropriate reference file and present the installation options in priority order (npm first, curl second, brew third).

Recommend the highest priority method that's available on the user's system

### Step 4: Execute Installation

As a DevOps Engineer, execute the user's chosen installation method:

1. Confirm the installation command with the user
2. Run the installation command using the Bash tool
3. Monitor the output for errors or warnings
4. Handle any installation issues (missing dependencies, permissions, etc.)

### Step 5: Verify Installation

After installation completes, verify the AI code agent was installed successfully:

1. Check the version using the verification command from the reference file
2. Verify the command is in PATH and executable
3. Report success or any issues to the user

### Step 6: Provide Setup Instructions

As a Tech Manager, provide the user with:

1. The command to launch the AI code agent
2. First-time setup instructions from the reference file

## Expected Output

After successfully running this skill:

1. The AI code agent CLI tool is installed on the system
2. The installation is verified with a version check
3. The user knows how to launch and set up the tool
4. Documentation links are provided

## Key Principles

- **Installation priority**: Prefer npm → curl → brew (in that order based on availability)
- **Platform awareness**: Always detect and use platform-appropriate commands
- **Multiple options**: Present multiple installation methods when available
- **Verification**: Always verify installation success before concluding
- **User choice**: Let the user choose their preferred installation method
- **Clear guidance**: Provide launch instructions and setup steps

## Common Issues

**Issue: Command not found after installation**
- Solution: Check if the installation directory is in PATH, restart shell, or reinstall

**Issue: Node.js version too old for npm installation**
- Solution: Update Node.js to version 18+ or use alternative installation method

**Issue: Homebrew not installed**
- Solution: Install Homebrew first or use alternative installation method

**Issue: Installation script fails**
- Solution: Check internet connection, try alternative method, or check system logs
