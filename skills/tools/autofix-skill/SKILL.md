---
name: autofix-skill
description: Comprehensive build error repair tool for C++, Rust, and Java projects using Soong, GN, Make, or CMake. Automatically fixes 30+ types of errors including headers, linking, APIs, and config.
---

# AutoFix-Skill

This skill integrates the AutoFix engine to intelligently repair build failures in AOSP, OpenHarmony, and Linux Kernel environments.

## capabilities

The skill operates by parsing build logs (or error strings), identifying the specific error type via regex/heuristics, and applying targeted fixes to the source code or build configuration files.

### 1. Symbol & Header Issues
- **Missing Header**: Adds `#include <...>` for standard and project headers.
- **Undeclared Identifier**: Import/include missing symbols.
- **Namespace Errors**: Fixes missing `std::` or namespace scopes.
- **Forward Declaration**: Inserts forward decls to resolve circular deps.

### 2. Linkage & Dependencies
- **Undefined Reference**: Adds libraries to `shared_libs`/`deps` in `Android.bp`/`BUILD.gn`.
- **Rust Crates**: Adds missing `rustlibs` implies.
- **Visibility**: Resolves visibility restrictions between modules.
- **Multiple Definitions**: Removes duplicate sources or link dependencies.
- **Vtable Issues**: Identifies missing virtual function implementations.

### 3. API & Type Safety
- **Signature Mismatch**: Fixes function call arguments (too few/many).
- **Type Conversion**: Suggests `static_cast`, `c_str()`, etc.
- **Const Correctness**: Fixes `const` qualifier mismatches.
- **Deprecated API**: Updates legacy API calls to modern equivalents.
- **Missing Override**: Adds `override` keyword to virtual destructors/methods.

### 4. Build Configuration
- **Syntax Errors**: Fixes typos in `Android.bp`/`BUILD.gn`.
- **Compiler Flags**: Removes unsupported/illegal flags.
- **Permissions**: Fixes `chmod +x` for build scripts.
- **Ninja Cache**: Cleans dirty ninja build states.

## Usage

### Fix from Log File (Recommended)
```bash
cd /absolute/path/to/autofix_skill
PYTHONPATH=. python3 -m src.cli fix --log /path/to/build.log --root /path/to/project_root
```

### Fix Single Error
```bash
cd /absolute/path/to/autofix_skill
PYTHONPATH=. python3 -m src.cli fix --error "fatal error: 'vector' file not found"
```

### Scan for Patterns
```bash
cd /absolute/path/to/autofix_skill
PYTHONPATH=. python3 -m src.cli scan
```

## Build Systems
- **Android**: `Android.bp` (Soong)
- **OpenHarmony**: `BUILD.gn` (GN)
- **Linux Kernel**: `Kbuild`/`Makefile`
- **General C++**: `CMakeLists.txt`
