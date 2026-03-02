---
name: atft-code-quality
description: Enforce lint, formatting, typing, testing, and security hygiene across the ATFT-GAT-FAN codebase.
proactive: true
---

# ATFT Code Quality Skill

## Mission
- Maintain production-readiness by catching regressions before merge or deployment.
- Automate formatting, static analysis, and targeted test execution.
- Surface remediation paths for failing checks with minimal GPU disruption.

## Trigger Phrases
- “Run quality gate”, “lint/format the repo”, “execute tests”, “type check”, “pre-commit”.
- Requests mentioning Ruff, mypy, pytest, security audit, or dependency hygiene.

## Quality Pipeline
1. **Workspace scan**  
   - `git status --short`  
   - `tools/project-health-check.sh --section quality`
2. **Formatting & Lint**  
   - `ruff check src/ --fix`  
   - `ruff format src/ tests/`
3. **Type Safety**  
   - `mypy src/gogooku3 scripts/`  
   - `pyright` if TypeScript integration touched.
4. **Testing Layers**  
   - `pytest tests/unit -n auto -q`  
   - `pytest tests/integration -m "not slow"`  
   - `python test_short_selling.py --strict` for risk module.
5. **Security & Secrets**  
   - `pip install bandit safety` (once) then `bandit -qr src/`  
   - `detect-secrets scan` with baseline `security/detect-secrets.baseline`.
6. **Pre-commit Sweeps**  
   - `pre-commit run --all-files`  
   - `pre-commit run --hook-stage manual conventional-pre-commit`.

## Specialized Workflows

### Fast Feedback (single file change)
- `ruff check src/gogooku3/<module>.py --fix`.
- `pytest tests/unit/test_<module>.py -k <case>`.
- `mypy src/gogooku3/<module>.py`.

### GPU-Sensitive Checks
- For CUDA kernels or Torch compile edits:  
  `pytest tests/integration/test_gpu_training.py::test_compile_path --maxfail=1`.
- Validate memory usage script: `python tools/gpu_memory_report.py --dry-run`.

### Dependency Hygiene
- `pip-compile requirements.in` (if edited).  
- `python tools/dependency_audit.py --fail-on-critical`.  
- Update lockfiles and note changes in `docs/ops/dependency_log.md`.

## Failure Handling
- **Lint failure** → reference Ruff rule from output, fix quickly; prefer `ruff --fix-only RULE`.
- **Type errors** → add precise type hints, update `mypy.ini` only with justification in docstring.
- **Flaky tests** → rerun with `pytest -k test_name --lf`; mark with `@pytest.mark.flaky` only when documented in `tests/README.md`.
- **Security findings** → rotate secrets, update `.env.example`, notify DevSecOps via `security/alerts.md`.

## Codex Collaboration
- Launch `./tools/codex.sh "Perform deep static audit of src/gogooku3"` for architectural refactors or elusive bug hunts.
- Use `codex exec --model gpt-5-codex "Review failing pytest logs and propose fixes"` when triaging stubborn CI failures.
- Reflect Codex recommendations into permanent lint/type rules and note significant changes in `quality/last_quality_report.md`.

## Exit Criteria
- All commands exit 0.
- Updated artifacts recorded in `quality/last_quality_report.md`.
- Provide summary + next steps in PR description or change log.
