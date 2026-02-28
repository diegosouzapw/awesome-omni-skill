---
name: update-codeql-query-dataflow-csharp
description: Upgrade C# CodeQL queries from legacy (v1) language-specific dataflow API to modern (v2) shared dataflow API while ensuring query result equivalence through test-driven development. Use this skill when modernizing C# dataflow queries to use the unified dataflow library.
---

# Update CodeQL Query Dataflow for C#

This skill guides you through migrating C# CodeQL queries from the legacy v1 (language-specific) dataflow API to the modern v2 (shared) dataflow API while ensuring query result equivalence.

## When to Use This Skill

- Migrating legacy C# queries using `DataFlow::Configuration` to modern `DataFlow::ConfigSig`
- Updating queries to use the shared dataflow library introduced in 2023
- Modernizing C# queries that use deprecated dataflow patterns
- Ensuring migrated queries produce identical results as the original

## Critical Success Factor: Query Result Equivalence

The most important outcome is ensuring the migrated query produces **exactly the same results** as the original query. Result changes due to query migration alone can cause:

- **Alert flapping**: Issues appearing and disappearing without code changes
- **False confidence**: Developers may lose trust in CodeQL analysis
- **Deployment issues**: CI/CD pipelines may fail due to new/changed alerts

**Test-Driven Development (TDD) is mandatory** for dataflow migration to guarantee result equivalence.

## Overview: v1 vs v2 Dataflow API

### Legacy v1 API (Language-Specific)

- **Configuration**: Uses `DataFlow::Configuration` classes
- **Predicates**: `isSource()`, `isSink()`, `isSanitizer()`, `isAdditionalTaintStep()`
- **Library**: Language-specific dataflow library (e.g., `semmle.code.csharp.dataflow.DataFlow`)
- **Nodes**: Language-specific node types (e.g., `ExprNode`, `ParameterNode`)

### Modern v2 API (Shared)

- **Configuration**: Uses `DataFlow::ConfigSig` signature modules with `DataFlow::Global<ConfigSig>`
- **Predicates**: `isSource()`, `isSink()`, `isBarrier()`, `isAdditionalFlowStep()`
- **Library**: Shared dataflow library (e.g., `semmle.code.csharp.dataflow.DataFlow2`)
- **Nodes**: Unified `DataFlow::Node` type with consistent semantics

### Key Differences

| Aspect        | v1 API                                                        | v2 API                                                                                            |
| ------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Configuration | `class Config extends DataFlow::Configuration`                | `module ConfigSig implements DataFlow::ConfigSig` + `module Config = DataFlow::Global<ConfigSig>` |
| Sanitizer     | `isSanitizer(DataFlow::Node node)`                            | `isBarrier(DataFlow::Node node)`                                                                  |
| Custom flow   | `isAdditionalTaintStep(DataFlow::Node n1, DataFlow::Node n2)` | `isAdditionalFlowStep(DataFlow::Node n1, DataFlow::Node n2)`                                      |
| Flow check    | `config.hasFlow(source, sink)`                                | `Config::flow(source, sink)`                                                                      |
| Flow path     | `config.hasFlowPath(source, sink)`                            | `Config::flowPath(source, sink)`                                                                  |

## Migration Workflow with TDD

### Phase 1: Establish Baseline (Pre-Migration)

#### Step 1: Understand Query and Capture Baseline

Review the existing v1 query: metadata (`@name`, `@kind`, `@id`), sources, sinks, barriers, and custom flow steps.

Run existing tests with `codeql_test_run` and **capture baseline results** - the migrated query must match these exactly:

```json
{
  "testPath": "<query-pack>/test/{QueryName}",
  "searchPath": ["<query-pack>"]
}
```

#### Step 2: Ensure Comprehensive Test Coverage

Add test cases for positive (vulnerable), negative (safe), and edge cases (sanitization, async/await, LINQ, properties). Update `.expected` file, then extract and run tests:

```json
{ "testPath": "<query-pack>/test/{QueryName}", "searchPath": ["<query-pack>"] }
```

#### Step 3: Create v2 Query Structure

Create new query file (or backup original and modify in place):

**Import v2 dataflow library**:

