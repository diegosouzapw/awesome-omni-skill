---
name: testing
description: "Design and implement tests (unit, integration, e2e, accessibility, i18n), improve coverage, and run test suites. Use when the user asks to add tests, write tests, improve coverage, run tests, test accessibility, test translations, or implement test strategy."
triggers:
  - "/test"
  - "add tests"
  - "write tests"
  - "test this"
  - "run tests"
  - "improve coverage"
  - "unit test"
  - "integration test"
  - "e2e test"
  - "test strategy"
  - "accessibility test"
  - "a11y test"
  - "test translations"
  - "i18n test"
  - "migration test"
---

# Testing Skill

## Core Philosophy

**"Tests document behavior and prevent regressions."**

Design and run tests that are fast, deterministic, and focused. Prefer testing behavior over implementation.

**⛔ TESTING IS MANDATORY - NOT OPTIONAL**

When working within the workflow skill:

- **NEVER** present tests as "optional" or "next steps"
- **NEVER** complete a workflow without tests passing
- **ALWAYS** write and run tests before commit
- **STOP** the workflow if tests are missing and write them first

For frontend/UI projects:

- Tests (`npm test` or equivalent) MUST pass
- Build (`npm run build` or equivalent) MUST succeed
- E2E tests via Playwright MCP MUST pass (if UI exists)

For backend projects (Go, Python, Ruby, etc.):

- Tests MUST pass before any commit

---

## Protocol

### 1. Understand Scope

- **Unit**: Single function/class in isolation (mocks for deps).
- **Integration**: Multiple components together (real or test doubles).
- **E2E**: Full stack or user flows (browser/API).

Choose the right level; avoid testing implementation details in unit tests.

**MANDATORY: For any UI-related testing, you MUST use the Playwright MCP server.** If the Playwright MCP is not configured, run `/setup` first to add it before proceeding with UI tests.

### 2. Test Design

- **Arrange** – Set up inputs and dependencies.
- **Act** – Call the code under test.
- **Assert** – Check outcomes and side effects.

One logical assertion per test when possible. Name tests by behavior: `it("returns 404 when resource is missing")` not `it("works")`.

### 3. Commands (by ecosystem)

| Context    | Command                                                     | Purpose                              |
| ---------- | ----------------------------------------------------------- | ------------------------------------ |
| Node/JS/TS | `npm test` or `npx vitest` / `jest`                         | Run test suite                       |
| Python     | `pytest` or `python -m pytest`                              | Run tests                            |
| Go         | `go test -race ./...`                                       | Run all packages with race detection |
| Rust       | `cargo test`                                                | Run tests                            |
| Generic    | Run the project’s test script from README or `package.json` | Respect project conventions          |

Run tests after changes. Fix or skip failing tests before adding new ones.

### 4. Coverage

- Use coverage only to find gaps, not as a target by itself.
- Prefer: critical paths, edge cases, and error handling.
- Report coverage when asked: `pytest --cov`, `npm run test:coverage`, `go test -race -cover ./...`, etc.

### 5. UI Testing with Playwright MCP (MANDATORY for UI)

**When the application has a UI, Playwright MCP is MANDATORY for E2E and UI testing.**

**Prerequisites:** Ensure Playwright MCP is configured via `/setup`. If not available, stop and run `/setup` before proceeding.

**Quick reference:** Playwright MCP provides browser automation tools (`browser_navigate`, `browser_click`, `browser_type`, `browser_screenshot`, `browser_snapshot`, `browser_wait_for`, etc.) for testing user flows, visual regression, and accessibility.

**For complete UI testing guide:** See [reference/UI_TESTING.md](reference/UI_TESTING.md), which covers:
- Playwright MCP capabilities and when to use them
- Example UI test flows (login, forms, multi-step wizards)
- Best practices (semantic selectors, waiting for dynamic content)
- Configuration and troubleshooting
- Integration with testing frameworks

### 6. Accessibility (a11y) Testing

Test that the application is usable by people with disabilities.

**Test types:** Automated (axe-core, pa11y, Lighthouse), Keyboard (tab order, focus), Screen reader (VoiceOver, NVDA, JAWS).

**Quick reference:** Test color contrast, alt text, ARIA labels, keyboard navigation, and screen reader announcements. Follow WCAG 2.1 Level AA guidelines.

