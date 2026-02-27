---
name: pattern-detection
description: Identify existing codebase patterns (naming conventions, architectural patterns, testing patterns) to maintain consistency. Use when generating code, reviewing changes, or understanding established practices. Ensures new code aligns with project conventions.
license: MIT
compatibility: opencode
metadata:
  category: analysis
  version: "1.0"
---

# Pattern Recognition

## When to Use

- Before writing new code to ensure consistency with existing patterns
- During code review to verify alignment with established conventions
- When onboarding to understand project-specific practices
- Before refactoring to preserve intentional design decisions

## Core Methodology

```sudolang
PatternDiscovery {
  State {
    representativeFiles
    discoveredPatterns
    patternSources
  }

  Constraints {
    Survey 3-5 representative files before writing new code.
    Patterns must be verified for intentionality before applying.
    Most authoritative source takes precedence.
  }

  /discover fileType => {
    1. Survey 3-5 files of $fileType
    2. Identify recurring structures in naming, organization, imports
    3. Verify intentionality via documentation or consistent application
    4. Catalog discovered patterns
  }

  /apply pattern, newCode => {
    require pattern is verified
    apply pattern conventions to newCode
  }
}

PatternSourcePriority {
  resolvePattern(query) {
    match availableSources {
      { sameModule: patterns } if patterns exist =>
        patterns  Most authoritative
      { styleGuide: documented } if documented =>
        documented  Explicit documentation
      { testFiles: patterns } if patterns exist =>
        patterns  Often reveal expected patterns
      { adjacentModules: patterns } if patterns exist =>
        patterns  Fallback when no direct examples
      _ => warn "No pattern sources found - document assumptions"
    }
  }
}
```

## Naming Convention Recognition

### File Naming Patterns

Detect and follow the project's file naming style:

| Pattern | Example | Common In |
|---------|---------|-----------|
| kebab-case | `user-profile.ts` | Node.js, Vue, Angular |
| PascalCase | `UserProfile.tsx` | React components |
| snake_case | `user_profile.py` | Python |
| camelCase | `userProfile.js` | Legacy JS, Java |

```sudolang
FileNamingDetector {
  detectNamingPattern(files) {
    patterns = files |> map(f => classifyNaming(f))
    dominant = patterns |> groupBy identity |> maxBy count

    match dominant {
      "kebab-case" => { style: "kebab-case", example: "user-profile.ts" }
      "PascalCase" => { style: "PascalCase", example: "UserProfile.tsx" }
      "snake_case" => { style: "snake_case", example: "user_profile.py" }
      "camelCase" => { style: "camelCase", example: "userProfile.js" }
      _ => warn "Mixed naming conventions detected"
    }
  }
}
```

### Function/Method Naming

Identify the project's verb conventions:

```sudolang
VerbConventions {
  dataAccess: "get" | "fetch" | "retrieve"
  creation: "create" | "add" | "new"
  mutation: "update" | "set" | "modify"
  deletion: "delete" | "remove" | "destroy"
  booleanPrefixes: ["is", "has", "can", "should"]

  detectVerbConvention(category, codebase) {
    usages = codebase |> extractFunctions |> filterByCategory(category)
    usages |> groupBy verb |> maxBy count |> first
  }
}
```

### Variable Naming

Detect pluralization and specificity patterns:

- Singular vs plural for collections (`user` vs `users` vs `userList`)
- Hungarian notation presence (`strName`, `iCount`)
- Private member indicators (`_private`, `#private`, `mPrivate`)

## Architectural Pattern Recognition

### Layer Identification

Recognize how the codebase separates concerns:

```sudolang
ArchitectureDetector {
  detectLayeringPattern(structure) {
    match structure.directories {
      dirs if hasAll(dirs, ["controllers", "models", "views"]) =>
        { pattern: "MVC", layers: ["controllers", "models", "views"] }
      dirs if hasAll(dirs, ["domain", "application", "infrastructure"]) =>
        { pattern: "Clean Architecture", layers: ["domain", "application", "infrastructure"] }
      dirs if hasAll(dirs, ["core", "adapters", "ports"]) =>
        { pattern: "Hexagonal", layers: ["core", "adapters", "ports"] }
      dirs if any dir starts with "features/" =>
        { pattern: "Feature-based", layers: extractFeatures(dirs) }
      dirs if hasAll(dirs, ["components", "services", "utils"]) =>
        { pattern: "Type-based", layers: ["components", "services", "utils"] }
      _ => { pattern: "Unknown", layers: dirs }
    }
  }
}
```

### Dependency Direction

Identify import patterns that reveal architecture:

- Which modules import from which (dependency flow)
- Shared vs feature-specific code boundaries
- Framework code vs application code separation

### State Management Patterns

Recognize how state flows through the application:

