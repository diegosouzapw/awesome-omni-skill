---
name: proof-driven-development
description: Run proof-driven development using Lean 4 for formal verification - comprehensive skill handling both design (planning) and execution (verification)
---

# SKILL: Proof-Driven Validation (Lean 4)

## Capability

Run Lean 4/Lake proof validation with tiered commands (CHECK/VALIDATE/GENERATE/REMEDIATE) and explicit exit codes. Provides formal verification of algorithms, data structures, and critical properties through machine-checked proofs. This skill handles both DESIGN (planning proofs from requirements) and EXECUTION (creating and verifying proofs).

## When to Use

- Verifying algorithm correctness (sorting, searching, graph algorithms)
- Proving safety properties (bounds, termination, invariants)
- Formalizing specifications before implementation
- Critical path verification requiring mathematical guarantees
- Documenting design decisions with machine-checked proofs

## Inputs

- Working directory with `lakefile.lean` or `.lean` sources (or requirements for new proofs)
- Lean 4 toolchain installed (via elan)
- Lake build system (bundled with Lean 4)

## Preconditions

```bash
# Verify Lean 4 toolchain
(command -v lake || command -v lean) >/dev/null || exit 11

# Verify Lean artifacts exist
fd -g 'lakefile.lean' -e lean . >/dev/null || exit 12
```

---

## Workflow Overview

```
PLAN -> CREATE -> VERIFY -> REMEDIATE -> SUCCESS
  ^                                         |
  +-----------------------------------------+

1. PLAN: Design proofs from requirements (theorem statements, dependencies)
2. CREATE: Generate .lean files with sorry placeholders
3. VERIFY: lake build, check compilation
4. REMEDIATE: Replace sorry with tactics
5. SUCCESS: Zero sorry, builds clean (exit 0)
```

---

## Phase 1: PLAN (Design Proofs from Requirements)

Design proof artifacts BEFORE implementation. Proofs guide implementation, not the reverse.

### 1.1 Understand Requirements

Parse user's task/requirement to identify proof candidates:

- **Safety properties**: "Bad things never happen" (no crashes, no corruption)
- **Liveness properties**: "Good things eventually happen" (termination, progress)
- **Functional correctness**: Algorithms produce correct results for ALL inputs
- **Invariants**: Properties that must hold throughout execution

### 1.2 Artifact Detection

Check for existing Lean 4 artifacts:

```bash
fd -e lean $ARGUMENTS
fd -g 'lakefile.lean' $ARGUMENTS
```

- If artifacts exist: analyze coverage gaps, plan extensions
- If no artifacts: proceed to design new proof architecture

### 1.3 Design Proof Architecture

Use thinking tools to plan the proof structure:

**Property Classification:**

```
Critical Path Properties (MUST prove):
- [Property 1]: {description} -> theorem {name}
- [Property 2]: {description} -> theorem {name}

Secondary Properties (Should prove):
- [Property 3]: {description} -> theorem {name}

Test-Only Properties (Too complex to prove):
- [Property 4]: {description} -> property test
```

**Lean 4 Project Structure:**

```
.outline/proofs/
├── lakefile.lean           # Build configuration
├── lean-toolchain          # v4.x.x
└── ProjectProofs/
    ├── Basic.lean          # Core definitions
    ├── Properties.lean     # Theorem statements
    └── Proofs/
        ├── Safety.lean     # Safety proofs
        ├── Liveness.lean   # Termination proofs
        └── Correctness.lean # Functional correctness
```

### 1.4 Design Theorem Statements

```lean
-- Template for theorem design
namespace ProjectProofs

-- From requirement: {requirement text}
-- Property: {what we're proving}
theorem {property_name} (params : Types) :
  {precondition} -> {postcondition} := by
  sorry  -- Proof to be constructed in CREATE phase

-- Example: Balance never negative
theorem balance_non_negative (acc : Account) (ops : List Operation) :
  valid_operations ops -> (apply_ops acc ops).balance >= 0 := by
  sorry

-- Example: Sort correctness
theorem sort_correct (xs : List Nat) :
  sorted (sort xs) /\ permutation xs (sort xs) := by
  sorry

end ProjectProofs
```

### 1.5 Plan Proof Strategies

| Theorem                | Tactic Strategy                         | Dependencies            |
| ---------------------- | --------------------------------------- | ----------------------- |
| `balance_non_negative` | induction on ops, omega for arithmetic  | `valid_operations`      |
| `sort_correct`         | induction on list, simp for permutation | `sorted`, `permutation` |

