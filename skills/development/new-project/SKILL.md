---
name: new-project
description: Scaffold a new full-stack project with backend (Scalingo) and Next.js frontend (Cloudflare Pages), initialize git/GitHub, and plan using GitHub issues.
title: /new-project
parent: Skills
permalink: /skills/new-project/
nav_order: 4
---

# New Project

Scaffold and bootstrap a new full-stack project. This skill is both the project creator and the foundation for how the project is planned and tracked — using GitHub issues as the planning database.

## Input Handling

- **Argument required**: project name (kebab-case), e.g. `/new-project my-app`
- Optionally followed by a description: `/new-project my-app "A tool that does X"`
- If no argument, ask the user for project name and description

## Your Task

### 1. Gather Information

Ask the user using AskUserQuestion:
- **Project description** (if not provided as argument)
- **Backend stack**: Python (FastAPI) or Node.js (NestJS)
- **Scalingo region**: osc-fr1 (default) or osc-secnum-fr1
- **GitHub visibility**: Public or private

### 2. Create Project Structure

```
~/dev/<project-name>/
├── api/                        # Backend
├── web/                        # Next.js frontend
├── docs/                       # Product documentation (empty)
├── CLAUDE.md                   # Project instructions for Claude Code
├── PLAN.md                     # Agent team workflow
└── README.md
```

### 3. Backend Setup (api/)

#### If Python (FastAPI):

Scaffold a minimal FastAPI app with:
- `uv` as package manager (pyproject.toml, uv.lock)
- Python 3.12 (.python-version)
- `Procfile` for Scalingo: `web: PYTHONPATH=. python -m uvicorn api.main:app --host 0.0.0.0 --port $PORT`
- Minimal FastAPI app entry point with a `/health` endpoint
- Dev tooling: `ruff` (linter/formatter), `ty` (type checker), `pytest`
- Standard .gitignore for Python

#### If Node.js (NestJS):

Scaffold a minimal NestJS app with:
- `pnpm` as package manager
- `Procfile` for Scalingo: `web: node dist/main.js`
- Minimal NestJS app with a `/health` endpoint
- Dev tooling: `eslint`, `prettier`
- Standard .gitignore for Node

### 4. Frontend Setup (web/)

Scaffold a Next.js 16 app with:
- `pnpm` as package manager
- TypeScript, Tailwind CSS 4, shadcn/ui
- OpenNext.js + Cloudflare Workers adapter (wrangler.toml, open-next.config.ts)
- Dev tooling: `eslint`, `prettier`
- Scripts: `dev`, `build`, `preview` (opennextjs-cloudflare), `deploy` (opennextjs-cloudflare)
- `.env.local` with `NEXT_PUBLIC_API_URL=http://localhost:8000/api`
- `.env.production` with `NEXT_PUBLIC_API_URL=https://<project-name>-api.<region>.scalingo.io/api`
- Minimal API client in `src/lib/api.ts` using `NEXT_PUBLIC_API_URL`
- Standard .gitignore for Next.js + Cloudflare

### 5. Root Files

- **CLAUDE.md**: Project-level instructions including:
  - Project overview and tech stack
  - How to run backend and frontend locally
  - Deployment targets (Scalingo for api/, Cloudflare Pages for web/)
  - Lint/test commands for each stack
- **README.md**: Project title, description, tech stack table, quick start commands
- **PLAN.md**: Agent team workflow with 4 phases:
  1. Discovery (product + devils-advocate)
  2. Design (product + devils-advocate)
  3. Build (backend-dev + frontend-dev, supervised by devils-advocate)
  4. Validate (devils-advocate)

Adapt roles and deliverables to the specific project.

### 6. Initialize Git and GitHub

1. `git init`
2. `git add -A && git commit -m "feat: scaffold project structure"`
3. `gh repo create <project-name> --<visibility> --source . --push`

### 7. Plan Using GitHub Issues

Use GitHub issues as the planning database. Create an initial set of issues based on the PLAN.md phases using `gh issue create`. The agent should document its plan, decisions, and progress as GitHub issues — not just at project creation but as an ongoing practice. Issues should be created and updated as the project evolves.

### 8. Install Dependencies

Run in parallel:
- Backend: `uv sync` (Python) or `pnpm install` (Node)
- Frontend: `pnpm install`

### 9. Final Report

Print a summary with project location, GitHub repo URL, created issue URLs, and quick start commands.

## Guidelines

**DO:**
- Keep the scaffold minimal - just enough to run and deploy
- Use modern tooling: uv/ruff/ty (Python), pnpm/eslint/prettier (Node/TS)
- Create a CLAUDE.md so Claude Code understands the project from the start
- Use GitHub issues as the single source of truth for planning
- Create the `docs/` directory (empty, ready for specs)

**DON'T:**
- Don't add business logic - just the bare scaffold
- Don't deploy anything - just set up the structure

## Example Usage

```
/new-project my-app
/new-project my-app "A marketplace for vintage furniture"
```
