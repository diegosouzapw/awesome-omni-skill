---
name: readme-generator
description: Generate a production-quality README.md by analyzing project structure, framework, and code
version: 1.0.0
author: Sovereign Skills
tags: [openclaw, agent-skills, automation, productivity, free, readme, documentation, generator]
triggers:
  - generate readme
  - create readme
  - write readme
  - readme generator
---

# readme-generator — Production-Quality README Generator

Analyze a project's structure and generate a comprehensive, framework-aware README.md.

## Steps

### 1. Analyze Project Structure

Read these files (if they exist):
- `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` — name, description, version, deps
- `tsconfig.json` — TypeScript config
- `docker-compose.yml` / `Dockerfile` — containerization
- `.github/workflows/` — CI/CD
- `LICENSE` / `LICENSE.md` — license type
- Entry points: `src/index.*`, `src/main.*`, `app.*`, `main.*`, `index.*`
- `tests/` / `test/` / `__tests__/` / `spec/` — test setup

```bash
# Get file tree (depth 3, ignore common dirs)
find . -maxdepth 3 -not -path '*/node_modules/*' -not -path '*/.git/*' -not -path '*/dist/*' -not -path '*/__pycache__/*' | head -100
# Windows alternative:
Get-ChildItem -Recurse -Depth 3 -Exclude node_modules,.git,dist,__pycache__ | Select-Object -First 100 FullName
```

### 2. Detect Framework & Ecosystem

| Signal | Framework |
|--------|-----------|
| `next.config.*` or `"next"` in deps | Next.js |
| `"express"` in deps | Express.js |
| `"fastapi"` in deps | FastAPI |
| `"django"` in deps | Django |
| `"flask"` in deps | Flask |
| `"react"` in deps (no next) | React (CRA/Vite) |
| `"vue"` in deps | Vue.js |
| `"svelte"` in deps | SvelteKit |
| `Cargo.toml` with `[[bin]]` | Rust CLI |
| `Cargo.toml` with `actix-web`/`axum` | Rust Web |
| `go.mod` | Go |

### 3. Determine Install & Run Commands

Based on detected ecosystem:

**Node.js:** Check for lockfiles to determine package manager.
- `pnpm-lock.yaml` → `pnpm install` / `pnpm dev`
- `yarn.lock` → `yarn` / `yarn dev`
- `package-lock.json` → `npm install` / `npm run dev`
- Read `scripts` in package.json for available commands.

**Python:** Check for pip, poetry, pipenv.
- `poetry.lock` → `poetry install` / `poetry run ...`
- `Pipfile` → `pipenv install` / `pipenv run ...`
- `requirements.txt` → `pip install -r requirements.txt`

**Rust:** `cargo build` / `cargo run`

**Go:** `go build` / `go run .`

### 4. Generate Badges

Build badge URLs from detected tools:

```markdown
![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Node](https://img.shields.io/badge/node-%3E%3D18-brightgreen)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue)
```

Only include badges for things actually detected. Common badges: license, language/runtime version, CI status, test coverage.

### 5. Assemble README

Use this structure:

```markdown
# Project Name

Brief description from package.json/pyproject.toml or inferred from code.

## Table of Contents
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [API Reference](#api-reference)  ← only if applicable
- [Configuration](#configuration)  ← only if env vars detected
- [Testing](#testing)
- [Deployment](#deployment)  ← only if Docker/CI detected
- [Contributing](#contributing)
- [License](#license)

## Features
- Bullet list of key capabilities (infer from code structure, routes, exports)

## Prerequisites
- Runtime version (node >= 18, python >= 3.10, etc.)
- Required system tools (Docker, database, etc.)

## Installation
[Package-manager-specific install commands from Step 3]

## Usage
[Dev/start commands, example API calls if it's a server]

## API Reference
[Only for libraries/APIs — list exported functions or endpoints]

## Configuration
[Environment variables — reference env-setup skill if complex]

## Testing
[Test runner command: npm test, pytest, cargo test, etc.]

## Contributing
1. Fork the repository
2. Create your feature branch (`git checkout -b feat/amazing-feature`)
3. Commit your changes (`git commit -m 'feat: add amazing feature'`)
4. Push to the branch (`git push origin feat/amazing-feature`)
5. Open a Pull Request

## License
[Detected license or "See LICENSE file"]
```

### 6. Tailor to Framework

- **Next.js**: Add sections for pages/app router, API routes, environment variables
- **Express/FastAPI**: Document route structure, middleware, API endpoints
- **React/Vue**: Document component structure, state management, build output
- **CLI tools**: Document command-line arguments and flags
- **Libraries**: Focus on API docs, installation, and usage examples

### 7. Output

Write to `README.md` in the project root. If one exists, ask user before overwriting — offer to write to `README.generated.md` instead.

## Edge Cases

- **Monorepo**: Generate a root README linking to sub-packages, plus per-package READMEs
- **Empty project**: Generate a minimal skeleton README with TODOs
- **No package manifest**: Infer from file extensions and directory structure
- **Existing README**: Ask before overwriting; diff and suggest additions

## Error Handling

| Error | Resolution |
|-------|-----------|
| Can't detect framework | Generate a generic README; ask user to specify |
| No description available | Use directory name; prompt user to add one |
| No license file | Note it's missing; suggest adding one |
| Very large project | Limit tree scan depth; focus on src/ and root config |

---
*Built by Clawb (SOVEREIGN) — more skills at [coming soon]*
