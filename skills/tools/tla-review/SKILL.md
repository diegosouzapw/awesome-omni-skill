---
name: tla-review
description: Comprehensive TLA+ specification review with checklist and automated validation
version: 1.0.0
allowed-tools: [Read, Grep, Write, mcp__plugin_tlaplus_tlaplus__tlaplus_mcp_sany_parse, mcp__plugin_tlaplus_tlaplus__tlaplus_mcp_sany_symbol, mcp__plugin_tlaplus_tlaplus__tlaplus_mcp_tlc_smoke]
---

# TLA+ Specification Review

Run a comprehensive review of your TLA+ specification including parsing, symbol extraction, smoke testing, and best practices checklist.

**IMPORTANT: Always use the MCP tools listed above. Never fall back to running Java or TLC commands via Bash.**

## Usage

```
/tla-review test-specs/Counter.tla
/tla-review test-specs/Counter.tla test-specs/Counter.cfg
/tla-review test-specs/Counter.tla --no-smoke
```

**Note:** If you typed `@path.tla` as the first argument, this skill strips the leading `@` and validates the file exists.

## What This Does

1. Validates and normalizes the spec path from the argument
2. Runs SANY parser to check syntax and semantics
3. Extracts symbols to analyze spec structure
4. Runs smoke test (unless `--no-smoke` flag present)
5. Generates comprehensive review report with recommendations

## Implementation

**Step 1: Normalize Spec Path**

Take the spec file path provided as the argument to this skill. If it starts with `@`, strip the leading `@`.

Print `Spec path: <spec_path>`

**Step 2: Validate File**

- Check path ends with `.tla`
- Use the Read tool to verify the file exists on disk
- If validation fails, print error and exit

**Step 3: Parse Flags**

Extract flags from the argument:
- `--no-smoke`: Skip smoke test (default: smoke enabled)

**Step 4: Determine CFG Argument**

Parse the second token from the argument (split by space, take second). If it ends with `.cfg`, treat it as the CFG_ARG.

**Step 5: Run SANY Parser**

Call `mcp__plugin_tlaplus_tlaplus__tlaplus_mcp_sany_parse` with `fileName=<spec_path>`

Store result:
- `PARSE_SUCCESS=true/false`
- `PARSE_ERRORS=<error list>`

**Step 6: Extract Symbols**

Call `mcp__plugin_tlaplus_tlaplus__tlaplus_mcp_sany_symbol` with:
- `fileName=<spec_path>`
- `includeExtendedModules=false`

Store result:
- `SYMBOLS=<symbol extraction result>`
- Extract: `CONSTANTS`, `VARIABLES`, `INIT`, `NEXT`, `SPEC`, `INVARIANTS`, `PROPERTIES`

**Step 7: Run Smoke Test (if enabled)**

If smoke is enabled (no `--no-smoke` flag):

Apply CFG selection algorithm (same as `/tla-smoke`):

**Phase 1: Ensure precondition**

Extract spec name and directory:
```
SPEC_DIR = dirname(SPEC_PATH)
SPEC_NAME = basename(SPEC_PATH, .tla)
```

Check preconditions in order:

1. If `SPEC_DIR/SPEC_NAME.cfg` exists:
   - Print `Phase 1: Spec.cfg exists`
   - Precondition satisfied

2. Else if `SPEC_DIR/MC<SPEC_NAME>.tla` AND `SPEC_DIR/MC<SPEC_NAME>.cfg` both exist:
   - Print `Phase 1: MC pair exists`
   - Precondition satisfied

3. Else if `CFG_ARG` is non-empty and exists:
   - Copy `CFG_ARG` to `SPEC_DIR/SPEC_NAME.cfg` (non-clobbering)
   - Print `Phase 1: Copied cfgArg to SPEC_NAME.cfg`
   - Precondition satisfied

4. Else if `SPEC_DIR/SPEC_NAME.generated.cfg` exists:
   - Copy it to `SPEC_DIR/SPEC_NAME.cfg` (non-clobbering)
   - Print `Phase 1: Copied generated cfg`
   - Precondition satisfied

5. Else:
   - Print `Smoke test skipped: No config file found`
   - Set `SMOKE_SKIPPED=true`
   - Continue to review

**Phase 2: Choose cfg**

If precondition satisfied:

1. If `CFG_ARG` is non-empty:
   - Resolve and use `CFG_ARG` (copy if needed, same as `/tla-smoke`)
   - Print `Phase 2: Using explicit cfgArg`

2. Else:
   - Use default cfg (`Spec.cfg` or `MCSpec.cfg`)
   - Print `Phase 2: Using default cfg`

Store final cfg path in `FINAL_CFG`.

Call `mcp__plugin_tlaplus_tlaplus__tlaplus_mcp_tlc_smoke` with:
- `fileName=<SPEC_PATH>`
- `cfgFile=<FINAL_CFG>`
- `extraJavaOpts=["-Dtlc2.TLC.stopAfter=3"]`

Store result:
- `SMOKE_SUCCESS=true/false`
- `SMOKE_VIOLATIONS=<violation list>`

**Step 8: Generate Review Report**

Print comprehensive review summary:

