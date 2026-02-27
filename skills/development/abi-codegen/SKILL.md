---
name: abi-codegen
description:
  Convert JSON ABI files to TypeScript const exports with proper typing. Use when working with smart contract ABIs,
  converting JSON to TypeScript, generating typed contract interfaces, or adding new contract ABIs to the SDK.
---

# JSON to TypeScript ABI Converter

## Purpose

Automates the conversion of JSON ABI files to TypeScript files with properly typed `const` exports, ensuring type safety
and consistency across the SDK.

## When to Use

- Adding new contract ABIs to the SDK
- Updating existing ABI files after contract changes
- Batch converting multiple ABI files
- Ensuring ABI TypeScript files match their JSON sources

## How It Works

The skill provides a conversion script that:

1. Reads JSON ABI file(s) from `src/evm/abi/`
2. Generates TypeScript files in `src/evm/releases/` with:
   - Named export using camelCase convention
   - `as const` assertion for full type inference
   - Preserved JSON structure
3. Automatically runs Biome formatting on generated files

## Usage

### Single File Conversion

```bash
bun .claude/skills/abi-codegen/scripts/codegen.ts src/evm/abi/lockup/v3.0/SablierLockup.json
```

### Multiple Files (Glob Pattern)

```bash
bun .claude/skills/abi-codegen/scripts/codegen.ts "src/evm/abi/lockup/v3.0/*.json"
```

### All ABIs in Directory

```bash
bun .claude/skills/abi-codegen/scripts/codegen.ts "src/evm/abi/**/*.json"
```

## File Path Conventions

### Input Path Pattern

```
src/evm/abi/{protocol}/{version}/{ContractName}.json
```

### Output Path Pattern

```
src/evm/releases/{protocol}/{version}/abi/{ContractName}.ts
```

### Naming Convention

- JSON file: `SablierLockup.json`
- TS export: `sablierLockupAbi`
- Pattern: PascalCase filename → camelCase + "Abi" suffix

## Examples

### Example 1: Convert Lockup ABI

**Input:** `src/evm/abi/lockup/v3.0/SablierLockup.json`

```json
[
  {
    "inputs": [
      { "internalType": "address", "name": "to", "type": "address" },
      { "internalType": "uint256", "name": "tokenId", "type": "uint256" }
    ],
    "name": "approve",
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  }
]
```

**Command:**

```bash
bun .claude/skills/abi-codegen/scripts/codegen.ts src/evm/abi/lockup/v3.0/SablierLockup.json
```

**Output:** `src/evm/releases/lockup/v3.0/abi/SablierLockup.ts`

```typescript
export const sablierLockupAbi = [
  {
    inputs: [
      { internalType: "address", name: "to", type: "address" },
      { internalType: "uint256", name: "tokenId", type: "uint256" },
    ],
    name: "approve",
    outputs: [],
    stateMutability: "nonpayable",
    type: "function",
  },
] as const;
```

### Example 2: Batch Convert All v3.0 Lockup ABIs

```bash
bun .claude/skills/abi-codegen/scripts/codegen.ts "src/evm/abi/lockup/v3.0/*.json"
```

**Output:**

```
✓ Converted: SablierLockup.json → SablierLockup.ts
✓ Converted: SablierLockupDynamic.json → SablierLockupDynamic.ts
✓ Converted: SablierLockupLinear.json → SablierLockupLinear.ts
✓ Converted: SablierLockupTranched.json → SablierLockupTranched.ts

Running Biome formatter...
✓ Formatted 4 files

Done! Converted 4 ABI files.
```

## Script Details

### Input Processing

- Accepts absolute or relative file paths
- Supports glob patterns via Bun's `Glob` API
- Validates JSON structure before conversion
- Handles missing or malformed files gracefully

### Output Generation

- Creates necessary directories automatically
- Preserves JSON structure exactly (no data loss)
- Applies consistent naming convention
- Adds `as const` for maximum type inference

### Post-Processing

- Runs `just biome-write` on all generated TypeScript files
- Ensures consistent formatting with project standards
- Reports any formatting errors

## Error Handling

The script handles common errors:

- **File not found**: Clear error message with file path
- **Invalid JSON**: Shows parsing error with line/column
- **Write permission**: Reports directory/file permission issues
- **Biome errors**: Displays formatting failures (non-fatal)

## Troubleshooting

### Script Not Found

```bash
# Ensure you're in the project root
pwd  # Should show .../sablier/sdk

# Verify script exists
ls .claude/skills/abi-codegen/scripts/codegen.ts
```

### Invalid JSON

```bash
# Validate JSON before converting
jq . src/evm/abi/lockup/v3.0/SablierLockup.json
```

### TypeScript Errors After Conversion

```bash
# Run type check to verify
just tsc-check

# If errors occur, check:
# 1. Is the JSON structure valid for an ABI?
# 2. Are all required fields present?
# 3. Run Biome again: just biome-write
```

### Glob Pattern Not Working

```bash
# Use quotes around glob patterns
bun .claude/skills/abi-codegen/scripts/codegen.ts "src/evm/abi/**/*.json"
#                                                 ^                    ^
#                                                 Quotes are required
```

## Integration with Workflow

### After Adding New ABIs

1. Place JSON ABI in `src/evm/abi/{protocol}/{version}/`
2. Run conversion script
3. Verify output in `src/evm/releases/{protocol}/{version}/abi/`
4. Run `just tsc-check` to verify types
5. Export from appropriate index file
6. Run `just test` to ensure no regressions

### Before Committing ABI Changes

```bash
# Convert all modified ABIs
bun .claude/skills/abi-codegen/scripts/codegen.ts "src/evm/abi/**/*.json"

# Verify TypeScript
just tsc-check

# Run tests
just test

# Commit both JSON and generated TS files
git add src/evm/abi/ src/evm/releases/
git commit -m "feat: update ABIs for {protocol} {version}"
```

## Technical Notes

### Why `as const`?

The `as const` assertion provides:

- **Literal types**: String values become literal types (e.g., `"function"` not `string`)
- **Readonly arrays**: Prevents accidental mutations
- **Better inference**: Viem uses literal types for better type checking
- **Autocomplete**: IDEs provide better suggestions

### Why Separate JSON and TS?

- **Source of truth**: JSON files are copied from Foundry artifacts
- **Version control**: Easy to see diffs in JSON changes
- **Type safety**: TypeScript provides compile-time checks
- **Build optimization**: TypeScript files can be tree-shaken

### Performance

- **Typical conversion**: <100ms per file
- **Batch processing**: ~500ms for 10 files
- **Biome formatting**: ~1-2s for all files
- **Total overhead**: Minimal, suitable for pre-commit hooks

## Related Files

- **Script**: `.claude/skills/abi-codegen/scripts/codegen.ts`
- **Source ABIs**: `src/evm/abi/{protocol}/{version}/*.json`
- **Output ABIs**: `src/evm/releases/{protocol}/{version}/abi/*.ts`
- **Build config**: `tsconfig.build.json` (includes ABI copy step)

## Future Enhancements

Potential improvements:

- Watch mode for automatic conversion on JSON changes
- Validation against official ABI schema
- Diff detection to only convert changed files
- Integration with Foundry broadcast parsing
- Automatic export statement generation in index files
