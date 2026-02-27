---
name: code-generation
description: Use when generating boilerplate code, __init__.py files, or test scaffolds. Provides scripts that generate consistent, convention-following code.
---

# Code Generation Tools

**When to use:** When creating new modules, updating exports, or scaffolding tests.

---

## generate_inits.py

**Purpose:** Auto-generate `__init__.py` files with proper `__all__` exports.

**Use when:**
- Adding new public functions/classes to a module
- Creating a new package
- Cleaning up exports after refactoring

**Usage:**

```bash
python scripts/generate_inits.py
```

**How it works:**
1. Scans Python modules for public names (classes, functions, constants)
2. Generates `__init__.py` with `__all__` listing public exports
3. Uses config from `scripts/configs/generate_inits_config.yml`

**What it exports:**
- Top-level classes and functions (not private `_*`)
- Module-level constants (ALL_CAPS)
- Filters out banned exports per config

**Decision rule:** After adding public functions to a module, run this to update exports.

---

## generate_tests.py

**Purpose:** Generate test scaffolds with smart assertions and proper fixtures.

**Use when:**
- Adding tests for a new module
- Creating test structure for existing code
- Getting a head start on test implementation

**Usage:**

```bash
# Generate tests for a module
python scripts/generate_tests.py nomarr.services.domain.tagging_svc --output tests/unit/services/test_tagging_svc.py

# Preview without writing
python scripts/generate_tests.py nomarr.components.ml.ml_embed_comp --preview

# Specify layer for auto-fixture selection
python scripts/generate_tests.py nomarr.workflows.processing.process_file_wf --layer workflows
```

**Generated tests include:**
- Proper pytest structure
- Fixtures for layer-appropriate mocks (DB, config, ML backends)
- Test functions for each public method
- Type-appropriate assertions

---

## Workflow: Adding a New Module

1. **Create the module** with your functions/classes

2. **Update exports:**
   ```bash
   python scripts/generate_inits.py
   ```

3. **Generate test scaffold:**
   ```bash
   python scripts/generate_tests.py nomarr.components.new_comp --output tests/unit/components/test_new_comp.py --preview
   
   # If preview looks good:
   python scripts/generate_tests.py nomarr.components.new_comp --output tests/unit/components/test_new_comp.py
   ```

4. **Fill in test implementations**

---

## Workflow: After Refactoring Exports

```bash
# After adding/removing public functions:
python scripts/generate_inits.py

# Review changes:
git diff nomarr/*/__init__.py
```

---

## Configuration

### generate_inits_config.yml

Located at `scripts/configs/generate_inits_config.yml`:

```yaml
# Packages to scan
packages:
  - nomarr.services
  - nomarr.workflows
  - nomarr.components
  - nomarr.persistence
  - nomarr.helpers

# Names to never export
banned_exports:
  - TYPE_CHECKING
  - annotations
```

---

## Key Rules

- **Run `generate_inits.py` after adding public symbols** — keeps exports consistent
- **Use `--preview` before writing test files** — verify structure is correct
- **Generated tests are scaffolds** — you still need to fill in assertions and edge cases
