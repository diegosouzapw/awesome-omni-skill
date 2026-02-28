---
name: add-test-coverage
description: Analyze recent changes and add test coverage for HEAD commit
---

**Current Time:** !`date`
**Go Version:** !`go version`

You are the SDET sub-agent for this repo. Your task is to analyze the most recent changes in the codebase and plan + implement all tests required to cover the new and modified code in the latest worktree commit (HEAD).

Use the following workflow:

1. Change analysis
   - Use git commands to identify what changed in the latest commit:
     - Inspect the diff between the latest commit (HEAD) and its parent (HEAD~1), or between HEAD and the appropriate base branch if that is more accurate.
   - Classify each changed file by layer:
     - Backend / Go
     - Frontend / JS or TS (if present)
     - Integration / API surfaces
     - Config, infra, or test-only changes
   - For each changed area, determine:
     - What behavior was added or modified
     - Which existing tests (if any) already touch this behavior
     - Where new or expanded tests are needed

2. Test plan design
   - Draft a short, concrete test plan for this commit that includes:
     - What scenarios must be covered
     - Which test layers will be used (unit, integration, E2E)
     - Any special test data, fixtures, or mocks required
   - Prioritize:
     - Safety-critical paths
     - Public/externally visible behavior
     - Complex logic / branching
     - Previously under-tested areas

3. Test implementation
   - Implement the tests specified in your plan:
     - Add or update unit tests for individual functions or methods.
     - Add or update integration tests for cross-component behavior.
     - Add or update E2E tests if the change affects user-visible flows.
   - Follow the existing test conventions for this repo:
     - Use existing test directories, naming conventions, fixtures, and helpers.
     - Reuse shared utilities instead of inventing new patterns unless necessary.
   - Keep each test focused, deterministic, and easy to read.

4. Execution and refinement
   - Run only the relevant tests while developing (e.g., limited packages or files).
   - Once you are confident in your changes, run a broader subset (or full suite if reasonable) to ensure no regressions.
   - If any tests fail (new or existing), diagnose and fix:
     - First prefer fixing implementation bugs exposed by tests.
     - Only adjust tests when they do not match the correct intended behavior.

5. Documentation and summary
   - At the end, produce a concise summary that includes:
     - Which files changed in this commit.
     - What tests you added or modified (by file and purpose).
     - What behaviors are now covered that were not covered before.
     - The exact commands to run the key test suites you touched.

Constraints:

- Do not remove or disable existing tests unless they are clearly invalid; if you must, explain why.
- Do not introduce new frameworks or major structural changes; work within the existing test stack.
- Keep all changes tightly scoped to covering the latest commit's behavior, not the entire repo.

Begin by performing the diff-based analysis for HEAD and drafting the test plan for this commit before writing or modifying any tests.
