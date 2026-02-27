---
name: pr-review
description: Perform automated code review on pull requests using Devin Review with regulatory compliance checks. Use this skill when reviewing migration PRs for correctness, when enforcing CIS Benchmark security compliance on code changes, when running automated bug detection and code quality analysis, or when triaging and auto-fixing review findings across enterprise modernization PRs.
---

# Automated PR Review

## Overview

This skill performs comprehensive automated code review on pull requests using Devin Review as the primary execution spoke. It goes beyond standard linting and formatting to provide deep semantic review: detecting bugs, security vulnerabilities, architectural drift, performance anti-patterns, and regulatory compliance violations. The skill integrates Devin Review's three core capabilities — auto-review for intelligent analysis, bug catcher for defect detection, and auto-fix for automated remediation — into a unified review pipeline that enforces DevinClaw guardrails on every PR.

The enterprise modernization program produces hundreds of PRs per sprint across multiple teams, repositories, and technology stacks. Manual code review cannot scale to this volume while maintaining consistent quality. DevinClaw's automated review ensures that every PR — whether produced by a human developer, a Devin Cloud session, or a parallel migration batch — receives the same rigorous security, quality, and compliance analysis before it reaches a human reviewer. This reduces human review burden to architectural decisions and business logic validation rather than catching syntax issues, security gaps, and test coverage shortfalls.

## What's Needed From User

- **PR identifier**: The pull request to review. Accepted formats:
 - GitLab merge request URL or number.
 - GitHub pull request URL or number.
 - Repository name and PR number (e.g., `enterprise-data-service#142`).
- **Repository context** (optional): If the repository has not been previously indexed by DeepWiki, the repository URL for initial indexing.
- **Review scope** (optional): Focus area for the review. Options:
 - `full` (default): Complete review including security, quality, performance, and compliance.
 - `security`: Security-focused review (CIS Benchmark, security controls, OWASP).
 - `migration`: Migration correctness review (for PL/SQL, COBOL, or API migration PRs).
 - `quality`: Code quality and pattern consistency review.
- **Auto-fix preference** (optional): Whether Devin Review should automatically create fix PRs for detected issues. Default: `suggest` (suggest fixes as review comments, do not auto-commit).
- **Compliance frameworks** (optional): Which compliance frameworks to validate against. Default: all applicable (CIS Benchmark, SOC 2 / ISO 27001, cloud security compliance).

## Procedure

1. **Retrieve PR context and index with DeepWiki**
 - Fetch the PR metadata via GitLab MCP or GitHub MCP:
 - PR title, description, author, base branch, and target branch.
 - Full diff (all changed files with context).
 - Linked Jira tickets or issue references.
 - CI/CD pipeline status if already running.
 - If the repository is not yet indexed in DeepWiki, trigger a full repository index.
 - If already indexed, refresh the index for the affected files and their dependencies.
 - Build a dependency impact map: which other files, services, or tests are affected by the changes in this PR.

2. **Generate review specification (SDD)**
 - Produce a review specification document that defines:
 - **Review scope**: Files changed, LOC added/modified/deleted, and affected modules.
 - **Risk assessment**: Sensitivity of the PR by risk level:
 - **Critical**: Changes to authentication, authorization, encryption, or safety-critical code.
 - **High**: Database migrations, API contract changes, new dependencies.
 - **Medium**: Business logic changes, new features, refactoring.
 - **Low**: Documentation, test-only changes, configuration.
 - **Applicable compliance checks**: Which benchmark IDs, security controls, and OWASP categories apply to the changed code.
 - **Review checklist**: Specific items to validate based on the change type and risk level.
 - Submit the review specification to Advanced Devin for review plan approval.

3. **Execute Devin Review auto-review**
 - Submit the PR to Devin Review via the `devin-review-mcp` server.
 - Devin Review performs deep semantic analysis:
 - **Logic correctness**: Detect logical errors, off-by-one bugs, null pointer risks, race conditions, and resource leaks.
 - **Pattern consistency**: Verify the code follows existing project patterns for error handling, logging, dependency injection, and naming conventions.
 - **API contract compliance**: For REST endpoints, verify that the implementation matches the OpenAPI specification. For database changes, verify migration scripts match the data model.
 - **Test adequacy**: Verify that test coverage meets the 80% threshold for changed code. Check that tests are meaningful (not just exercising code without assertions).
 - **Documentation**: Verify that new public APIs have documentation, new configuration has examples, and complex logic has explanatory comments.
 - Collect all findings with severity levels: Critical, High, Medium, Low, Info.