**For complete a11y testing guide:** See [reference/A11Y_TESTING.md](reference/A11Y_TESTING.md), which covers:
- Automated, keyboard, and screen reader testing
- WCAG 2.1 criteria (Perceivable, Operable, Understandable, Robust)
- Common issues and fixes
- Tools and resources
- Integration with CI/CD

### 7. Internationalization (i18n) Testing

Test that the application works correctly across languages and locales.

**Test types:** Translation coverage (no hardcoded text), Locale formatting (dates, numbers, currencies), RTL support (Arabic, Hebrew), Text expansion, Pluralization, Character encoding (Unicode, no mojibake).

**Quick reference:** Use pseudo-localization to catch hardcoded strings, test with longest translations (German), test RTL layouts (Arabic), verify pluralization rules per language.

**For complete i18n testing guide:** See [reference/I18N_TESTING.md](reference/I18N_TESTING.md), which covers:
- Translation coverage and pseudo-localization
- Locale formatting (dates, numbers, currencies)
- RTL (right-to-left) support testing
- Text expansion and UI breakage detection
- Pluralization for different languages
- Character encoding and mojibake detection
- Tools (i18next, FormatJS, Crowdin, Lokalise)

### 8. Database Migration Testing

Test that migrations work correctly and are reversible:

| Test               | Purpose                                            |
| ------------------ | -------------------------------------------------- |
| **Up migration**   | Schema changes apply without error                 |
| **Down migration** | Rollback works; schema returns to previous state   |
| **Data integrity** | Existing data survives migration; constraints hold |
| **Idempotency**    | Running migration twice doesn't fail or corrupt    |

**Migration testing in CI:**

1. Start with clean DB
2. Run all migrations up
3. Seed with test data
4. Run latest migration down, then up again
5. Verify data integrity

### 9. LLM Evaluation and Testing

Test LLM-powered features (skills, agents, AI-assisted workflows) with specialized approaches.

**Challenges:** Non-deterministic outputs, semantic correctness, evaluation complexity, cost/latency.

**Quick reference:** Test behavior/outcomes (not exact output), use temperature=0 for deterministic tests, mock LLM calls for unit tests, run expensive E2E tests selectively, use scoring-based evaluation for quality.

**For complete LLM evaluation guide:** See [reference/LLM_EVALUATION.md](reference/LLM_EVALUATION.md), which covers:
- Testing challenges with LLMs (non-determinism, evaluation metrics, flaky tests)
- Test types (unit with mocks, integration, E2E with real LLM, regression/snapshot)
- Deterministic vs scoring-based tests
- Prompt evaluation frameworks (PromptFoo, LangSmith)
- Skill validation patterns (outcome-based, property-based, LLM-as-judge, differential)
- Common pitfalls (over-fitting, flaky tests, testing implementation details)
- Example test patterns and tools

### 10. Output

When adding or updating tests, provide:

- Short rationale (what behavior is covered).
- How to run the new/updated tests.
- Any new dependencies or config (e.g. test framework, env vars).

---

## Checklist

**⛔ MANDATORY (workflow cannot proceed without these):**

- [ ] Tests written for new/changed code
- [ ] All tests pass
- [ ] Build succeeds (frontend/compiled languages)
- [ ] **UI: E2E tests via Playwright MCP pass** (mandatory if UI exists)

**Quality checks:**

- [ ] Tests are deterministic (no flake from time/random/network).
- [ ] No tests that only assert implementation details.
- [ ] Failures give clear messages (assertions describe expected vs actual).
- [ ] Test file/names follow project layout and naming.
- [ ] Accessibility tests pass (axe-core or equivalent); keyboard navigation works.
- [ ] i18n tested: no hardcoded strings, locale formatting correct, RTL supported (if applicable).
- [ ] Database migrations tested: up, down, and data integrity verified.

**⚠️ NEVER output "Optional: Add tests" - testing is MANDATORY.**

---

## Cross-Skill Integration

| Situation | Skill to invoke | How |
|-----------|----------------|-----|
| UI/E2E tests needed | Playwright MCP | Run `/setup` first, then use `browser_navigate`, `browser_click`, etc. |
| Test reveals security issue | **security-reviewer** skill | Read `skills/security-reviewer/SKILL.md` |
| Test coverage in CI pipeline | **ci-cd** skill | Read `skills/ci-cd/SKILL.md` |
| Need to write tests before refactoring | **refactoring** skill | Read `skills/refactoring/SKILL.md` |

**See also:** **debugging** skill for investigating flaky tests caused by timing/race conditions.
