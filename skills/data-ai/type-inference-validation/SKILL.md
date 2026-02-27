---
name: type-inference-validation
description: Static type inference and validation for navigation paths
source: Type theory, gradual typing, structure refinement
license: MIT
trit: -1
gf3_triad: "type-inference-validation (-1) ⊗ tuple-nav-composition (0) ⊗ constraint-generalization (+1)"
status: Production Ready
---

# Type Inference Validation

## Core Concept

Static type validation that **rejects invalid path compositions before caching**. Every Navigator path must prove type compatibility with its input structure—this skill enforces that proof.

Works alongside Möbius filtering as a second line of defense: Möbius eliminates topological impossibilities, Type Inference eliminates structural impossibilities.

## Why Type Validation?

Consider this dangerous scenario:
```julia
data_numbers = [1, 2, 3, 4, 5]
data_dict = Dict("x" => 10, "y" => 20)

# This path makes sense for numbers
nav1 = @late_nav([ALL, pred(iseven)])  # Type ✓: Vector → Numbers → Numbers

# But what if someone mistakenly uses it on the dict?
nav_select(nav1, data_dict, x -> [x])
# => IndexError! (dicts don't support ALL enumeration this way)
```

**Type Inference Validation prevents this at compile time**, not runtime.

## Architecture

### Type System

Each Navigator carries a **type signature**:
```
Navigator{InputType, OutputType, PathSteps}
```

Examples:
```julia
nav_vector = Navigator{Vector, Vector, [ALL, pred(f)]}
nav_dict = Navigator{Dict, Vector, [keypath(k), ALL]}
nav_sexp = Navigator{SExpression, Symbol, [sexp_nth(2), SEXP_HEAD]}
```

### Type Refinement Pipeline

```
Input Type
    ↓
@late_nav(path_expr) triggers validation
    ↓
Type each step in path:
  [ALL] : Vector{T} → T
  [keypath(k)] : Dict{K,V} → V
  [pred(f)] : T → T (preserves type)
  [INDEX(i)] : Vector{T} → T
  [SEXP_HEAD] : SExpr → Symbol
    ↓
Unify step outputs with next step inputs
    ↓
Final output type computed
    ↓
Cache with type signature
    ↓
Accept or Reject
```

### Type Incompatibility Detection

| Step | Input Type | Output Type | Valid? |
|------|-----------|------------|--------|
| `[ALL]` | Vector{T} | T | ✓ |
| `[ALL]` | Dict{K,V} | ⚠️ (ambiguous) | ✗ |
| `[keypath(k)]` | Dict{K,V} | V | ✓ |
| `[keypath(k)]` | Vector{T} | ✗ | ✗ |
| `[pred(f)]` | T | T | ✓ |
| `[INDEX(i)]` | Vector{T} | T | ✓ |
| `[INDEX(i)]` | Dict | ✗ | ✗ |

## API

### Type Inference

**`infer_type(path_expr, input_type) :: Result{OutputType, Error}`**
Computes the output type of a path given an input type.

```julia
infer_type([ALL, pred(iseven)], Vector{Int})
# => Result(Int)  # outputs Int values

infer_type([keypath("x"), ALL], Dict{String, Vector{Int}})
# => Result(Int)  # navigates to x, then enumerates integers

infer_type([keypath("x")], Vector{Int})
# => Result(TypeError("Cannot apply keypath to Vector"))
```

**`validate_path(navigator::Navigator, input_type) :: Bool`**
Checks if a Navigator is compatible with a given input type.

```julia
nav = @late_nav([ALL, pred(f)])
validate_path(nav, Vector{Int})      # => true
validate_path(nav, Dict{String, Int})  # => false ✗

# Causes @late_nav to reject the Navigator before caching
```

### Type Signatures

**`navigator_signature(nav::Navigator) :: TypeSignature`**
Returns the full type signature of a cached Navigator.

```julia
nav = @late_nav([ALL, pred(iseven)])
sig = navigator_signature(nav)

# => TypeSignature(
#     input: Vector{T},
#     output: T,
#     constraints: [iseven(T)],
#     steps: [[ALL], [pred(iseven)]],
#     valid_for: Vector{Int}, Vector{BigInt}, ...
#   )
```

