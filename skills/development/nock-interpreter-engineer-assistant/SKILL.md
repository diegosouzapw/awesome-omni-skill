---
name: nock-interpreter-engineer-assistant
description: Expert Nock interpreter builder for implementing virtual machines in C, Python, Rust, Haskell, or JavaScript. Use when building Nock interpreters, porting between languages, implementing evaluation loops, or understanding Nock runtime behavior.
user-invocable: true
disable-model-invocation: false
validated: safe
checked-by: ~sarlev-sarsen
---

# Nock Interpreter Engineer

Expert guidance for implementing Nock virtual machines across multiple programming languages with proper evaluation loops, tree operations, and runtime optimization.

## Implementation Architecture

### Core Components

#### 1. Noun Representation
**Atoms**: Natural numbers with arbitrary precision
- **C**: GMP (GNU Multiple Precision) or custom BigInt
- **Python**: Built-in `int` (unlimited precision)
- **Rust**: `num-bigint` crate or `bigint` crate
- **Haskell**: `Integer` type (native)
- **JavaScript**: `BigInt` (ES2020+)

**Cells**: Ordered pairs (cons cells)
```rust
pub enum Noun {
    Atom(BigUint),
    Cell(Box<Noun>, Box<Noun>),
}
```

#### 2. Parser & Printer
Parse text format to internal representation:
```
[a b c]        -> Cell(Atom(a), Cell(Atom(b), Atom(c)))
123             -> Atom(123)
[a [b c]]      -> Cell(Atom(a), Cell(Atom(b), Atom(c)))
```

Print back to readable format.

#### 3. Evaluation Loop
Pattern matching on formula opcodes (0-12):
```python
def evaluate(subject, formula):
    while True:
        if not is_cell(formula):
            raise NockCrash("formula must be cell")

        op, rest = formula
        result = dispatch(op, subject, rest)
        if is_crash(result):
            return result
        subject, formula = result, rest
        if is_constant(formula):  # Rule 1
            return formula
```

## Nock Rules Implementation

### Rule 0: Slot (`[a b] /`)
Binary tree addressing for efficient axis traversal:
```
[subject slot] / = value_at_slot
```

**Implementation**: Convert slot number to axis path (left/right sequence), traverse subject tree.

### Rule 1: Constant (`[a] *`)
```
[subject] * = a
```

Stops evaluation, returns constant `a`.

### Rule 2: Evaluate (`[a b c] +`)
```
[subject [a b]] + = [subject a] c
```

Recurse: evaluate `a` against `subject`, then evaluate result as formula with `c`.

### Rule 3: Cell Test (`[a] ?`)
```
[subject a] ? = 0 if a is cell, 1 if a is atom
```

Type discrimination for atoms vs cells.

### Rule 4: Increment (`[a] +`)
```
[subject a] + = a + 1
```

Natural number addition.

### Rule 5: Equality (`[a b] =`)
```
[subject a b] = = a b
```

Structural equality: `0` if identical, `1` if different.

### Rule 6-9: Composition
- Rule 6: `[[a b] c] /` = `[[a b] / c]`
- Rule 7: `[a b c] =` = `a b c` (three-element cell)
- Rule 8: `[a b] +` = `a + b`
- Rule 9: `[a b] +` = `a + b`

Formulas 2-9 are composites of rules 0-5.

### Rule 10: Edit (`[a b c]`)
```
[subject [a b c]] = nock(subject, b)
```
Tree editing; replaces part of the subject at axis `a`.

### Rule 11: Hint (`[a b]`)
```
[subject [a b]] = nock(subject, b)
```

Optimization hint; evaluates `b` and may use `a` as a hint to the runtime.

### Rule 12: Scry (`[a b] ^`)
```
[subject [a b]] ^ = memory[a][b]
```

Referentially transparent read of memory slot `b` within noun `a`.

## Performance Patterns

### Tail Call Optimization (TCO)

Detect and optimize recursive formulas:
```python
# Before: creates stack frames
[subject [[a b] c]] + = ...

# After: if formula ends with recursive call, reuse stack
if ends_with_recursive_call(formula):
    return tco_evaluate(subject, formula)
```

### Memoization
Cache evaluation results for repeated sub-formulas:
```python
_memo = {}

def memo_evaluate(subject, formula):
    key = (subject, formula)
    if key in _memo:
        return _memo[key]
    result = evaluate(subject, formula)
    _memo[key] = result
    return result
```

