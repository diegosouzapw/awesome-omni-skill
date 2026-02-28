---
name: oracle
description: Use the @steipete/oracle CLI to bundle a prompt plus the right files and get a second-model review (API or browser) for debugging, refactors, design checks, or cross-validation.
---

# Oracle (CLI) — best use

Oracle bundles your prompt + selected files into one "one-shot" request so another model can answer with real repo context (API or browser automation). Treat outputs as advisory: verify against the codebase + tests.

## Recommended Modes (in order of preference)

### Option 1: Gemini Browser Mode (RECOMMENDED)

Most reliable browser mode. Uses cookie-based HTTP client (no Chrome automation needed).

```bash
# Pre-authorize Keychain first (one-time, enter password and click "Always Allow")
security find-generic-password -s "Chrome Safe Storage" -w

# Run with Gemini
oracle --engine browser --model gemini-3-pro -p "<task>" --file "src/**"
```

**Requirements**: Logged into gemini.google.com in Chrome. Close Chrome before first run.

### Option 2: ChatGPT Manual Paste

**ChatGPT browser automation is unreliable** (DevTools connection issues, session token problems). Use `--render --copy` instead:

```bash
# Bundle and copy to clipboard
oracle --render --copy -p "<task>" --file "src/**"
```

Then manually paste into ChatGPT. This bypasses all browser automation issues.

### Option 3: API Mode

Requires API keys but most reliable for programmatic use:

```bash
# Set API keys
export OPENAI_API_KEY="your-key"      # For GPT-5.x
export GEMINI_API_KEY="your-key"      # For Gemini
export ANTHROPIC_API_KEY="your-key"   # For Claude

# Run with API
oracle --engine api --model gpt-5.2-pro -p "<task>" --file "src/**"
oracle --engine api --model gemini-3-pro -p "<task>" --file "src/**"
oracle --engine api --model claude-4.5-sonnet -p "<task>" --file "src/**"
```

**Note**: API runs require explicit user consent due to usage costs.

## Quick Reference

| Model | Mode | Command |
|-------|------|---------|
| Gemini 3 Pro | Browser | `oracle --engine browser --model gemini-3-pro -p "..." --file "..."` |
| GPT-5.2 Pro | Manual | `oracle --render --copy -p "..." --file "..."` → paste in ChatGPT |
| Any model | API | `oracle --engine api --model <model> -p "..." --file "..."` |

## Commands

```bash
# Preview (no tokens spent)
oracle --dry-run summary -p "<task>" --file "src/**"

# Token/cost check
oracle --dry-run summary --files-report -p "<task>" --file "src/**"

# Gemini browser (recommended)
oracle --engine browser --model gemini-3-pro -p "<task>" --file "src/**"

# Manual paste for ChatGPT
oracle --render --copy -p "<task>" --file "src/**"

# API mode
oracle --engine api -p "<task>" --file "src/**"
```

## Attaching files (`--file`)

`--file` accepts files, directories, and globs. Pass multiple times; comma-separated also works.

**Include**:
```bash
--file "src/**"           # directory glob
--file src/index.ts       # literal file
--file docs --file README.md
```

**Exclude** (prefix with negation):
```bash
--file "src/**" --file '!src/**/*.test.ts' --file '!**/*.snap'
```

**Defaults**:
- Ignores: `node_modules`, `dist`, `coverage`, `.git`, `.turbo`, `.next`, `build`, `tmp`
- Honors `.gitignore`
- Files > 1 MB rejected

## Budget

- Target: keep total input under ~196k tokens
- Use `--files-report` to spot token hogs

## Sessions

Sessions stored under `~/.oracle/sessions`. If a run detaches:
```bash
oracle status --hours 72        # List sessions
oracle session <id> --render    # Reattach
```

Use `--slug "<3-5 words>"` for readable session IDs.

## Prompt Tips

Oracle starts with **zero** project knowledge. Include:
- Project briefing (stack, build/test commands)
- Key directories and entrypoints
- Exact error text (verbatim)
- Constraints ("don't change X", "must keep public API")
- Desired output format

## Safety

- Don't attach secrets (`.env`, credentials, auth tokens)
- Prefer minimal context: fewer files + better prompt beats whole-repo dumps

## Troubleshooting

### macOS Keychain blocks Gemini browser mode

**Symptom**: `Failed to read macOS Keychain (Chrome Safe Storage): Timed out after 3000ms`

**Fix**: Pre-authorize Keychain:
```bash
security find-generic-password -s "Chrome Safe Storage" -w
# Enter password, click "Always Allow"
```

### Chrome cookies not found (Gemini)

**Symptom**: `missing __Secure-1PSID/__Secure-1PSIDTS`

**Fixes**:
1. Log into gemini.google.com in Chrome
2. Close Chrome completely before running Oracle
3. Specify profile path if using multiple profiles:
   ```bash
   oracle --engine browser --model gemini-3-pro \
     --browser-cookie-path ~/Library/Application\ Support/Google/Chrome/Profile\ 2/Cookies \
     -p "..." --file "..."
   ```

### ChatGPT browser mode not working

**Symptom**: `Cannot GET /json/list`, `Inspected target navigated or closed`, DevTools errors

**Cause**: ChatGPT browser automation uses Chrome DevTools Protocol which is unreliable. It launches a new Chrome instance with a temp profile, and session tokens often don't transfer properly.

**Solution**: Don't use ChatGPT browser mode. Use one of:
1. `--render --copy` then manually paste into ChatGPT
2. API mode with `OPENAI_API_KEY`
3. Gemini browser mode instead