4. **Execute security compliance scan**
 - Run the security benchmark scanner via `benchmark-scanner-mcp` against all changed files:
 - **INPUT-001 Input Validation**: Verify all user inputs are validated with whitelist patterns.
 - **SEC-XXX Input Sanitization**: Verify parameterized queries for all database access. Flag any string concatenation in SQL.
 - **CRYPTO-001 Encryption at Rest**: Verify sensitive data fields use appropriate encryption.
 - **TLS-001 Encryption in Transit**: Verify TLS configuration for any new network communication.
 - **AUDIT-001 Audit Logging**: Verify security events are logged in structured JSON format.
 - **HEADERS-001 Error Handling**: Verify error responses do not leak stack traces, internal paths, or sensitive data.
 - Run validation via `security-controls-mcp`:
 - Map each changed file to applicable security control families (access control, audit, authentication, communications, integrity).
 - Validate control implementation for each mapped control.
 - Run dependency vulnerability scan:
 - Check new/updated dependencies against NVD for Critical and High CVEs.
 - Generate SBOM delta (new components added by this PR).
 - Run secret scanner (TruffleHog/GitLeaks) against the full diff.
 - Collect all security findings with benchmark IDs, security control references, and CVE identifiers.

5. **Execute bug catcher analysis**
 - Run Devin Review's bug catcher module on the diff:
 - **Null safety**: Detect potential null pointer exceptions, uninitialized variables, and missing null checks.
 - **Resource management**: Detect unclosed connections, file handles, streams, and transactions.
 - **Concurrency**: Detect race conditions, deadlock potential, and thread-unsafe operations.
 - **Error handling**: Detect swallowed exceptions, generic catch blocks, and missing error propagation.
 - **Type safety**: Detect type coercion issues, unchecked casts, and generic type erasure risks.
 - **SQL correctness**: For database changes, detect N+1 queries, missing indexes on queried columns, and incorrect JOIN conditions.
 - **Migration correctness**: For PL/SQL or COBOL migration PRs, detect semantic differences between the original and converted code (Oracle NULL handling, COBOL decimal precision, etc.).
 - Each bug finding includes: file, line number, severity, description, and suggested fix.

