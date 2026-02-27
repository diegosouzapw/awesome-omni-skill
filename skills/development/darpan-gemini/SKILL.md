---
name: darpan-gemini
description: Use when coding or reviewing the Moqui-based Darpan component to follow local conventions, documentation practices, and verification steps; keep work inside runtime/component/darpan/** and apply both AGENTS files.
---

# Darpan Gemini Skill

Use this skill whenever coding or reviewing the Darpan component so Gemini follows local conventions without re-reading the full repo. Frontmatter provides the trigger; the instructions below are what to follow.

## Scope and priorities
- Keep changes within `runtime/component/darpan/**`; avoid framework/base components unless explicitly asked.
- Default to best-practice implementation and system design; confirm if the user says prior changes were just a test.
- Always cross-check both `AGENTS.md` files (repo root for personal prefs, component-level for technical guardrails).

## Execution checklist
- Before coding: re-read this file plus both `AGENTS.md` files; keep scope to `runtime/component/darpan/**`.
- Documentation: when behavior or signatures change, update the closest doc under `runtime/component/darpan/docs/**` with inputs/outputs, defaults, examples, JSONPath, and operational impacts.
- Services/entities: keep service XML in sync with entities (in/out parameters typed, defaults described); avoid navigation/menu changes unless explicitly requested.
- Verification: lint/validate XML after edits; run targeted `./gradlew test` when touching logic; use `runtime/component/darpan/data/test/test-json-reconciliation.groovy` + `runtime/schemas/json/` for reconciliation changes.
- Safety: confirm intent if the user said prior changes were only a test; avoid framework/runtime paths outside the component; keep secrets out of logs and commits.

## Where to implement
- Business logic: `runtime/component/darpan/src/**`, exposed via `runtime/component/darpan/service/*.xml`.
- Entities: `runtime/component/darpan/entity/*.xml`; keep service and docs in sync with schema changes.
- Docs: update nearest feature doc under `runtime/component/darpan/docs/**` with behavior changes, examples, and operational impacts.

## Moqui + service patterns
- Define explicit `in-parameters`/`out-parameters` with types/defaults/descriptions; prefer `component://`/`runtime://` paths.
- Keep XML thin (routing/validation); move heavy logic to Groovy or `src/**`. Use Groovy for loops, branching, Spark, or library calls.
- For uploads, accept `org.apache.commons.fileupload.FileItem`; persist under `runtime://tmp/**` with sanitized names.

## Reconciliation and Spark
- Keep Spark sessions alive (stop only in `finally` for one-off scripts); avoid `collect()` on large data, prefer `limit`/`select` or disk writes.
- Persist/unpersist DataFrames explicitly; keep `sparkMaster`/`sparkAppName` configurable.
- Standardize IDs as `compare_id` (string, non-null, trimmed); use anti-joins for diffs and stream outputs (CSV via `coalesce(1)` + rename, JSON via `toLocalIterator()` writers).
- Name outputs with `yyyyMMdd-HHmmss`, sanitize `[A-Za-z0-9._-]`, suffix on collision; return both `diffLocation` and `diffFileName`.
- Resolve locations via `ec.resource.getLocationReference(...)`; create paths with `makeDirectory`/`makeFile`; prefer runtime paths.

## JSON schema handling
- Store schemas in `runtime/schemas/json/*.schema.json`; reference by filename and document JSONPath in `runtime/component/darpan/docs/**`.
- Prefer library-backed validation (Jackson + json-schema-validator); avoid bespoke implementations.
- Resolve simple keys to JSONPath via `JsonSchemaServices.resolve#JsonPathForKey`; default to `$.id` only with `processingWarnings`.

## UI and pagination
- Use Moqui built-in pagination widgets/styles; avoid custom HTML pagination.
- If multiple paginated lists exist, isolate pagination per list (standalone list screens in dynamic containers).
- For UI tweaks, limit scope to page display; avoid navigation/menu changes unless asked.

## Logging, errors, safety
- Log with context (service name, file labels, counts) without secrets; mask credentials and connection strings.
- Validate inputs early (`IllegalArgumentException` with clear messages); normalize strings (`?.toString()?.trim()`) and booleans via `normalizeBool`.
- Treat config and `mcp.json` as sensitive; do not commit secrets.

## Java 17 compatibility
- Preserve Spark/Java flags (`--add-exports=java.base/sun.nio.ch=ALL-UNNAMED`) in Docker entrypoints and docs.

## Testing and verification
- Tests are opt-in; run `./gradlew test` or targeted tasks when touching logic.
- After XML edits (entities/services/screens), run an XML verification step/lint and fix errors immediately.
- Use `runtime/component/darpan/data/test/test-json-reconciliation.groovy` and `runtime/schemas/json/` for quick validation when changing reconciliation paths.
