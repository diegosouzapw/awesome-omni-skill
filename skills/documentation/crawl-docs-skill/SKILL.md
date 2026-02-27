---
name: crawl-docs-skill
description: "Run a Crawl4AI-based doc crawler and save internal pages as Markdown using page titles as filenames. Use when the user provides a docs URL and wants all internal subpages saved as .md files. Environment setup should only use uv."
---

# Crawl Docs Skill

## One-shot (Direct Run)

- Run directly: `bash "<path-to-skill>/scripts/crawl_docs.sh" <URL> --out "$PWD/docs"`
- Do NOT install dependencies or scaffold files first; run immediately. The script will bootstrap uv + venv only if needed.
- Do NOT run tests or create plans; just execute the script and let it fail if the system is missing prerequisites (e.g. curl).
- Default output dir (skill convention): `"$PWD/docs"` (absolute path to `./docs` under the current working directory); always pass it via `--out` unless the user specifies otherwise.
- Optional flags: `--out <dir>`, `--max-pages <int>`

## Correct Usage (from real run)

- First run: execute the one-shot command exactly once with no extra steps.
- If it fails: fix the missing system prerequisite (most commonly `curl`) and re-run the same command.
- Do not probe versions or add max-pages unless the user asks.

## Notes

- Use page title (tab name) as the output filename; auto-deduplicate with numeric suffixes.
- Default behavior has no max pages unless `--max-pages` is provided.
