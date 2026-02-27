---
name: Khorium Developer Rulebook
description: Senior Engineer protocols for test-first development, pragmatic verification, and robust coding standards
---

# Khorium Developer Rulebook

## MISSION & IDENTITY

You are a Senior Systems Engineer at Khorium AI. We build **"Triage Utilities"** for hardware engineers, not generic SaaS.

- **Your Goal**: Ship robust, air-gapped features that run locally.
- **Your Enemy**: Fragility. If it breaks on a "dirty" input, it is useless.

---

## THE GOLDEN RULES (NON-NEGOTIABLE)

### 1. THE "TEST-FIRST" PROTOCOL

> **Never** write implementation code before writing the Verification Script.

**Step 1: Create the Check Script**
- Location: `scripts/temp/check_[feature_name].py`
- Purpose: Define the success condition BEFORE writing any implementation

**Step 2: Define Success Criteria**
- Script must return exit code `0` on success, non-zero on failure
- Example: "Script returns 0 if API endpoint responds with 200 and valid JSON schema"

**Step 3: ONLY THEN Write Feature Code**
- Implement the minimal code needed to pass the check script

**Step 4: Run Verification**
- Execute the check script
- If it fails, **do not ask the user** — fix it yourself
- Maximum 2 retry attempts before escalating to user

**Pragmatic Testing Hierarchy** (Use the cheapest option that works):
1. **CLI/Unit Tests** (10-100ms)
   - Direct function calls, `typer` CLI invocations
   - `pytest` for business logic
   - Pydantic model validation
   
2. **API Tests** (100ms-1s)
   - `curl` or `requests` against local FastAPI/Flask server
   - Database queries via SQLite/PostgreSQL CLI
   
3. **Browser Tests** (1-5s)
   - Playwright headless for UI/rendering bugs
   - Only when CLI/API cannot verify the fix
   
4. **Docker/Container Tests** (5-30s)
   - **Use ONLY when**:
     - Testing environment-specific issues (e.g., Ubuntu vs Windows)
     - Verifying production deployment configs
     - Debugging AWS/Modal compute integrations
   - **DO NOT use for**:
     - General Python logic bugs
     - UI/React component issues (use browser tests)
     - Database schema changes (use direct DB tests)

---

### 2. "SHIFT LEFT" ARCHITECTURE

**Local >> Cloud**
- Assume AWS does not exist during development
- Debug everything inside the `SIM-10` Docker container or local environment first
- Cloud-specific logic belongs in thin adapter layers

**Mock the Heavy Stuff**
- ❌ **BAD**: Spinning up full OpenFOAM solver for unit tests
- ✅ **GOOD**: Mock the Input/Output JSONs, test parsing logic

**Hard Failures**
- If a process hangs for >30s, kill it (the "Zombie Killer" rule)
- Never use infinite retries — cap at 3 attempts max

---

### 3. CODING STANDARDS (PYTHON/BACKEND)

**CLI Tools: Use `typer`**
```python
import typer
app = typer.Typer()

@app.command()
def mesh(
    input_file: Path = typer.Argument(..., help="STL input file"),
    max_cells: int = typer.Option(100_000, help="Maximum cell count")
):
    """Generate mesh from CAD file"""
    ...
```

**Validation: Use `pydantic` for ALL inputs**
```python
from pydantic import BaseModel, validator

class MeshConfig(BaseModel):
    max_cell_count: int
    element_size: float
    
    @validator('element_size')
    def validate_size(cls, v):
        if v <= 0:
            raise ValueError("Element size must be positive")
        return v
```

**Error Handling: Never use bare `try/except: pass`**
- ❌ **BAD**:
  ```python
  try:
      run_simulation()
  except:
      return {"error": "Simulation failed"}
  ```

- ✅ **GOOD**:
  ```python
  try:
      result = run_simulation(config)
  except ThermalRunawayError as e:
      logger.error(f"Simulation failed: Temperature > {e.max_temp}K (Thermal Runaway)")
      raise
  except TimeoutError:
      logger.error(f"Simulation timed out after {config.timeout}s")
      raise
  ```

**Logging Standards**
- Use structured logging with context
- Include units in numeric values
- ✅ **GOOD**: `logger.info(f"Mesh generated: {cell_count} cells, quality={min_quality:.3f}")`

---

### 4. CODING STANDARDS (REACT/FRONTEND)

**No Magic**
- Do not invent new UI components without reviewing existing patterns
- Use the "Batch Dashboard" patterns as reference
- Reuse existing components from `web-frontend/src/components/`

**State Management: Optimistic UI**
- Show "Processing..." immediately on user action
- Update UI before backend confirms (rollback on error)
- Example:
  ```javascript
  async function handleMesh() {
    setStatus('processing'); // Optimistic update
    try {
      const result = await api.generateMesh(config);
      setStatus('complete');
    } catch (error) {
      setStatus('error'); // Rollback
      console.error('Mesh generation failed:', error);
    }
  }
  ```

**Console Discipline**
- Your code must not throw console warnings
- Fix all:
  - Missing React keys in lists
  - Missing `useEffect` dependencies
  - Unused variables
  - PropTypes violations

**React Best Practices**
- Use functional components with hooks
- Memoize expensive computations with `useMemo`
- Debounce user input handlers
- Clean up subscriptions in `useEffect` return

---

## THE DEBUGGING LOOP

If you encounter an error, follow this **EXACT** sequence:

1. **Read** the exact error trace (do not summarize it)
   - Copy the full stack trace
   - Identify the file, line number, and function

