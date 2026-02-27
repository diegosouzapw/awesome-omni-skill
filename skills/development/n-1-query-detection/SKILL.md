---
name: N+1 Query Detection
description: Detect N+1 query patterns in GORM repository and service code — identify loops that execute queries, missing preloads, and unbounded fetches
category: performance
version: 1.0.0
triggers:
  - repository-change
  - service-change
  - performance-review
globs: "**/repository/**,**/service/**"
---

# N+1 Query Detection Skill

Scan repository and service code for N+1 query patterns that cause performance degradation.

## Trigger Conditions
- Repository or service files are modified
- Performance review is requested
- New data access patterns are introduced
- User invokes with "N+1 check" or "n-plus-1-query-detection"

## Input Contract
- **Required:** Path to repository and/or service files
- **Optional:** GORM configuration for preload analysis

## Output Contract
- List of N+1 patterns found with file:line references
- Severity assessment (critical for high-volume paths, low for admin-only)
- Fix recommendation (Preload, Joins, or batch query)
- Unbounded query warnings (missing Limit)

## Tool Permissions
- **Read:** Repository files, service files, model relationship definitions
- **Write:** None (read-only analysis)
- **Search:** Grep for loop patterns, `.Find()`, `.First()`, relationship access

## Execution Steps

1. **Scan for loops with queries**: Find patterns where a database query is inside a `for` loop
2. **Check preloads**: For each query that accesses relationships, verify `Preload()` or `Joins()` is used
3. **Check unbounded queries**: Find `.Find()` calls without `.Limit()` on list endpoints
4. **Assess severity**: Rate each finding by the expected data volume (high-traffic endpoints are critical)
5. **Recommend fixes**: Suggest specific GORM patterns (`Preload`, `Joins`, `SelectinLoad`)
6. **Report**: Produce findings with code examples for the fix

## Common Patterns (Anti-patterns)

```go
// N+1: Query inside loop
users, _ := db.Find(&users)
for _, u := range users {
    db.Where("user_id = ?", u.ID).Find(&orders) // N+1!
}

// Fix: Preload
db.Preload("Orders").Find(&users)
```

## References
- `.cursor/rules/122-gorm-conventions.mdc`
- `.cursor/rules/047-performance-optimization.mdc`
