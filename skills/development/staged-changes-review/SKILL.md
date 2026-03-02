---
name: staged-changes-review
description: Checklist-driven review of staged Git changes with deterministic rule scanning and semantic analysis. Use when the user wants to review staged changes, check for errors before commit, or validate code quality before committing.
allowed-tools: Bash, Read, Grep, Glob
metadata:
  version: "2.1.0"
---

# Staged Changes Review

Checklist-driven, deterministic review of Git staged changes. Produces reproducible findings by combining grep-based scanning with closed-question semantic analysis.

## 输出要求

**重要：本技能必须使用中文输出所有分析结果和建议。**

- **语言**: 始终使用中文回复
- **编码**: UTF-8
- **风格**: 专业、简洁，发现必须附带代码证据

## Workflow (8 Steps)

Execute steps in strict order: **0 → 0.5 → 1 → 2 → 2.5 → 3 → 4 → 5**. Do not skip or reorder any step.

### Step 0: Collect Changes

```bash
git diff --cached
git diff --cached --stat
git diff --cached --name-status
```

If no staged changes exist, output "无暂存变更，审查终止。" and stop.

### Step 0.5: Detect Project Profile

Detect the project type **once** to determine which rule categories to activate.

```bash
# Detect framework
python3 -c "
import json, sys
try:
    with open('package.json') as f:
        d = json.load(f)
    deps = {**d.get('dependencies', {}), **d.get('devDependencies', {})}
    if 'next' in deps:
        print('react-nextjs')
    elif 'react' in deps:
        print('react-app')
    else:
        print('generic')
except:
    print('generic')
" 2>/dev/null || echo "generic"

# Count staged file types
git diff --cached --name-only | grep -cE '\.(tsx|jsx)$' || echo 0
git diff --cached --name-only | grep -cE '\.py$' || echo 0
```

Record the detected **profile** — it determines active rule categories:

| Profile | Condition | Active Rule Categories |
|---------|-----------|----------------------|
| **react-nextjs** | `next` in dependencies | SEC + REACT + PERF + ASYNC + STR + LOGIC/BREAK |
| **react-app** | `react` but no `next` | SEC + REACT + ASYNC + STR + LOGIC/BREAK |
| **python-generic** | Python files majority, no package.json | SEC + STR + LOGIC/BREAK |
| **generic** | No package.json or detection failed | SEC + STR + LOGIC/BREAK |

**REACT-\*** and **PERF-\*** rules are **skipped entirely** for `python-generic` or `generic` profiles. If detection fails, default to `generic`.

### Step 1: Classify Files

Classify every staged file into priority tiers using the rules in `references/review-rules.md` §1.

- Assign each file exactly one priority (P0–P4).
- **Skip P4 files entirely** — do not read or scan them.
- Sort remaining files by priority (P0 first, P3 last).
- Record the file list for the report's "审查范围" section.

### Step 2: Deterministic Scan

For each file (P0 → P3 order), execute **every applicable** deterministic rule from `references/review-rules.md` §2.

**Execution method:**
1. Use grep patterns from `references/language-patterns.md` matching the file's language.
2. Run each grep against the **staged diff** (`git diff --cached -- <file>`) or the file itself.
3. If a pattern matches, record a finding immediately — no subjective filtering.
4. Use the severity from `references/review-rules.md` §4 — do NOT modify severity.

**MANDATORY**: Every rule × file combination must be executed or explicitly marked N/A (wrong file type or inactive profile). Do not skip rules because "they seem unlikely."

### Step 2.5: Impact Scope Analysis

For P0/P1 files, run a one-time impact scope assessment and record results for the "影响范围" section.

```bash
# 1. New/modified exported symbols in staged diff
git diff --cached -U0 -- "*.ts" "*.tsx" | grep -E "^\+export (function|const|class|interface|type)"

# 2. API route changes
git diff --cached --name-only | grep -E "app/api/|pages/api/"

# 3. Server Action changes (security impact)
git diff --cached -- "*.ts" "*.tsx" | grep -E '^[+-].*("use server"|'"'"'use server'"'"')'

# 4. Environment variable changes
git diff --cached -- ".env*" 2>/dev/null | grep -E "^[+-][A-Z_]+" | head -10
```

