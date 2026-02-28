---
name: AILANG Debug
description: Debug AILANG code errors. Use when you encounter type errors, parse errors, or runtime failures in AILANG programs.
---

# AILANG Debug

Fix common AILANG errors quickly.

## Quick Reference

| Error | Cause | Fix |
|-------|-------|-----|
| `undefined variable: print` | Not in entry module | Use entry module or `import std/io (print)` |
| `undefined variable: println` | Wrong function name | Use `print` not `println` |
| `undefined variable: map` | Not imported | `import std/list (map)` or write recursive |
| `No instance for Num[string]` | `print(42)` | Use `print(show(42))` |
| `expected }, got let` | Missing semicolon | Add `;` between statements |
| `unexpected token: for` | No loops in AILANG | Use recursion instead |
| `unexpected token: in` | No `for x in xs` | Use `match xs { ... }` |

## Decision Tree

```
Error message?
│
├─ "undefined variable: X"
│   └─ Is X a builtin?
│       ├─ Yes → Import it: ailang builtins list | grep X
│       └─ No → Check spelling, define it
│
├─ "expected }, got ..."
│   └─ Missing semicolon between statements
│       Fix: let x = 1; let y = 2; x + y
│
├─ "No instance for Num[string]"
│   └─ Passing number to string function
│       Fix: print(show(42)) not print(42)
│
├─ "unexpected token: for/while/in"
│   └─ AILANG has no loops!
│       Fix: Use recursion with match
│
└─ Parse error with braces
    └─ Unmatched { } or missing expression
        Fix: Check all blocks are closed
```

## Common Fixes

### 1. Missing Semicolons

```ailang
-- WRONG
export func main() -> () ! {IO} {
  let x = 10
  let y = 20
  print(show(x + y))
}

-- CORRECT (semicolons between statements)
export func main() -> () ! {IO} {
  let x = 10;
  let y = 20;
  print(show(x + y))
}
```

### 2. Print Needs String

```ailang
-- WRONG: print expects string
print(42)

-- CORRECT: convert with show()
print(show(42))
```

### 3. No Loops - Use Recursion

```ailang
-- WRONG: no for loops
for i in range(5) { print(show(i)) }

-- CORRECT: recursive function
export func printRange(n: int) -> () ! {IO} {
  if n <= 0 then () else {
    print(show(n));
    printRange(n - 1)
  }
}
```

### 4. Import Standard Library

```ailang
-- WRONG: map not in scope
let doubled = map(\x. x * 2, nums)

-- CORRECT: import from std/list
import std/list (map)
let doubled = map(\x. x * 2, nums)

-- OR: write it yourself (recursion)
export func myMap[a,b](f: func(a) -> b, xs: [a]) -> [b] {
  match xs {
    [] => [],
    hd :: tl => f(hd) :: myMap(f, tl)
  }
}
```

## Debugging Workflow

1. **Type-check first** (faster feedback):
   ```bash
   ailang check file.ail
   ```

2. **Read error location** - line:column tells you where

3. **Check the pattern** above for your error type

4. **Use REPL** for quick tests:
   ```bash
   ailang repl
   > show(42)
   > 1 + 2
   > :type \x. x * 2
   ```

5. **List builtins** to find imports (**CLI is source of truth**):
   ```bash
   # SOURCE OF TRUTH: Full documentation with examples
   ailang builtins list --verbose --by-module

   # Search for specific function with full docs
   ailang builtins list --verbose | grep -A 10 "map"

   # See a specific module's functions
   ailang builtins list --verbose --by-module | grep -A 30 "std/list"
   ```

## Resources

**Always prefer CLI commands** (`ailang prompt`, `ailang builtins list --verbose`) over static docs - they're always up-to-date.

See [resources/error_catalog.md](resources/error_catalog.md) for additional error patterns.