### Thinking Tool Integration

```
Use sequential-thinking for:
- Decomposing complex theorems into lemmas
- Planning proof dependencies
- Ordering proof obligations

Use actor-critic-thinking for:
- Challenging proof approaches
- Evaluating alternative tactics
- Assessing proof completeness

Use shannon-thinking for:
- Identifying proof gaps
- Risk of incompleteness
- Tactic selection uncertainty
```

---

## Phase 2: CREATE (Generate Validation Artifacts)

```bash
# Create .outline/proofs directory
mkdir -p .outline/proofs

# Initialize Lake project if needed
test -f .outline/proofs/lakefile.lean || {
  cd .outline/proofs
  lake init proofs
  cd ../..
}
```

### Generate Proof Files from Plan

Create theorem files with `sorry` placeholders from the plan design:

```lean
-- .outline/proofs/{Module}.lean
-- Generated from plan design

import Mathlib.Tactic

/-!
# {Module Name}

## Source Requirements
{traceability from plan}

## Properties Being Proved
{from plan design document}
-/

-- Theorem: {property from plan}
-- Traces to: {requirement reference}
theorem property_name : {statement from plan} := by
  sorry -- To be completed

-- Supporting lemma
lemma helper_lemma : {statement} := by
  sorry
```

---

## Phase 3: VERIFY (Validation)

### Basic (Precondition Check)

```bash
# Verify toolchain
(command -v lake || command -v lean) >/dev/null || exit 11

# Verify artifacts exist
fd -g 'lakefile.lean' -e lean .outline/proofs >/dev/null || exit 12
```

### Intermediate (Build Validation)

```bash
# Lake project build
cd .outline/proofs && lake build || exit 13

# Standalone files
fd -e lean .outline/proofs -x lean --make {} || exit 13
```

### Advanced (Full Verification)

```bash
# Lake project with tests
cd .outline/proofs && lake test || exit 13

# Check for incomplete proofs
rg -n '\bsorry\b' .outline/proofs/ && {
  echo "Incomplete proofs found - proceeding to remediation"
}
```

### Full Verification Sequence

```bash
cd .outline/proofs
lake build 2>&1 | tee build.log
SORRY_COUNT=$(rg -c '\bsorry\b' . || echo "0")
echo "Remaining sorry count: $SORRY_COUNT"
test "$SORRY_COUNT" = "0" || exit 13
```

---

## Phase 4: REMEDIATE (Fix Issues)

### Complete `sorry` Placeholders

For each `sorry` found, apply appropriate tactics:

| Goal Type   | Recommended Tactics                 |
| ----------- | ----------------------------------- |
| Equality    | `rfl`, `simp`, `rw [h]`             |
| Arithmetic  | `linarith`, `omega`, `ring`         |
| Inductive   | `induction`, `cases`, `constructor` |
| Existential | `use x`, `exists x`                 |
| Universal   | `intro h`, `intros`                 |
| Complex     | `aesop`, `decide`, `native_decide`  |

### Resolving `sorry` Step by Step

**Step 1: Understand the goal**

```lean
theorem example_theorem : P := by
  sorry  -- Hover or check goal in editor
```

**Step 2: Try simple tactics first**

```lean
-- For equalities
theorem eq_example : 1 + 1 = 2 := by rfl
theorem eq_example2 (h : a = b) : a = b := h

-- For arithmetic
theorem arith_example (x : Nat) (h : x > 0) : x >= 1 := by omega

-- For logic
theorem logic_example : P or not P := by decide  -- if decidable
```

**Step 3: Use case analysis or induction**

```lean
theorem induct_example : forall n : Nat, n + 0 = n := by
  intro n
  induction n with
  | zero => rfl
  | succ n ih => simp [Nat.succ_add, ih]
```

**Step 4: Add intermediate lemmas**

```lean
theorem complex_theorem : P := by
  have h1 : Q := by exact proof_of_Q
  have h2 : Q -> P := by exact proof_of_implication
  exact h2 h1
```

### Termination Hints

