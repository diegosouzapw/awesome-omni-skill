---
name: biome
description: Configure BiomeJS for projects - linting, formatting, and code style setup. Use when the user asks to set up biome, configure linting or formatting, migrate from eslint or prettier, enforce code style, or add biome to a project.
license: MIT
metadata:
  author: agentskills
  version: "1.0.0"
allowed-tools: Bash(npm:*, npx:*)
---

# BiomeJS Configuration

Set up and configure BiomeJS for linting, formatting, and code style enforcement.

## Version Discovery

**Always discover the latest version before doing anything.** Never hardcode a biome version.

```bash
npm view @biomejs/biome version
```

Use the discovered version for:
- Installing: `npm install --save-dev --save-exact @biomejs/biome@<version>`
- Schema URL: `https://biomejs.dev/schemas/<version>/schema.json`

## New Project Setup

1. **Discover the latest version** (see above)
2. **Install biome**:
   ```bash
   npm install --save-dev --save-exact @biomejs/biome@<version>
   ```
3. **Create `biome.json`** at the project root using the baseline configuration below
4. **Add package.json scripts** (see Scripts section)
5. **Verify** with `npx biome check .`

## Migration / Upgrade

When a project already has biome configured:

1. Discover the latest version
2. Update the dependency: `npm install --save-dev --save-exact @biomejs/biome@<version>`
3. Update the `$schema` URL in `biome.json` to match the new version
4. Run `npx biome check .` to verify nothing breaks
5. Review and modernize the config — only override what differs from biome's defaults

## Baseline Configuration

Use this as the starting point for all projects. Adapt based on project context (see Adaptive Rules below).

```json
{
  "$schema": "https://biomejs.dev/schemas/<version>/schema.json",
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "semicolons": "always"
    }
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "complexity": {
        "noExcessiveCognitiveComplexity": "warn"
      },
      "correctness": {
        "noUnusedImports": "error",
        "noUnusedVariables": "warn",
        "useArrayLiterals": "error"
      },
      "style": {
        "noDefaultExport": "error",
        "noCommonJs": "error",
        "noNamespaceImport": "warn",
        "useFilenamingConvention": {
          "level": "error",
          "options": {
            "strictCase": true,
            "filenameCases": ["camelCase", "kebab-case"]
          }
        }
      },
      "suspicious": {
        "noConsole": "warn"
      }
    }
  },
  "css": {
    "linter": {
      "enabled": true
    },
    "formatter": {
      "enabled": true,
      "quoteStyle": "single"
    }
  },
  "organizeImports": {
    "enabled": true
  }
}
```

## Adaptive Rules

Adjust the baseline based on what you detect in the project:

### No CSS in project
If there are no CSS, SCSS, or similar files, remove the `css` section entirely.

### Next.js / Remix / Astro / SvelteKit
These frameworks require default exports for pages and routes. Disable the rule or scope it:
```json
"style": {
  "noDefaultExport": "off"
}
```
Or use `overrides` to allow default exports only in route files:
```json
"overrides": [
  {
    "includes": ["src/app/**", "src/pages/**", "app/**", "pages/**"],
    "linter": {
      "rules": {
        "style": {
          "noDefaultExport": "off"
        }
      }
    }
  }
]
```

### CommonJS projects
If the project uses `"type": "commonjs"` or lacks `"type": "module"` in package.json and uses `require()`:
```json
"style": {
  "noCommonJs": "off"
}
```

### Tailwind CSS
No special biome config needed — Tailwind works with biome out of the box. Just ensure CSS linting is enabled if the project has CSS files.

### Monorepo
Place `biome.json` at the workspace root. Use `overrides` for package-specific rules if needed.

### Test files
Allow `noConsole` in test files:
```json
"overrides": [
  {
    "includes": ["**/*.test.*", "**/*.spec.*", "**/__tests__/**"],
    "linter": {
      "rules": {
        "suspicious": {
          "noConsole": "off"
        }
      }
    }
  }
]
```

## Package.json Scripts

Add these scripts to the project's `package.json`:

```json
{
  "scripts": {
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "format": "biome format --write ."
  }
}
```

## Verification

After setup, always verify:

```bash
npx biome check .
```

If there are many existing violations, consider using `--write` to auto-fix what biome can:

```bash
npx biome check --write .
```

For remaining issues that can't be auto-fixed, either fix them or adjust rules as appropriate for the project.
