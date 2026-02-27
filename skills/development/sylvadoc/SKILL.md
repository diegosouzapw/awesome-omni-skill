---
name: sylvadoc
description: Les préférences de Sylvadoc pour la confection de projets web en Nuxt/Vue/Typescript.
metadata:
  author: Sylvadoc
  version: "2026.1.29"
---

# Les préférences de Sylvadoc

Ces skills documentent les préférences de Sylvadoc pour la confection de projets web modernes en JavaScript/TypeScript, notamment avec Vue, Nuxt, Vite, et d'autres outils populaires.

## Quick Summary

| Category | Preference                     |
|----------|--------------------------------|
| Package Manager | pnpm                           |
| Language | TypeScript (strict mode)       |
| Module System | ESM (`"type": "module"`)       |
| Linting & Formatting | ESLint & Prettier              |
| Testing | Vitest                         |
| Documentation | VitePress (in `docs/`)         |

---

## Stack principal

### Package Manager (pnpm)

Utiliser pnpm comme package manager.

```json
{
  "packageManager": "pnpm@latest"
}
```

### TypeScript (Strict Mode)

Toujours utiliser TypeScript avec le mode strict activé.

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true
  }
}
```

### Unit Testing (Vitest)

Utiliser Vitest pour les tests unitaires.

```json
{
  "scripts": {
    "test": "vitest"
  }
}
```

**Conventions:**

- Place test files next to source files: `foo.ts` → `foo.test.ts` (same directory)
- High-level tests go in `tests/` directory in each package
- Use `describe` and `it` API (not `test`)
- Use `expect` API for assertions
- Use `assert` only for TypeScript null assertions
- Use `toMatchSnapshot` for complex output assertions
- Use `toMatchFileSnapshot` with explicit file path and extension for language-specific output (exclude those files from linting)

---

## References

### Configurations et fichiers communs

| Topic      | Description | Reference                                                      |
|------------|-------------|----------------------------------------------------------------|
| eslint     | ESLint flat config for formatting and linting | [sylvadoc-eslint-config](references/sylvadoc-eslint-config.md) |
| .gitignore | Preferred .gitignore for JS/TS projects | [gitignore](references/gitignore.md)                           | |
| -------    |-------------|----------------------------------------------------------------|

### Développement Web

| Topic | Description | Reference |
|-------|-------------|-----------|
| css development | Preferences for writing CSS with modern features | [css-development](references/css-development.md)               |
| html development | Preferences for writing semantic and accessible HTML | [html-development](references/html-development.md)             |
| app development | Preferences for Vue/Vite/Nuxt/UnoCSS web applications | [app-development](references/app-development.md)   |
|-------|-------------|-----------|