- Global stores (Redux, Vuex, MobX patterns)
- React Context usage patterns
- Service layer patterns for backend state
- Event-driven vs request-response patterns

## Testing Pattern Recognition

### Test Organization

Identify how tests are structured:

| Pattern | Structure | Example |
|---------|-----------|---------|
| Co-located | `src/user.ts`, `src/user.test.ts` | Common in modern JS/TS |
| Mirror tree | `src/user.ts`, `tests/src/user.test.ts` | Traditional, Java-style |
| Feature-based | `src/user/`, `src/user/__tests__/` | React, organized features |

```sudolang
TestPatternDetector {
  detectTestOrganization(projectRoot) {
    match projectRoot {
      root if hasColocatedTests(root) =>
        { organization: "co-located", testPath: "same directory as source" }
      root if hasMirrorTree(root) =>
        { organization: "mirror-tree", testPath: "tests/ mirrors src/" }
      root if hasFeatureTests(root) =>
        { organization: "feature-based", testPath: "__tests__/ in feature dirs" }
      _ => warn "Test organization unclear - check existing tests"
    }
  }

  detectTestNamingStyle(testFiles) {
    match testFiles |> extractDescriptions |> classify {
      "BDD" => { style: "BDD", example: "it('should return user when found')" }
      "descriptive" => { style: "descriptive", example: "test('getUser returns user when id exists')" }
      "function-focused" => { style: "function-focused", example: "test_get_user_returns_user_when_found" }
      _ => warn "Mixed test naming styles detected"
    }
  }
}
```

### Test Structure Patterns

Recognize Arrange-Act-Assert or Given-When-Then patterns:

- Setup block conventions (beforeEach, fixtures, factories)
- Assertion style (expect vs assert vs should)
- Mock/stub patterns (jest.mock vs sinon vs manual)

## Code Organization Patterns

### Import Organization

Identify import ordering and grouping:

```sudolang
ImportPatternDetector {
  commonPatterns: [
    "External packages first, internal modules second",
    "Grouped by type (React, libraries, local)",
    "Alphabetized within groups",
    "Absolute imports vs relative imports preference"
  ]

  detectImportPattern(files) {
    imports = files |> flatMap(f => extractImports(f))

    match imports {
      i if hasExternalFirstPattern(i) =>
        { ordering: "external-first", grouping: detectGrouping(i) }
      i if hasTypeGrouping(i) =>
        { ordering: "type-grouped", grouping: ["framework", "libraries", "local"] }
      i if isAlphabetized(i) =>
        { ordering: "alphabetized", grouping: "none" }
      _ => { ordering: "unstructured", grouping: "none" }
    }
  }
}
```

### Export Patterns

Recognize module boundary conventions:

- Default exports vs named exports preference
- Barrel files (index.ts re-exports) presence
- Public API definition patterns

### Comment and Documentation Patterns

Identify documentation conventions:

- JSDoc/TSDoc presence and style
- Inline comment frequency and style
- README conventions per module/feature

## Best Practices

```sudolang
PatternRecognitionRules {
  Constraints {
    Follow existing patterns even if imperfect - consistency trumps preference.
    Document deviations explicitly when breaking patterns intentionally.
    Pattern changes require migration - do not introduce without updating existing code.
    Check tests for patterns too - test code reveals expected conventions.
    Prefer explicit over implicit - when unclear, ask or document assumptions.
  }

  require New code matches discovered patterns.
  require Deviations have documented justification.
  require Migration plan exists before introducing new patterns.

  warn Mixed naming conventions in same codebase.
  warn New architectural patterns without team consensus.
  warn Patterns assumed from other projects.
  warn Test patterns ignored when writing implementation.
  warn Special files that do not follow established structure.
}
```

## Anti-Patterns to Avoid

```sudolang
PatternAntiPatterns {
  violations: [
    "Mixing naming conventions in the same codebase",
    "Introducing new architectural patterns without team consensus",
    "Assuming patterns from other projects apply here",
    "Ignoring test patterns when writing implementation",
    "Creating 'special' files that don't follow established structure"
  ]

  checkForViolation(change) {
    match change {
      c if mixesNamingConventions(c) =>
        { violation: true, type: "mixed-naming", action: "Use consistent naming" }
      c if introducesNewPattern(c) and not hasTeamConsensus(c) =>
        { violation: true, type: "unapproved-pattern", action: "Get team approval" }
      c if assumesExternalPatterns(c) =>
        { violation: true, type: "foreign-pattern", action: "Check local conventions" }
      c if ignoresTestPatterns(c) =>
        { violation: true, type: "test-mismatch", action: "Align with test patterns" }
      c if createsSpecialFile(c) =>
        { violation: true, type: "special-file", action: "Follow established structure" }
      _ => { violation: false }
    }
  }
}
```

## References

- `examples/common-patterns.md` - Concrete examples of pattern recognition in action
