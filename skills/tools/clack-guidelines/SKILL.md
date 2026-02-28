---
name: clack-guidelines
description: Comprehensive guide for building beautiful interactive command-line interfaces using Clack. Use when creating CLI tools with text input, selections, autocomplete, progress tracking, and streaming output.
---

# Clack Guidelines

## Overview

This skill provides guidance for building beautiful interactive command-line interfaces using Clack. It covers common patterns for text input, selections, autocomplete, progress tracking, streaming output, and creating complete interactive CLI applications.

## Quick Start

### Installation

```bash
npm install @clack/prompts
```

### Basic Program

```javascript
import { text, isCancel, cancel } from "@clack/prompts";

const projectPath = await text({
  message: "Where should we create your project?",
  placeholder: "./my-awesome-project",
  initialValue: "./sparkling-solid",
  validate: (value) => {
    if (!value) return "Please enter a path.";
    if (value[0] !== ".") return "Please enter a relative path.";
  },
});

if (isCancel(projectPath)) {
  cancel("Operation cancelled.");
  process.exit(0);
}

console.log(`Creating project at: ${projectPath}`);
```

### Complete Interactive CLI

```javascript
import * as p from "@clack/prompts";
import color from "picocolors";

async function createApp() {
  console.clear();
  p.intro(color.bgCyan(color.black(" create-myapp ")));

  const config = await p.group(
    {
      name: () =>
        p.text({
          message: "What is your project name?",
          placeholder: "my-awesome-app",
          validate: (value) => {
            if (!value) return "Project name is required";
          },
        }),

      framework: () =>
        p.select({
          message: "Choose your framework",
          options: [
            { value: "react", label: "React", hint: "Popular" },
            { value: "vue", label: "Vue" },
            { value: "svelte", label: "Svelte" },
          ],
        }),

      typescript: () =>
        p.confirm({
          message: "Use TypeScript?",
          initialValue: true,
        }),
    },
    {
      onCancel: () => {
        p.cancel("Setup cancelled");
        process.exit(0);
      },
    },
  );

  console.log(config);
}

createApp().catch(console.error);
```

## Common Patterns

### Text Input

Single-line text input with validation and placeholders.

```javascript
import { text } from "@clack/prompts";

const name = await text({
  message: "Enter your name",
  placeholder: "John Doe",
  validate: (value) => {
    if (!value) return "Name is required";
  },
});
```

### Password Input

Secure password input with masking.

```javascript
import { password } from "@clack/prompts";

const userPassword = await password({
  message: "Provide a password",
  mask: "•",
  validate: (value) => {
    if (!value) return "Please enter a password.";
    if (value.length < 8) return "Password should have at least 8 characters.";
  },
});
```

### Autocomplete Single Selection

Type-ahead search with real-time filtering.

```javascript
import { autocomplete } from "@clack/prompts";

const country = await autocomplete({
  message: "Select a country",
  options: [
    { value: "us", label: "United States", hint: "NA" },
    { value: "ca", label: "Canada", hint: "NA" },
    { value: "uk", label: "United Kingdom", hint: "EU" },
  ],
  placeholder: "Type to search countries...",
  maxItems: 8,
  initialValue: "us",
});
```

### Autocomplete Multi-Select

Type-ahead search with multi-selection.

```javascript
import { autocompleteMultiselect } from "@clack/prompts";

const frameworks = await autocompleteMultiselect({
  message: "Select frameworks (type to filter)",
  options: [
    { value: "react", label: "React", hint: "Frontend/UI" },
    { value: "vue", label: "Vue.js", hint: "Frontend/UI" },
    { value: "nextjs", label: "Next.js", hint: "React Framework" },
  ],
  placeholder: "Type to filter...",
  maxItems: 8,
  initialValues: ["react", "nextjs"],
  required: true,
});
```

### Single Selection Menu

Choose one option from a scrollable list.