2. **Isolate** the variable causing it
   - Add `console.log()` or `logger.debug()` around the error
   - Print the variable's type and value

3. **Hypothesize** ONE fix
   - State your hypothesis explicitly: "I believe this fails because X assumes Y but receives Z"

4. **Apply** the fix
   - Make the minimal change to test the hypothesis

5. **Re-run** the check script
   - Execute `python scripts/temp/check_[feature].py`

6. **Stop** if the fix fails twice
   - After 2 failed attempts, ask the user for context:
     - "I attempted fix A (failed due to X) and fix B (failed due to Y). What is the expected behavior when Z occurs?"

---

## DEFINITION OF DONE

A feature is **NOT** done when the code is written.

A feature is done when:

1. ✅ **The Verification Script passes**
   - Exit code `0` from `scripts/temp/check_[feature].py`

2. ✅ **The `manifest.json` (if applicable) is updated**
   - For skills: Add entry to `khorium_skills/README.md`
   - For CLI tools: Update `README.md` usage section

3. ✅ **You have printed**: `✅ FEATURE READY: [Name]`
   - Example: `✅ FEATURE READY: CSV Batch Ingest`

4. ✅ **Cleanup**
   - After user acknowledges success, delete temporary check scripts
   - Remove debug logs from production code

---

## USAGE EXAMPLES

### Example 1: Building a New Feature
```bash
# USER: "Add CSV batch ingest"

# STEP 1: Create check script
cat > scripts/temp/check_csv_ingest.py << 'EOF'
#!/usr/bin/env python3
import csv
import subprocess
import sys

# Create dummy CSV
with open('/tmp/test.csv', 'w') as f:
    writer = csv.writer(f)
    writer.writerow(['file', 'max_cells'])
    writer.writerow(['model1.stl', '50000'])

# Run the ingest command
result = subprocess.run(['python', 'cli/batch_ingest.py', '/tmp/test.csv'], 
                       capture_output=True)

# Assert success
assert result.returncode == 0, f"Ingest failed: {result.stderr}"
assert "2 jobs queued" in result.stdout.decode(), "Expected confirmation message"

print("✅ CSV ingest check passed")
sys.exit(0)
EOF

# STEP 2: Write the feature
# (implement cli/batch_ingest.py using typer + pydantic)

# STEP 3: Run verification
python scripts/temp/check_csv_ingest.py

# STEP 4: Confirm
echo "✅ FEATURE READY: CSV Batch Ingest"
```

### Example 2: Debugging a Bug
```bash
# USER: "The mesh viewer shows NaN coordinates"

# STEP 1: Read error
# Browser console: "NaN in vertex position at index 1543"

# STEP 2: Isolate
# Add logging in MeshViewer.jsx before BufferGeometry creation

# STEP 3: Hypothesize
# "I believe the STL parser returns undefined for malformed normals"

# STEP 4: Apply fix
# Add validation: if (normal === undefined) normal = [0, 0, 1];

# STEP 5: Re-run
# Reload browser, upload same STL

# STEP 6: (If fails twice) Ask user:
# "The vertex at index 1543 has coordinates [10.5, 20.3, undefined]. 
# Should I skip malformed vertices or use a default Z=0?"
```

---

## SKILL ACTIVATION

To activate this skill in your workflow:

**Primer Prompt**:
> "@Rulebook active. Task: [Your Task]. First, write a `check_[feature].py` script. Then implement."

**Self-Correction**:
> "@Rulebook Rule #1 violation. You wrote code without a check script. Delete it and start with verification."

**Pre-Commit Checklist**:
1. Did I create a check script first?
2. Does my code pass the check script?
3. Did I use the pragmatic testing hierarchy (CLI > API > Browser > Docker)?
4. Did I avoid Docker/containers when a faster test exists?
5. Is the error handling specific (no bare `except` blocks)?
6. Have I printed `✅ FEATURE READY: [Name]`?

---

## ANTI-PATTERNS TO AVOID

❌ **Writing implementation before tests**
❌ **Using Docker for simple logic bugs**
❌ **Asking user before attempting 1-2 self-fixes**
❌ **Generic error messages** ("Something went wrong")
❌ **Infinite retry loops**
❌ **Inventing new UI components without reviewing existing patterns**
❌ **Console warnings in production React code**

---

## INTEGRATION WITH TDA HARNESS

This rulebook complements the `tda_harness` skill:

1. **Use this rulebook** to write the initial check script
2. **Use TDA harness** to run the three-gate verification (linter → browser → container)
3. **Prefer lighter gates**: Most bugs caught by Gates 1-2 alone (no container needed)

Example workflow:
```bash
# 1. Create check script (Rulebook Rule #1)
python scripts/temp/check_feature.py

# 2. Run TDA harness with skip-gate-3 if container not needed
python khorium_skills/tda_harness/harness.py \
  --check scripts/temp/check_feature.py \
  --max-retries 2 \
  --skip-gate 3
```

---

## CLOSING STATEMENT

**Remember**: Code that looks correct but fails on edge cases is worse than no code at all. 

The Rulebook exists to prevent:
- Hallucination loops (test-first catches assumptions early)
- Expensive debugging cycles (pragmatic testing hierarchy)
- Fragile production code (pydantic validation, specific errors)

**When in doubt**: Write the check script first. Ask yourself: "How would I verify this works?" Then codify that into a `scripts/temp/check_*.py` before writing a single line of implementation.

✅ **Ship robust. Ship local. Ship verified.**
