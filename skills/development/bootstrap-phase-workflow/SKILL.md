---
name: bootstrap-phase-workflow
description: Integrate the vibe/mature phase workflow into a project
disable-model-invocation: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---

# Bootstrap Phase Workflow

Integrate the two-phase (vibe → mature) development workflow into the current project. The phase-workflow repo is located at `../phase-workflow` relative to the target project.

## Steps

### 1. Copy scaffold files

Copy everything from `../phase-workflow/scaffold/` into the project root, preserving directory structure:

- `.claude/phase`
- `.claude/settings.json`
- `.claude/skills/vibe/SKILL.md`
- `.claude/skills/phase/SKILL.md`
- `.husky/pre-commit`
- `.husky/commit-msg`
- `.lintstagedrc.json`
- `commitlint.config.js`

Make `.husky/pre-commit` and `.husky/commit-msg` executable.

If `.claude/settings.json` already exists, merge the hooks from the scaffold into the existing file rather than overwriting.

### 2. Detect project structure

Read through the project to determine:

- **Entry points**: Find the main entry files (e.g., `src/main.tsx`, `src/index.ts`, `api/*.ts`). Look at existing `package.json` scripts and `tsconfig*.json` for hints.
- **Source directories**: Identify which directories contain source code (e.g., `src/`, `api/`, `lib/`).
- **Core modules**: Identify 3-5 files that contain the most important business logic — these are candidates for mutation testing in the mature skill.
- **TSConfig files**: Find tsconfig files used for different source directories (e.g., `tsconfig.app.json`, `tsconfig.api.json`).
- **Existing ESLint config**: Check if one already exists.

### 3. Generate knip.json

Read `../phase-workflow/templates/knip.json` as a reference. Create `knip.json` in the project root with the detected entry points and project globs.

### 4. Generate eslint.config.js

Read `../phase-workflow/templates/eslint.config.js` as a reference. Create `eslint.config.js` adapted to the project:

- One config block per source directory
- Each block references the correct tsconfig
- Frontend blocks include react-hooks and react-refresh plugins
- Backend/API blocks omit react plugins
- All blocks use `strictTypeChecked`

If the project already has an ESLint config, upgrade it to `strictTypeChecked` and add the phase-workflow rules rather than replacing it entirely.

### 5. Generate mature skill

Read `../phase-workflow/templates/mature-SKILL.md` as a reference. Create `.claude/skills/mature/SKILL.md` with the `{{CORE_MODULES}}` placeholder replaced with the actual core module paths detected in step 2.

### 6. Update package.json

Read `../phase-workflow/templates/devDependencies.json` and `../phase-workflow/templates/scripts.json`. Merge them into the project's `package.json`:

- Add missing devDependencies (don't overwrite existing versions)
- Add missing scripts (don't overwrite existing scripts)
- Ensure `"prepare": "husky"` is present

### 7. Install dependencies

Run `npm install`.

### 8. Generate CLAUDE.md

If the project doesn't have a `CLAUDE.md`, create one. If it does, add the phase workflow section.

The CLAUDE.md should include:

- **Two-Phase Development Model** section (copy from the scaffold — this is standard)
- **Architecture** section — generate a text diagram by reading through the project's source files
- **Project Structure** section — list the key directories and what they contain
- **Conventions** section — note TypeScript strict mode, ESLint strictTypeChecked, complexity limit, and any project-specific patterns observed
- **TypeScript Design Guidance** section (copy from the scaffold — this is standard):
  - Prefer discriminated unions over boolean flags
  - Use `satisfies` over `as`
  - Let TypeScript infer return types
  - Type-narrow errors in catch blocks
  - Avoid `!` non-null assertions

### 9. Confirm

Tell the user what was set up and that the project is now in **vibe** phase. Mention that `/vibe`, `/mature`, and `/phase` skills are available.
