---
name: mojo
description: "Mojo development patterns for high-performance computing: SIMD vectorization, zero-copy Python interop, GIL-free parallelism, C FFI (external_call, DLHandle), Python C-API extensions, and Hatch build integration. Use when writing Mojo kernels or hybrid Python-Mojo projects."
---

# Mojo (High-Performance Computing)

## Scope

- Work in `src/mo/` (Mojo source) and `src/py/` (Python wrapper).
- Build high-performance kernels for numeric, AI, and data-intensive workloads.
- Develop Python extensions using Mojo's native C-API features.
- Call C/C++ libraries via CFFI (`external_call`, `DLHandle`).
- Zero-copy data exchange between Python and Mojo.

## Core Rules

### Strict Typing

- **Prefer `fn` over `def`:** Strict type checking, predictable performance, deterministic behavior.
- Explicit types for all function arguments and return values.
- Use `alias` for compile-time constants.

```mojo
fn dot_product(a: SIMD[DType.float32, 8], b: SIMD[DType.float32, 8]) -> Float32:
    return (a * b).reduce_add()
```

### Memory Safety

- Leverage ownership semantics: `owned`, `borrowed`, `inout`.
- Document every use of `UnsafePointer` with a safety comment.
- Prefer stack allocation over heap when size is known at compile time.

```mojo
fn process(borrowed data: Tensor[DType.float32]) -> Tensor[DType.float32]:
    # borrowed: read-only, no copy, caller retains ownership
    var result = Tensor[DType.float32](data.shape())
    # ... transform ...
    return result^  # Transfer ownership to caller
```

### Tooling

- Use `uv` exclusively for Python environment management. Never `pixi` or `conda`.
- Use `mojo` CLI for compilation and testing.

## Docs Retrieval Policy (Modular)

Use Modular docs as the primary source of truth for Mojo behavior, MAX integration details, and assistant workflows.

Resolution order for reference context:

1. Use scoped reference files first:
- https://docs.modular.com/llms-mojo.txt
- https://docs.modular.com/llms-mojo-python.txt
- https://docs.modular.com/llms-mojo-kernel.txt
- https://docs.modular.com/llms-max.txt
2. Use https://docs.modular.com/llms.txt for broad discovery.
3. Use https://docs.modular.com/llms-full.txt only when scoped files are insufficient.

Doc-linking requirements:

- Prefer deep links to exact manual/API pages instead of only top-level doc hubs.
- Include source links when behavior depends on implementation details.
- Avoid version-pinned changelog anchors; link to the latest changelog root.

## Assistant Integration

Place shared Modular guidance in the assistant root context file for your environment:

- Codex / Gemini CLI: `AGENTS.md`
- Claude Code / Cline / Roo: `CLAUDE.md`
- Cursor: `.cursorrules` or `.cursor/rules`
- Windsurf: `.windsurfrules` or `.windsurf/rules`

For Codex-style instructions, require links under:

- MAX docs: `https://docs.modular.com/max/...`
- Mojo docs: `https://docs.modular.com/mojo/...`
- Source code: `https://github.com/modular/modular/tree/main/...`

## Performance: SIMD + Parallelize

### SIMD-First Vectorization

Replace scalar loops with SIMD operations:

```mojo
from algorithm import vectorize

fn relu_simd(inout tensor: Tensor[DType.float32]):
    alias simd_width = simdwidthof[DType.float32]()
    let zero = SIMD[DType.float32, simd_width](0)

    @parameter
    fn _relu[width: Int](idx: Int):
        let val = tensor.load[width=width](idx)
        tensor.store(idx, val.max(zero))

    vectorize[_relu, simd_width](tensor.num_elements())
```

**Rules:**
- Use `simdwidthof` to auto-detect hardware SIMD width.
- Use `@parameter` for compile-time loop specialization.
- Benchmark SIMD vs scalar — verify vectorization gives real speedup.

### GIL-Free Parallelism

True multi-core scaling without Python's GIL:

