---
name: evolve-skill
version: 1.0.0
created: 2026-02-02
last-evolved: null
evolution-count: 0
feedback-count: 0
description: Analyze feedback and evolve skills through structured improvement. The meta-skill that makes other skills better.
argument-hint: [skill-name] or [--all] or [rollback skill-name] or [versions skill-name]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Evolve Skill

Analyze feedback and improve skills through structured evolution. This meta-skill enables skills to learn and improve over time.

## Arguments

- `$0` - Skill name to evolve (e.g., `add-chart`)
- Or special commands:
  - `--all` - Analyze all skills
  - `--analyze-only {skill}` - Just show analysis, no changes
  - `rollback {skill}` - Rollback to previous version
  - `rollback {skill} {version}` - Rollback to specific version
  - `versions {skill}` - List available versions

---

## Mode 1: Evolve a Skill

### Step 1: Gather Feedback

Read all feedback files for the target skill:

```
.claude/skills/.feedback/${$0}/*.yaml
```

Parse each feedback file and aggregate:
- Outcome distribution (success/partial/failed)
- Common strengths (patterns that appear multiple times)
- Common corrections (fixes that were needed repeatedly)
- Common suggestions (frequently requested improvements)

### Step 2: Analyze Patterns

Categorize improvements by type:

1. **Instructions** - Steps that need clarification or reordering
2. **Templates** - Code patterns that need updating
3. **Verification** - Missing checklist items
4. **Tools** - Tool permissions that should change

Look for patterns:
- If 2+ feedback entries mention the same correction → likely needed
- If 2+ feedback entries suggest the same improvement → worth considering
- If success rate < 70% → skill needs significant improvement

### Step 3: Propose Improvements

Present proposals to the user using `AskUserQuestion`:

```
Based on {N} feedback entries for '{skill-name}':
- Success rate: {X}%
- Common corrections: {list}
- Common suggestions: {list}

Proposed improvements:
1. {Improvement 1} - rationale from feedback
2. {Improvement 2} - rationale from feedback
...

Which improvements should we apply?
```

Options should include:
- Apply all proposed improvements
- Select specific improvements
- Skip (just update metadata)

### Step 4: Apply Changes (with approval)

For each approved improvement:

1. **Save current version to .versions/**
   ```
   cp SKILL.md .versions/{current-version}.md
   ```

2. **Edit SKILL.md** with the improvement
   - Be precise with edits
   - Maintain skill structure and formatting
   - Add new sections if needed

3. **Update metadata:**
   ```yaml
   version: {new-version}  # Bump appropriately
   last-evolved: {today's date}
   evolution-count: {previous + 1}
   ```

4. **Update CHANGELOG.md:**
   ```markdown
   ## [{new-version}] - {date}

   ### Changed
   - {Description of change 1}
   - {Description of change 2}

   ### Feedback-driven
   Based on {N} feedback entries. Key patterns addressed:
   - {Pattern 1}
   - {Pattern 2}
   ```

### Step 5: Commit Changes

Create a git commit with:

```bash
git add .claude/skills/${$0}/
git commit -m "evolve(${$0}): {brief description}

Based on {N} feedback entries:
- {Key improvement 1}
- {Key improvement 2}

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
```

### Step 6: Report

Tell the user:
- Changes applied
- New version number
- Suggest testing the skill with `/skill-name` to verify

---

## Mode 2: Rollback

### `rollback {skill}` - Previous Version

1. Read current version from SKILL.md frontmatter
2. Find previous version in `.versions/`
3. Copy previous version to SKILL.md
4. Update metadata (decrement evolution-count)
5. Add CHANGELOG entry noting rollback
6. Commit with message: `revert(${skill}): rollback to {version}`

### `rollback {skill} {version}` - Specific Version

1. Verify version exists in `.versions/{version}.md`
2. Copy that version to SKILL.md
3. Update metadata appropriately
4. Add CHANGELOG entry
5. Commit

---

## Mode 3: List Versions

### `versions {skill}`

1. List all files in `.versions/` directory
2. Show version, date (from filename or metadata), and brief description
3. Indicate current version

Output format:
```
Versions for '{skill}':
  1.0.0 - 2026-02-02 - Initial release
  1.1.0 - 2026-02-15 - Added ErrorBoundary pattern (current)
```

---

## Mode 4: Analyze All

### `--all`

1. Scan all skill directories
2. For each skill with feedback:
   - Calculate success rate
   - Count pending feedback entries
   - Identify skills most in need of evolution

Output format:
```
Skill Evolution Status:
  add-chart      - 5 feedback entries, 60% success, needs evolution
  add-api-route  - 2 feedback entries, 100% success, healthy
  add-dashboard  - 0 feedback entries, no data
  ...
```

Recommend which skills to evolve first.

---

## Version Numbering

Use semantic versioning:

- **Patch (1.0.x)** - Small fixes, clarifications
  - Typo fixes
  - Minor wording improvements
  - Adding missing details

- **Minor (1.x.0)** - New content, significant improvements
  - New sections added
  - Template code updated
  - Verification steps added

- **Major (x.0.0)** - Breaking changes (rare for skills)
  - Complete restructure
  - Argument changes
  - Fundamental approach change

---

## Feedback Analysis Patterns

### Detecting Instruction Issues
```yaml
# If corrections mention "order" or "sequence"
corrections:
  - "Had to reorder the steps"
  - "Step 3 should come before Step 2"
→ Instruction ordering issue
```

### Detecting Template Issues
```yaml
# If corrections mention specific code
corrections:
  - "Fixed import order"
  - "Added missing ErrorBoundary"
→ Template needs updating
```

### Detecting Missing Guidance
```yaml
# If suggestions mention "add" or "include"
suggestions:
  - "Add section about error handling"
  - "Include z-index best practices"
→ Missing documentation
```

---

## Verification Checklist

After evolution:

1. [ ] Previous version saved to `.versions/`
2. [ ] SKILL.md updated with improvements
3. [ ] Version bumped appropriately
4. [ ] `last-evolved` date updated
5. [ ] `evolution-count` incremented
6. [ ] CHANGELOG.md entry added
7. [ ] Git commit created
8. [ ] User notified of changes

After rollback:

1. [ ] Correct version restored
2. [ ] Metadata updated
3. [ ] CHANGELOG notes rollback
4. [ ] Git commit created