### Polymorphic Inference

**`polymorphic_infer(path_expr) :: PolymorphicType`**
Infers the most general type for a path (before knowing input type).

```julia
polymorphic_infer([ALL, pred(iseven)])
# => ∀T. (T ∈ Enumerable, T ∈ HasEven) → T

# This means: works for ANY type T that:
# - Can be enumerated (Vector, Set, List, etc.)
# - Has an iseven() method defined
```

## Integration with Caching

Cache keys now include type information:
```julia
cache_key = hash((path_expr, inferred_type))

# Different input types → different cache entries
nav_vec = @late_nav([ALL, pred(f)])   # cache for Vector
nav_set = @late_nav([ALL, pred(f)])   # cache for Set (if compatible)
# => Different NavigatorObjects, despite same path!
```

## Refinement Types

Supports **refinement types** for predicates:
```julia
# pred(iseven) refines Int → EvenInt
# pred(x -> x > 5) refines Int → IntGt5

nav = @late_nav([ALL, pred(x -> x > 5)])
# Type: Vector{Int} → IntGt5
# (output is proven to be > 5)
```

Refinement types enable:
- **Automatic constraint propagation** downstream
- **Proof that outputs satisfy predicates** without re-checking
- **Composition of constraints** with type safety

## Type Mismatch Examples

### ❌ INVALID: Wrong container type
```julia
nav = @late_nav([keypath("name"), ALL])
# Type error: keypath returns a string, ALL requires enumerable

# KeyPath{Dict, String} → String
# ALL{String} → ✗ (String not enumerable in that way)
```

### ❌ INVALID: Incompatible predicate
```julia
nav = @late_nav([ALL, pred(x -> x > 5)])
# If input is Vector{String}, predicate fails
# Type error: `(String > 5)` is nonsense
```

### ✓ VALID: Polymorphic path
```julia
nav = @late_nav([ALL, pred(identity)])  # identity always works
# Type: ∀T. Vector{T} → T
# Works for ANY vector type
```

## Error Messages

**Type validation errors are clear**:
```
TypeError: Path composition invalid
  Step 1: [ALL]
    Input: Vector{Int}
    Output: Int
    ✓ Type check passed

  Step 2: [keypath("x")]
    Input: Int
    Output: ???
    ✗ TypeError: Cannot apply keypath to Int
       keypath requires Dict or record type
       Got: Int

Suggestion: Remove [keypath("x")] or ensure input is Dict/Struct
```

## Performance

All type checking happens at **compile time** (during `@late_nav` expansion):
| Operation | Complexity | Notes |
|-----------|-----------|-------|
| infer_type | O(\|path\|) | Single pass through steps |
| validate_path | O(1) | Cached signature lookup |
| polymorphic_infer | O(\|path\|) | Compute most general type |
| **Runtime overhead** | O(0) | Zero cost—all checks are static |

## Composition with Other Skills

**Works with Möbius Filtering**:
1. Möbius filters topological impossibilities
2. Type Inference filters structural impossibilities
3. Both must pass for path to compile

**Works with Color Envelopes**:
Type signatures include color trit information:
```julia
nav = @late_nav([ALL, pred(f)])  # trit = -1 (filtering)
sig = navigator_signature(nav)
# => TypeSignature(..., trit: -1)

# Type system respects color: composition must balance trits AND types
```

**Enables Constraint Generalization** (next skill):
Once types are proven, constraints can be composed and generalized safely.

## Related Skills

- **möbius-path-filtering** - Topological validation (works alongside this)
- **tuple-nav-composition** - Uses type signatures to compose products
- **constraint-generalization** - Builds on type-proven constraints
- **specter-navigator-gadget** - Uses validated types for safe composition

## References

- Gradual typing: "Gradual Typing for Functional Languages" - Siek & Taha
- Refinement types: "Liquid Haskell: Haskell as a Theorem Prover" - Jhala et al.
- Type inference: "Algorithm W: Inference of Data Types" - Damas & Milner
