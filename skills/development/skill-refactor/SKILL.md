---
name: skill-refactor
description: Transform and refactor existing skills by applying pattern substitutions (package managers, hosting platforms, build tools, testing frameworks). Use when: adapting a skill to use different tools (e.g., pnpm→bun, cloudflare→vercel), migrating project patterns, or aligning skills with your tech stack.
---

# Skill Refactor

Transform existing skills by applying pattern substitutions across all skill files.

## How to Use

```bash
/skill-refactor <transform1> <transform2> [skill-name]
```

**Arguments:**
- `<transform>`: One or more transformations to apply (see list below)
- `[skill-name]`: Optional target skill name (defaults to skill in current context)

**Examples:**

```bash
/skill-refactor bun
/skill-refactor bun vercel
/skill-refactor bun vercel my-skill
/skill-refactor vitest motion
```

## Workflow

### 1. Parse Arguments
Extract transformation types and optional target skill name from command arguments.

### 2. Identify Target Skill
If a skill name is provided, search for it in `.claude/skills/` or `.github/skills/`.
Otherwise, detect the target skill from conversation context (recently read or edited skill files).
If no skill can be identified, ask the user to specify which skill to transform.

### 3. Analyze Target Skill
List all files in the skill directory to understand its structure:
- `SKILL.md` (main skill file)
- `references/` (documentation references)
- `scripts/` (initialization scripts)
- `templates/` (code templates)
- `assets/` (images, diagrams, etc.)

Read the main `SKILL.md` to understand the current state of the skill.

### 4. Load Transformation Patterns
Read the relevant reference files based on the transformation arguments:
- `references/package-managers.md` for: `pnpm`, `bun`, `npm`, `yarn`
- `references/hosting.md` for: `vercel`, `cloudflare`, `netlify`
- `references/build-tools.md` for: `webpack`, `vite`, `esbuild`, `rollup`
- `references/testing.md` for: `jest`, `vitest`, `playwright`, `node:test`

Build a list of search/replace patterns from the reference files.

### 5. Apply Transformations
For each file in the target skill (SKILL.md, references/, scripts/, templates/):
1. Read the file content
2. Apply all relevant search/replace patterns
3. Write the updated content back to the file

Skip binary files in `assets/` (not text files).

### 6. Report Changes
After completing the transformations, report:
- List of files that were modified
- Summary of transformations applied
- Any files that were skipped (binary files, etc.)

## Available Transformations

### Package Managers
Transform between: `pnpm`, `bun`, `npm`, `yarn`

Patterns include: command patterns, lock file references, config file references, documentation phrases.

### Hosting Platforms
Transform between: `vercel`, `cloudflare`, `netlify`

Patterns include: environment variable names, deployment commands, platform-specific imports, configuration file patterns.

### Build Tools
Transform between: `webpack`, `vite`, `esbuild`, `rollup`

Patterns include: CLI command patterns, configuration file formats, plugin/import patterns, build output references.

### Testing Frameworks
Transform between: `jest`, `vitest`, `playwright`, `node:test`

Patterns include: test syntax, assertion patterns, mock/fixture patterns, configuration files.

## Examples

### Convert from pnpm to bun

```bash
/skill-refactor bun
```

Before:
```markdown
pnpm add @vercel/blob
```

After:
```markdown
bun add @vercel/blob
```

### Convert from Vercel to Cloudflare

```bash
/skill-refactor cloudflare my-skill
```

Before:
```typescript
import { put } from '@vercel/blob';
const url = process.env.VERCEL_URL;
```

After:
```typescript
import { put } from '@cloudflare/workers-types';
const url = env.CLOUDFLARE_URL;
```

### Multiple transformations

```bash
/skill-refactor bun vite vitest
```

Converts to: Bun (package manager), Vite (build tool), Vitest (testing framework).

## Notes

- This skill performs text-based pattern replacement. Always review changes after transformation.
- Some transformations may require additional manual adjustments (e.g., configuration file formats differ significantly).
- Binary files in `assets/` are never modified.
- New transformation categories can be added by creating new reference files in `references/`.
