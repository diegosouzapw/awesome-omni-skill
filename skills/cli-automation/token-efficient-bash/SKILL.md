---
name: token-efficient-bash
description: Write compact bash scripts using exec tracing pattern. Triggers when writing bash scripts with 3+ sequential commands. The exec 2>&1 + set -xeuo pipefail pattern eliminates echo statements via automatic command tracing, reducing script size by 40-60%.
version: 0.1.0
---

# Token-Efficient Bash Scripting

Write bash scripts that provide automatic progress diagnostics and fail-fast error handling without manual echo statements or verbose error reporting.

## The Pattern

For sequential bash commands (3+ steps), use this header with intent comment:

```bash
# Intent: high-level description of what this block does
exec 2>&1
set -xeuo pipefail

# Script commands here
```

**Intent comment required:** First line must be a comment explaining the block's purpose (before `exec 2>&1`). This provides context for why the script exists, while `set -x` handles per-command tracing.

## Per-Command Comments Not Needed

The `set -x` flag eliminates the need for per-command WHAT comments - the traced output documents what each command does:

```bash
# Reorganize source files into target directory
exec 2>&1
set -xeuo pipefail

mkdir -p target/
mv source/* target/
ln -s ../target source
```

**Output shows exactly what happens:**
```
+ mkdir -p target/
+ mv source/* target/
+ ln -s ../target source
```

Each command is printed before execution. No need for `# Create directory` or `# Move files` comments after each line. The traced output IS the per-command documentation.

**Two types of comments:**
- **Intent comment (required):** First line explains overall purpose of the block
- **Per-command comments (not needed):** Individual WHAT comments eliminated by `set -x` tracing
- **Exception:** Complex logic may still warrant inline comments explaining WHY (business logic, non-obvious decisions), not WHAT (commands being run)

## How It Works

### `exec 2>&1`
Redirects stderr to stdout, merging all output into single stream.

**Purpose:** Ensures tracing output (`set -x` writes to stderr) appears in correct order with command output.

**Result:** Unified output stream with interleaved commands and results.

### `set -xeuo pipefail`

Four orthogonal flags:

**`-x` (xtrace):**
- Print each command before execution
- Prefixed with `+` in output
- **This is the key to token economy** - eliminates echo statements
- Shows actual commands with expanded variables

**`-e` (errexit):**
- Exit immediately if any command returns non-zero
- Prevents error cascades
- No need for `|| exit 1` after every command

**`-u` (nounset):**
- Error on undefined variable references
- Catches typos and missing environment variables
- Prevents silent failures from `$TYPO` expanding to empty string

**`-o pipefail`:**
- Pipeline fails if ANY command in pipe fails
- Normally pipes return exit status of last command only
- Example: `grep pattern file | wc -l` fails if grep fails

## When to Use

**Use this pattern for:**
- Sequential operations (3+ commands)
- File operations requiring error checking (mv, cp, ln, mkdir)
- Multi-step git workflows (add, commit, push)
- Setup/teardown sequences
- Any script where you'd otherwise write echo statements

**Do NOT use for:**
- Single commands (just run the command)
- Parallel operations (use multiple Bash tool calls)
- Interactive commands (set -x interferes with prompts)
- Large process substitutions (tracing overhead)
- Commands requiring complex error handling (use traditional if/else)

## Strict Mode Caveats

Some commands have expected non-zero exits that would trigger `set -e`. Handle with `|| true`.

### Commands with Normal Non-Zero Exits

**`grep` - exits 1 when no match:**

```bash
# Search for pattern and continue processing
exec 2>&1
set -xeuo pipefail

# grep returns 1 when pattern not found - this is normal
grep "pattern" file.txt || true

# Continue with other commands
echo "Processing complete"
```

**`diff` - exits 1 when files differ:**

```bash
# Compare files and continue with other operations
exec 2>&1
set -xeuo pipefail

# diff returns 1 when files differ - expected behavior
diff file1.txt file2.txt || true

# Other commands still fail-fast
```

**Other common cases:**
- `test` / `[` / `[[` - false condition returns 1
- `git diff --exit-code` - exits 1 if differences exist
- `make` with certain targets
- Background jobs that may fail

### Bash Arithmetic Expansion

**Problem:** `(( ))` returns exit status equal to the expression value.

```bash
# Increment counter (BROKEN)
exec 2>&1
set -xeuo pipefail

count=0
((count++))  # FAILS - expression evaluates to 0, triggers set -e
```

**Solution 1:** Use `|| true`

```bash
# Increment counter with let
exec 2>&1
set -xeuo pipefail

count=0
let count++ || true  # Safe
```

**Solution 2:** Use `$(( ))` (arithmetic substitution, not expansion)

```bash
# Increment counter with arithmetic substitution
exec 2>&1
set -xeuo pipefail

count=0
count=$((count + 1))  # Safe - always succeeds
```

### Conditional Logic with Expected Failures

When a command failure is part of normal flow:

```bash
# Check configuration and branch on result
exec 2>&1
set -xeuo pipefail

# Check if pattern exists (grep exit 1 is expected)
if grep -q "pattern" config.yaml || true; then
    echo "Pattern found"
else
    echo "Pattern not found"
fi

# Or use grep in condition directly (if handles exit status)
if grep -q "pattern" config.yaml; then
    echo "Found"
fi  # No || true needed in if condition
```

**Worked examples:** Read `references/examples.md` for file operations, git workflow, and setup-with-expected-failures examples with output.

**Directory changes:** Read `references/directory-changes.md` for trap-based and subshell approaches to safe directory changes.

**Anti-patterns:** Read `references/anti-patterns.md` for common mistakes (redundant error handling, single-command overhead, blanket error suppression).