6. **Validate guardrail compliance**
 - Check all DevinClaw hard gates against the PR:
 - **HG-001 Tests Pass**: Verify CI pipeline test results (or trigger test execution if pipeline hasn't run).
 - **HG-002 CIS Benchmark Clean**: Verify zero High findings from step 4.
 - **HG-003 Lint/Format**: Verify linter and formatter report zero errors in changed files.
 - **HG-004 Coverage**: Verify code coverage >= 80% for new/modified code.
 - **HG-005 No Secrets**: Verify zero findings from secret scanner.
 - **HG-006 Dependencies**: Verify no Critical/High CVEs in new dependencies.
 - **HG-007 Signed Commits**: Verify all commits in the PR branch are GPG-signed.
 - **HG-008 Audit Trail**: Verify session audit log is present and complete.
 - **HG-009 License Compliance**: Verify no GPL/AGPL dependencies in Apache-licensed modules.
 - **HG-010 Security Controls**: Verify applicable security controls are implemented.
 - Produce a guardrail compliance report: pass/fail per gate with details on any failures.

7. **Generate auto-fix suggestions or PRs**
 - For each finding that Devin Review can automatically fix:
 - **Auto-fix mode (`auto`)**: Create a fix commit directly on the PR branch. Applicable for: formatting issues, simple null checks, missing input validation, missing test assertions.
 - **Suggest mode (`suggest`)**: Post the fix as a review comment with a code suggestion. The PR author can accept or reject.
 - **Manual mode (`manual`)**: Only report the finding. No fix suggested. Used for architectural issues and complex logic bugs.
 - Auto-fixes must pass all guardrails themselves. Every auto-fix commit triggers a re-run of guardrail checks.
 - Group related fixes into a single commit (e.g., all formatting fixes in one commit, all null check additions in another).

8. **Compose and post review summary**
 - Compile all findings into a structured review summary posted as a PR comment:
 - **Overall verdict**: Approve, Request Changes, or Comment.
 - **Guardrail status**: Pass/fail table for all 10 hard gates.
 - **Finding summary**: Count by severity (Critical/High/Medium/Low/Info).
 - **Security findings**: Listed with benchmark IDs and security control references.
 - **Bug findings**: Listed with file, line, and suggested fix.
 - **Quality findings**: Listed with pattern violation details.
 - **Auto-fix status**: Which findings were auto-fixed, which need manual attention.
 - Post inline comments on specific lines for each finding.
 - If the verdict is "Approve" and all guardrails pass, add the `devin-approved` label to the PR.
 - If the verdict is "Request Changes", list the blocking issues that must be resolved.
 - Update DeepWiki with review patterns and common findings for the repository.

9. **Monitor re-review cycle**
 - After the PR author addresses findings and pushes new commits:
 - Automatically re-trigger the review pipeline on the updated diff.
 - Only review the new changes (incremental review) plus verify that previous findings were resolved.
 - Update the review summary comment with the new status.
 - Continue the re-review cycle until all Critical and High findings are resolved.
 - Track the number of review cycles. If a PR exceeds 3 review cycles without resolution, escalate to the tech lead per the escalation policy.

## Specifications

- **Review latency**: Auto-review must complete within 15 minutes for PRs under 500 LOC changed. Up to 30 minutes for larger PRs.
- **Finding severity levels**:
 - **Critical**: Security vulnerability, data loss risk, or production-breaking bug. Blocks merge.
 - **High**: Significant bug, performance issue, or compliance violation. Blocks merge.
 - **Medium**: Code quality issue, pattern inconsistency, or minor bug. Does not block merge but should be addressed.
 - **Low**: Style suggestion, naming improvement, or documentation gap. Does not block merge.
 - **Info**: Informational observation about the code. No action required.
- **Blocking policy**: PRs with any Critical or High findings cannot receive `devin-approved` status.
- **Auto-fix scope**: Auto-fix is limited to mechanical changes (formatting, simple null checks, missing imports). Logic changes and architectural modifications are always manual.
- **Rate limit**: Maximum 20 reviews per hour per organization (Devin Review rate limit).
- **Inline comments**: Maximum 50 inline comments per review. If more findings exist, group related findings and prioritize by severity.
- **Review persistence**: All review findings, verdicts, and audit data are stored in the OpenClaw ledger for minimum 1 year per organizational data retention policy.
- **Re-review**: Incremental re-reviews should complete in under 5 minutes for fix-only changes.
- **Multi-repo PRs**: If a feature spans multiple repositories, review each PR independently but cross-reference findings that affect the integration boundary.

## Advice and Pointers

- For migration PRs (PL/SQL to PostgreSQL, COBOL to Java), pay special attention to semantic equivalence. Syntax may be correct while behavior differs. Key areas: NULL handling, date/time arithmetic, string comparison semantics, and implicit type conversions.
- The most common security finding in enterprise modernization PRs is string-concatenated SQL queries. This is always a Critical finding. Even if the inputs appear controlled, parameterize all queries — defense in depth.
- When reviewing Spring Boot PRs, check that `@Transactional` boundaries are correct. A common bug is placing `@Transactional` on private methods (which Spring proxies cannot intercept) or missing it on methods that perform multiple database writes.
- For React/TypeScript PRs, verify that useEffect dependency arrays are complete. Missing dependencies cause stale closure bugs that are extremely difficult to diagnose in production.
- Auto-fix works best for deterministic, mechanical issues. Never enable auto-fix mode for repositories with complex pre-commit hooks or custom linting rules that might conflict with the fixes.
- When the same finding appears across multiple PRs in the same repository, this indicates a systemic pattern issue. Escalate to the tech lead for a project-level fix rather than fixing it PR by PR.
- Review database migration PRs with extra scrutiny. A flawed migration script deployed to production can be catastrophic. Verify: idempotency, rollback capability, data preservation, and index coverage.
- For PRs that add new API endpoints, always verify that authentication and authorization are enforced. It is easy to add a new controller method and forget the `@PreAuthorize` or `@Secured` annotation, leaving the endpoint publicly accessible.
- Devin Review's bug catcher is most effective on Java and Python code. For COBOL-converted code, supplement with manual review of numeric precision and decimal handling, as these are areas where automated analysis has known limitations.
- Track review metrics over time: average findings per PR, most common finding categories, and fix turnaround time. These metrics feed into the DeepWiki knowledge base and improve future review accuracy.


## Self-Verification Loop (Devin 2.2)

After completing the primary procedure:

1. **Self-verify**: Run all applicable verification gates:
 - Build/test gates: verify PR branch builds cleanly, all tests pass on PR branch
 - Security gates: CIS Benchmark scan on changed files, dependency vulnerability check on new dependencies
 - Review gates: all Critical/High findings resolved or documented as accepted risk
2. **Auto-fix**: If any verification gate fails, attempt automated repair — adjust code, configuration, or test fixtures to resolve the failure.
3. **Re-verify**: Run all verification gates again after fixes. Confirm each gate transitions from FAIL to PASS.
4. **Escalate**: If auto-fix fails after 2 attempts, escalate to human reviewer with a complete evidence pack. Include the failing gate identifier, error output, attempted fixes, and root cause hypothesis.

## Artifact Contract

Every stage of this skill produces paired outputs for machine-consumable handoff:

| Stage | Markdown Output | JSON Output |
|-------|----------------|-------------|
| Diff Analysis | `diff_analysis.md` | `diff_analysis.json` |
| Bug Detection | `bug_detection.md` | `bug_detection.json` |
| Compliance Check | `compliance_check.md` | `compliance_check.json` |
| Review Report | `review_report.md` | `review_report.json` |

JSON outputs must conform to the schema defined in `audit/artifact-schemas/`. Markdown outputs are the human-readable narrative; JSON outputs are the machine-consumable contract consumed by the next stage or by OpenClaw for artifact validation.

## Evidence Pack

On completion, produce `evidence-pack.json` containing:

```json
{
 "session_id": "<Devin session identifier>",
 "timestamp": "<ISO 8601 completion time>",
 "skill_id": "devinclaw.pr_review.v1",
 "artifacts": [
 {
 "filename": "<output file>",
 "sha256": "<SHA-256 hash of file contents>",
 "stage": "<which stage produced this artifact>"
 }
 ],
 "verification": {
 "gates_run": ["<gate_1>", "<gate_2>"],
 "gates_passed": ["<gate_1>", "<gate_2>"],
 "gates_failed": [],
 "auto_fix_attempts": 0,
 "test_summary": {"passed": 0, "failed": 0, "skipped": 0},
 "scan_summary": {"critical": 0, "high": 0, "medium": 0, "low": 0}
 },
 "knowledge_updates": [
 {
 "action": "created|updated",
 "knowledge_id": "<Devin knowledge entry ID>",
 "summary": "<what was learned>"
 }
 ],
 "escalations": [
 {
 "gate": "<failing gate>",
 "reason": "<why auto-fix failed>",
 "evidence": "<link to error output>"
 }
 ]
}
```

## Escalation Policy

- **Divergence threshold**: 0.35 — if parallel verification sessions disagree beyond this threshold on key findings, escalate to human reviewer with both evidence packs for adjudication.
- **Human approval required for**: database migration PR approval, authentication/authorization changes, API contract modifications, safety-critical code changes.
- **Auto-escalate on**: Any security finding rated HIGH or CRITICAL, any risk of data loss or corruption, any changes to authentication or authorization logic, any modification to safety-critical code paths (DO-178C applicable systems).

## Forbidden Actions

- Do not approve a PR that has any unresolved Critical or High findings. All blocking findings must be fixed or documented as accepted risk with explicit tech lead approval.
- Do not skip the security compliance scan. Every PR must be scanned for CIS Benchmark, security controls, and dependency vulnerabilities regardless of the change size.
- Do not auto-fix logic bugs or architectural issues. Auto-fix is restricted to mechanical, deterministic changes (formatting, imports, simple null checks).
- Do not modify files outside the PR's diff scope. Review and auto-fix actions must be limited to the files changed in the PR.
- Do not post review comments containing sensitive information (secrets, PII, internal infrastructure details). Review comments are visible to all repository collaborators.
- Do not override or bypass guardrail gate failures. If a guardrail fails, the finding must be resolved or escalated — never suppressed.
- Do not merge PRs on behalf of the author. This skill reviews and comments; merge decisions are made by human engineers with appropriate authorization.
- Do not re-review the same PR more than 5 times without escalation. Repeated review cycles indicate a fundamental issue that requires human intervention.
- Do not store or cache PR source code outside the review session sandbox. All code access is scoped to the review session and must not persist after the review completes.
- Do not disable or reduce the severity of security findings. finding severity levels are defined by industry security and cannot be overridden by the review tool.
