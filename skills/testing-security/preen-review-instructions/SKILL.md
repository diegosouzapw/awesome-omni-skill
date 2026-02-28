---
name: preen-review-instructions
description: Audit and update code review instructions (REVIEW.md, .gemini/INSTRUCTIONS.md)
---


# Preen Review Instructions

Audit the review instructions in `REVIEW.md` and `.gemini/INSTRUCTIONS.md` for completeness, accuracy, and sync with codebase patterns. Update instructions to reflect current conventions.

## When to Run

Run this skill when:

- Adding new patterns or conventions to the codebase
- Changing API security requirements
- Adding new package types (e.g., new shared libraries)
- After significant architectural changes
- During regular preen passes to catch drift

## Source Files

| File                       | Purpose                                              |
| -------------------------- | ---------------------------------------------------- |
| `REVIEW.md`                | Master review instructions (repo root)               |
| `.gemini/INSTRUCTIONS.md`  | Gemini Code Assist specific instructions             |
| `CLAUDE.md`                | Claude Code conventions (source of truth)            |
| `AGENTS.md`                | Codex conventions (source of truth)                  |
| `compliance/`              | Compliance documentation (security controls)         |
| `compliance/infrastructure-controls.md` | Cross-framework sentinel index         |

## Discovery Phase

### 1. Check REVIEW.md sections exist and are current

```bash
echo "=== REVIEW.md sections ==="
rg '^##' REVIEW.md | head -20

echo "=== Last modified ==="
git log -1 --format="%ci" -- REVIEW.md
```

### 2. Find new patterns not covered in review instructions

```bash
# New route files not mentioned
echo "=== Recent route additions ==="
git log --since="30 days ago" --name-only --pretty=format: -- 'packages/api/src/routes/**/*.ts' | sort -u | head -10

# New shared components not mentioned
echo "=== Recent component additions ==="
git log --since="30 days ago" --name-only --pretty=format: -- 'packages/*/src/components/**/*.tsx' | sort -u | head -10

# New package directories
echo "=== Package structure ==="
ls -d packages/*/ 2>/dev/null | head -10
```

### 3. Check for rules in CLAUDE.md/AGENTS.md not in REVIEW.md

```bash
# TypeScript rules
echo "=== CLAUDE.md TypeScript mentions ==="
rg -i 'typescript|any|cast|@ts-' CLAUDE.md | head -10

# Security rules
echo "=== CLAUDE.md security mentions ==="
rg -i 'security|auth|permission|injection' CLAUDE.md | head -10
```

### 4. Check .gemini/INSTRUCTIONS.md sync with REVIEW.md

```bash
echo "=== Gemini INSTRUCTIONS sections ==="
rg '^##' .gemini/INSTRUCTIONS.md | head -20

# Compare section counts
echo "=== Section comparison ==="
echo "REVIEW.md sections: $(rg '^##' REVIEW.md | wc -l)"
echo "Gemini sections: $(rg '^##' .gemini/INSTRUCTIONS.md | wc -l)"
```

### 5. Verify review scripts reference instructions

```bash
echo "=== Review script prompt analysis ==="
rg -A5 'PROMPT=' scripts/solicitClaudeCodeReview.sh | head -15
rg -A5 'PROMPT=' scripts/solicitCodexReview.sh 2>/dev/null | head -15 || true
```

### 6. Check security and compliance coverage in review instructions

```bash
# Check REVIEW.md has security section
echo "=== REVIEW.md security coverage ==="
rg -c 'Security|OWASP|injection|auth' REVIEW.md

# Check for compliance documentation references
echo "=== Compliance references in REVIEW.md ==="
rg -c 'compliance/|sentinel|TL-' REVIEW.md

# Check infrastructure-controls.md exists and has sentinels
echo "=== Infrastructure controls sentinel count ==="
rg -c 'TL-[A-Z]+-[0-9]+' compliance/infrastructure-controls.md

# Check for framework parity
echo "=== Framework document counts ==="
for fw in HIPAA NIST.SP.800-53 SOC2; do
  echo "$fw policies: $(ls compliance/$fw/policies/*.md 2>/dev/null | wc -l | tr -d ' ')"
done

# Check REVIEW.md mentions sentinel workflow
echo "=== Sentinel workflow in REVIEW.md ==="
rg 'Adding New Sentinel|COMPLIANCE_SENTINEL' REVIEW.md | head -5
```

## Issue Categories

| Category                           | Severity | Action                                       |
| ---------------------------------- | -------- | -------------------------------------------- |
| Missing section for new pattern    | Medium   | Add section to REVIEW.md, sync to Gemini     |
| Outdated package structure         | Low      | Update package list in REVIEW.md             |
| Gemini instructions drift          | Medium   | Sync .gemini/INSTRUCTIONS.md with REVIEW.md  |
| Security rule missing              | High     | Add security guidance from CLAUDE.md         |
| Review script not using instruct.  | Medium   | Update script to include REVIEW.md content   |
| Missing compliance section         | High     | Add compliance documentation guidance        |
| Missing sentinel workflow          | High     | Add instructions for adding new sentinels    |
| Compliance framework parity gap    | Medium   | Ensure all frameworks have equal coverage    |

## Fix Strategies

### Adding a New Section

1. Identify the pattern or convention to document
2. Add section to `REVIEW.md` under appropriate heading
3. Add condensed version to `.gemini/INSTRUCTIONS.md`
4. Update version history at bottom of `REVIEW.md`

### Syncing Files

```bash
# Compare section headers
diff <(rg '^##' REVIEW.md | sort) <(rg '^##' .gemini/INSTRUCTIONS.md | sort)
```

Ensure key sections from `REVIEW.md` have corresponding entries in `.gemini/INSTRUCTIONS.md`.