```ql
import csharp
import semmle.code.csharp.dataflow.TaintTracking
import semmle.code.csharp.security.dataflow.SqlInjectionQuery
```

**Define configuration signature**:

```ql
/**
 * Configuration for detecting [vulnerability description]
 */
module MyConfigSig implements DataFlow::ConfigSig {
  predicate isSource(DataFlow::Node source) {
    // Define sources - user-controllable input
    source instanceof RemoteFlowSource or
    exists(Parameter p |
      p.fromSource() and
      source.asParameter() = p
    )
  }

  predicate isSink(DataFlow::Node sink) {
    // Define sinks - dangerous operations
    exists(MethodCall call |
      call.getTarget().hasName("FromSqlRaw") and
      call.getAnArgument() = sink.asExpr()
    )
  }

  predicate isBarrier(DataFlow::Node node) {
    // Define barriers - sanitization/validation
    // Renamed from isSanitizer in v1
    node instanceof SanitizerGuard or
    exists(MethodCall sanitize |
      sanitize.getTarget().hasName(["Sanitize", "Validate", "Encode"]) and
      DataFlow::localFlow(sanitize, node)
    )
  }

  predicate isAdditionalFlowStep(DataFlow::Node node1, DataFlow::Node node2) {
    // Define custom flow steps
    // Renamed from isAdditionalTaintStep in v1
    // Example: flow through string formatting
    exists(MethodCall format |
      format.getTarget().hasName("Format") and
      format.getAnArgument() = node1.asExpr() and
      format = node2.asExpr()
    )
  }
}
```

**Instantiate global flow module**:

```ql
module MyConfig = TaintTracking::Global<MyConfigSig>;
```

**Update query select**:

```ql
from DataFlow::Node source, DataFlow::Node sink
where MyConfig::flow(source, sink)
select sink, "Dataflow from $@ to sink", source, "user input"
```

For path queries (`@kind path-problem`), use `PathNode` from the instantiated module:

```ql
import DataFlow::PathGraph

from MyConfig::PathNode source, MyConfig::PathNode sink
where MyConfig::flowPath(source, sink)
select sink.getNode(), source, sink, "Dataflow from $@ to sink", source.getNode(), "user input"
```

#### Step 4: Apply C#-Specific Migration Patterns

**RemoteFlowSource** (v2 pattern):

```ql
// v1 pattern
source.asExpr().(Parameter).fromSource()

// v2 pattern
source instanceof RemoteFlowSource
```

**Barrier Guards** (renamed from sanitizers):

```ql
// v1 pattern
predicate isSanitizer(DataFlow::Node node) {
  // guard logic
}

// v2 pattern
predicate isBarrier(DataFlow::Node node) {
  // same guard logic
}
```

