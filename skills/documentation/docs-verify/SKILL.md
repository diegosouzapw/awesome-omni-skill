---
name: docs-verify
description: Validates documentation quality and freshness — checks for broken links, stale content, llms.txt sync, image issues, heading hierarchy, and badge URLs. Runs locally or in CI. Use to catch documentation decay before it reaches users.
version: "1.3.0"
---

# Documentation Verifier

## Philosophy

Generating documentation is a solved problem. **Preventing documentation decay** is not. This skill validates that generated docs remain accurate, linked, and fresh over time.

## Verification Checks

### 1. Markdown Lint

Check heading hierarchy and structural consistency across all documentation files.

```bash
# Find all documentation Markdown files
find . -name "*.md" -not -path "./.git/*" -not -path "./node_modules/*" | sort
```

For each file, verify:

- **Heading hierarchy** — H1 > H2 > H3 without skipping levels (no H1 > H3). Critical for RAG chunking and GEO.
- **Single H1** — Only one H1 per document (the title)
- **Consistent formatting** — No trailing whitespace, consistent list markers, blank lines around headings
- **No bare URLs** — Links should use `[text](url)` format, not raw URLs in prose

Report format:
```
Markdown Lint:
  ✓ README.md — 0 issues
  ⚠ docs/guides/configuration.md:45 — heading level skipped (H2 → H4)
  ⚠ CONTRIBUTING.md:23 — trailing whitespace
```

### 2. Link Validation

Check all internal and external links in documentation files.

**Internal links (relative paths and anchors):**

```bash
# Extract relative links from Markdown files
grep -roE '\[([^\]]*)\]\(([^)]+)\)' docs/ README.md CONTRIBUTING.md CHANGELOG.md 2>/dev/null
```

For each relative link:
- Verify the target file exists on disk
- Verify anchor links (`#section-name`) match an actual heading in the target file
- Check for case-sensitivity issues (common on Linux, invisible on macOS)

**External links (URLs):**

For each external URL found in documentation:
- Check HTTP status code (200 OK, 301 redirect, 404 not found)
- Timeout after 10 seconds per URL
- Skip URLs behind authentication (GitHub private repos, paywalled content)
- Flag any 404s or 5xx errors

Report format:
```
Link Validation:
  Checked: 45 links (32 internal, 13 external)
  ✓ 42 valid
  ✗ README.md:89 — docs/guides/migration.md (file not found)
  ✗ CONTRIBUTING.md:34 — #setup-instructions (anchor not found, did you mean #development-setup?)
  ⚠ README.md:12 — https://example.com/old-docs (301 redirect → https://example.com/docs)
```

#### Enhanced Detection Patterns

Four patterns that cause silent breakage not caught by basic link checking:

**1. Case-sensitivity (Linux hidden bugs)**

On Linux, verify the file path matches case exactly. macOS silently accepts wrong-case paths — Linux does not:

```bash
# If find returns a result that differs in case from the link, flag it
find . -iname "$(basename "$link_path")" | grep -v "^$link_path$" 2>/dev/null
```

**2. Fragment-only anchors (same-file jumps)**

For links like `[see setup](#setup-section)`:
- Extract the heading text and convert to GitHub anchor format: lowercase, spaces to hyphens, strip punctuation
- Search the target file for the matching heading
- Flag "did you mean #X?" when no match but a close heading exists (Levenshtein distance ≤ 2)

**3. Redirect chains**

Flag external URLs that redirect through more than 1 hop:

```bash
curl -Ls -o /dev/null -w "%{url_effective} (via %{num_redirects} redirects)\n" "$url"
# Flag if num_redirects > 1
```

**4. Relative links in nested documents**

For links found in `docs/guides/*.md`, resolve relative paths from the document's directory, not the repo root:

```bash
doc_dir=$(dirname "$doc_file")
resolved="$doc_dir/$relative_link"
[ -f "$resolved" ] || echo "✗ $doc_file — $relative_link (resolved to $resolved, not found)"
```

### 3. llms.txt Sync Check

Verify that `llms.txt` references match actual files on disk.

```bash
# Extract file paths from llms.txt
grep -oE '\./[^ ]+\.md' llms.txt 2>/dev/null | while read -r path; do
  [ -f "$path" ] && echo "✓ $path" || echo "✗ $path (file not found)"
done
```

Also check:
- Every Markdown file in the repo is represented in llms.txt (no orphaned docs)
- Descriptions in llms.txt match the actual file content (first paragraph check)
- llms-full.txt (if present) is not stale — compare modification time against source files

Report format:
```
llms.txt Sync:
  ✓ 12/12 referenced files exist
  ⚠ docs/guides/deployment.md not listed in llms.txt (orphaned doc)
  ⚠ llms-full.txt is 14 days older than README.md — may need regeneration
```

### 4. Image Validation

Check that all referenced images exist and are properly formatted.

```bash
# Extract image references from Markdown files
grep -roE '!\[([^\]]*)\]\(([^)]+)\)' docs/ README.md 2>/dev/null
```