```
═══════════════════════════════════════════════════════════
TLA+ SPECIFICATION REVIEW
═══════════════════════════════════════════════════════════

Spec: <SPEC_PATH>

─────────────────────────────────────────────────────────
1. SYNTAX & SEMANTICS (SANY Parser)
─────────────────────────────────────────────────────────

<if PARSE_SUCCESS>
Parsing successful. No syntax errors.
<else>
Parsing failed. Errors found:
<PARSE_ERRORS>
<endif>

─────────────────────────────────────────────────────────
2. STRUCTURE ANALYSIS (Symbol Extraction)
─────────────────────────────────────────────────────────

Constants: <CONSTANTS or "None">
Variables: <VARIABLES or "None">
Init: <INIT or "Not detected">
Next: <NEXT or "Not detected">
Spec: <SPEC or "Not detected">
Invariants: <INVARIANTS or "None">
Properties: <PROPERTIES or "None">

<if no INIT or no NEXT or no SPEC>
Warning: Missing behavior specification
  - Ensure Init, Next, and Spec are defined
  - Or define INIT/NEXT in .cfg file
<endif>

<if CONSTANTS non-empty>
Warning: Constants require assignment
  - Edit .cfg file to assign concrete values
  - Example: CONSTANT MaxValue = 10
<endif>

─────────────────────────────────────────────────────────
3. SMOKE TEST (3-second simulation)
─────────────────────────────────────────────────────────

<if SMOKE_SKIPPED>
Skipped (no config file or --no-smoke flag)
<else if SMOKE_SUCCESS>
Smoke test passed
  CFG used: <FINAL_CFG>
  No violations found in random simulation
<else>
Smoke test failed
  CFG used: <FINAL_CFG>
  Violations detected:
<SMOKE_VIOLATIONS>
<endif>

─────────────────────────────────────────────────────────
4. BEST PRACTICES CHECKLIST
─────────────────────────────────────────────────────────

<Check and report on:>

Module documentation
  - Does module have header comment explaining purpose?
  - Are complex operators documented?

Type invariants
  - Are type invariants defined for all variables?
  - Example: TypeInvariant == var \in ExpectedType

Safety properties
  - Are safety invariants defined?
  - Do they cover critical correctness conditions?

Liveness properties
  - Are liveness properties defined if needed?
  - Example: <>[]Termination

Constant bounds
  - Are constants bounded to reasonable values?
  - Large constants cause state explosion

Symmetry
  - Can symmetry sets reduce state space?
  - Example: SYMMETRY SymmetrySet

State constraints
  - Are state constraints needed to limit exploration?
  - Example: CONSTRAINT StateConstraint

─────────────────────────────────────────────────────────
5. RECOMMENDATIONS
─────────────────────────────────────────────────────────

<Generate specific recommendations based on findings:>

<if PARSE_ERRORS>
-> Fix syntax errors before proceeding
<endif>

<if no config file>
-> Run: /tla-symbols <SPEC_PATH>
<endif>

<if CONSTANTS non-empty and no config>
-> Assign constant values in .cfg file
<endif>

<if SMOKE_VIOLATIONS>
-> Fix violations found in smoke test
-> Run: /tla-check for full counterexample
<endif>

<if SMOKE_SUCCESS or SMOKE_SKIPPED>
-> Run: /tla-check for exhaustive verification
<endif>

<if no INVARIANTS>
-> Consider adding type and safety invariants
<endif>

<if no PROPERTIES>
-> Consider adding liveness properties if applicable
<endif>

═══════════════════════════════════════════════════════════
REVIEW COMPLETE
═══════════════════════════════════════════════════════════
```

## Example Output

```
═══════════════════════════════════════════════════════════
TLA+ SPECIFICATION REVIEW
═══════════════════════════════════════════════════════════

Spec: test-specs/Counter.tla

─────────────────────────────────────────────────────────
1. SYNTAX & SEMANTICS (SANY Parser)
─────────────────────────────────────────────────────────

Parsing successful. No syntax errors.

─────────────────────────────────────────────────────────
2. STRUCTURE ANALYSIS (Symbol Extraction)
─────────────────────────────────────────────────────────

Constants: MaxValue
Variables: count
Init: Init
Next: Next
Spec: Spec
Invariants: TypeInvariant, BoundInvariant
Properties: None

Warning: Constants require assignment
  - Edit .cfg file to assign concrete values
  - Example: CONSTANT MaxValue = 10

─────────────────────────────────────────────────────────
3. SMOKE TEST (3-second simulation)
─────────────────────────────────────────────────────────

Smoke test passed
  CFG used: test-specs/Counter.cfg
  No violations found in random simulation

─────────────────────────────────────────────────────────
4. BEST PRACTICES CHECKLIST
─────────────────────────────────────────────────────────

Module documentation - Header comment present
Type invariants - TypeInvariant defined
Safety properties - BoundInvariant defined
Liveness properties - None defined (may not be needed)
Constant bounds - MaxValue = 10 (reasonable)
Symmetry - Not applicable for this spec
State constraints - Not needed (small state space)

─────────────────────────────────────────────────────────
5. RECOMMENDATIONS
─────────────────────────────────────────────────────────

-> Run: /tla-check for exhaustive verification
-> Consider adding liveness properties if termination matters

═══════════════════════════════════════════════════════════
REVIEW COMPLETE
═══════════════════════════════════════════════════════════
```