**Library Extensions** (C#-specific):

```ql
// For custom library dataflow in C#
import semmle.code.csharp.dataflow.LibraryTypeDataFlow

class MyLibraryFlow extends LibraryTypeDataFlow {
  override predicate callableFlow(
    CallableFlowSource source,
    CallableFlowSink sink,
    SourceDeclarationCallable c,
    boolean preservesValue
  ) {
    // Define flow through library methods
  }
}
```

**ASP.NET Patterns**:

```ql
// Web input sources
source.asExpr() instanceof WebInput or
exists(Parameter p |
  p.getAnAttribute().getType().hasName("FromBodyAttribute") and
  source.asParameter() = p
)

// Controller action sinks
exists(MethodCall redirect |
  redirect.getTarget().hasName(["Redirect", "RedirectToAction"]) and
  sink.asExpr() = redirect.getAnArgument()
)
```

#### Step 5: Compile and Test

Compile with `codeql_query_compile`, then run tests with `codeql_test_run` to validate result equivalence.

#### Step 6: Debug and Refine

If tests fail, analyze differences: missing sources (v2 `RemoteFlowSource` coverage), barrier semantics, flow steps, or node conversions. Use `codeql_query_run` with PrintAST for debugging. Iterate until tests pass.

#### Step 7: Finalize

Once tests pass: extract helper predicates, update QLDoc comments, format with `codeql_query_format`, install pack dependencies with `codeql_pack_install`, and run full test suite to ensure no regressions.

## C#-Specific Considerations

### ASP.NET Core Patterns

```ql
// Controller actions as sources
exists(Method m, Parameter p |
  m.getDeclaringType().getABaseType*().hasName("Controller") and
  p = m.getAParameter() and
  source.asParameter() = p
)
```

### Entity Framework Operations

```ql
// Database sinks
exists(MethodCall call |
  call.getTarget().hasName(["FromSqlRaw", "ExecuteSqlRaw", "ExecuteSqlCommand"]) and
  sink.asExpr() = call.getAnArgument()
)
```

### LINQ Expressions

```ql
// Flow through LINQ
exists(LambdaExpr lambda |
  node1.asExpr() = lambda.getAParameter() and
  node2.asExpr() = lambda.getExpressionBody()
)
```

### Async/Await Patterns

```ql
// Async method flow
exists(AwaitExpr await |
  node1.asExpr() = await.getExpr() and
  DataFlow::localFlow(await, node2)
)
```

## MCP Tools Reference

- **`codeql_test_extract`**: Extract test databases from C# code
- **`codeql_test_run`**: Run query tests and validate result equivalence
- **`codeql_query_compile`**: Compile queries and check for v2 API syntax errors
- **`codeql_query_format`**: Format migrated query code
- **`codeql_query_run`**: Run PrintAST for debugging AST structure
- **`codeql_pack_install`**: Install pack dependencies
- **`codeql_bqrs_decode`**: Decode query results for analysis

## Common Migration Pitfalls

❌ **Don't:**

- Migrate without establishing comprehensive baseline tests
- Accept test differences without understanding root cause
- Skip testing edge cases (async, LINQ, properties)
- Ignore custom flow steps that may need adjustment
- Assume v1 and v2 semantics are identical
- Forget to update query metadata and documentation

✅ **Do:**

- Create extensive test coverage before migration
- Verify exact result equivalence after migration
- Test all C# language features (async, LINQ, properties, etc.)
- Review official migration guides and documentation
- Use TDD to catch regressions early
- Document migration decisions in code comments
- Format code consistently with `codeql_query_format`

## Quality Checklist

Before considering migration complete:

- [ ] Baseline test results captured from original v1 query
- [ ] Comprehensive test coverage (positive, negative, edge cases)
- [ ] All v1 imports replaced with v2 imports
- [ ] `DataFlow::Configuration` replaced with `DataFlow::ConfigSig`
- [ ] `isSanitizer` renamed to `isBarrier`
- [ ] `isAdditionalTaintStep` renamed to `isAdditionalFlowStep`
- [ ] `hasFlow`/`hasFlowPath` replaced with `flow`/`flowPath`
- [ ] Query compiles without errors or warnings
- [ ] All tests pass with exact result match to baseline
- [ ] No regressions in other queries (full test suite passes)
- [ ] Query properly formatted with `codeql_query_format`
- [ ] QLDoc comments updated to reflect v2 API
- [ ] C#-specific patterns properly migrated
- [ ] Performance is acceptable (similar to v1 query)

## Official Documentation

For detailed v2 API information, consult:

- [New dataflow API for writing custom CodeQL queries](https://github.blog/changelog/2023-08-14-new-dataflow-api-for-writing-custom-codeql-queries/) - Official announcement and migration guide
- [Analyzing data flow in C#](https://codeql.github.com/docs/codeql-language-guides/analyzing-data-flow-in-csharp/) - C#-specific dataflow documentation

## Related Skills

- [Create CodeQL Query Unit Test for C#](../create-codeql-query-unit-test-csharp/SKILL.md) - For creating comprehensive tests
- [Test-Driven CodeQL Query Development](../create-codeql-query-tdd-generic/SKILL.md) - TDD methodology for queries

## Success Criteria

Your dataflow migration is successful when:

1. ✅ All tests pass with exact result equivalence to v1 baseline
2. ✅ Query uses v2 API consistently (`ConfigSig`, `isBarrier`, `isAdditionalFlowStep`)
3. ✅ No deprecated v1 patterns remain
4. ✅ All C#-specific patterns properly migrated
5. ✅ Query compiles without warnings
6. ✅ Full test suite passes (no regressions)
7. ✅ Code is clean, formatted, and well-documented
8. ✅ Performance is acceptable for production use
