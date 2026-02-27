---
name: patterns/adapter
description: Adapter (Wrapper) Pattern pattern for C development
---

# Adapter (Wrapper) Pattern

Convert interface of one module to interface expected by another. Enables integration of incompatible interfaces without modifying either.

## ikigai Application

**wrapper.h:** Adapts system calls to mockable interface:
- `posix_read_()` wraps `read()`
- `curl_easy_perform_()` wraps libcurl
- `yyjson_read_file_()` wraps yyjson

**Purpose:** Real implementations in production, mock implementations in tests via weak symbol linking.

**LLM providers:** Adapt different API formats (OpenAI, Anthropic) to unified internal message format.

**Benefit:** External dependencies isolated behind stable internal interface. Swap implementations without changing callers.

**Testing:** Inject failures, control responses, verify calls.