```javascript
import { select } from "@clack/prompts";

const projectType = await select({
  message: "Pick a project type",
  initialValue: "ts",
  maxItems: 5,
  options: [
    { value: "ts", label: "TypeScript" },
    { value: "js", label: "JavaScript" },
    { value: "rust", label: "Rust" },
    { value: "go", label: "Go" },
  ],
});
```

### Multi-Select Menu

Select multiple options from a list.

```javascript
import { multiselect } from "@clack/prompts";

const tools = await multiselect({
  message: "Select additional tools",
  initialValues: ["prettier", "eslint"],
  required: true,
  options: [
    { value: "prettier", label: "Prettier", hint: "recommended" },
    { value: "eslint", label: "ESLint", hint: "recommended" },
    { value: "stylelint", label: "Stylelint" },
  ],
});
```

### Confirmation Dialog

Binary yes/no choice.

```javascript
import { confirm } from "@clack/prompts";

const shouldInstallDeps = await confirm({
  message: "Install dependencies?",
  active: "Yes, please",
  inactive: "No, skip",
  initialValue: false,
});
```

### Progress Bar

Visual progress indicator for long-running operations.

```javascript
import { progress } from "@clack/prompts";
import { setTimeout } from "node:timers/promises";

const downloadProgress = progress({
  style: "block",
  max: 100,
  size: 40,
});

downloadProgress.start("Downloading packages");

for (let i = 0; i < 100; i += 10) {
  await setTimeout(500);
  downloadProgress.advance(10);
  downloadProgress.message(`Downloaded ${i + 10}%`);
}

downloadProgress.stop("Download complete!");
```

### Spinner Loading Indicator

Display progress for long-running operations.

```javascript
import { spinner } from "@clack/prompts";
import { setTimeout } from "node:timers/promises";

const s = spinner();

s.start("Installing packages via pnpm");
await setTimeout(2000);

s.message("Downloading dependencies");
await setTimeout(1500);

s.stop("Installation complete!");
```

### Task Log with Live Output

Display streaming output that clears on success and persists on error.

```javascript
import { taskLog } from "@clack/prompts";
import { spawn } from "node:child_process";

const log = taskLog({
  title: "Running npm install",
  limit: 10,
  retainLog: true,
  spacing: 1,
});

const npmInstall = spawn("npm", ["install"]);

npmInstall.stdout.on("data", (data) => {
  log.message(data.toString(), { raw: true });
});

npmInstall.on("close", (code) => {
  if (code === 0) {
    log.success("Installation complete!");
  } else {
    log.error("Installation failed!", { showLog: true });
  }
});
```

### Streaming Output for LLMs

Stream dynamic content from async iterables.

```javascript
import { stream } from "@clack/prompts";
import { setTimeout } from "node:timers/promises";

await stream.step(
  (async function* () {
    const words = "Building your application with modern tools".split(" ");
    for (const word of words) {
      yield word;
      yield " ";
      await setTimeout(100);
    }
  })(),
);
```

### Grouped Prompts with Sequential Execution

Execute multiple prompts in sequence with shared state.

```javascript
import * as p from "@clack/prompts";
import color from "picocolors";

p.intro(color.bgCyan(color.black(" create-app ")));

const project = await p.group(
  {
    path: () =>
      p.text({
        message: "Where should we create your project?",
        placeholder: "./sparkling-solid",
        validate: (value) => {
          if (!value) return "Please enter a path.";
          if (value[0] !== ".") return "Please enter a relative path.";
        },
      }),

    type: ({ results }) =>
      p.select({
        message: `Pick a project type within "${results.path}"`,
        initialValue: "ts",
        options: [
          { value: "ts", label: "TypeScript" },
          { value: "js", label: "JavaScript" },
          { value: "rust", label: "Rust" },
        ],
      }),

    tools: () =>
      p.multiselect({
        message: "Select additional tools",
        options: [
          { value: "prettier", label: "Prettier", hint: "recommended" },
          { value: "eslint", label: "ESLint", hint: "recommended" },
        ],
      }),

    install: () =>
      p.confirm({
        message: "Install dependencies?",
        initialValue: true,
      }),
  },
  {
    onCancel: () => {
      p.cancel("Operation cancelled.");
      process.exit(0);
    },
  },
);

console.log(project.path, project.type, project.tools, project.install);
```

