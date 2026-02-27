---
name: python-dev
description: Python development standards and practices for zero-fabrication, test-driven development with strict quality gates. Use when working on Python projects that require rigorous testing, linting, and architecture standards with real integrations only.
---

# Python Development Standards

This skill provides comprehensive Python development standards focused on real implementations, rigorous testing, and zero fabrication.

## Repository & Project Layout

**Required structure**
```
README.md
requirements.txt
.gitignore
run              # Python file with argparse (no .py), executable
src/             # all source code
output/          # gitignored runtime outputs
  testing/       # gitignored test output/logs/artifacts
local/           # gitignored large downloads/artifacts
```

**Directory constraints**
- Shallow structure preferred. Once you use subdirectories, place peer modules at the same nesting level.
- Max 20 files per directory.
- No experimental scripts or alt versions at root or anywhere else; use branches for iteration.

**Example `.gitignore`**
```
output/
local/
__pycache__/
*.pyc
```

---

## Coding Standards

**Naming policy**
- Never use a name not in the dictionary
- Prefer snake_case for all identifiers - especially in the database

**Commenting policy**
- No docstrings. Use regular comments only.
- Every class/function must end with a **comment block** with line and two fields: a name and a description.
  - `# ##################################################################`
  - `# <short human name>`
  - `# <concise why + key intent in prose>`
- Keep functions tiny and obviously correct.
- A function with a loop should primarily loop and call a named helper.
- A multi-step function should delegate each step to named helpers.

**proc name**
there's a libary setproctitle, which should be pip installed - every app at the start
of main should do 
    setproctitle.setproctitle('meaningful title')

**Comment-block example**
```python
# ##################################################################
# clean repository
# restore the working tree to a fresh-clone state by
# removing untracked files, resetting changes, and ensuring only
# canonical files remain.
def cleanRepository() -> None:
    run_shell("git reset --hard")
    run_shell("git clean -xfd")
    ensure_only_expected_files()
```

**Static, namespaced utilities example**
```python
# ##################################################################
# text operations namespace
# stateless text helpers grouped for discoverability
class TextOperations:
    # ##################################################################
    # wrap
    # makes sure text is split into lines of less than this width, but
    # without breaking words unless there is no other choice
    @staticmethod
    def wrap(text: str, width: int) -> str:
        return textwrap.fill(text, width)
```

---

## DRY in Practice

- Centralize repeated patterns: logging, color output, constants, conversions.
- After writing code, extract commonality; then migrate other files to the new helper.

**Example: color output**
```python
# ##################################################################
# print header
# make sure headers are always cyan
def print_header(text: str) -> None:
    print(f"{Fore.CYAN}{text}{Style.RESET_ALL}")
    logger.info(f"----{text}")
```

---

## Error Handling

- Catch exceptions only where you can make a meaningful decision:
  - Entry points (fail the task, show the error)
  - Long processing loops (log context, continue to next item)
- Elsewhere, catch only to add context and re-raise.
- For network work, use standard backoff utilities; keep these in one reusable module.
- Never log or raise a constant string; always add context.

**Processing-loop example**
```python
# ##################################################################
# process batch
# ensure one bad item doesn't abort a long batch while logging context for triage
def process_batch(items: list[str]) -> None:
    for item in items:
        try:
            process_one(item)
        except Exception as err:
            logger.error("process_one failed item=%r err=%s", item, err)
            continue
```

**Entry-point exception policy**
```python
# ##################################################################
# top-level entry
# single place to convert exceptions into exit codes and logs
def main() -> int:
    try:
        run_pipeline()
        return 0
    except Exception as err:
        logger.exception("Pipeline failed: %s", err)
        return 1
```

---

## Secrets & Configuration

**Secrets**
- Secrets come only from **keyring** or **AWS Parameter Store**.
- The literal word `keyring` must **never appear in tests**.
- If secret retrieval fails, allow natural failure; do not override or stub.

**Configuration**
- Internal apps: avoid config files. Encode base configuration in code.
- Environment-specific values via environment variables only.

**Example**
```python
# ##################################################################
# get api key
# centralized secret access with explicit failure; sane default via env override
def get_api_key(name: str) -> str:
    key = keyring.get_password("app", name)
    if not key:
        raise RuntimeError(f"Missing secret for {name}")
    return key

DEFAULT_TIMEOUT_SEC = int(os.getenv("APP_TIMEOUT_SEC", "15"))
```

---

## Testing Policy (Real, Integration-First, Pytest, Per-File Only)

