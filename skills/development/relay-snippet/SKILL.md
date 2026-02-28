---
name: relay-snippet
description: Save a useful, cool, interesting, or novel code snippet to the Data Relay. Use this when a piece of code encountered or written during a session is worth preserving — elegant patterns, clever solutions, reusable utilities, or instructive examples. When you spot a snippet-worthy moment during work, propose it to March first — if he agrees, invoke this skill to save it.
allowed-tools: Read, Write, Glob, Grep, Bash
---

## Vault Path

Resolve the Data Relay vault path before any file operations:

```bash
jq -r '.["data-relay"].path' ~/.claude/domains.json | sed "s|~|$HOME|"
```

Use the resolved absolute path (`<vault>`) for all file operations below. Do NOT pass `~` to Glob, Read, or Write — they require absolute paths.

## Instructions

Save a code snippet to `<vault>/local/snippets/`.

The snippet: $ARGUMENTS

1. Read `<vault>/tags.md` to check available subject tags.
2. Read `<vault>/meta/templates/snippet.md` for the file format spec.
3. Write the snippet as a **code file in the target language**, using the format defined in the template. All metadata (description, source, date, tags) and commentary (explanation, references, notes) go in **line comments**.
4. Use a lowercase, hyphenated filename with the appropriate language extension (e.g., `bounded-retry-with-backoff.rs`, `bitemporal-exclusion.sql`).

## Relevant languages

- **Professional:** TypeScript (primary), Python, SQL
- **Primary interests:** TypeScript, Python, Lua, OCaml
- **Actively learning:** Rust, Gleam
- **Adjacent:** Erlang/Elixir (via BEAM ecosystem)

Snippets in these languages are in scope. Snippets in other languages can still qualify if the principle transfers to March's work or learning.

## Selection criteria

A snippet is worth saving when it meets most of these:

- **It demonstrates a principle, not just a technique.** The code is a vehicle for an idea. If you can't articulate the *why* in the header comments, it's probably not worth saving.
- **It transfers.** The insight applies beyond the specific language or codebase. A snippet that only works in one context is a Stack Overflow answer, not a knowledge artifact.
- **It connects to active work or learning.** Typestates connect to marchhouse, bitemporal constraints connect to Numeric's data platform, `use` expressions connect to learning Gleam. Brilliant but irrelevant code doesn't clear the bar.
- **There's a surprise in it.** Something non-obvious — a twist, a constraint that forces an elegant solution, an insight that reframes how you think about a problem. If the code does exactly what you'd expect, it's not teaching anything.
- **It's re-readable.** Future-you can open the file in six months and follow it without reconstructing the original conversation.

Be cautious about saving things that are merely *clever*. Cleverness without a transferable principle is trivia.

## Quality bar

- The snippet should be **self-contained** — understandable without reading the full codebase it came from.
- **Explain the why**, not just the what. A snippet without context is just syntax.
- **Use `[[wikilinks]]`** in comments to reference related notes or codex entries in the vault.
- To introduce a new subject tag not in `tags.md`, ask March for approval first.
