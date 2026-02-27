---
name: safety-checker
description: Identifies unsafe operations in Zig code including pointer casts, bounds checking, null pointer dereferences, and undefined behavior. Use when writing low-level code, reviewing safety-critical sections, or debugging crashes.
---

# Safety Checker

This skill helps identify unsafe operations and potential undefined behavior in Zig code, ensuring memory safety and correctness.

## Zig Safety Principles

Zig provides safety by default in Debug and ReleaseSafe builds:
- Bounds checking on arrays and slices
- Integer overflow detection
- Null pointer checks for optionals
- Alignment verification

**However**, unsafe operations can bypass these checks and cause undefined behavior.

## Unsafe Operations to Watch For

### 1. Pointer Casts Without Validation

#### Type Punning with @ptrCast

```zig
// DANGEROUS: No validation of pointer validity
pub fn setHandler(self: *Server, handler: *anyopaque) void {
    self.handler = handler;
}

pub fn getHandler(self: *Server) *Handler {
    const handler_ptr = self.handler.?;
    return @ptrCast(@alignCast(handler_ptr));  // Unsafe cast!
}
```

**Problems:**
- No type safety: `*anyopaque` can point to anything
- No alignment verification before cast
- No null check (using `.?` panics)
- Could point to wrong type

**Better approach:**

```zig
// SAFE: Use proper optional type
pub const Server = struct {
    handler: ?*Handler,  // Type-safe optional pointer

    pub fn setHandler(self: *Server, handler: *Handler) void {
        self.handler = handler;  // Type checked
    }

    pub fn getHandler(self: *Server) ?*Handler {
        return self.handler;  // No cast needed
    }
};
```

**If type erasure is truly needed:**

```zig
// SAFER: Validate before casting
pub fn getHandler(self: *Server) ServerError!*Handler {
    const handler_ptr = self.handler orelse return error.NoHandler;

    // Validate alignment
    const addr = @intFromPtr(handler_ptr);
    if (addr % @alignOf(Handler) != 0) {
        return error.InvalidAlignment;
    }

    // Cast with explicit alignment
    const aligned_ptr: *align(@alignOf(Handler)) anyopaque = @alignCast(handler_ptr);
    return @ptrCast(aligned_ptr);
}
```

### 2. Missing Bounds Checks

#### Array/Slice Access

```zig
// DANGEROUS: No bounds check
pub fn readIntBig(buffer: []const u8, comptime T: type) T {
    const size = @sizeOf(T);
    if (buffer.len < size) @panic("Buffer too small");  // Panic!

    const bytes = buffer[0..size];
    return std.mem.readInt(T, bytes[0..size], .big);
}
```

**Problems:**
- Panic on invalid input (should return error)
- Potential crash if panic is reached in release mode
- No way for caller to handle error

**Better approach:**

```zig
// SAFE: Return error, not panic
pub fn readIntBig(buffer: []const u8, comptime T: type) ParseError!T {
    const size = @sizeOf(T);
    if (buffer.len < size) return error.BufferTooSmall;

    const bytes = buffer[0..size];
    return std.mem.readInt(T, bytes[0..size], .big);
}
```

#### Unchecked Indexing

```zig
// DANGEROUS: Assumes index is valid
pub fn getRegister(self: *Server, index: u8) Value {
    return self.registers[index];  // Could be out of bounds!
}

// SAFE: Check bounds explicitly
pub fn getRegister(self: *Server, index: u8) ServerError!Value {
    if (index >= self.registers.len) return error.RegisterOutOfBounds;
    return self.registers[index];
}

// ALSO SAFE: Use optional for invalid indices
pub fn getRegister(self: *Server, index: u8) ?Value {
    if (index >= self.registers.len) return null;
    return self.registers[index];
}
```

### 3. Unsafe Pointer Arithmetic

```zig
// DANGEROUS: Manual pointer arithmetic
pub fn advance(ptr: [*]const u8, offset: usize) [*]const u8 {
    return ptr + offset;  // No bounds check!
}

// SAFE: Use slices with bounds
pub fn advance(slice: []const u8, offset: usize) ?[]const u8 {
    if (offset >= slice.len) return null;
    return slice[offset..];
}
```

### 4. Uninitialized Memory

```zig
// DANGEROUS: Reading uninitialized memory
pub fn createValue() Value {
    var value: Value = undefined;  // Uninitialized!
    if (condition) {
        value = Value.makeInt(42);
    }
    return value;  // May return uninitialized data!
}

// SAFE: Initialize before use
pub fn createValue() Value {
    if (condition) {
        return Value.makeInt(42);
    }
    return Value.nil();  // Always initialized
}
```

### 5. Null Pointer Dereference

```zig
// DANGEROUS: Force-unwrapping optionals
pub fn process(self: *Server) void {
    const handler = self.handler.?;  // Panics if null!
    handler.handle(self);
}

// SAFE: Check for null
pub fn process(self: *Server) ServerError!void {
    const handler = self.handler orelse return error.NoHandler;
    try handler.handle(self);
}

// ALSO SAFE: Use if for optional handling
pub fn process(self: *Server) void {
    if (self.handler) |handler| {
        handler.handle(self) catch |err| {
            std.log.err("Handle failed: {}", .{err});
        };
    }
}
```

### 6. Integer Overflow

