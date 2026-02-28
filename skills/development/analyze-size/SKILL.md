---
name: analyze-size
description: Analyze codebase size and language distribution using cloc. Use when user wants to understand codebase scale, primary languages, code composition, or assess project complexity. Provides total LOC, size category, language breakdown percentages, and key insights.
---

# Analyze Size

**Role:** Analyze codebase size, language distribution, and code metrics using `cloc` (Count Lines of Code).

**Goal:** Provide comprehensive understanding of codebase scale, primary languages, and code distribution to help assess project complexity and composition.

## Workflow

1. **Run cloc** with these flags:

```bash
cloc . --exclude-dir=node_modules,.git,dist,build,coverage,out,.next,.turbo --quiet 2>/dev/null || echo "cloc not found. Please install: brew install cloc (macOS) or apt-get install cloc (Linux)"
```

2. **Interpret results**
   - Identify total lines of code (excluding blank lines and comments)
   - Determine language distribution (files, code lines per language)
   - Calculate percentage breakdown of languages
   - Assess codebase size category

3. **Provide analysis** using this format:

```
## Codebase Size Analysis

**Total Lines of Code:** X,XXX lines

**Size Category:** [tiny (<1K) / small (1-10K) / medium (10-50K) / large (50-100K) / very large (>100K)]

**Language Distribution:**
- Language1: XX.X% (X,XXX lines)
- Language2: XX.X% (X,XXX lines)
- ...

**Key Insights:**
- Brief observation about codebase composition
- Any notable patterns or characteristics (e.g., high documentation ratio, test coverage indicators)
- Suggestions if relevant (e.g., "Primarily a TypeScript/React project")

[Include the raw cloc output table for reference]
```

## Constraints

- Focus on source code and meaningful configuration files
- Distinguish between source code, configuration, and documentation
- Provide context-appropriate insights based on language mix
- If cloc not installed, inform user with installation instructions