- Each `x.py` has `x_test.py` beside it.
- Use **Pytest** only; no `__main__` blocks.
- All tests are real end-to-end or integration tests. No smoke checks.
- **Run only the specific test(s) for the file you are working on.**
  - Do **not** run the full suite with the tool. `dazpycheck` runs everything at the end.
  - Example: while developing `src/text/wrap.py`, run `pytest src/text/wrap_test.py::test_wrap_text`.
- Write all test logs and artifacts to `output/testing/`.
- Expensive external actions: design a plugin interface and provide at least two real implementations (e.g., Physical vs In-Memory). Run the same test suite on both. In production, you may wire either.
- For LLM calls or costly requests, use **memoization** keyed by full parameters. Identical calls hit cache, but single runs remain fully real.

**Plugin pattern example**
```python
# ##################################################################
# check printer
# this prints out an actual check, or cheque - we define it as an
# interface so we can have several different implementations
class CheckPrinter:

    # ##################################################################
    # check printer
    # this prints out an actual check, or cheque
    def print_check(self, data: dict) -> None: ...

# ##################################################################
# check printer physical
# for use when sending out cheques to clients
class CheckPrinterPhysical(CheckPrinter):

    # ##################################################################
    # check printer
    # this prints out an actual check, or cheque
    def print_check(self, data: dict) -> None:
        send_to_usb_printer(data)

# ##################################################################
# check printer memory
# for testing, uat and simulation, we don't want real checks to be
# printed out - so we just print them to memory and log them
class CheckPrinterMemory(CheckPrinter):
    def __init__(self) -> None:
        self.printed: list[dict] = []

    # ##################################################################
    # check printer
    # this prints out an actual check, or cheque
    def print_check(self, data: dict) -> None:
        self.printed.append(data)


# ##################################################################
# tests of CheckPrinter
# any time we have an interface we want tests that test each implementation, to make sure that
# they all work exactly the same
@pytest.mark.parametrize("impl_cls",[CheckPrinterPhysical, CheckPrinterMemory])
def test_prints_identically(impl_cls):
    impl = impl_cls()
    impl.print_check({"amount": 100})
    # assertions…
```

**LLM memoization example**
```python
from functools import lru_cache

# ##################################################################
# llm_complete
# we memoize this because calls to the llm are expensive - this is occasionally useful in production, but
# particularly helps with the speed of tests - if ever time we call the llm in a test we use the same prompt
# then the test suite only ever ends up calling the llm once
@lru_cache(maxsize=256)
def llm_complete(model: str, prompt: str) -> str:
    return call_llm(model=model, prompt=prompt)
```

**Per-file run example (tool behavior)**
```
# Good: focused
pytest -q src/text/wrap_test.py::test_wrap_text --maxfail=1 --disable-warnings > output/testing/wrap_test.log 2>&1

# Bad: full-suite duplication during development
pytest
```

---

## Linting, Warnings, and `dazpycheck`

- Treat all warnings as errors, including deprecations.
- Line length is 120.
- Run linter after each file, then run only the tests for that file (logging to `output/testing/`).
- At the end of the task, run `dazpycheck` which runs the full test suite and final gates.
- **No commits** without full `dazpycheck` pass.

**Command examples**
```bash
ruff check --line-length 120
pytest -W error -q src/text/wrap_test.py::test_wrap_text > output/testing/wrap_test.log 2>&1
./run check
```

---

## Run Facade (Python Argparse Only)

The run tool is a **Python file named `run`** (no `.py` extension) using **argparse**. It orchestrates by invoking Python modules/commands, not shelling out core logic. It must not contain business logic.

**Example `run` (Python argparse)**
```python
#!/usr/bin/env python3
import argparse
import subprocess
import sys
from pathlib import Path

TEST_OUT = Path("output/testing")

# ##################################################################
# run
# executes a shell command
def _run(cmd: list[str]) -> int:
    return subprocess.call(cmd)

# ##################################################################
# test
# the user ran 'run test' - so we want to run all the pytests
def command_test(args: argparse.Namespace) -> int:
    # Per-file test only; write logs under output/testing
    TEST_OUT.mkdir(parents=True, exist_ok=True)
    target = args.target
    log = TEST_OUT / (target.replace('/', '_').replace('::','_') + ".log")
    rc = _run(["pytest", "-q", target, "--maxfail=1", "--disable-warnings"])
    return rc

# ##################################################################
# lint
# the user ran 'run lint' - so we want to run ruff
def command_lint(_: argparse.Namespace) -> int:
    return _run(["ruff", "check", "--line-length", "120"])

# ##################################################################
# check
# the user ran 'run check' - so we want to run dazpycheck
def command_check(_: argparse.Namespace) -> int:
    # Full-suite and gates only at the end
    return _run(["dazpycheck"])

# ##################################################################
# main
# the main run app - just parse the arguments and delegate to the right thing
def main(argv: list[str]) -> int:
    parser = argparse.ArgumentParser()
    sub = parser.add_subparsers(dest="cmd", required=True)

    p_test = sub.add_parser("test", help="Run a single test target (file or test function)")
    p_test.add_argument("target", help="e.g. src/text/wrap_test.py::test_wrap_text")
    p_test.set_defaults(func=command_test)

    p_lint = sub.add_parser("lint", help="Run linter")
    p_lint.set_defaults(func=command_lint)

    p_check = sub.add_parser("check", help="Run full suite and gates (dazpycheck)")
    p_check.set_defaults(func=command_check)

    args = parser.parse_args(argv)
    return args.func(args)

# ##################################################################
#
# standard python pattern for dispatching main

if __name__ == "__main__":
    sys.exit(main(sys.argv[1:]))
```