```lean
-- Explicit termination measure
def factorial : Nat -> Nat
  | 0 => 1
  | n + 1 => (n + 1) * factorial n
termination_by n => n

-- Custom well-founded relation
def ackermann : Nat -> Nat -> Nat
  | 0, m => m + 1
  | n + 1, 0 => ackermann n 1
  | n + 1, m + 1 => ackermann n (ackermann (n + 1) m)
termination_by n m => (n, m)

-- Decreasing proof
def merge : List a -> List a -> List a
  | [], ys => ys
  | xs, [] => xs
  | x :: xs, y :: ys =>
    if x <= y then x :: merge xs (y :: ys)
    else y :: merge (x :: xs) ys
termination_by xs ys => xs.length + ys.length
```

---

## Commands Reference

### Basic Commands

| Command   | Purpose                        | Usage                                                                   |
| --------- | ------------------------------ | ----------------------------------------------------------------------- |
| CHECK     | Verify toolchain and artifacts | `(command -v lake \|\| command -v lean) >/dev/null \|\| exit 11`        |
| VALIDATE  | Run proofs or build            | `test -f lakefile.lean && lake test \|\| fd -e lean -x lean --make {}`  |
| GENERATE  | Build without tests            | `test -f lakefile.lean && lake build \|\| fd -e lean -x lean --make {}` |
| REMEDIATE | Find incomplete proofs         | `rg -n '\\bsorry\\b' . && exit 13 \|\| exit 0`                          |

### Lake Commands Reference

```bash
# Initialize new project
lake init <project-name>

# Build and verify all proofs
lake build

# Build specific target
lake build <package>:<target>

# Check file without building dependencies
lake env lean --run src/Proofs.lean

# Interactive proof development (LSP mode)
lake env lean --server

# Update dependencies
lake update

# Clean build artifacts
lake clean

# Run tests
lake test

# Generate documentation
lake build :docs
```

### Quick Validation Pipeline

```bash
# Full verification
lake build && rg -c '\bsorry\b' . && exit 13 || echo "All proofs verified"
```

---

## Tactic Selection Decision Tree

Use this guide to select the appropriate tactic for your proof goal:

| Goal Type              | Tactic                   | When to Use                                          |
| ---------------------- | ------------------------ | ---------------------------------------------------- |
| `a = a`                | `rfl`                    | Definitional/reflexive equality - **try this first** |
| Rewrite with `a = b`   | `rw [h]`                 | Apply equality to rewrite goal; most common          |
| Simplification         | `simp only [...]`        | Apply lemmas; **avoid bare `simp`**                  |
| Linear arithmetic      | `linarith`               | Linear inequalities, bounds proofs                   |
| Nonlinear integer      | `omega`                  | Integer constraints; Lean 4's strongest              |
| Ring equations         | `ring`                   | Polynomial ring identities                           |
| Decidable propositions | `decide`                 | Computational goals; **only for small**              |
| Case analysis          | `cases x`                | Split on constructors                                |
| Induction              | `induction x`            | Prove base + inductive step                          |
| Exists goal            | `exact <witness, proof>` | Provide witness and proof                            |
| Automation             | `aesop`                  | Last resort; **watch for timeouts**                  |
| Intermediate lemma     | `have h : T := proof`    | Add hypothesis to context                            |

### Complete Tactics Reference

```lean
-- Introduction
intro h          -- Introduce hypothesis
intros           -- Introduce all binders

-- Application
apply f          -- Apply function/lemma to goal
exact e          -- Provide exact proof term
constructor      -- Apply data constructor

-- Rewriting
rw [eq]          -- Rewrite using equality
simp [lemmas]    -- Simplify using lemmas
simp only [...]  -- Controlled simplification (preferred)

-- Case/Induction
cases x          -- Case split on x
induction x      -- Induction on x
  | base => ...  -- Base case
  | step ih => ...  -- Inductive case with hypothesis

-- Local context
have h : T := e  -- Add hypothesis to context
let x := e       -- Local definition

-- Automation
aesop            -- Automated proof search
linarith         -- Linear arithmetic
omega            -- Nonlinear integer arithmetic
ring             -- Polynomial ring identity
decide           -- Computational decision
norm_num         -- Numeric simplification

-- Meta
sorry            -- Placeholder (MUST eliminate before shipping)
```

---

## Proof Organization Patterns

### Module Structure Template