For each image reference:
- **File exists** — Verify the image file is on disk (for relative paths)
- **Alt text present** — Flag images with empty alt text (`![]()`)
- **Absolute URLs for registries** — If the project is published to npm/PyPI, images must use absolute URLs (`https://raw.githubusercontent.com/...`), not relative paths
- **File size** — Flag images over 1MB (GitHub has a 10MB file limit, but large images slow page load)

Report format:
```
Image Validation:
  ✓ docs/images/demo.gif — exists, alt text: "Quick start demo", 2.3MB
  ⚠ docs/images/architecture.svg — empty alt text
  ✗ README.md:15 — assets/screenshot.png (file not found)
  ⚠ README.md:15 — relative image path, will break on npm (use absolute URL)
```

### 5. Freshness Check

Flag documentation files that haven't been updated recently. Uses `git log` to check the last modification date.

```bash
# Check last modification date for each doc file
for f in README.md CONTRIBUTING.md CHANGELOG.md docs/guides/*.md; do
  if [ -f "$f" ]; then
    last_modified=$(git log -1 --format="%ci" -- "$f" 2>/dev/null)
    echo "$f: $last_modified"
  fi
done
```

**Staleness thresholds (configurable):**

| File | Warning | Stale |
|------|---------|-------|
| README.md | 90 days | 180 days |
| CHANGELOG.md | 30 days (if releases exist) | 90 days |
| CONTRIBUTING.md | 180 days | 365 days |
| docs/guides/*.md | 90 days | 180 days |
| SECURITY.md | 180 days | 365 days |

Compare against latest commit date, not calendar date — a dormant project with no commits shouldn't trigger freshness warnings.

Report format:
```
Freshness Check:
  ✓ README.md — updated 12 days ago
  ⚠ docs/guides/deployment.md — last updated 95 days ago (threshold: 90 days)
  ✗ CONTRIBUTING.md — last updated 14 months ago (stale)
  · CHANGELOG.md — 2 releases since last update (v1.3.0, v1.4.0)
```

### 6. Feature Coverage Sync

Compare features mentioned in README against actual code. Reuses the `feature-benefits` skill's extraction workflow.

1. Load `feature-benefits` skill and run the feature extraction
2. Parse README.md features section for listed features
3. Cross-reference:
   - **Undocumented features** — code evidence exists, but not in README
   - **Over-documented features** — claimed in README, but no code evidence

Report format:
```
Feature Coverage: 8 documented / 10 detected (80%)
  Missing from README:
    - WebSocket support — found in src/ws.ts
    - Rate limiting — found in src/middleware/ratelimit.ts
  Over-documented:
    - "AI-powered suggestions" — no code evidence found
```

### 7. Badge URL Validation

Verify that shields.io badges in README return valid responses.

```bash
# Extract badge URLs from README
grep -oE 'https://img\.shields\.io/[^)]+' README.md 2>/dev/null
```

For each badge URL:
- Fetch the URL and check for HTTP 200
- Flag badges that return error SVGs (e.g., "invalid" or "not found")
- Check that badge links point to valid destinations

Report format:
```
Badge Validation:
  ✓ build status — 200 OK (passing)
  ✓ npm version — 200 OK (1.4.1)
  ✗ coverage — 200 OK but shows "unknown" (codecov may not be configured)
  ⚠ downloads — 301 redirect (badge URL format may be outdated)
```

## Quality Score

After running all verification checks, calculate a numeric quality score. The score gives users a single number to track and improve — modelled on the grading approach used in documentation quality tooling across the ecosystem.

### Scoring Dimensions

| Dimension | Max | Deductions |
|-----------|-----|-----------|
| Completeness | 30 | -5 per missing Tier 1 file (README, LICENSE, CONTRIBUTING, issue templates, PR template), -3 per missing Tier 2 file (CHANGELOG, SECURITY, CODE_OF_CONDUCT, llms.txt, AGENTS.md), -1 per missing Tier 3 file (ROADMAP, CITATION.cff, .cursorrules) |
| Structure | 20 | -5 if heading hierarchy skipped anywhere, -5 if hero missing required parts (one-liner + explanatory sentence + badges), -5 if no 4-question framework evident, -5 if single H1 rule violated |
| Freshness | 15 | -5 per stale file (>180 days since last update), -3 per warning file (>90 days) |
| Link Health | 20 | -5 per broken internal link (file not found), -3 per broken external link (404/5xx), -2 per broken anchor |
| Evidence | 15 | -5 if feature coverage below 70%, -5 per over-documented feature (claims without code evidence), -3 per missing benefit translation in features section |

### Score Calculation

```
score = 100
for each check result:
  apply deductions from the table above
score = max(0, score)
grade = lookup(score)
```

### Grade Bands

| Score | Grade | Label |
|-------|-------|-------|
| 90–100 | A | Ship-ready |
| 80–89 | B | Minor fixes needed |
| 70–79 | C | Needs work |
| 60–69 | D | Significant gaps |
| <60 | F | Not ready |

### Report Format

Append the score to the standard verification report:

```
📊 Documentation Quality Score: 74/100 (C — Needs work)

Breakdown:
  Completeness:   22/30  (-5 SECURITY.md missing, -3 ROADMAP.md missing)
  Structure:      20/20  ✓
  Freshness:      12/15  (-3 docs/guides/deployment.md stale)
  Link Health:    15/20  (-5 README.md:89 broken internal link)
  Evidence:        5/15  (-5 feature coverage 62%, -5 "AI-powered" claim without code evidence)

To reach grade B (80+): Fix the broken link (+5) and add SECURITY.md (+5).
```

Always include the actionable "To reach next grade" suggestion showing the 1–2 highest-impact fixes.

### CI Integration

When run with `ci` argument, export the score for pipeline use:

```bash
echo "PITCHDOCS_SCORE=74" >> "$GITHUB_OUTPUT"
echo "PITCHDOCS_GRADE=C" >> "$GITHUB_OUTPUT"
```

Accept `--min-score N` to fail the CI job if the score falls below a threshold:

```
/docs-verify ci --min-score 70
```

### 8. Token Audit

Estimate token cost for all skill files in `.claude/skills/`. Flag skills that may be degrading context efficiency.

```bash
# Estimate token cost for all skills
for skill_dir in .claude/skills/*/; do
  skill_file="$skill_dir/SKILL.md"
  if [ -f "$skill_file" ]; then
    words=$(wc -w < "$skill_file")
    tokens=$(echo "$words * 13 / 10" | bc)
    name=$(basename "$skill_dir")
    echo "${tokens} tokens — ${name}"
  fi