```mojo
from algorithm import parallelize

fn parallel_transform(inout data: Tensor[DType.float32], num_workers: Int):
    let chunk_size = data.num_elements() // num_workers

    @parameter
    fn _worker(worker_id: Int):
        let start = worker_id * chunk_size
        let end = min(start + chunk_size, data.num_elements())
        for i in range(start, end):
            data[i] = expensive_compute(data[i])

    parallelize[_worker](num_workers)
```

### Combined SIMD + Parallel

```mojo
fn process_large_dataset(inout data: Tensor[DType.float32]):
    alias simd_width = simdwidthof[DType.float32]()

    @parameter
    fn _chunk(chunk_id: Int):
        let offset = chunk_id * CHUNK_SIZE

        @parameter
        fn _vectorized[width: Int](idx: Int):
            let val = data.load[width=width](offset + idx)
            data.store(offset + idx, transform(val))

        vectorize[_vectorized, simd_width](CHUNK_SIZE)

    parallelize[_chunk](data.num_elements() // CHUNK_SIZE)
```

## Zero-Copy Python Interop

### Via __array_interface__

Exchange data with NumPy without copying:

```mojo
fn from_numpy(np_array: PythonObject) -> Tensor[DType.float32]:
    # Extract raw pointer from NumPy's array interface
    let interface = np_array.__array_interface__
    let data_ptr = interface["data"][0].to_int()
    let shape = interface["shape"]

    # SAFETY: np_array must remain alive while this tensor exists.
    # The pointer is valid for shape[0] * sizeof(float32) bytes.
    let ptr = UnsafePointer[Float32](address=data_ptr)
    return Tensor[DType.float32](ptr, shape[0].to_int())
```

### Returning Data to Python

```mojo
fn to_numpy(tensor: Tensor[DType.float32]) -> PythonObject:
    let np = Python.import_module("numpy")
    # Create NumPy array from Mojo tensor data
    return np.frombuffer(
        tensor.data().as_bytes(),
        dtype=np.float32
    ).reshape(tensor.shape())
```

## Python Extensions (C-API)

### Module Entry Point

```mojo
from python.module import PythonModuleBuilder

@export
fn PyInit_my_module() -> PythonObject:
    var builder = PythonModuleBuilder("my_module")
    builder.add_function("dot_product", dot_product_wrapper)
    builder.add_function("relu", relu_wrapper)
    return builder.build()
```

### Function Wrappers

```mojo
fn dot_product_wrapper(args: PythonObject) -> PythonObject:
    let a = args[0]  # NumPy array
    let b = args[1]  # NumPy array
    let result = dot_product_simd(
        from_numpy(a),
        from_numpy(b)
    )
    return to_numpy(result)
```

## C FFI (Calling C Libraries from Mojo)

### Static External Calls

Use `external_call` for compile-time linked C functions:

```mojo
from sys.ffi import external_call

fn get_time() -> Float64:
    # Calls C's clock_gettime via libc (linked at compile time)
    return external_call["clock", Float64]()

fn allocate_aligned(size: Int, alignment: Int) -> UnsafePointer[UInt8]:
    # SAFETY: Caller must free with aligned_free/free.
    return external_call["aligned_alloc", UnsafePointer[UInt8]](alignment, size)
```

**Rules:**
- First parameter is the C function name as a string literal.
- Second parameter is the Mojo return type.
- Remaining parameters are the arguments, which must map to C-compatible types.
- Only works for functions available at link time (libc, system libraries).

### Dynamic Library Loading (DLHandle)

Load shared libraries at runtime for plugin-style integration:

```mojo
from sys.ffi import DLHandle, c_char

fn load_custom_library():
    # Open shared library
    var lib = DLHandle("./libcustom_ops.so")

    # Get function pointer by name
    var compute_fn = lib.get_function[fn (UnsafePointer[Float32], Int) -> Float32](
        "custom_compute"
    )

    # Call the C function
    var data = UnsafePointer[Float32].alloc(1024)
    var result = compute_fn(data, 1024)

    data.free()
    # lib closes on drop (RAII)
```

