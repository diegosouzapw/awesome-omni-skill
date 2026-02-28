---
name: opentangl
description: Not a code generator — an entire dev team. You write the vision, it ships the code. Autonomous builds, PRs, reviews, and merges across multiple repos. Point it at any JS/TS project and a product vision. It plans features, writes code, verifies builds, creates PRs, reviews diffs, and merges — autonomously. Manages multiple repos as one product. Use when you want to ship code without writing it. AI code generation, autonomous development, workflow automation, multi-repo orchestration, TypeScript, JavaScript, GitHub, OpenAI, Anthropic, Claude, GPT, LLM, devtools, CI/CD, pull requests, code review.
homepage: https://github.com/8co/opentangl
category: development
tags: ["ai-agents", "code-generation", "autonomous-development", "multi-repo", "typescript", "javascript", "github", "pull-requests", "openai", "anthropic", "claude", "gpt", "llm", "devtools", "workflow-automation", "ci-cd", "code-review", "codegen"]
metadata: {"clawdbot":{"emoji":"🤖","requires":{"bins":["node","git","gh"]}}}
---

# OpenTangl

Configure a self-driving development loop for any JavaScript/TypeScript project. This skill detects your project setup, generates configuration files, and prepares OpenTangl to run autonomously.

**Follow these steps in order.** Complete each step fully before moving to the next. Wait for user confirmation at every gate before proceeding. Do not skip steps or combine them — the user needs to complete actions on their end between steps.

## Prerequisites

The user must have OpenTangl cloned and installed before using this skill. If they haven't, provide these commands for them to run:

```
git clone https://github.com/8co/opentangl.git
cd opentangl
npm install
```

**Do not run these commands on the user's behalf.** Wait for confirmation that OpenTangl is installed.

Once confirmed, verify the required tools are present. Run each check and report the results:

- **Node.js** ≥ 18 — run `node --version` and show the output
- **git** — run `git --version` and show the output
- **GitHub CLI** — run `gh auth status` and show the output (needed for PR creation and merging)

Report all results to the user. If anything is missing, tell them exactly how to install it and stop until resolved.

## Step 1 — Determine the Target Project

Ask the user:

> Are you improving **(a)** an existing project, or **(b)** starting from scratch?

### Path A: Existing Project

1. Ask: **"Where is your project?"** Accept a path. If they say "this directory," use cwd.
2. Tell the user you'll read config files in their project directory to detect the setup. Only inspect files in the directory the user provided — do not scan outside it. Check:
   - **Type**: `tsconfig.json` → TypeScript, `vite.config.ts` → Vite, `next.config.*` → Next.js, `serverless.yml` → Serverless
   - **Package manager**: `package-lock.json` → npm, `yarn.lock` → yarn, `pnpm-lock.yaml` → pnpm
   - **Build/test commands**: Read `package.json` scripts for `build`, `test`, `lint`, `typecheck`
   - **Source dirs**: Default to `src/` if it exists
   - **Target branch**: Check `git symbolic-ref refs/remotes/origin/HEAD` or look for `main` vs `master`
3. Show everything you detected and confirm with the user before proceeding.
4. Ask: "Are there other repos that are part of this same product?" If yes, repeat detection for each.

### Path B: New Project

Tell the user to scaffold and initialize their project before continuing. Suggest the appropriate tool based on what they want to build:

- React + Vite: `npm create vite@latest {name} -- --template react-ts`
- Next.js: `npx create-next-app@latest {name} --typescript`
- Express: create `package.json` + `src/index.ts` manually

They should also initialize git and create a GitHub repo:

```
cd {name}
git init && git add . && git commit -m "Initial scaffold"
gh repo create {name} --public --source . --push
```

**Do not run these commands on the user's behalf.** Once they confirm the project exists with a GitHub remote, continue.

## Step 2 — Generate projects.yaml

Create `projects.yaml` in the OpenTangl root directory. Each project entry needs:

