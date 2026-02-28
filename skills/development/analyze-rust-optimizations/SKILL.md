---
name: analyze-rust-optimizations
description: This skill performs thorough analysis of Rust libraries to find optimization opportunities. It should be used when reviewing Rust code for performance improvements, memory efficiency, or when profiling indicates bottlenecks. Focuses on runtime performance and memory usage through dynamic profiling tools and static code analysis.
---

<objective>
Analyze Rust libraries for optimization opportunities with focus on runtime performance and memory usage. Uses dynamic profiling tools when available, falls back to static analysis, and produces prioritized findings with a detailed report.
</objective>

<quick_start>
Run profiling tools first, then analyze code:

```bash
# Check for benchmarks
ls benches/ 2>/dev/null || ls **/bench*.rs 2>/dev/null

# Run benchmarks if available
cargo bench

# Generate flamegraph (if installed)
cargo flamegraph --bench <benchmark_name>

# Check binary/dependency bloat
cargo bloat --release
cargo bloat --release --crates
```

If tools missing, see `<tools_setup>` for installation.
</quick_start>

<process>
**Phase 1: Tool Detection & Dynamic Analysis**

1. **Check available tools**:
   ```bash
   command -v flamegraph && echo "flamegraph: installed"
   cargo bloat --version 2>/dev/null && echo "cargo-bloat: installed"
   command -v samply && echo "samply: installed"
   command -v heaptrack && echo "heaptrack: installed"
   ```

2. **Run benchmarks** (if `benches/` or `#[bench]` exists):
   ```bash
   cargo bench -- --save-baseline before
   ```

3. **CPU profiling** (pick available tool):
   - `cargo flamegraph` - generates SVG flamegraph
   - `samply record cargo run --release` - sampling profiler
   - `perf record cargo run --release && perf report` - Linux perf

4. **Memory profiling**:
   - `heaptrack cargo run --release` - heap allocations
   - `valgrind --tool=massif` - memory over time
   - `DHAT` via `#[global_allocator]` instrumentation

**Phase 2: Static Code Analysis**

Scan the codebase for these patterns (priority order):

1. **Hot path allocations**: `.clone()`, `.to_string()`, `.to_vec()` in loops
2. **Unnecessary allocations**: `String` where `&str` suffices, `Vec` where slice works
3. **Missing `#[inline]`**: Small functions called across crate boundaries
4. **Inefficient iterators**: `.collect()` followed by iteration, multiple `.iter()` passes
5. **Box/Arc overuse**: Heap allocation where stack would work
6. **HashMap/BTreeMap inefficiency**: Missing `with_capacity()`, poor hash functions
7. **String formatting in hot paths**: `format!()` allocates, prefer `write!()`
8. **Bounds checking**: Array indexing vs `.get_unchecked()` in verified-safe paths
9. **Memory layout**: Large structs, poor field ordering, excessive padding

**Phase 3: Generate Report**

Produce two outputs:
1. **Prioritized findings list** - ranked by estimated impact
2. **Detailed report** - comprehensive markdown with sections
</process>

<optimization_patterns>
**Runtime Performance (Priority 1)**

| Pattern | Problem | Solution |
|---------|---------|----------|
| `.clone()` in loops | Repeated heap allocation | Borrow, use `Cow<T>`, or hoist clone |
| `collect().iter()` | Intermediate allocation | Chain iterators directly |
| Missing `#[inline]` | Function call overhead | Add `#[inline]` or `#[inline(always)]` |
| `format!()` in hot path | Allocates String | Use `write!()` to existing buffer |
| `HashMap` default hasher | SipHash is slow | Use `FxHashMap` or `ahash` |
| Recursive without TCO | Stack growth, cache misses | Convert to iteration |
| Virtual dispatch (`dyn Trait`) | Indirect call overhead | Use generics or enum dispatch |

**Memory Usage (Priority 2)**

