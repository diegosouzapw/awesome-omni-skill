---
name: fix-markdown
description:
  Fix lint, formatting, and prose issues in markdown files using
  Prettier and Vale. Use when the user or agent needs to fix lint,
  formatting, and prose issues in markdown files.
---

# Agent protocol: Fix markdown file

**`GOAL`**: use prettier and vale to fix lint, formatting, and prose
issues in markdown files.

**`WHEN`**: use when the user or agent needs to fix lint, formatting, or
improve prose in markdown files.

**`NOTE`**: strictly follow E-Prime directive (avoid "`to be`" verbs)
when writing or correcting prose.

## References

The following reference files serve as strict guidelines when updating
prose.

Locate the reference files in the `references` folder.

- **`references/e-prime-directive.md`**: _The E-Prime Communication
  Protocol defining the rules for avoiding `to be` verbs._

## Primary directives

### Formatting and linting sequence

- **Format first & last**: Always run prettier before analysis and after
  edits
- **E-Prime compliance**: Strictly follow
  `references/e-prime-directive.md` when writing/correcting prose
- **Vale cycle**: Run vale iteratively (lint → fix → verify) until no
  issues remain
- **Research complex issues**: Use search tools or Perplexity for
  unfamiliar lint problems

### Tool commands

- Format: `prettier --write <file_path>`
- Sync rules: `vale sync`
- Lint: `vale --no-wrap --output=JSON <file_path>`

### Vale fixing guidelines

- **Path wrapping**: Wrap filenames, `URIs`, `URLs`, and paths in
  backticks
- **Context awareness**: Check line numbers - issues may appear as
  `substrings` in technical terms
- **False positives**: Wrap acronyms, names, and proper nouns in
  backticks
- **Headings**: Use sentence case (capitalize first character only)
- **Passive voice**: Follow E-Prime directive to remove "`to be`" verbs
- **Follow hints**: Use Vale's issue links when needed

## Efficiency directives

- Batch operations on file groups, avoid individual file processing
- Use parallel execution when possible
- Target only requested files
- Reduce token usage in all operations

## Task management

For complex tasks: use `todo` system to break down, plan, and optimize
workflow.

## Workflow

- Run `vale sync` to update lint rules
- Run `prettier --write` for baseline formatting
- Study file and apply Vale path directives (wrap all paths/filenames in
  backticks)
- Iteratively run `vale`, fix issues, and verify until no issues remain
- Run `prettier --write` for final formatting
- **`DONE`**

## Output

**Files modified:**

- Target markdown files - Formatted and lint-free

**Status communication:**

- Reports number of issues fixed
- Confirms Vale shows zero remaining issues
- Lists file paths processed