```lean
-- .outline/proofs/Algorithm.lean

import Mathlib.Tactic

/-!
# Algorithm Correctness

## Properties to Prove
- Termination: All recursive calls decrease
- Correctness: Output matches specification
- Efficiency: Bounded by O(n log n)

## Theorem Hierarchy
  Termination
    +-- helper_decreases
    +-- base_case_terminates
  Correctness
    +-- correctness_on_empty
    +-- correctness_on_single
    +-- correctness_main
-/

-- Supporting lemma (prove first)
lemma helper_lemma : forall n, P n := by
  intro n
  sorry

-- Main theorem (depends on lemma)
theorem main_property : forall xs, correct xs := by
  intro xs
  induction xs with
  | nil => simp [helper_lemma]
  | cons x xs ih =>
    -- Use helper_lemma and inductive hypothesis
    sorry
```

### Proof by Induction Template

```lean
theorem prop : forall n, P n := by
  intro n
  induction n with
  | zero =>
    -- Base case: prove P 0
    simp [definition]
  | succ n ih =>  -- ih : P n
    -- Inductive case: use ih to prove P (n + 1)
    rw [definition]
    exact ih  -- or apply helper lemmas
```

### Proof by Cases Template

```lean
theorem prop (h : x = a or x = b) : Q x := by
  cases h with
  | inl h_eq =>
    -- Case: x = a, h : x = a
    rw [h_eq]
    sorry
  | inr h_eq =>
    -- Case: x = b, h : x = b
    rw [h_eq]
    sorry
```

---

## Exit Codes

| Code | Meaning                     | Resolution                             |
| ---- | --------------------------- | -------------------------------------- |
| 0    | All proofs verified         | Success                                |
| 11   | Lean/Lake not installed     | Install via `elan`                     |
| 12   | No Lean artifacts found     | Create `.lean` files or run PLAN phase |
| 13   | Incomplete proofs (`sorry`) | Complete proofs using tactics          |
| 14   | Coverage/totality gaps      | Add missing cases or lemmas            |

---

## Troubleshooting Guide

### Common Issues

| Symptom                            | Cause                                 | Resolution                                     |
| ---------------------------------- | ------------------------------------- | ---------------------------------------------- |
| Exit 11                            | Lean 4 / Lake not found               | `elan install` or `brew install lean4`         |
| Exit 12                            | No .lean files in `.outline/proofs/`  | Run PLAN phase first to generate artifacts     |
| Exit 13                            | `sorry` in proofs                     | Replace `sorry` with appropriate tactics       |
| Exit 14                            | Missing lemmas                        | Add helper lemmas from plan design             |
| `unknown identifier`               | Missing import                        | Add `import Mathlib.Tactic` or specific module |
| `type mismatch`                    | Proof term doesn't match goal         | Add type ascription or derive required form    |
| `failed to prove termination`      | Recursion not structurally decreasing | Add `termination_by` clause                    |
| `maximum recursion depth exceeded` | Infinite loop in tactic               | Use `simp only [...]` instead of `simp`        |
| `tactic 'aesop' failed`            | Search space too large                | Break down goal or add intermediate `have`     |

### Tactic Selection Decision Tree (Troubleshooting)

```
Goal Shape -> Recommended Tactic
---------------------------------
a = a                         -> rfl (try first)
rewrite with known eq h       -> rw [h]
trivial simplification        -> simp only [lemma1, lemma2]
linear arithmetic             -> linarith
nonlinear integer             -> omega
ring/field identity           -> ring
decidable goal (small)        -> decide
case analysis on x            -> cases x
structural induction on n     -> induction n
exists goal                   -> exact (witness, proof)
need intermediate fact        -> have h : T := proof
stuck (last resort)           -> aesop (watch for timeouts)
```

### Debugging Commands

```bash
# Interactive debugging in Lean REPL
cd .outline/proofs && lake env lean --run Main.lean

# Check specific theorem interactively
#check theorem_name
#print theorem_name
#reduce expression

# Evaluate expressions
#eval 1 + 1
#eval decide (2 < 3)

# Find tactics/theorems
#check @Nat.add_comm
example : 1 + 2 = 2 + 1 := Nat.add_comm 1 2

# Verbose build with error details
lake build --verbose 2>&1 | tee debug.log

# Find all incomplete proofs
rg -n '\bsorry\b' .outline/proofs/

# Check specific file for errors
lean --make .outline/proofs/Module.lean 2>&1

# Lake clean rebuild
lake clean && lake build
```

### Common Pitfalls and Solutions

#### Pitfall 1: Stuck Proofs with `sorry`

**Problem:** Goal won't close; using `sorry` as placeholder.

**Solution:**