### Updating Review Scripts

If `solicitClaudeCodeReview.sh` doesn't reference `REVIEW.md`:

```bash
# Read and include instructions
INSTRUCTIONS=$(cat REVIEW.md | head -100)
PROMPT="Review using these guidelines:
$INSTRUCTIONS

[Rest of prompt...]"
```

### Adding Security and Compliance Guidance

When security or compliance guidance is missing from review instructions:

1. **Check `/preen-api-security` skill** for security patterns to document
2. **Check `compliance/infrastructure-controls.md`** for sentinel patterns
3. **Add security section** covering OWASP top 10 awareness
4. **Add compliance section** explaining when to update compliance docs
5. **Add sentinel workflow** explaining how to add new security controls

Required content for security section:

- Authentication/authorization check requirements
- SQL injection prevention (parameterized queries)
- Input validation requirements
- Sensitive data handling
- OWASP Top 10 quick reference

Required content for compliance section:

- Document triad structure (policy/procedure/control-map)
- Sentinel naming convention (`TL-<CATEGORY>-<NUMBER>`)
- When compliance updates are required (new auth, new audit, infra changes)
- How to add new sentinels to `infrastructure-controls.md`

### Updating Compliance Documentation Workflow

When new security controls need compliance documentation:

1. **Add sentinel** to `compliance/infrastructure-controls.md` with:
   - Sentinel ID following naming convention
   - Control description
   - Location (file path)
   - Framework mappings (SOC2, NIST, HIPAA)

2. **Create/update document triad** in each framework:
   - `compliance/<FW>/policies/NN-topic-policy.md`
   - `compliance/<FW>/procedures/NN-topic-procedure.md`
   - `compliance/<FW>/technical-controls/NN-topic-control-map.md`

3. **Update POLICY_INDEX.md** in each framework directory

4. **Add COMPLIANCE_SENTINEL comments** in policy documents:

```markdown
<!-- COMPLIANCE_SENTINEL: TL-NEW-001 | policy=path | procedure=path | control=description -->
```

## Workflow

1. **Discovery**: Run discovery commands to find gaps
2. **Prioritize**: Focus on missing security guidance first
3. **Create branch**: `git checkout -b docs/review-instructions-<date>`
4. **Update REVIEW.md**: Add missing sections/patterns
5. **Sync .gemini/INSTRUCTIONS.md**: Update to match
6. **Validate**: Run discovery again to confirm coverage
7. **Commit and merge**: Run `/commit-and-push`, then `/enter-merge-queue`

If no issues found during discovery, do not create a branch.

## Guardrails

- Do not remove existing sections without explicit justification
- Keep `.gemini/INSTRUCTIONS.md` concise (Gemini has token limits)
- Maintain sync between `REVIEW.md` and `.gemini/INSTRUCTIONS.md`
- Do not duplicate CLAUDE.md/AGENTS.md content verbatim - reference or summarize

## Quality Bar

- All major codebase patterns documented in `REVIEW.md`
- `.gemini/INSTRUCTIONS.md` covers same topics as `REVIEW.md` (condensed)
- Security guidance complete and up-to-date
- Version history current
- No contradictions between instruction files
- Compliance documentation workflow documented (sentinel system, document triads)
- OWASP Top 10 awareness included
- Instructions for adding new security controls and sentinels present
- Cross-references to `compliance/infrastructure-controls.md` and `/preen-compliance-docs`

## Metric

The quality metric is the count of missing or stale sections:

```bash
# Count potential gaps
GAPS=0

# Check for missing sections
[ -z "$(rg 'TypeScript Standards' REVIEW.md)" ] && GAPS=$((GAPS + 1))
[ -z "$(rg 'API Security' REVIEW.md)" ] && GAPS=$((GAPS + 1))
[ -z "$(rg 'React Standards' REVIEW.md)" ] && GAPS=$((GAPS + 1))
[ -z "$(rg 'Database Performance' REVIEW.md)" ] && GAPS=$((GAPS + 1))
[ -z "$(rg 'Testing Standards' REVIEW.md)" ] && GAPS=$((GAPS + 1))

# Check for security and compliance coverage
[ -z "$(rg 'Security and Compliance' REVIEW.md)" ] && GAPS=$((GAPS + 1))
[ -z "$(rg 'OWASP' REVIEW.md)" ] && GAPS=$((GAPS + 1))
[ -z "$(rg 'Adding New Sentinel' REVIEW.md)" ] && GAPS=$((GAPS + 1))
[ -z "$(rg 'COMPLIANCE_SENTINEL' REVIEW.md)" ] && GAPS=$((GAPS + 1))
[ -z "$(rg 'infrastructure-controls.md' REVIEW.md)" ] && GAPS=$((GAPS + 1))

# Check gemini security coverage
[ -z "$(rg 'Security' .gemini/INSTRUCTIONS.md)" ] && GAPS=$((GAPS + 1))
[ -z "$(rg 'compliance/' .gemini/INSTRUCTIONS.md)" ] && GAPS=$((GAPS + 1))

# Check gemini sync
REVIEW_SECTIONS=$(rg '^## ' REVIEW.md | wc -l)
GEMINI_SECTIONS=$(rg '^## ' .gemini/INSTRUCTIONS.md | wc -l)
[ "$GEMINI_SECTIONS" -lt $((REVIEW_SECTIONS / 2)) ] && GAPS=$((GAPS + 1))

echo "$GAPS"
```

## Token Efficiency

```bash
# Limit discovery output
git log --since="30 days ago" --name-only --pretty=format: | head -30

# Suppress validation output
git commit -S -m "message" >/dev/null
git push >/dev/null
```

On failure, re-run without suppression to see errors.
