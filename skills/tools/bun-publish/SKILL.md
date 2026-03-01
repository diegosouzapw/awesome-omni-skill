---
model: haiku
description: Publish package to npm with Bun build
args: [--dry-run] [--access <level>] [--provenance]
allowed-tools: Bash, Read
argument-hint: [--dry-run] [--access public]
disable-model-invocation: true
created: 2025-12-21
modified: 2025-12-21
reviewed: 2025-12-21
name: bun-publish
---

# /bun:publish

Publish package to npm registry after building with Bun.

## Parameters

- `--dry-run`: Preview publish without executing
- `--access`: Access level (`public` or `restricted`)
- `--provenance`: Enable supply chain provenance signing

## Context

Detect package configuration:
```bash
cat package.json | jq '{name, version, publishConfig, bin, files, scripts: {prepublishOnly: .scripts.prepublishOnly}}'
```

## Execution

### Pre-publish Validation

```bash
# Type check
bun run tsc --noEmit 2>&1 | head -20

# Build
bun run build

# Verify tarball contents
npm pack --dry-run
```

### Publish

**Dry run (default for first attempt):**
```bash
npm publish --dry-run {{ "--access " + ACCESS if ACCESS }}
```

**Actual publish:**
```bash
# Standard package
npm publish {{ "--access " + ACCESS if ACCESS }} {{ "--provenance" if PROVENANCE }}

# Scoped package (auto-detect from name starting with @)
npm publish --access public {{ "--provenance" if PROVENANCE }}
```

## Scoped Package Detection

If `package.json` name starts with `@`:
- Automatically add `--access public` unless explicitly restricted
- Warn if `publishConfig.access` is not set in package.json

## Post-publish

1. Display published version: `npm view <package> version`
2. Show installation command: `npm install <package>`
3. Link to npm package page

## Error Handling

**402 Payment Required:**
- Scoped packages require `--access public` for public registry
- Add `publishConfig.access: "public"` to package.json

**Missing authentication:**
- Run `npm login` to authenticate
- Or set `NPM_TOKEN` environment variable