done | sort -rn
```

Thresholds (from `doc-standards` rule):
- Under 3,000 tokens: healthy
- 3,000–5,000 tokens: acceptable, monitor
- Over 5,000 tokens: flag for refactor — split into SKILL.md + SKILL-extended.md

Report format:
```
Token Audit:
  ✓ feature-benefits — 2,847 tokens (healthy)
  ⚠ pitchdocs-suite — 4,312 tokens (acceptable — monitor)
  ✗ docs-verify — 5,890 tokens (exceeds 5k — consider splitting)
```

### 9. Security Scan

Scan generated documentation for content that should never appear in public repos. AI-generated docs can accidentally surface internal paths, credentials, or proprietary configuration.

```bash
# Scan all docs for common credential patterns
grep -rn -E "(api[_-]?key|secret[_-]?key|password|token|bearer|private[_-]?key)" \
  README.md CONTRIBUTING.md CHANGELOG.md docs/ AGENTS.md CLAUDE.md \
  --include="*.md" -i 2>/dev/null
```

For each match, classify as:
- **Placeholder** (e.g., `YOUR_API_KEY`, `<your-token>`) — acceptable
- **Env var reference** (e.g., `$API_KEY`, `process.env.SECRET`) — acceptable
- **Real credential value** — block immediately, do not write to file, inform user

Additional checks:
- **Internal paths** — absolute paths like `/Users/`, `/home/`, `C:\Users\` suggest a dev machine path leaked in
- **Internal hostnames** — IP addresses like `192.168.`, `10.0.`, `172.16.`, `localhost:PORT` outside a code example context
- **Package names that don't exist** — if the README references a package name, verify it exists on the relevant registry to avoid dependency confusion vectors

Report format:
```
Security Scan:
  ✓ No credential patterns detected
  ⚠ README.md:45 — internal path: /Users/developer/projects/... (likely leaked from codebase scan)
  ✗ CLAUDE.md:12 — credential pattern: "token: ghp_abc123..." — review immediately
```

## CI-Friendly Output

When run with the `ci` argument, output results in a format suitable for CI/CD pipelines:

- Exit code 0: all checks pass
- Exit code 1: any check has errors (not warnings)
- Machine-readable output with file:line format for easy IDE integration

```
ERROR: README.md:89 — broken link: docs/guides/migration.md (file not found)
ERROR: llms.txt:5 — sync: ./ROADMAP.md (file not found)
WARN: docs/guides/deployment.md — stale: last updated 95 days ago
WARN: README.md:15 — image: relative path will break on npm
```

## GitHub Actions Workflow Template

For projects that want to run docs verification in CI:

```yaml
name: docs
on:
  pull_request:
    paths:
      - '**.md'
      - 'docs/**'
      - 'llms.txt'
      - 'llms-full.txt'

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Lint Markdown
        uses: DavidAnson/markdownlint-cli2-action@v18
        with:
          globs: '**/*.md'

      - name: Check links
        uses: lycheeverse/lychee-action@v2
        with:
          args: --exclude-private --no-progress docs/ README.md CONTRIBUTING.md CHANGELOG.md
          fail: true
```

## Anti-Patterns

- **Don't ignore warnings** — a broken link today becomes a confused user tomorrow
- **Don't run external link checks on every commit** — run them on PRs and weekly schedules to avoid rate limiting
- **Don't fix docs in a separate PR from code changes** — docs updates should accompany the code that changes behaviour
- **Don't suppress freshness warnings without reviewing** — stale docs erode trust faster than missing docs