### Task Execution with Spinners

Run multiple tasks with automatic spinner management.

```javascript
import * as p from "@clack/prompts";
import { setTimeout } from "node:timers/promises";

await p.tasks([
  {
    title: "Installing dependencies",
    task: async (message) => {
      message("Downloading packages");
      await setTimeout(1000);
      message("Building native modules");
      await setTimeout(500);
      return "Installed 127 packages";
    },
  },
  {
    title: "Running linter",
    task: async () => {
      // Run linter
      return "No issues found";
    },
  },
]);
```

### Logging Messages

Display formatted status messages with different severity levels.

```javascript
import { log } from "@clack/prompts";
import color from "picocolors";

log.info("Starting build process...");
log.step("Compiling TypeScript");
log.success("Build completed successfully!");
log.warn("Some dependencies are outdated");
log.error("Failed to connect to database");

// Custom symbol
log.message("Custom message", {
  symbol: color.cyan("→"),
});
```

### Intro, Outro, and Notes

Frame your CLI sessions with beautiful headers.

```javascript
import { intro, outro, note } from "@clack/prompts";
import color from "picocolors";

intro(color.bgCyan(color.black(" my-cli-tool v2.0.0 ")));

// ... prompts and operations ...

note("cd my-project\nnpm install\nnpm run dev", "Next steps");

outro(
  `Problems? ${color.underline(color.cyan("https://github.com/myorg/myproject/issues"))}`,
);
```

### Custom Key Bindings

Configure alternative key mappings for navigation.

```javascript
import { updateSettings } from "@clack/prompts";

// Enable WASD navigation
updateSettings({
  aliases: {
    w: "up",
    s: "down",
    a: "left",
    d: "right",
  },
});
```

### Non-Interactive Mode Detection

Handle CI environments where prompts aren't supported.

```javascript
import { text, block } from "@clack/prompts";

// In CI environments, prompts automatically use defaults
const name = await text({
  message: "Project name?",
  defaultValue: "default-project",
});

// block() utility prevents input in non-TTY environments
const unblock = block();
// Perform operations
unblock();
```

### Cancellation and Error Handling

Robust error handling for user cancellations and validation errors.

```javascript
import * as p from "@clack/prompts";

async function setupProject() {
  try {
    const responses = await p.group({
      name: () =>
        p.text({
          message: "Project name?",
          validate: (v) => {
            if (!v) return "Required";
          },
        }),

      confirm: ({ results }) =>
        p.confirm({
          message: `Create "${results.name}"?`,
        }),
    });

    // Check if any step was cancelled
    if (p.isCancel(responses.name) || p.isCancel(responses.confirm)) {
      p.cancel("Setup cancelled");
      return;
    }

    if (!responses.confirm) {
      p.outro("Setup aborted");
      return;
    }

    // Proceed with setup
    p.outro("Project created!");
  } catch (error) {
    p.log.error(`Setup failed: ${error.message}`);
    process.exit(1);
  }
}

setupProject();
```

## Advanced Features

### Custom Render Function with @clack/core

Build custom prompts with full control over rendering.

```javascript
import { TextPrompt } from "@clack/core";
import color from "picocolors";

const customPrompt = new TextPrompt({
  validate: (value) => (value.length < 3 ? "Too short!" : undefined),
  render() {
    const title = `>>> ${color.bold("Enter your name")}:`;
    const input = this.valueWithCursor || color.dim("(empty)");

    switch (this.state) {
      case "error":
        return `${title}\n${color.red(input)}\n${color.red(this.error)}`;
      case "submit":
        return `${title} ${color.green(this.value)}`;
      case "cancel":
        return `${title} ${color.strikethrough(this.value)}`;
      default:
        return `${title}\n${color.cyan(input)}`;
    }
  },
});

const result = await customPrompt.prompt();
console.log(`Result: ${result}`);
```

