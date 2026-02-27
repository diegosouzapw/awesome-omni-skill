---
name: apply-key-principles
description: Apply and validate the 7 Key Principles to code - TDD, Fail Fast, Modular, Reuse, Open Source, No Debt, Excellence. Checks code compliance and suggests improvements. Use during code review or refactoring.
allowed-tools: [Read, Grep, Glob, Bash]
---

# apply-key-principles

**Skill Type**: Validator/Sensor
**Purpose**: Validate code compliance with 7 Key Principles
**Prerequisites**: Code exists to validate

---

## Agent Instructions

You are validating code against the **7 Key Principles**.

**The 7 Key Principles**:
1. **Test Driven Development** - "No code without tests"
2. **Fail Fast & Root Cause** - "Break loudly, fix completely"
3. **Modular & Maintainable** - "Single responsibility, loose coupling"
4. **Reuse Before Build** - "Check first, create second"
5. **Open Source First** - "Suggest alternatives, human decides"
6. **No Legacy Baggage** - "Clean slate, no debt"
7. **Perfectionist Excellence** - "Best of breed only"

**Your role**: Check code compliance and report violations.

---

## Validation Checks

### Principle #1: Test Driven Development

**Check**: Does code have tests?

**Validation**:
```bash
# For each file in src/, check if tests exist
for file in src/**/*.py; do
  test_file="tests/$(basename $file | sed 's/\.py$//')"
  if [ ! -f "${test_file}_test.py" ]; then
    echo "VIOLATION: $file has no tests"
  fi
done
```

**✅ Pass**:
- Every production file has corresponding test file
- Tests were written first (git history shows RED before GREEN commits)
- Coverage >= 80%

**❌ Fail**:
- Code without tests
- Tests written after code (git history shows)
- Coverage < 80%

---

### Principle #2: Fail Fast & Root Cause

**Check**: Does code fail loudly?

**Validation**:
```python
# Look for silent failures
grep -rn "except.*pass" src/          # Empty except blocks (silent failures)
grep -rn "return None" src/ | grep -v "Optional"  # Silent None returns
grep -rn "# TODO: error handling" src/  # Deferred error handling
```

**✅ Pass**:
- Exceptions raised for invalid states
- Assertions check preconditions
- Specific error messages
- Logging for debugging

**❌ Fail**:
- Silent failures (empty except blocks)
- Generic error messages
- Swallowing exceptions
- No logging

---

### Principle #3: Modular & Maintainable

**Check**: Is code modular?

**Validation**:
```bash
# Check file/function sizes
find src -name "*.py" -exec wc -l {} \; | awk '$1 > 300'  # Files > 300 lines

# Check cyclomatic complexity
radon cc src/ -a | grep "F"  # Functions with F rating (too complex)
```

**✅ Pass**:
- Files < 300 lines
- Functions < 50 lines
- Cyclomatic complexity <= 10
- Single responsibility per module
- Low coupling

**❌ Fail**:
- Large files (> 300 lines)
- Large functions (> 50 lines)
- High complexity (> 10)
- Mixed responsibilities

---

### Principle #4: Reuse Before Build

**Check**: Is this functionality already available?

**Validation**:
```bash
# Search for similar code in project
grep -rn "function_name" src/

# Check for duplicate code
jscpd src/  # Copy-paste detector
```

**✅ Pass**:
- No duplicate code
- Reusing existing functions/classes
- Using standard libraries where appropriate

**❌ Fail**:
- Duplicate code blocks
- Reimplementing existing functionality
- Not using available libraries

---

### Principle #5: Open Source First

**Check**: Could we use an open source library?

**Validation**:
- For custom implementations, check if library exists
- Document decision if building custom

**✅ Pass**:
- Using well-maintained libraries
- Documented decision to build custom (ADR)
- Libraries chosen after research

**❌ Fail**:
- Building custom without research
- Reinventing wheel (custom date parser, custom validation, etc.)

---

### Principle #6: No Legacy Baggage

**Check**: Is code clean of technical debt?

**Validation**:
```bash
# Unused imports
grep -rn "^import\|^from" src/ | check_usage

# Dead code
find_functions_with_zero_callers src/

# Commented code
grep -rn "# " src/ | grep -v "^#" | check_if_code

# Complexity
radon cc src/ -a | grep -E "C|D|E|F"
```

**✅ Pass**:
- No unused imports
- No dead code
- No commented-out code
- Complexity <= 10
- No TODOs without tickets

**❌ Fail**:
- Any technical debt present

**If FAIL**: Invoke `prune-unused-code`, `simplify-complex-code`