---

## Prohibited Patterns (Zero-Tolerance)

**Forbidden words in code/tests/comments:**
`simulate`, `mock`, `fake`, `pretend`, `placeholder`, `stub`, `dummy`, `sleep`, `todo`

**Forbidden method name fragments:**
`_simulate_`, `_mock_`, `_fake_`, `_stub_`, `_dummy_`, `_sleep_`

**Forbidden comments:**
`# TODO: replace with real implementation`

**Consequences:**
- Any appearance of the above indicates failure of the task.

**What to do instead:**
- If a dependency is missing or a system is unavailable, stop and report a blocker with exact requirements to proceed.

**Blocker example**
```
Cannot proceed: Postgres unreachable at $DATABASE_URL
Tried 3 retries with 1/2/4s backoff
Required next step: provide credentials and network access
```

---

## Architecture Heuristics

**Separation of concerns**
- Business rules are thin adapters over generic utilities.
- Cluster by domain (text/, net/, io/, llm/).

**Stateless first**
- Prefer pure, parameter-defined functions. Memoize when appropriate.

**File and function size signals**
- If a file exceeds ~200–300 lines or a function exceeds ~25 lines, refactor.

**Interfaces before conditionals**
- If you foresee two modes (physical/memory, online/offline), define an interface and two real implementations. Avoid branching inside one class.

**Examples that guide structure**
```python
# text/wrap.py
def wrap_text(text: str, width: int) -> str: ...
# loan_system/term_loan.py
def wrap_description(description: str) -> str: return wrap_text(description, 80)

# net/http_client.py
def get_json(url: str, timeout: int) -> dict: ...
# loan_system/catalog.py
def fetch_catalog() -> dict: return get_json(BASE_URL + "/catalog", DEFAULT_TIMEOUT_SEC)
```

---

## Examples Library (Ready-to-Copy)

**Memoized expensive call**
```python
# ##################################################################
# memoized embedding
# cut repeated external cost without faking calls; cache key is full parameter tuple
@lru_cache(maxsize=1024)
def embed(model: str, text: str) -> list[float]:
    return call_embed(model=model, text=text)
```

**Loop + helper**
```python
# ##################################################################
# normalise all
# go through and normalise each item in the list
def normalize_all(rows: list[str]) -> list[str]:
    return [normalize(r) for r in rows]

# ##################################################################
# normalise
# make sure there's exactly one space between words
def normalize(s: str) -> str:
    return " ".join(s.strip().split())
```

---

## Workflow Integration

When Python code needs to execute n8n workflows, use the following pattern:

```python
import requests

def execute_n8n_workflow(workflow_id: str, data: dict = None) -> dict:
    payload = {
        "authKey": "c9543cecb08a4f84644110bedf91b4b04493d95e21b528508d94107c99178b28",
        "workflowId": workflow_id
    }
    if data:
        payload["data"] = data

    response = requests.post(
        "https://BFC910259E30AA1A89A40802CF16112CE.asuscomm.com:11133/webhook/run/workflow",
        json=payload
    )
    response.raise_for_status()
    return response.json()

# Example usage:
# execute_n8n_workflow("ppfQrpbtEExtanYe")  # Without data
# execute_n8n_workflow("WORKFLOW_ID", {"key": "value"})  # With data
```

---

## Development Workflow

**For each file change:**
1. Write or modify code following the standards above
2. Run linter: `ruff check --line-length 120`
3. Run tests for that specific file: `pytest src/path/file_test.py`
4. Write test output to `output/testing/`
5. Fix any issues and repeat steps 2-4

**At task completion:**
1. Run `./run check` (which executes `dazpycheck`)
2. Ensure all tests pass
3. Ensure no warnings or lint errors
4. Only then commit

**Key Principles:**
- Verify success through **real, individual tests** for the file being worked on
- Write test output to `output/testing/`
- Clean the repository and re-run checks until zero errors
- `dazpycheck` must be green before any commit
