---
name: algorithms
description: Master algorithm design, common patterns, optimization techniques, and problem-solving strategies. Learn to solve any computational challenge efficiently.
sasmp_version: "1.3.0"
bonded_agent: 02-algorithms-expert
bond_type: PRIMARY_BOND
---

# Algorithms Skill

## Skill Metadata

```yaml
skill_config:
  version: "1.0.0"
  category: problem-solving
  prerequisites: [cs-foundations]
  estimated_time: "8-12 weeks"
  difficulty: intermediate-advanced

  parameter_validation:
    pattern:
      type: string
      enum: [search, sort, dp, greedy, backtrack, divide-conquer, graph]
      required: true
    language:
      type: string
      enum: [python, java, cpp, javascript]
      default: python

  retry_config:
    max_attempts: 3
    backoff_strategy: exponential
    initial_delay_ms: 500

  observability:
    log_level: INFO
    metrics: [pattern_usage, solution_complexity, hint_count]
```

---

## Quick Start

Become a master problem solver through systematic algorithm design.

### Common Algorithm Patterns

**Searching & Sorting (Week 1)**
- Linear search O(n)
- Binary search O(log n)
- Merge sort O(n log n)
- Quick sort O(n log n) average
- Heap sort O(n log n)

**Divide & Conquer (Week 2)**
- Split problem into subproblems
- Solve recursively
- Combine results
- Analyze with recurrence relations

**Dynamic Programming (Weeks 3-4)**
- Identify overlapping subproblems
- Define state and transitions
- Memoization vs tabulation
- Classic problems: Fibonacci, knapsack, LCS

**Greedy Algorithms (Week 5)**
- Make optimal local choice
- Examples: Huffman coding, Dijkstra, activity selection
- When to use: optimal substructure + greedy choice property

**Backtracking (Week 6)**
- Explore all solutions
- Backtrack when dead end
- Pruning branches
- N-Queens, permutations, sudoku solver

---

## Problem-Solving Framework

1. **Understand**: Read problem, identify constraints, find examples
2. **Plan**: Recognize pattern, think brute force, optimize
3. **Code**: Write clean code, handle edge cases
4. **Optimize**: Improve time/space complexity
5. **Verify**: Test, verify analysis

---

## Must-Know Problems

- **Array Problems**: Two sum, max subarray, product of array
- **String Problems**: Longest substring, pattern matching, anagrams
- **Tree Problems**: Traversal, BST, LCA, path sum
- **Graph Problems**: Shortest path, cycle detection, topological sort
- **DP Problems**: Fibonacci, knapsack, coin change, edit distance
- **Math Problems**: Prime numbers, GCD, power

---

## Troubleshooting

| Issue | Root Cause | Resolution |
|-------|------------|------------|
| TLE | Wrong complexity class | Find better algorithm |
| WA | Edge case missing | Test empty, single, max inputs |
| RE | Index out of bounds | Add bounds checking |
| Stack overflow | Deep recursion | Convert to iteration |

---

## Complexity Quick Reference

| Problem | Best | Average | Worst |
|---------|------|---------|-------|
| Merge sort | O(n log n) | O(n log n) | O(n log n) |
| Quick sort | O(n log n) | O(n log n) | O(n²) |
| Binary search | O(1) | O(log n) | O(log n) |
| Linear search | O(1) | O(n/2) | O(n) |

---

## Interview Tips

- Communicate your approach
- Discuss time/space trade-offs
- Code cleanly on the first try
- Test with examples
- Optimize if time permits
