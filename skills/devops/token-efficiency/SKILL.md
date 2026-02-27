---
name: token-efficiency
description: Minimize token consumption when reading files and gathering context
autoload: true
---

# Token Efficiency

**Load this skill at session start.** Token waste impacts cost and speed.

## OpenCode Read Protection

OpenCode requires reading a file before editing/writing it (for existing files). **Partial reads satisfy this requirement** - but you must read the section you intend to change.

**Why this matters:** The protection prevents overwriting content you haven't seen. If you read lines 1-50 but edit lines 200-250, you risk corrupting code you never reviewed.

```
# CORRECT - read the area you're changing
read({ filePath: "src/app.ts", offset: 200, limit: 50 })
edit({ filePath: "src/app.ts", oldString: "// line 220 content", newString: "..." })

# WRONG - reading unrelated section then editing elsewhere
read({ filePath: "src/app.ts", offset: 0, limit: 50 })
edit({ filePath: "src/app.ts", oldString: "// line 220 content", newString: "..." })  # Dangerous!
```

## File Reading Guidelines

### When Full Reads Are Acceptable
- Small files (<50 lines)
- Data files you need to analyze completely (JSON, CSV, config)
- Files where you need to understand the full structure
- New/unfamiliar codebases where you need context

### When to Use Partial Reads
- Large files (>100 lines) where you're editing a specific section
- Files you've already read in this session
- When grep/codegraph already told you the line numbers

### Tool Priority for Finding Code

1. `grep` → Find line numbers and file paths
2. `codegraphcontext` → Find symbol definitions and relationships
3. `glob` → Find file paths by pattern
4. `read` with `offset`/`limit` → Get targeted content

### Read Tool Usage

```
# Full read - fine for small files or when you need everything
read({ filePath: "config.json" })

# Partial read - preferred for large files
read({ filePath: "src/app.ts", offset: 100, limit: 50 })
```

## Edit vs Write Tool Selection

### Use `edit` when:
- Modifying existing code (most common case)
- You know the exact text to replace
- Changing a specific section of a large file

**Edit workflow:**
```
1. grep/codegraph → find location (line ~245)
2. read with offset/limit → get the lines around your target (read what you'll change)
3. edit({ filePath, oldString: "<exact match from step 2>", newString: "<replacement>" })
```

**Important:** Your `oldString` must come from what you actually read. Don't guess at content.

### Use `write` when:
- Creating new files (no prior read needed)
- Replacing entire small file content
- File is small AND you're changing most of it

### Token Comparison

| Scenario | edit approach | write approach |
|----------|---------------|----------------|
| Change 10 lines in 500-line file | Read ~50 lines | Read 500 lines |
| Change 100 lines in 500-line file | Read ~120 lines | Read 500 lines |
| Create new 50-line file | N/A | Write 50 lines (no read) |
| Rewrite small config (20 lines) | Either works | Either works |

## Common Patterns

**Editing a function you found with grep:**
```
grep "handleRequest" src/ → found at src/handler.ts:145
read({ filePath: "src/handler.ts", offset: 140, limit: 50 })
edit({ filePath: "src/handler.ts", oldString: "...", newString: "..." })
```

**Creating a new file:**
```
write({ filePath: "src/new-feature.ts", content: "..." })
# No prior read needed for new files
```

**Analyzing a data file:**
```
# Full read is appropriate here
read({ filePath: "data/config.json" })
```

## Anti-Patterns to Avoid

❌ Reading 500-line file to change 5 lines (use partial read)
❌ Using `write` to modify existing large files (use `edit`)
❌ Re-reading same file sections repeatedly (remember what you read)
❌ Reading full file "just in case" when grep already found the lines
