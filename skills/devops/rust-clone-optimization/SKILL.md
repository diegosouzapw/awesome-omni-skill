---
name: rust-clone-optimization
description: Use when optimizing Rust code to remove unnecessary .clone() calls, when asked to improve memory efficiency, or when reviewing code for excessive cloning
---

# Rust Clone Optimization

## Overview

Remove unnecessary `.clone()` calls by using borrowing, reordering, or minimal structural changes. **Prioritize simple solutions over clever ones.**

## When to Use

- Asked to "remove clones" or "optimize memory"
- Reviewing Rust code for efficiency
- Seeing many `.clone()` calls in hot paths

**When NOT to use:**
- `Arc::clone()` / `Rc::clone()` - these are cheap, intentional
- Cold paths with small data - optimization not worth complexity
- Prototyping phase - correctness before performance

## Analysis Hierarchy

**Try techniques in this order. Stop at the first that works.**

```
1. REORDER    → Move usage before the value is needed elsewhere
2. BORROW     → Pass &T or &str instead of owned
3. COW        → Cow<'_, str> for "sometimes owned"
4. STRUCTURAL → Change struct to hold references (last resort)
```

**Never jump to step 4 before exhausting 1-3.**

## Quick Reference

| Clone Pattern | First Try | Example |
|---------------|-----------|---------|
| Clone then move original | Reorder: use clone target first | Move `state.modal = ...` after using `login_state` |
| `.map(\|x\| x.field.clone())` | Check if caller accepts `&str` | `from_entry(entry, &name)` not `from_entry(entry, name.clone())` |
| Clone to avoid borrow conflict | Reorder operations | Extract data before mutable borrow |
| Clone in struct field assignment | Check if `Cow<str>` fits | `name: Cow<'a, str>` |
| Clone inside loop | Clone once outside | `let s = x.clone(); for _ in 0..n { use(&s) }` |

## Technique 1: Reorder

**Before:**
```rust
let username = login_state.username.clone();
let password = login_state.password.clone();
state.modal = Modal::LoginPending;  // Invalidates login_state
api.send(ApiCommand::Login { username, password });
```

**After:**
```rust
// Send BEFORE changing modal - no clone needed
let cmd = ApiCommand::Login {
    username: login_state.username.clone(),  // Still need if Login takes owned
    password: login_state.password.clone(),
};
state.modal = Modal::LoginPending;
api.send(cmd);
```

**Check:** Does `ApiCommand::Login` need owned `String`? If it accepts `&str`, remove clones entirely.

## Technique 2: Borrow

**Before:**
```rust
fn process(name: String) { println!("{}", name); }
let task_name = task.name.clone();
process(task_name);
```

**After:**
```rust
fn process(name: &str) { println!("{}", name); }  // Accept reference
process(&task.name);  // No clone
```

**When to use:** Function only reads the value, doesn't store it.

## Technique 3: Cow

**Before:**
```rust
struct Form {
    name: String,  // Always owned
}
// Must clone even for borrowed data
form.name = borrowed_str.to_string();
```

**After:**
```rust
use std::borrow::Cow;

struct Form<'a> {
    name: Cow<'a, str>,  // Borrowed OR owned
}
// No clone for borrowed
form.name = Cow::Borrowed(borrowed_str);
// Clone only when needed
form.name = Cow::Owned(compute_name());
```

**When to use:** Struct sometimes gets owned data, sometimes borrowed. Adds lifetime parameter.

## Technique 4: Structural (Last Resort)

Change struct to hold references:

```rust
// Before: owns data
struct EntryForm { task_name: String }

// After: borrows data
struct EntryForm<'a> { task_name: &'a str }
```

**Costs:**
- Adds lifetime parameters throughout
- May require significant refactoring
- Can complicate APIs

**Only use when:** Hot path with large data AND techniques 1-3 don't apply.

## Common Mistakes

| Mistake | Why It's Wrong |
|---------|----------------|
| Using `std::mem::replace` to avoid clone | Over-engineered; try reordering first |
| Removing Arc/Rc clones | These are O(1), intentional for sharing |
| Adding lifetimes everywhere | Complexity cost often exceeds clone cost |
| Optimizing cold paths | Profile first; don't optimize what doesn't matter |

## Audit Process

1. **Find clones:** `rg '\.clone\(\)'` in target directory
2. **Categorize each:**
   - Arc/Rc → Skip (intentional)
   - Hot path → Prioritize
   - Cold path → Low priority
3. **For each hot-path clone:**
   - Can I reorder? → Do it
   - Can caller accept `&T`? → Change signature
   - Need "sometimes owned"? → Use `Cow`
   - None work? → Consider if optimization is worth structural change
4. **Verify:** `cargo test` after each change

## Checklist

- [ ] Identified Arc/Rc clones (skip these)
- [ ] Prioritized by hot path vs cold path
- [ ] Tried reordering before other techniques
- [ ] Checked if callers can accept references
- [ ] Only used Cow/structural changes when simpler options failed
- [ ] Ran tests after changes
- [ ] Kept changes minimal - no over-engineering