```yaml
projects:
  - id: my-app                          # Short kebab-case ID (used in CLI flags)
    name: my-app                        # Human-readable name
    path: ../my-app                     # Relative path from OpenTangl root to the project
    type: react-vite                    # Project type (see below)
    description: React dashboard app    # One-line description
    scan_dirs:
      - src                             # Directories containing source code
    skip_patterns:
      - node_modules
      - dist
      - "*.test.*"
    verify:                             # Commands that must pass before committing
      - command: npm
        args: [run, build]
    package_manager: npm                # npm | yarn | pnpm
    merge:
      target_branch: main               # Branch PRs merge into
```

**Supported types:** `typescript-node`, `serverless-js`, `serverless-ts`, `react-vite`, `react-next`, `express` (or any descriptive string).

For **multi-project setups**, add an `environment` field to group related projects under a shared vision:

```yaml
  - id: my-api
    environment: my-product
    # ...
  - id: my-frontend
    environment: my-product
    # ...
```

## Step 3 — Create the Vision Doc

Create `docs/environments/{environment}/product-vision.md` (use the project `id` as environment name for single projects, or the `environment` field for multi-project).

The vision doc has two sections:

### Origin & Direction (human-authored, never modified by OpenTangl)

Ask the user to describe:
- **What This Is** — 2-3 sentences about the project
- **Where It's Going** — long-term direction, 6-12 months out
- **What Matters Most** — 3-5 principles guiding decisions

### Current Priorities (maintained by OpenTangl after each run)

Ask: **"What are the first 3-5 things you want built or improved?"**

Write them as Active Initiatives:

```markdown
### Active Initiatives

1. **{Priority}** — {What and why}
   - Status: not started
```

If the user isn't sure, offer to read the codebase and suggest priorities.

## Step 4 — Configure the LLM

The user needs to create a `.env` file in the OpenTangl root with their API key. **Do not accept or handle API keys directly** — provide the template and let the user create the file themselves.

First, verify that `.env` appears in the project's `.gitignore` by reading the file. If it does not, add it and tell the user.

Then provide the appropriate template for the user to fill in:

**For OpenAI:**
```
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o
DEFAULT_AGENT=openai
```

**For Anthropic (Claude):**
```
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_MODEL=claude-sonnet-4-20250514
DEFAULT_AGENT=anthropic
```

Tell the user: "Create a `.env` file in the OpenTangl root and paste one of the templates above with your key. This file is gitignored and will never be committed."

Wait for confirmation before continuing.

## Step 5 — Prepare the First Run

Initialize an empty task queue:

```bash
mkdir -p tasks
echo "tasks: []" > tasks/queue.yaml
```

Then provide the user with the command to start the autopilot. **Do not run this command on the user's behalf** — show it and let them execute it:

For a single project:

```
npx tsx src/cli.ts autopilot --projects {project-id} --cycles 1 --feature-ratio 0.8
```

For multi-project:

```
npx tsx src/cli.ts autopilot --projects {api-id},{ui-id} --cycles 1 --feature-ratio 0.8
```

**What happens during a cycle:**
1. OpenTangl reads the vision doc and scans the codebase
2. It proposes tasks aligned with the vision
3. It executes each task — writes code, runs verification
4. It creates PRs, reviews them with the LLM, merges if clean
5. It updates the vision doc with progress

Tell the user to review the results after the first run — check the generated PRs and the updated vision doc.

## Troubleshooting

- **"No pending tasks"** — The queue is empty. Run autopilot to have the LLM propose tasks, or add more specific priorities to the vision doc.
- **Build failures** — OpenTangl retries up to 3 times with error feedback. If all attempts fail, the task is marked failed and skipped.
- **Escalated PRs** — The LLM reviewer flagged critical concerns. Check the GitHub issue it created for details.
- **"OPENAI_API_KEY is required"** — Create `.env` and add your key (see Step 4).
- **Merge conflicts** — OpenTangl has a built-in conflict resolver. If it can't resolve automatically, the PR is escalated for human review.
