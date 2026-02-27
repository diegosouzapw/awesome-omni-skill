---
name: better-leetcode
description: Optimize and refactor LeetCode solutions to be more elegant, readable, and idiomatic. Use when the user provides code from a LeetCode problem and wants to improve code quality, simplify logic, enhance variable naming, or make the solution more concise and maintainable. Also use when the user asks to "optimize", "refactor", "improve", or "clean up" LeetCode code.
---

# Better LeetCode

Refactor LeetCode solutions to follow clean code principles: clear logic, intuitive variable names, minimal comments, and idiomatic patterns.

## Core Principles

1. **Clear over clever**: Prefer readable solutions that are easy to understand at first glance
2. **Minimal comments**: Code should be self-documenting through good naming and clear structure
3. **Idiomatic patterns**: Use well-known algorithms and data structures with their standard implementations
4. **Concise variable names**: Use short but meaningful names that follow common conventions

## Refactoring Guidelines

### Variable Naming

Use standard, memorable variable names for common patterns:

- **Pointers**: `slow`, `fast`, `left`, `right`, `i`, `j`, `k`
- **Lists/Arrays**: `nums`, `arr`, `res`, `ans`
- **Trees**: `root`, `node`, `left`, `right`
- **Graphs**: `graph`, `node`, `visited`, `adj`
- **Strings**: `s`, `t`, `word`, `pattern`
- **Counters**: `count`, `cnt`, `freq`
- **Results**: `res`, `ans`, `result`

### Code Structure

**Do:**
- Keep functions focused and simple
- Use early returns to reduce nesting
- Prefer while loops for unclear iteration counts
- Use for loops with range for known iterations
- Initialize variables close to usage

**Avoid:**
- Excessive comments explaining obvious logic
- Overly complex one-liners that sacrifice readability
- Unnecessary temporary variables
- Deep nesting (>3 levels)

### Common Patterns

**Two Pointers:**
```python
slow = fast = head
while fast and fast.next:
    fast = fast.next.next
    slow = slow.next
```

**Sliding Window:**
```python
left = 0
for right in range(len(s)):
    # expand window
    while condition:
        # shrink window
        left += 1
```

**Binary Search:**
```python
left, right = 0, len(nums) - 1
while left <= right:
    mid = (left + right) // 2
    if nums[mid] == target:
        return mid
    elif nums[mid] < target:
        left = mid + 1
    else:
        right = mid - 1
```

**DFS (Tree):**
```python
def dfs(node):
    if not node:
        return
    # process node
    dfs(node.left)
    dfs(node.right)
```

**BFS:**
```python
from collections import deque

queue = deque([start])
while queue:
    node = queue.popleft()
    # process node
    if condition:
        queue.append(next_node)
```

## Optimization Process

When refactoring code:

1. **Analyze**: Identify the algorithm/pattern being used
2. **Simplify**: Remove unnecessary complexity and redundancy
3. **Rename**: Apply standard variable naming conventions
4. **Structure**: Organize code for maximum readability
5. **Verify**: Ensure logic correctness is maintained

Focus on readability and maintainability over micro-optimizations unless performance is explicitly required.