```zig
// DANGEROUS: Unchecked arithmetic in ReleaseFast
pub fn allocate(size: usize, count: usize) ![]u8 {
    const total = size * count;  // Can overflow!
    return try allocator.alloc(u8, total);
}

// SAFE: Use checked arithmetic
pub fn allocate(size: usize, count: usize) ![]u8 {
    const total = std.math.mul(usize, size, count) catch {
        return error.ArithmeticOverflow;
    };
    return try allocator.alloc(u8, total);
}

// ALSO SAFE: Wrapping arithmetic when overflow is intended
pub fn hash(value: u32) u32 {
    return value *% 31;  // *% = wrapping multiplication
}
```

### 7. Alignment Issues

```zig
// DANGEROUS: Assuming alignment
pub fn readU32(bytes: []const u8) u32 {
    const ptr: *const u32 = @ptrCast(bytes.ptr);  // May be misaligned!
    return ptr.*;
}

// SAFE: Use std.mem functions
pub fn readU32(bytes: []const u8) !u32 {
    if (bytes.len < 4) return error.BufferTooSmall;
    return std.mem.readInt(u32, bytes[0..4], .little);
}

// SAFE: Check alignment explicitly
pub fn readU32(bytes: []align(4) const u8) u32 {
    const ptr: *const u32 = @ptrCast(bytes.ptr);  // Compiler ensures alignment
    return ptr.*;
}
```

### 8. Unsafe Memory Operations

```zig
// DANGEROUS: @memcpy with wrong size
pub fn copy(dest: []u8, src: []const u8) void {
    @memcpy(dest, src);  // Panics if dest.len < src.len
}

// SAFE: Check sizes first
pub fn copy(dest: []u8, src: []const u8) !void {
    if (dest.len < src.len) return error.DestinationTooSmall;
    @memcpy(dest[0..src.len], src);
}
```

## Safety Checklist

### When Writing Low-Level Code:

- [ ] No `@ptrCast` without validation
- [ ] Check alignment before pointer casts
- [ ] Validate buffer sizes before access
- [ ] Use errors instead of panics for invalid input
- [ ] Initialize all variables before use
- [ ] Check for null before unwrapping optionals
- [ ] Use checked arithmetic for user-controlled values
- [ ] Verify alignment assumptions
- [ ] Validate sizes for `@memcpy`, `@memset`

### When Reviewing Code:

- [ ] Look for `@ptrCast` - is it necessary? Is it safe?
- [ ] Look for `@alignCast` - is alignment guaranteed?
- [ ] Check all array/slice indexing - bounds checked?
- [ ] Look for `.?` unwrapping - can it be null?
- [ ] Check for `undefined` - is it initialized on all paths?
- [ ] Look for panics - should they be errors instead?
- [ ] Check arithmetic - can it overflow?
- [ ] Verify `@memcpy` sizes match

## Red Flags

### Immediate Review Needed:

1. **`@ptrCast` without validation** - Almost always wrong
2. **`@panic` for user input** - Should be errors
3. **`.?` on optional without null check** - May crash
4. **Array access without bounds check** - May crash
5. **`undefined` without initialization on all paths** - UB
6. **Unchecked arithmetic on user input** - May overflow

### Consider Refactoring:

1. **Type erasure with `*anyopaque`** - Prefer proper types
2. **Manual pointer arithmetic** - Use slices instead
3. **Assuming alignment** - Make it explicit
4. **Complex pointer operations** - Simplify if possible

## Testing Safety

Test edge cases and error conditions:

```zig
test "register access out of bounds" {
    var server = try Server.init(std.testing.allocator);
    defer server.deinit();

    // Should return error, not crash
    try std.testing.expectError(
        ServerError.RegisterOutOfBounds,
        server.getRegister(255)
    );
}

test "stack overflow protection" {
    var server = try Server.init(std.testing.allocator);
    defer server.deinit();

    // Fill stack
    while (server.sp < server.stack.len) {
        try server.push(Value.makeInt(0));
    }

    // Next push should error, not crash
    try std.testing.expectError(
        ServerError.StackOverflow,
        server.push(Value.makeInt(1))
    );
}

test "buffer too small handling" {
    const data = [_]u8{ 1, 2 };  // Only 2 bytes

    // Should error, not crash
    try std.testing.expectError(
        ParseError.BufferTooSmall,
        readIntBig(&data, u32)  // Needs 4 bytes
    );
}
```

## Build Modes and Safety

- **Debug**: All safety checks enabled (bounds, overflow, etc.)
- **ReleaseSafe**: Safety checks enabled, optimizations on
- **ReleaseFast**: Safety checks **disabled** for performance
- **ReleaseSmall**: Like ReleaseFast but optimizes for size

**Golden Rule:** Code should be correct in ReleaseFast even though safety checks are disabled. Never rely on runtime checks for correctness in release builds.

```zig
// BAD: Relies on Debug mode checks
pub fn access(slice: []u8, index: usize) u8 {
    return slice[index];  // Crashes in ReleaseFast if out of bounds!
}

// GOOD: Explicit check in all modes
pub fn access(slice: []u8, index: usize) ?u8 {
    if (index >= slice.len) return null;
    return slice[index];
}
```

## Summary

Safety in Zig is about:
1. **Validation** - Check inputs and bounds explicitly
2. **Error handling** - Return errors, don't panic
3. **Type safety** - Avoid type erasure when possible
4. **Explicit intent** - Make assumptions clear in types
5. **Testing** - Test error paths and edge cases

**When in doubt, prefer safety over performance.** Optimize only when profiling proves it necessary.