```lean
-- Add debugging
theorem stuck : P := by
  intro x
  -- Debug: inspect goal and context
  trace "{x}"
  sorry

-- Use exploration commands in REPL
#check expr        -- Verify type
#eval expr         -- Compute value (if decidable)
#reduce expr       -- Full reduction
```

#### Pitfall 2: Type Mismatch

**Problem:** `rw [h]` fails; types don't align.

**Solution:**

```lean
-- Bad: assumes h : a = b but need different form
theorem foo (h : f a = f b) : P := by
  rw [h]  -- Error: type mismatch

-- Good: derive what you need first
theorem foo (h : f a = f b) : P := by
  have : a = b := sorry  -- Derive needed form
  rw [this]
```

#### Pitfall 3: Non-Termination

**Problem:** Recursion doesn't terminate; build hangs.

**Solution:**

```lean
-- Add explicit termination measure
def foo (n : Nat) : P := by
  induction n with
  | zero => rfl
  | succ n ih => simp [ih]
  termination_by n  -- Explicit measure
```

#### Pitfall 4: Over-Generalization with `simp`

**Problem:** `simp` oversimplifies or creates unprovable goals.

**Solution:**

```lean
-- Bad: simp alone is unpredictable
theorem foo : P := by simp  -- Might fail later

-- Good: be explicit
theorem foo : P := by
  simp only [definition]  -- Only specific lemmas
  exact bar
```

---

## Performance Tips

### Avoid Exponential Blowup

```lean
-- SLOW: simp with many lemmas
theorem slow : P := by
  simp [a, b, c, d, e, f, g, h, ...]  -- Can timeout

-- FAST: targeted simplification
theorem fast : P := by
  simp only [a, b]
  rw [c]
  exact d
```

### Use `decide` Sparingly

```lean
-- OK for small goals
theorem small : 2 + 2 = 4 := by decide

-- AVOID for large computations
-- theorem big : fib 100 = ... := by decide  -- Very slow!
```

### Increase Timeout for Complex Proofs

```lean
set_option maxHeartbeats 100000  -- Increase solver timeout
```

---

## Style Guidelines

1. **`by` placement:** Same line or next line with indent
   ```lean
   -- GOOD
   theorem foo : P := by rfl

   -- GOOD (multi-line)
   theorem foo : P := by
     intro x
     simp
   ```

2. **Indentation:** 2 spaces; consistent for focused subgoals
   ```lean
   theorem foo : P and Q := by
     constructor
     . -- Subgoal 1: P
       simp
     . -- Subgoal 2: Q
       exact bar
   ```

3. **Naming:** `property + qualifier`
   - `add_assoc`, `list_append_length`
   - `helper_preserves_invariant`
   - Avoid: `l1`, `aux`, `temp`

---

## When NOT to Use Proof-Driven Development

| Scenario                       | Better Alternative                  |
| ------------------------------ | ----------------------------------- |
| Simple CRUD operations         | Design-by-contract (runtime checks) |
| Rapidly changing requirements  | Test-driven (easier to update)      |
| Performance-critical hot paths | Type-driven (compile-time only)     |
| UI/UX code                     | Property-based testing              |
| Prototyping / exploration      | Test-driven development             |
| Team unfamiliar with Lean      | Design-by-contract + property tests |

## Complementary Approaches

- **Type-driven + Proof-driven**: Use Idris 2 for implementation types, Lean 4 for complex invariants
- **Validation-first + Proof-driven**: Quint for state machine, Lean 4 for mathematical properties
- **Test-driven + Proof-driven**: Property tests for exploration, proofs for critical invariants

---

## Integration Points

- **Quint:** Verify Quint-specified properties in Lean 4
- **Rust:** Translate verified algorithms to Rust implementation
- **Testing:** Generate test cases from proof traces

---

## Safety

- Read-only operations; no file mutations during validation
- Abort on exit >= 11 or if safety concerns raised (code 3)
- Proofs are machine-checked; no runtime overhead

---

## Output Report

Provide:

- Files created in `.outline/proofs/`
- Build status (pass/fail)
- `sorry` count before/after remediation
- Theorem verification status
- Traceability update (requirement -> theorem -> proof status)

---

## Resources

- [Theorem Proving in Lean 4](https://leanprover.github.io/theorem_proving_in_lean4/)
- [Mathlib Documentation](https://leanprover-community.github.io/mathlib4_docs/)
- [Lean 4 Tactics Guide](https://leanprover-community.github.io/contribute/style.html)
