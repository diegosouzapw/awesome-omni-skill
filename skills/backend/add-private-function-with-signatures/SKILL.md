---
name: add-private-function-with-signatures
description: Add private functions from game DLLs (server.dll/engine.dll) to metamod plugins using signature scanning and symbol lookup. Use when adding new private function hooks that require IDA Pro analysis, signature creation, cross-platform support (Windows/Linux), and runtime function pointer filling.
---

# Add Private Function With Signatures

## Overview

This skill guides the process of adding private functions from game engine DLLs to metamod plugins. It covers IDA Pro analysis, signature creation, function type definition, and runtime hooking for both Windows and Linux platforms.

## Workflow

### Step 1: Analyze Function with IDA Pro

Use IDA Pro MCP tools to analyze the target function:

```bash
# Find the function
mcp__ida-pro-mcp__lookup_funcs("FunctionName")

# Get decompiled code
mcp__ida-pro-mcp__decompile("0xAddress")

# Get disassembly
mcp__ida-pro-mcp__disasm("0xAddress", max_instructions=30)

# Extract signature bytes
mcp__ida-pro-mcp__get_bytes({"addr": "0xAddress", "size": 50})

# Verify signature uniqueness (CRITICAL STEP!)
# Convert bytes to IDA search format (use signature_converter.py script)
python scripts/signature_converter.py spaced "\\x83\\xEC\\x2C\\xA1\\x2A\\x2A\\x2A\\x2A"
# Output: 83 EC 2C A1 ?? ?? ?? ??

# Search in IDA Pro with converted pattern
mcp__ida-pro-mcp__find_bytes("83 EC 2C A1 ?? ?? ?? ?? 33 C4")
# Result should show ONLY ONE match - your target function
# If multiple matches found, extend signature or use more specific bytes
```

**Critical**: Always verify signature uniqueness using `find_bytes`. If the pattern matches multiple functions, the signature is too generic and must be made more specific.

**Tip**: Use `scripts/signature_converter.py` to convert between formats:
```bash
# Convert C string to IDA format
python scripts/signature_converter.py spaced "\\x83\\xEC\\x2C"

# Convert IDA format to C string
python scripts/signature_converter.py c_string "83 EC 2C"

# Show all formats
python scripts/signature_converter.py all "83EC2C"
```

### Step 2: Define Function Type

Edit `fallguys/serverdef.h`:

```cpp
// Function signature from IDA: ReturnType __callingconv FunctionName(params)
typedef ReturnType(CALLING_CONVENTION *fnFunctionName)(ParamTypes);
PRIVATE_FUNCTION_EXTERN(FunctionName);
```

**Calling convention mapping**:
- `SC_SERVER_DECL` - fastcall (Windows) / cdecl (Linux), for class methods
- `SC_SERVER_CDECL` - cdecl (both platforms), for C functions
- Default - cdecl

For new calling conventions, see [calling-conventions.md](references/calling-conventions.md).

### Step 3: Add Function Signatures

Edit `fallguys/signatures.h`:

**Windows signature**:
```cpp
#ifdef _WIN32
#define FunctionName_Signature "\\x83\\xEC\\x2C\\xA1\\x2A\\x2A\\x2A\\x2A..."
```

**Linux signature and symbol**:
```cpp
#else
#define FunctionName_Signature "\\x55\\x57\\x56\\x53\\x83\\xEC\\x5C..."
#define FunctionName_Symbol "_Z23FunctionNameMangledName"
```

See [signature-patterns.md](references/signature-patterns.md) for signature creation guidelines.

### Step 4: Define Global Function Pointer

Edit `fallguys/server_hook.cpp` (or relevant .cpp file):

```cpp
PRIVATE_FUNCTION_DEFINE(ExistingFunction1);
PRIVATE_FUNCTION_DEFINE(ExistingFunction2);
PRIVATE_FUNCTION_DEFINE(FunctionName);  // Add new line
```