For each exported symbol found (up to 5 symbols), check reference count:

```bash
# Replace SYMBOL_NAME with actual symbol
grep -rl "SYMBOL_NAME" --include="*.ts" --include="*.tsx" src/ 2>/dev/null | wc -l
```

If no impact scope items are detected, note "无显著影响范围变更" in the report.

### Step 3: Semantic Review

For each file (P0 → P3 order), answer **every applicable** semantic question from `references/review-rules.md` §3.

**Execution method:**
1. Read the file's staged diff and surrounding context (use `Read` tool for full file when needed).
2. For each semantic rule, answer the closed question with exactly one of:
   - **YES** — issue found. MUST provide: file:line + evidence snippet.
   - **NO** — reviewed, no issue found.
   - **N/A** — rule does not apply to this file type or change.
3. Only YES answers produce findings. Use the severity from §4.

**MANDATORY**: Answer every question. Do not skip questions because context seems insufficient — answer N/A with a brief reason instead.

### Step 4: Merge & Deduplicate

1. Collect all findings from Steps 2 and 3.
2. Generate a fingerprint for each finding: `{ruleId}:{file}:{line}`.
3. Deduplicate by fingerprint — if the same fingerprint appears from both a deterministic and semantic rule, keep the one with higher severity.
4. Group findings by severity: CRITICAL → HIGH → MEDIUM → LOW.
5. Apply output budget limits from `references/report-template.md` §3.

### Step 5: Generate Report

Output the report using the exact structure from `references/report-template.md` §2.

1. **审查范围**: file counts and rule counts.
2. **发现总览**: severity summary table.
3. **结论**: auto-select from `references/review-rules.md` §5 based on highest severity found.
4. **分级发现**: findings in `references/report-template.md` §1 format, grouped by severity.
5. **文件审查矩阵**: per-file summary table.

## Execution Constraints (MANDATORY)

These constraints are non-negotiable. Violating any constraint invalidates the review.

1. **No skipping**: Every rule must be evaluated against every applicable file. Document N/A explicitly.
2. **No free-form analysis**: Only report findings that match a defined rule ID. Do not invent new categories.
3. **Forced answers**: Every semantic question must receive YES, NO, or N/A. Leaving a question unanswered is forbidden.
4. **Evidence lock**: Every YES finding must cite a specific file:line and include a code snippet. Findings without evidence are invalid.
5. **Severity is immutable**: Use the severity from the mapping table. Do not upgrade or downgrade based on subjective judgment.
6. **Conclusion is templated**: The conclusion must be one of the four templates in review-rules.md §5. Do not write a custom conclusion.
7. **Fingerprint required**: Every finding must include a fingerprint in `{ruleId}:{file}:{line}` format.

## Context Budget Strategy

When staged changes are large (>500 lines or >15 files):

- **P0 files**: Always read full content — security-critical.
- **P1 files**: Read full diff. If a single file diff >200 lines, focus on: function signatures, exported symbols, error handling blocks, and security-sensitive patterns.
- **P2 files**: Read diff only. Focus on SEC rules and BREAK-003/BREAK-004.
- **P3 files**: Scan diff headers only. Apply STR rules only.
- If total context would exceed capacity, process files strictly by priority and stop at capacity with a note: "剩余 N 个低优先级文件未审查".

**REACT/ASYNC rule sampling limit**: For rules that require Read tool for semantic verification (REACT-001, REACT-003, ASYNC-001, ASYNC-002), if grep returns more than 5 matches, take the **first 5 matches only** and note "N 处未验证（采样限制）". This prevents excessive Read tool calls.

## Validator Interface Contract

This skill's output is consumed by `staged-review-validator`. The following fields are required for compatibility:

- Finding title must include `[ruleId]` prefix (e.g., `[SEC-003] SQL 拼接风险`)
- Each finding must include a `指纹` (Fingerprint) field
- Severity uses 4 levels: CRITICAL / HIGH / MEDIUM / LOW
- Report structure follows `references/report-template.md` §2

## References

- `references/review-rules.md` — Rule definitions, file classification, severity mapping
- `references/report-template.md` — Finding format, report structure, output budget
- `references/language-patterns.md` — Language-specific grep patterns by rule ID
