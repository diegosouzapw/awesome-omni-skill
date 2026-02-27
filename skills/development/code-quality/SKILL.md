---
name: code-quality
description: A protocol for performing a deep code quality analysis using static analysis tools and formatting. It covers C++ (cppcheck) and React/JavaScript (ESLint).
license: MIT
metadata:
  author: GitHub Copilot
  version: "2.0"
---


## Agentic Command: /check

When the user writes `/check` in chat, immediately trigger the full code-quality protocol below, without further confirmation. This command acts as a shortcut for a comprehensive code review and static analysis. The agent should proceed automatically, applying all steps unless the user specifies otherwise.

---
## Agentic Code Analysis and Quality Protocol

As an AI assistant, your goal is to perform a comprehensive code quality check using industry-standard tools. When asked to "review the code," "check for bugs," or "improve code quality," follow this protocol.

### Step 1: Identify File Type and Scope

1.  Determine the language of the file(s) to be reviewed:
    *   C++ (`.cpp`, `.h`)
    *   React/JavaScript/CSS (`.jsx`, `.js`, `.scss`)
2.  Ask the user if you should apply formatting changes directly or just provide suggestions.

### Step 2: Run Static Analysis

This is the core of the code review. Run the appropriate tool to find bugs, vulnerabilities, and code smells.

#### For C++ Firmware (`src/` directory):

1.  **Run Static Analysis:** Execute the PlatformIO `check` command.
    ```bash
    platformio check
    ```
2.  Analyze the output for any reported defects, vulnerabilities, or performance issues.

#### For React/JavaScript Frontend (`web/` directory):

1.  **Check for ESLint setup:**
    *   Look for an `.eslintrc.*` file in the root or `web/` directory.
    *   Check `web/package.json` for a `lint` script.
2.  **If ESLint is not configured:**
    *   Inform the user that ESLint is not set up.
    *   Ask for permission to set it up: "I've noticed ESLint isn't configured for the frontend code. It's a powerful tool for finding bugs. Would you like me to set it up for you?"
    *   If yes, proceed to install `eslint` and a standard configuration (e.g., `eslint-plugin-react`). Create a basic `.eslintrc.js` and add a `lint` script to `package.json`.
3.  **If ESLint is configured:**
    *   Execute the linting command.
        ```bash
        npm run lint --prefix web
        ```
    *   Analyze the output for any errors or warnings.

### Step 3: Apply Automatic Formatting

After the deep analysis, apply standard code formatting for consistency.

#### For C++ Firmware:

1. **Formatting:** If the project provides a formatting script (e.g., `scripts/format.sh`) or a documented process, use that. Otherwise, use `clang-format` directly:
    ```bash
    clang-format -i src/**/*.cpp src/**/*.h
    ```
    *(Adjust the file pattern as needed for your project structure.)*
2. If no formatting tool is available, inform the user and offer to help set up `clang-format` with a standard configuration file (e.g., `.clang-format`).

#### For React/JavaScript Frontend:
```bash
npm run format --prefix web
```
*(Note: If the `format` script doesn't exist, offer to set it up with Prettier.)*

### Step 4: Report Findings and Suggestions

1.  **Summarize Static Analysis Results:** Report the critical findings from `cppcheck` or `ESLint`. For each issue, provide the file, line number, and a description of the problem.
2.  **Summarize Formatting:** Mention that the code has been automatically formatted for consistency.
3.  **Offer to Fix:** Ask the user if they would like you to attempt to fix the issues found by the static analysis tools.