This expands to:
```cpp
fnFunctionName g_pfn_FunctionName;
fnFunctionName g_call_original_FunctionName;
```

### Step 5: Fill Function Pointer in Meta_Attach

Edit `fallguys/meta_api.cpp`:

**Windows branch**:
```cpp
#ifdef _WIN32
FILL_FROM_SIGNATURE(server, FunctionName);
```

**Linux Sven Co-op 5.16+ branch**:
```cpp
if (CreateInterface("SCServerDLL003", nullptr) != nullptr)
{
    FILL_FROM_SIGNATURE(server, FunctionName);
}
```

**Linux Sven Co-op 5.15 branch**:
```cpp
else
{
    FILL_FROM_SYMBOL(server, FunctionName);
}
```

**Fill macros**:
- `FILL_FROM_SIGNATURE(module, name)` - Scan by signature
- `FILL_FROM_SYMBOL(module, name)` - Lookup by symbol
- `FILL_FROM_SIGNATURED_CALLER_FROM_END(module, name, offset)` - Scan caller signature

**Module options**: `server` (server.dll/so), `engine` (hw.dll/so)

### Step 6: Call Function in Business Code

Always check pointer before calling:

```cpp
if (g_pfn_FunctionName)
{
    g_pfn_FunctionName(arg1, arg2);
}
```

**Example usage**:
```cpp
ASEXT_RegisterDocInitCallback([](CASDocumentation *pASDoc) {
    if (g_pfn_RegisterSCScriptColor24)
    {
        g_pfn_RegisterSCScriptColor24(pASDoc);
    }
});
```

## Checklist

- [ ] Analyze function with IDA Pro (decompile, disasm, bytes)
- [ ] Verify signature uniqueness in IDA Pro
- [ ] Define calling convention macro (if new) in `asext/include/asext_api.h`
- [ ] Define function typedef in `fallguys/serverdef.h`
- [ ] Add Windows signature in `fallguys/signatures.h`
- [ ] Add Linux signature and symbol in `fallguys/signatures.h`
- [ ] Add `PRIVATE_FUNCTION_DEFINE` in `fallguys/server_hook.cpp`
- [ ] Add Windows `FILL_FROM_SIGNATURE` in `fallguys/meta_api.cpp`
- [ ] Add Linux 5.16 `FILL_FROM_SIGNATURE` in `fallguys/meta_api.cpp`
- [ ] Add Linux 5.15 `FILL_FROM_SYMBOL` in `fallguys/meta_api.cpp`
- [ ] Call function with null check in business code
- [ ] Compile and test (Windows and Linux)
- [ ] Verify function found in logs

## Key Files

- `asext/include/asext_api.h` - Calling convention macros
- `fallguys/serverdef.h` - Function type definitions
- `fallguys/signatures.h` - Signatures and symbols
- `fallguys/server_hook.cpp` - Function pointer definitions
- `fallguys/meta_api.cpp` - Function pointer filling
- `metamod/signatures_template.h` - Macro definitions

## References

- [calling-conventions.md](references/calling-conventions.md) - Detailed calling convention guide
- [signature-patterns.md](references/signature-patterns.md) - Signature creation best practices
- [troubleshooting.md](references/troubleshooting.md) - Common issues and solutions

## Scripts

### signature_converter.py

Converts between three signature formats:
- **C String**: `\x83\xEC\x2C\xA1\x2A` (for code)
- **Spaced**: `83 EC 2C A1 ??` (for IDA Pro)
- **Compact**: `83EC2CA12A` (compressed)

**Usage**:
```bash
# Convert to IDA Pro format
python scripts/signature_converter.py spaced "\x83\xEC\x2C"

# Convert to C string format
python scripts/signature_converter.py c_string "83 EC 2C"

# Show all formats
python scripts/signature_converter.py all "83EC2C"

# Get help
python scripts/signature_converter.py --help
```
