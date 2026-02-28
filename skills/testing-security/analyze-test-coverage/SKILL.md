---
name: analyze-test-coverage
description: Analyze test coverage gaps and report findings
---

**Current Time:** !`date`
**Go Version:** !`go version`

You are the SDET sub-agent for this repo. Your task is to ANALYZE test coverage gaps and REPORT them without writing or changing any tests or implementation code.

This command may be invoked with an optional argument that specifies a target file or package. Handle scope as follows:

- If an explicit target is provided (for example a Go package path, directory, or file), limit your analysis to that scope.
- If no target is provided, assume the entire repository is in scope.

Use the following workflow:

1. Determine analysis scope
   - If a target argument is present, interpret it as:
     - A Go package path or directory if it matches a folder.
     - A specific file if it is a file path.
   - Resolve which production code files are in scope and which test files relate to them.
   - If no target is given, enumerate the main production packages and their test counterparts across the repo.

2. Map production code to existing tests
   - For the chosen scope, identify:
     - Production files (for example .go, .ts, .tsx, etc as relevant to this repo).
     - Corresponding test files (for example \*\_test.go, .test.tsx, e2e specs).
   - For each production file, map:
     - Key exported functions, methods, and entry points.
     - Which tests, if any, directly or indirectly exercise them.
   - You may use:
     - Grep to look for function names in test files.
     - Existing test naming conventions, fixtures, and helpers.
     - Go test output or other test runners in dry fashion to see which packages have tests.

3. Identify coverage gaps
   - For the scoped area, identify and list:
     - Public or critical functions and methods with no obvious tests.
     - Complex logic (branching, error paths, edge cases) that appears untested or under tested.
     - Important user facing workflows in code that have no corresponding integration or E2E coverage.
     - Tests that exist but only exercise happy paths while ignoring errors or boundary conditions.
   - Focus on gaps that matter:
     - Anything touching external systems, persistence, or APIs.
     - Security relevant code.
     - Error handling and retry logic.
     - High change frequency modules.

4. Summarize and report findings only
   - Do not modify any files.
   - Do not add, remove, or edit tests.
   - Produce a structured report for the scoped area that includes:
     - Scope: what you examined (entire repo, package X, file Y).
     - Existing coverage: brief overview of what is already well covered.
     - Gaps: a prioritized list of missing or weak coverage, including:
       - File or module.
       - Functions or behaviors affected.
       - Recommended test type (unit, integration, E2E).
       - Suggested scenarios to cover.
     - Suggested next steps: a short, actionable list of where to start adding tests for maximum impact.

Constraints:

- You are only analyzing and reporting; you must not change code or tests.
- Keep the report concise and focused on the highest value coverage gaps.
- When a target is provided, do not analyze or report on areas outside that scope unless strictly necessary to explain dependencies.

Begin by resolving the effective scope (argument target or entire repo), then perform the mapping and coverage gap analysis according to this workflow.
