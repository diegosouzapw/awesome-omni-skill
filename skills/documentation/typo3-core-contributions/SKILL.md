---
name: typo3-core-contributions
description: "Use when analyzing TYPO3 Forge issues, submitting patches to Gerrit, or contributing documentation to TYPO3 Core."
---

# TYPO3 Core Contributions

Guide for TYPO3 Core contribution workflow from account setup to patch submission.

## When to Use

- Forge issue URLs (e.g., `https://forge.typo3.org/issues/105737`)
- Contributing patches, fixing TYPO3 bugs
- Gerrit review workflow, rebasing, CI failures

## Prerequisites

```bash
scripts/verify-prerequisites.sh
```

Check: TYPO3.org account, Gerrit SSH, Git config (email must match Gerrit!)

## Workflow Overview

1. **Setup**: Account → Environment (`scripts/setup-typo3-coredev.sh`)
2. **Branch**: `git checkout -b feature/105737-fix-description`
3. **Analyze Issue**: Understand deeply before coding (see below)
4. **Develop**: Implement, write tests, validate with typo3-conformance-skill
5. **Verify CI**: Run full test suite locally, ensure all checks pass
6. **Commit**: Follow format, include `Resolves: #<issue>` + `Releases:`
7. **Submit**: `git push origin HEAD:refs/for/main` (starts as WIP)
8. **CI Check**: Wait for Gerrit CI, fix ALL failures before marking ready
9. **Mark Ready**: Remove WIP state only when all CI jobs pass
10. **Update**: Amend + push (preserve Change-Id!)
11. **Verify Releases**: After approval, test in all target branches

## Phase 3: Analyze Issue Deeply

**Before writing any code**, thoroughly understand the problem:

1. **What is broken?** - Identify the exact behavior vs expected behavior
2. **Why is it broken?** - Trace the root cause in the codebase
3. **Reproduction steps** - Document minimal steps to reproduce
4. **Affected versions** - Check which branches have the issue (main, 13.4, 12.4)
5. **Related code** - Review existing tests and similar implementations
6. **Edge cases** - Consider what else might be affected

**This analysis prevents wasted time** on incomplete fixes or patches that don't address the actual problem.

## Phase 5 & 8: CI Verification

**Before marking ready for review**:

1. Run tests locally: `./Build/Scripts/runTests.sh -s unit && ./Build/Scripts/runTests.sh -s functional`
2. Check code style: `./Build/Scripts/cglFixMyCommit.sh`
3. Run PHPStan: `./Build/Scripts/runTests.sh -s phpstan`
4. After push: Wait for ALL Gerrit CI jobs to complete
5. **Read actual job logs** for any failures - never guess!
6. Fix ALL issues in one patchset before marking ready

See `references/gerrit-workflow.md` for CI debugging details.

## Phase 11: Verify in Target Branches

**After receiving +2 approval**, before merge:

1. Check your `Releases:` line (e.g., `main, 13.4, 12.4`)
2. Cherry-pick to each target branch locally
3. Verify fix works on each version
4. Ensure no version-specific issues (API differences, etc.)

This prevents broken backports and ensures the fix works everywhere it's needed.

## Commit Format

```
[TYPE] Subject line (imperative, max 52 chars)

Description explaining how and why.

Resolves: #12345
Releases: main, 13.4, 12.4
```

**Types**: `[BUGFIX]`, `[FEATURE]`, `[TASK]`, `[DOCS]`, `[SECURITY]`, `[!!!]`

## Related Skills

- **typo3-ddev-skill**: Development environment
- **typo3-testing-skill**: Test writing
- **typo3-conformance-skill**: Code quality validation

## References

| Topic | File |
|-------|------|
| Account setup | `references/account-setup.md` |
| Commit format | `references/commit-message-format.md` |
| Gerrit workflow | `references/gerrit-workflow.md` |
| Troubleshooting | `references/troubleshooting.md` |

---

> **Contributing:** https://github.com/netresearch/typo3-core-contributions-skill