**When to use DLHandle:**
- Loading vendor-specific acceleration libraries (CUDA, oneDNN, MKL).
- Plugin architectures where the library isn't known at compile time.
- Wrapping existing C/C++ libraries without recompilation.

### C Struct Mapping

Map C structs for FFI interop:

```mojo
@register_passable("trivial")
struct CTimeSpec:
    var tv_sec: Int64
    var tv_nsec: Int64

    fn __init__(out self):
        self.tv_sec = 0
        self.tv_nsec = 0

fn get_monotonic_time() -> CTimeSpec:
    var ts = CTimeSpec()
    # SAFETY: CTimeSpec layout matches C's struct timespec.
    # CLOCK_MONOTONIC (1) is always available on POSIX systems.
    external_call["clock_gettime", Int32](Int32(1), UnsafePointer.address_of(ts))
    return ts
```

**Rules:**
- Use `@register_passable("trivial")` for C-compatible structs (no destructor, copyable by memcpy).
- Ensure field layout matches the C struct exactly (same types, same order).
- Use `Int32`, `Int64`, `Float32`, `Float64` for exact-width C type mapping.

### C String Handling

```mojo
from sys.ffi import c_char

fn call_c_with_string(name: String):
    # Convert Mojo String to null-terminated C string
    var c_str = name.unsafe_cstr_ptr()

    # SAFETY: c_str is valid for the lifetime of `name`.
    external_call["puts", Int32](c_str)

fn read_c_string(ptr: UnsafePointer[c_char]) -> String:
    # SAFETY: ptr must point to a valid null-terminated C string.
    return String(ptr)
```

### Callback Patterns (C Calling Mojo)

Pass Mojo functions as C callbacks:

```mojo
alias CCallback = fn (UnsafePointer[Float32], Int) -> None

fn my_callback(data: UnsafePointer[Float32], len: Int) -> None:
    # Process data from C callback
    for i in range(len):
        data[i] = data[i] * 2.0

fn register_with_c_library():
    # Pass Mojo function as C function pointer
    external_call["register_callback", None](my_callback)
```

**Rules:**
- Callback functions must use C-compatible types only.
- Use `@always_inline("never")` if the callback needs a stable address.
- Document lifetime requirements — the callback must not outlive its data.

### Linking C Libraries

```bash
# Link system library at compile time
mojo build --link-lib "m" src/main.mojo          # libm (math)
mojo build --link-lib "ssl" src/main.mojo         # libssl

# Link custom shared library
mojo build --link-lib "custom_ops" --link-path "./lib" src/main.mojo
```

### C FFI Type Mapping

| C Type | Mojo Type |
|--------|-----------|
| `int` / `int32_t` | `Int32` |
| `long` / `int64_t` | `Int64` |
| `float` | `Float32` |
| `double` | `Float64` |
| `size_t` | `Int` |
| `char*` | `UnsafePointer[c_char]` |
| `void*` | `UnsafePointer[NoneType]` |
| `T*` | `UnsafePointer[T]` |
| `const T*` | `UnsafePointer[T]` (enforce read-only by convention) |
| `bool` | `Bool` |

## Build System (hatch-mojo)