### Event-Driven Custom Prompt

Subscribe to prompt events for advanced interactions.

```javascript
import { TextPrompt } from "@clack/core";

const prompt = new TextPrompt({
  render() {
    return `Input: ${this.valueWithCursor}`;
  },
});

// Listen to events
prompt.on("value", (value) => {
  console.log(`Current value: ${value}`);
});

prompt.on("submit", () => {
  console.log("User submitted the form");
});

prompt.on("cancel", () => {
  console.log("User cancelled");
});

const result = await prompt.prompt();
```

### Complete CLI Application Example

Full-featured setup wizard combining multiple prompt types.

```javascript
import { setTimeout } from "node:timers/promises";
import * as p from "@clack/prompts";
import color from "picocolors";

async function createApp() {
  console.clear();

  p.intro(color.bgCyan(color.black(" create-myapp ")));

  const config = await p.group(
    {
      name: () =>
        p.text({
          message: "What is your project name?",
          placeholder: "my-awesome-app",
          validate: (value) => {
            if (!value) return "Project name is required";
            if (!/^[a-z0-9-]+$/.test(value)) {
              return "Use only lowercase letters, numbers, and hyphens";
            }
          },
        }),

      framework: () =>
        p.select({
          message: "Choose your framework",
          options: [
            { value: "react", label: "React", hint: "Popular" },
            { value: "vue", label: "Vue" },
            { value: "svelte", label: "Svelte" },
            { value: "solid", label: "Solid" },
          ],
        }),

      features: () =>
        p.multiselect({
          message: "Select features",
          required: false,
          options: [
            { value: "router", label: "Router" },
            { value: "state", label: "State Management" },
            { value: "testing", label: "Testing Setup" },
            { value: "ci", label: "CI/CD Pipeline" },
          ],
        }),

      typescript: () =>
        p.confirm({
          message: "Use TypeScript?",
          initialValue: true,
        }),

      install: () =>
        p.confirm({
          message: "Install dependencies now?",
          initialValue: true,
        }),
    },
    {
      onCancel: () => {
        p.cancel("Setup cancelled");
        process.exit(0);
      },
    },
  );

  // Execute setup tasks
  await p.tasks([
    {
      title: "Creating project structure",
      task: async () => {
        // Create project files
        return `Created ${config.name}`;
      },
    },
    {
      title: "Installing dependencies",
      task: async (message) => {
        message("Resolving packages...");
        await setTimeout(1000);
        message("Downloading...");
        await setTimeout(2000);
        return "Installed 42 packages";
      },
      enabled: config.install,
    },
  ]);

  // Display next steps
  const nextSteps = [
    `cd ${config.name}`,
    config.install ? "" : "npm install",
    "npm run dev",
  ]
    .filter(Boolean)
    .join("\n");

  p.note(nextSteps, "Next steps");

  p.outro(
    `All set! ${color.underline(color.cyan("https://docs.example.com"))}`,
  );
}

async function createProjectFiles(config) {
  // Implementation...
}

createApp().catch(console.error);
```

## Detailed API Reference

For comprehensive API documentation including all prompt types, configuration options, and advanced features, see the detailed API reference:

**[references/clack-api.md](references/clack-api.md)**

This reference includes:
- Complete prompt type documentation
- Configuration options for each prompt
- Validation and error handling patterns
- Streaming and async patterns
- Custom key bindings and settings
- Integration with @clack/core
- TypeScript support
- And more

## Resources

### references/
This skill includes detailed API documentation:

- **clack-api.md** - Comprehensive API reference for Clack prompts, organized by prompt type with code examples for each feature.