---

### Principle #7: Perfectionist Excellence

**Check**: Is this excellent code?

**Validation**:
```bash
# Naming quality
grep -rn " x " src/  # Single-letter variables
grep -rn " temp" src/  # Temp variables

# Documentation
find src -name "*.py" -exec grep -L '"""' {} \;  # Files without docstrings

# Type hints (Python)
grep -rn "def " src/ | grep -v " -> "  # Functions without return types

# Style compliance
pylint src/ --errors-only
black src/ --check
```

**✅ Pass**:
- Clear, descriptive names
- Comprehensive documentation
- Type hints/annotations
- Follows style guide (PEP 8, etc.)
- Code review ready

**❌ Fail**:
- Vague naming
- Missing documentation
- No type hints
- Style violations

---

## Output Format

**When all principles satisfied**:

```
[APPLY KEY PRINCIPLES]

Validating code against 7 Key Principles...

✅ Principle #1: Test Driven Development
   - All files have tests
   - Tests written first (git history verified)
   - Coverage: 95.2%

✅ Principle #2: Fail Fast & Root Cause
   - No silent failures
   - Specific error messages
   - Comprehensive logging

✅ Principle #3: Modular & Maintainable
   - Max file size: 187 lines (< 300)
   - Max function size: 23 lines (< 50)
   - Max complexity: 6 (< 10)

✅ Principle #4: Reuse Before Build
   - No duplicate code
   - Using standard libraries (bcrypt, datetime)
   - Searched codebase first

✅ Principle #5: Open Source First
   - Using bcrypt library (not custom hashing)
   - Documented in ADR-001

✅ Principle #6: No Legacy Baggage
   - Tech debt: 0 violations
   - No unused imports
   - No dead code
   - No commented code
   - Max complexity: 6

✅ Principle #7: Perfectionist Excellence
   - Clear naming (validate_email, normalize_email)
   - Comprehensive docstrings
   - Type hints on all functions
   - Style: PEP 8 compliant

Result: 7/7 Principles Satisfied ✅

Quality: EXCELLENT 🔥
Ready for commit/deployment
```

**When violations found**:

```
[APPLY KEY PRINCIPLES - VIOLATIONS FOUND]

❌ Principle #6: No Legacy Baggage

Violations (5):
  1. Unused import: import hashlib (src/auth.py:3)
  2. Dead function: legacy_hash_password() (src/auth.py:67-74)
  3. Commented code: Lines 120-135 (src/auth.py)
  4. High complexity: login() complexity 14 (src/auth.py:89)
  5. TODO without ticket: # TODO: Add rate limiting (src/auth.py:145)

❌ Principle #7: Perfectionist Excellence

Violations (3):
  1. Missing docstring: _check_password() (src/auth.py:156)
  2. No type hint: def validate(email) (src/auth.py:178)
  3. Vague naming: variable 'x' (src/auth.py:192)

Result: 5/7 Principles Satisfied ⚠️

Violations: 8 total
Quality: NEEDS IMPROVEMENT

Actions Required:
  1. Invoke 'prune-unused-code' to fix Principle #6
  2. Fix naming, docs, types for Principle #7
  3. Re-run validation after fixes

Blocked: Fix violations before commit
```

---

## Prerequisites Check

Before invoking:
1. Code exists to validate
2. Testing tools available (for Principle #1)

---

## Configuration

```yaml
plugins:
  - name: "@aisdlc/principles-key"
    config:
      principles:
        enforce_tdd: true
        enforce_fail_fast: true
        enforce_modular: true
        enforce_reuse_first: true
        enforce_open_source_first: true
        enforce_no_legacy: true
        enforce_excellence: true
        block_on_violation: true

      thresholds:
        max_file_lines: 300
        max_function_lines: 50
        max_complexity: 10
        min_coverage: 80
```

---

## Notes

**Why apply principles?**
- **Operational enforcement** (not just aspirational)
- **Measurable** (can check compliance automatically)
- **Quality gate** (blocks bad code)
- **Continuous validation** (run on every commit)

**Principles manifest in code**:
1. TDD → Tests exist, coverage high
2. Fail Fast → Exceptions, assertions, logging
3. Modular → Small files/functions, low complexity
4. Reuse → No duplication, using libraries
5. Open Source → Libraries documented
6. No Debt → Zero unused code, low complexity
7. Excellence → Clear naming, docs, types

**Homeostasis Goal**:
```yaml
desired_state:
  all_seven_principles_satisfied: true
  violations: 0
  code_quality: excellent
```

**"Excellence or nothing"** 🔥