Use [hatch-mojo](https://pypi.org/project/hatch-mojo/) — a Hatch build hook plugin for compiling Mojo sources into Python extensions, shared libraries, executables, and other artifacts.

### Minimal Setup

```toml
# pyproject.toml
[build-system]
build-backend = "hatchling.build"
requires = ["hatchling", "hatch-mojo"]

[project]
name = "my-package"
version = "0.1.0"
requires-python = ">=3.11"

[[tool.hatch.build.targets.wheel.hooks.mojo.jobs]]
name = "core"
input = "src/mo/my_pkg/core.mojo"
emit = "python-extension"
module = "my_pkg._core"
include-dirs = ["src/mo"]
```

```bash
# Build wheel with compiled Mojo extension
hatch build -t wheel
```

### Job Types (emit)

| `emit` Value | Output | Use Case |
|--------------|--------|----------|
| `python-extension` | `.so`/`.pyd` importable by Python | Primary — Mojo kernels callable from Python |
| `shared-lib` | `.so`/`.dylib` | Shared library for FFI consumers |
| `static-lib` | `.a` | Static linking into other builds |
| `executable` | Binary | CLI tools written in Mojo |
| `object` | `.o` | Object file for custom linking |

### Multiple Jobs with Dependencies

```toml
[tool.hatch.build.targets.wheel.hooks.mojo]
parallel = true         # Compile independent jobs concurrently
fail-fast = true        # Stop on first failure

# Reusable config profiles
[tool.hatch.build.targets.wheel.hooks.mojo.profiles.common]
include-dirs = ["src/mo"]

[[tool.hatch.build.targets.wheel.hooks.mojo.jobs]]
name = "core"
input = "src/mo/my_pkg/core.mojo"
emit = "python-extension"
module = "my_pkg._core"
profiles = ["common"]

[[tool.hatch.build.targets.wheel.hooks.mojo.jobs]]
name = "ops"
input = "src/mo/my_pkg/ops.mojo"
emit = "python-extension"
module = "my_pkg._ops"
profiles = ["common"]
depends-on = ["core"]    # Compiles after core

[[tool.hatch.build.targets.wheel.hooks.mojo.jobs]]
name = "cli"
input = "src/mo/cli.mojo"
emit = "executable"
depends-on = ["core"]
install = { kind = "scripts", path = "my-cli" }
```

### Platform-Conditional Compilation

```toml
[[tool.hatch.build.targets.wheel.hooks.mojo.jobs]]
name = "unix-accel"
input = "src/mo/accel_unix.mojo"
emit = "shared-lib"
platforms = ["linux", "darwin"]          # Skip on Windows
install = { kind = "package", path = "my_pkg/lib" }

[[tool.hatch.build.targets.wheel.hooks.mojo.jobs]]
name = "cuda-kernels"
input = "src/mo/cuda_*.mojo"            # Glob expansion
emit = "shared-lib"
marker = "platform_machine == 'x86_64'" # PEP 508 marker
install = { kind = "package", path = "my_pkg/cuda" }
```

### Global Configuration Options

| Key | Default | Description |
|-----|---------|-------------|
| `mojo-bin` | auto-detect | Path to mojo binary |
| `parallel` | `false` | Compile independent jobs concurrently |
| `fail-fast` | `true` | Stop on first failure (parallel mode) |
| `skip-editable` | `true` | Skip compilation for editable installs |
| `clean-before-build` | `false` | Remove build dir before compiling |
| `clean-after-build` | `false` | Remove build dir after compiling |
| `build-dir` | `.hatch_mojo` | Working directory for artifacts |
| `include` / `exclude` | `[]` | Global git-style glob filters |

### Mojo Binary Discovery

Priority order:
1. `mojo-bin` config value
2. `HATCH_MOJO_BIN` environment variable
3. `mojo` on `PATH`
4. `.venv/bin/mojo` relative to project root

### Install Mappings (non-Python artifacts)

For jobs with `emit` other than `python-extension`, specify where to install:

| `kind` | Description | Example `path` |
|--------|-------------|----------------|
| `package` | Inside Python package dir | `my_pkg/lib` |
| `scripts` | As executable script | `my-cli` |
| `data` | As package data | `data` |
| `root` | At package root | `lib` |
| `force-include` | Arbitrary path | `vendor/lib` |

### Manual Compilation (without hatch-mojo)

```bash
# Build shared library directly
mojo build --emit shared-lib src/mo/module.mojo -o src/py/package/_module.so

# Build standalone binary
mojo build src/mo/main.mojo -o dist/main

# Run directly
mojo run src/mo/main.mojo
```

## Testing

### Mojo Tests

```mojo
fn test_dot_product():
    let a = SIMD[DType.float32, 4](1.0, 2.0, 3.0, 4.0)
    let b = SIMD[DType.float32, 4](5.0, 6.0, 7.0, 8.0)
    let result = dot_product(a, b)
    assert_almost_equal(result, 70.0)
```

```bash
mojo test src/mo/tests/
```

### Python-Mojo Boundary Tests

```python
# tests/test_module.py
import numpy as np
from my_package._my_module import dot_product

def test_dot_product():
    a = np.array([1.0, 2.0, 3.0, 4.0], dtype=np.float32)
    b = np.array([5.0, 6.0, 7.0, 8.0], dtype=np.float32)
    result = dot_product(a, b)
    np.testing.assert_almost_equal(result, 70.0)

def test_large_array_performance():
    """Verify Mojo kernel is faster than NumPy for large arrays."""
    a = np.random.randn(1_000_000).astype(np.float32)
    b = np.random.randn(1_000_000).astype(np.float32)

    import time
    start = time.perf_counter()
    result_mojo = dot_product(a, b)
    mojo_time = time.perf_counter() - start

    start = time.perf_counter()
    result_numpy = np.dot(a, b)
    numpy_time = time.perf_counter() - start

    np.testing.assert_almost_equal(result_mojo, result_numpy, decimal=1)
    # Mojo should be competitive or faster
    print(f"Mojo: {mojo_time:.6f}s, NumPy: {numpy_time:.6f}s")
```

```bash
uv run pytest tests/
```

## Project Structure

```
project/
├── pyproject.toml            # hatch-mojo config lives here
├── src/
│   ├── mo/                   # Mojo source
│   │   ├── my_pkg/
│   │   │   ├── core.mojo     # Python extension (emit: python-extension)
│   │   │   ├── ops.mojo      # SIMD/parallel kernels
│   │   │   └── layers.mojo   # Compute layers
│   │   └── tests/            # Mojo unit tests
│   └── py/                   # Python package
│       └── my_package/
│           ├── __init__.py   # Re-exports from _core
│           └── py.typed      # PEP 561
├── tests/                    # Python integration tests
├── benchmarks/               # Performance comparisons
└── .hatch_mojo/              # Build artifacts (gitignored)
```

## Conventions

- `fn` over `def` for all Mojo functions unless Python interop requires `def`.
- Explicit types everywhere — no type inference for function signatures.
- Document `UnsafePointer` usage with safety comments.
- Benchmark against NumPy/Python baselines and document results.
- Use `@parameter` for compile-time specialization.
- Keep Python wrappers thin — compute logic stays in Mojo.
- Use `uv` for all Python tooling (never `pip`, `pixi`, or `conda`).

## Official References

- https://docs.modular.com/max/coding-assistants
- https://docs.modular.com/max/coding-assistants/codex/
- https://docs.modular.com/llms-mojo.txt
- https://docs.modular.com/llms-mojo-python.txt
- https://docs.modular.com/llms-mojo-kernel.txt
- https://docs.modular.com/llms-max.txt
- https://docs.modular.com/llms.txt
- https://docs.modular.com/mojo/
- https://docs.modular.com/mojo/changelog/
- https://docs.modular.com/mojo/manual/testing/
- https://docs.modular.com/mojo/manual/python/mojo-from-python/
- https://docs.modular.com/mojo/stdlib/ffi/external_call/
- https://github.com/modular/modular/blob/main/.github/agents/AGENTS.md
- https://github.com/modular/modular/blob/main/.github/agents/CLAUDE.md
- https://pypi.org/project/hatch-mojo/

## Shared Styleguide Baseline

- Use shared styleguides for generic language/framework rules to reduce duplication in this skill.
- [General Principles](https://github.com/cofin/flow/blob/main/templates/styleguides/general.md)
- [Python](https://github.com/cofin/flow/blob/main/templates/styleguides/languages/python.md)
- Keep this skill focused on tool-specific workflows, edge cases, and integration details.
