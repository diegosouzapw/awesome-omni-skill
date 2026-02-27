---
name: rsmap
description: Search and explore Rust codebases using rsmap indexes
user-invocable: true
---

You are an assistant that uses rsmap to index and explore Rust codebases.

rsmap generates a multi-layered, LLM-friendly index of Rust projects.

## Index Files

- overview.md       — Layer 0: Crate metadata, module tree with descriptions
- api-surface.md    — Layer 1: All item signatures (bodies stripped), grouped by module
- relationships.md  — Layer 2: Trait impls, error chains, module deps, type hotspots
- index.json        — Layer 3: Fully-qualified path → file:line lookup table
- annotations.toml  — LLM-facing descriptions (note/stale/removed per item)
- cache.json        — BLAKE3 hashes for incremental rebuilds

## Searching the Index

This is the core workflow. When the user asks about a Rust codebase that has
an rsmap index, search the index files instead of scanning source files directly.

**Find where something lives:**
Search overview.md for crate and module names to orient yourself.

**Find types, functions, traits, signatures:**
Search api-surface.md. It contains every item signature in the codebase,
grouped by module with `<!-- file: ... -->` comments. Grep for type names,
function names, or trait names to find their signatures and which module
they belong to.

**Understand architecture and cross-cutting concerns:**
Search relationships.md for trait implementations (what types implement a
trait), module dependencies (what depends on what), error chains (From impls),
and type hotspots (types used across many modules).

**Jump to source:**
Search index.json for an item's fully-qualified path to get its exact file
path and line range. Then read the actual source when the signature alone
isn't enough.

**General approach:**
1. Search the index to find what you need
2. Only read source files when the index doesn't have enough detail
3. Use index.json to go from item name → file:line → source

## Generating an Index

```
rsmap generate --path <RUST_PROJECT> --output <OUTPUT_DIR>
```

Use --no-cache to force a full rebuild.

## Annotation Workflow

When asked to annotate a codebase:
1. Run `rsmap annotate export` to get unannotated/stale items
2. Read the exported TOML, fill in `note` fields with concise descriptions
3. Run `rsmap annotate import <FILE>` to merge annotations back
4. Re-run `generate` to update the index with new annotations

## Visualization

The relationships.md file contains a "Key Types" section showing types
referenced across the most modules. This data can be used to generate
bar charts or other visualizations when requested.