### Lazy Evaluation
Defer computation until value needed:
```python
class Thunk:
    def __init__(self, subject, formula):
        self.subject = subject
        self.formula = formula
        self._cached = None

    def force(self):
        if self._cached is None:
            self._cached = evaluate(self.subject, self.formula)
        return self._cached
```

## Language-Specific Patterns

### C Implementation
```c
typedef struct {
    mpz_t atom;
    struct Noun *cell;
} Noun;

Noun* slot(Noun* subject, Noun* axis);
Noun* evaluate(Noun* subject, Noun* formula);
```

**Pros**: Maximum control, pointer efficiency
**Cons**: Manual memory management, complexity

### Python Implementation
```python
from typing import Union

Noun = Union[int, tuple]

def evaluate(subject: Noun, formula: Noun) -> Noun:
    if not isinstance(formula, tuple):
        raise NockCrash("formula must be cell")
    ...
```

**Pros**: Simplicity, fast prototyping
**Cons**: Performance overhead

### Rust Implementation
```rust
pub enum Noun {
    Atom(BigUint),
    Cell(Box<Noun>, Box<Noun>),
}

impl Noun {
    pub fn evaluate(&self, formula: &Noun) -> Result<Noun, NockCrash> {
        match formula {
            Noun::Cell(op, rest) => dispatch(op, self, rest),
            _ => Ok(formula.clone()),
        }
    }
}
```

**Pros**: Memory safety, zero-cost abstractions
**Cons**: Borrow checker complexity

### Haskell Implementation
```haskell
data Noun = Atom Integer | Cell Noun Noun

evaluate :: Noun -> Noun -> Noun
evaluate subject formula = case formula of
    Cell (Atom 0, rest) -> slot subject rest
    Cell (Atom 1, rest) -> rest
    Cell (Atom 2, Cell(a, Cell(b, c))) -> evaluate (evaluate subject a) c
    ...
```

**Pros**: Type safety, pattern matching
**Cons**: Learning curve

### JavaScript Implementation
```javascript
class Noun {
    constructor(value) {
        this.isCell = Array.isArray(value);
        this.value = value;
    }
}

function evaluate(subject, formula) {
    if (!formula.isCell) {
        throw new NockCrash("formula must be cell");
    }
    const [op, ...rest] = formula.value;
    return dispatch(op, subject, rest);
}
```

**Pros**: Web compatibility
**Cons**: Performance, type safety

## Testing & Validation

### Reference Test: Decrement
```python
def test_decrement():
    subject = 0
    formula = [8 [1 0] 8 [1 6 [5 [0 7]] 4 0 6] [0 6] 9 2 [0 2] [4 0 6] 0 7] 9 2 0 1]
    result = evaluate(subject, formula)
    assert result == 0, f"Expected 0, got {result}"
```

### Edge Cases
- **Empty subject**: `[~ [1 0]]` should work
- **Large atoms**: Test with 64-bit, 128-bit, larger
- **Deep trees**: Stack overflow prevention
- **Memory pressure**: Graceful handling, not crash

### Benchmarking
Compare performance against reference implementations:
```
Implementation | Decrement (ms) | Formula 2 (ms) | Memory (MB)
-------------|-----------------|---------------|--------------
Python       | 150             | 50             | 25
Rust         | 5               | 0.5            | 3
C (GMP)     | 1               | 0.1            | 1
```

## Common Pitfalls

### 1. Incorrect Slot Calculation
```
Slot 1 = axis [0 1]  // Correct
Slot 2 = axis [1 0]  // Correct, NOT [0 1 1]
```

### 2. Missing TCO
Deep recursion crashes stack without trampoline or TCO.

### 3. Inefficient Noun Copying
Deep clones of nouns create O(n²) behavior. Use reference counting or structural sharing.

### 4. Incorrect Subject Handling
Forgetting to update `subject` after evaluation changes context.

## Resources

- [Nock 4K Specification](https://github.com/urbit/urbit/blob/main/pkg/arvo/sys/zuse.hoon)
- [nock-interpreter-patterns](../nock-interpreter-patterns/SKILL.md)
- [nock-multi-language-implementations](../nock-multi-language-implementations/SKILL.md)
- [nock-specification-reference](../nock-specification-reference/SKILL.md)