| Pattern | Problem | Solution |
|---------|---------|----------|
| `String` for static text | Heap allocation | Use `&'static str` or `Cow<str>` |
| `Vec<T>` without capacity | Repeated reallocation | `Vec::with_capacity(n)` |
| Large enum variants | Wastes memory on small variants | Box large variants |
| `Box<dyn Trait>` | Heap + vtable overhead | Consider enum dispatch |
| Struct field ordering | Padding waste | Order fields large-to-small |
| `Arc` where `Rc` suffices | Extra atomic overhead | Use `Rc` if single-threaded |
| Owned in struct fields | Forces cloning | Consider borrowing with lifetime |
</optimization_patterns>

<tools_setup>
**Install profiling tools if missing:**

```bash
# Flamegraph (CPU profiling visualization)
cargo install flamegraph
# Linux: sudo apt install linux-perf
# macOS: works via dtrace

# cargo-bloat (binary size analysis)
cargo install cargo-bloat

# samply (sampling profiler)
cargo install samply

# heaptrack (memory profiling) - Linux only
sudo apt install heaptrack heaptrack-gui

# criterion (micro-benchmarking)
# Add to Cargo.toml: criterion = "0.5"
```

**For accurate profiling:**
```toml
# Cargo.toml - add release profile with debug info
[profile.release]
debug = true
```
</tools_setup>

<output_format>
**1. Prioritized Findings List**

```markdown
## Optimization Findings (Prioritized)

### Critical (High Impact)
1. **[PERF]** `src/parser.rs:142` - `.clone()` inside hot loop, ~500k calls/sec
2. **[MEM]** `src/cache.rs:89` - HashMap without capacity, grows 12 times

### Important (Medium Impact)
3. **[PERF]** `src/lib.rs:34` - Missing `#[inline]` on public API hot path
4. **[MEM]** `src/types.rs:15-28` - Struct padding wastes 24 bytes per instance

### Minor (Low Impact)
5. **[PERF]** `src/utils.rs:67` - `format!()` could use `write!()`
```

**2. Detailed Report Structure**

```markdown
# Rust Optimization Analysis Report

## Executive Summary
- Total findings: X
- Critical: Y | Important: Z | Minor: W
- Estimated performance gain: ...

## Profiling Results
### CPU Profile
[Flamegraph or hotspot analysis]

### Memory Profile
[Allocation patterns, peak usage]

## Findings by Category

### Runtime Performance
[Detailed findings with code snippets and fixes]

### Memory Usage
[Detailed findings with before/after layouts]

## Recommended Changes
[Prioritized action items with effort estimates]

## Appendix
- Tool versions used
- Benchmark results
- Profiling methodology
```
</output_format>

<static_analysis_commands>
Use these to find common patterns:

```bash
# Find .clone() in potential hot paths
rg '\.clone\(\)' --type rust -n

# Find allocating methods in loops
rg 'for .* in|while|loop' -A 10 --type rust | rg '\.to_string\(\)|\.to_vec\(\)|\.clone\(\)'

# Find HashMap/Vec without capacity hints
rg 'HashMap::new\(\)|Vec::new\(\)' --type rust -n

# Find format! macro usage
rg 'format!\(' --type rust -n

# Find missing inline attributes on pub functions
rg '^pub fn \w+' --type rust -n

# Find large enums (check for Box opportunities)
rg '^pub enum|^enum' -A 20 --type rust

# Find dyn Trait usage
rg 'dyn \w+' --type rust -n
```
</static_analysis_commands>

<success_criteria>
Analysis is complete when:

- [ ] Available profiling tools identified (or install instructions provided)
- [ ] Dynamic profiling run (benchmarks, flamegraph, memory profiler)
- [ ] Static analysis completed for all optimization patterns
- [ ] Findings prioritized by impact (Critical/Important/Minor)
- [ ] Each finding includes file:line reference
- [ ] Each finding includes specific fix recommendation
- [ ] Prioritized findings list generated
- [ ] Detailed markdown report generated
- [ ] Report includes methodology and tool versions
</success_criteria>
