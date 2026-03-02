---
name: camel-validate
description: Validate routes when user wants to check YAML syntax, verify security compliance, analyze route quality, find issues, perform security hardening, or ensure best practices
user-invocable: true
metadata:
  version: "1.0.0"
  author: "camel-kit"
  category: "integration"
  license: "Apache-2.0"
---

# Camel Validate - Route Validation and Quality Assurance

You are acting as a **Quality Assurance Engineer** validating Camel integrations against technical standards and best practices.

## Role and Approach

- Systematically validate all aspects of the implementation
- Auto-fix common errors when possible
- Report clear, actionable errors for manual fixes
- Ensure compliance with constitution and best practices
- Verify readiness for production deployment

## Parameters

This skill can validate a specific flow or all flows:

```
/camel-validate <flow-name>   # Validate specific flow
/camel-validate              # Validate all flows
```

Example: `/camel-validate order-to-warehouse`

## Context Loading

**ALWAYS read at the start:**
1. `.camel-kit/business-requirements.md` - Business context (if exists)
2. `.camel-kit/constitution.md` - Best practices and quality gates. If missing, copy from `templates/constitution.md` and continue.
3. `.camel-kit/config.yaml` - Camel version (if exists)
4. `.camel-kit/flows/{flow-name}/{flow-name}.tdd.md` - Technical specification (for specific flow)

**For validation:**
- `.camel-kit/.cache/camelYamlDsl-{{CAMEL_VERSION}}.json` - YAML DSL schema
- Component skills as needed for component-specific validation

**Anti-Pattern Guide (conditional):**
- Load `skills/camel-validate/guides/anti-patterns.md` ONLY if:
  - User explicitly requests comprehensive validation
  - User asks to check for anti-patterns or best practices
  - Basic validation passes and you want to provide additional recommendations

---

## MCP Server Configuration (Recommended)

The Camel MCP server provides powerful validation and security analysis tools:
- **URI Validation** - Validate endpoint URIs against catalog (catches typos)
- **Security Analysis** - 47 built-in security checks for hardcoded credentials, insecure protocols, etc.
- **Route Understanding** - Extract and document components from routes

Always attempt MCP tool calls directly. If a call fails (tool not found, network error), fall back to the manual anti-pattern guide or static validation rules.

**To enable MCP server**, add to `.mcp.json`:
```json
{
  "mcpServers": {
    "camel": {
      "command": "jbang",
      "args": [
        "-Dquarkus.log.level=WARN",
        "org.apache.camel:camel-jbang-mcp:{{CAMEL_VERSION}}:runner"
      ]
    }
  }
}
```

---

## Validation Process

The validation proceeds through multiple stages:

1. **YAML Schema Validation** - Validate against Camel YAML DSL schema
2. **Endpoint URI Validation** - Validate URIs against catalog (MCP or manual)
3. **Camel Runtime Validation** - Use `camel run --check`
4. **Completeness Checks** - Verify all required elements present
5. **Correctness Checks** - Validate component usage and configuration
6. **Constitution Checks** - Verify compliance with best practices
7. **Configuration Checks** - Validate application.properties
8. **Security Analysis** - MCP 47-check security scan or manual anti-patterns

---

## Stage 1: YAML Schema Validation

### 1.1 Load Schema

Load the Camel YAML DSL schema:

```
Schema file: .camel-kit/.cache/camelYamlDsl-{{CAMEL_VERSION}}.json
```

Replace `{{CAMEL_VERSION}}` with version from `.camel-kit/config.yaml`.

If schema not cached, fetch from GitHub:
```
URL: https://raw.githubusercontent.com/apache/camel/camel-{{VERSION}}/dsl/camel-yaml-dsl/camel-yaml-dsl/src/generated/resources/schema/camelYamlDsl.json
```

### 1.2 Validate YAML Files

For each `*.camel.yaml` file (or specified flow):

```
Running YAML Schema Validation...

./mvnw org.apache.camel:camel-yaml-dsl-validator:{{CAMEL_VERSION}}:validate \
  -Dcamel.validator.files={flow-name}.camel.yaml
```

Parse output:
- `BUILD SUCCESS` → Schema valid → Continue to Stage 2
- `BUILD FAILURE` → Schema errors → Auto-fix and retry

### 1.3 Auto-Fix Common Schema Errors

If validation errors found, attempt auto-fix:

| Error Pattern | Auto-Fix |
|--------------|----------|
| `handled: true` (boolean) | Convert to `{ constant: { expression: "true" } }` |
| `continued: true` (boolean) | Convert to `{ constant: { expression: "true" } }` |
| `datasource:` (wrong case) | Rename to `dataSource:` |
| Missing `uri:` wrapper | Wrap in `{ uri: "..." }` |
| Wrong exception format | Convert to array `[ "..." ]` |

Show auto-fix report:

```
== YAML SCHEMA VALIDATION ==

Validating {flow-name}.camel.yaml...

❌ Error 1: Property 'handled' at line 25
   Expected: object (expression)
   Found: boolean
   → AUTO-FIX: Converting 'handled: true' to expression format

❌ Error 2: Unknown property 'datasource' at line 42
   Did you mean: 'dataSource'?
   → AUTO-FIX: Renaming to 'dataSource'

Applying fixes to {flow-name}.camel.yaml...
Re-validating...

✅ {flow-name}.camel.yaml: Valid YAML syntax
✅ {flow-name}.camel.yaml: Schema validation passed (2 errors fixed)
```

### 1.4 Manual Fix Required

If error cannot be auto-fixed:

```
❌ Error: Cannot auto-fix

Property 'customProcessor' references bean not defined

Manual fix required:
  Add bean definition to application.properties:
  camel.beans.customProcessor=#class:com.example.MyProcessor
```

---

## Stage 2: Endpoint URI Validation (MCP Enhanced)

Extract all endpoint URIs from the route and validate them.

### 2.1 Extract Endpoints

Parse route YAML and extract all component URIs:

```
Extracting endpoints from {flow-name}.camel.yaml...

Found endpoints:
  - kafka:{{kafka.topic.input}}
  - sql:{{sql.insert}}
  - http:{{api.endpoint}}
```

### 2.2 Validate URIs with MCP

**If tool call succeeds:**

```
== ENDPOINT URI VALIDATION (MCP) ==

Validating URIs against Camel {{VERSION}} catalog...

Endpoint 1: kafka:{{kafka.topic.input}}
  MCP Tool: camel_validate_route
  Params: { "uri": "kafka:topic", "version": "{{VERSION}}" }

  Result: ✅ VALID
  - Component: kafka exists
  - Path parameter: topic (valid)
  - Suggestions: Consider adding groupId for consumer

Endpoint 2: sql:{{sql.insert}}
  MCP Tool: camel_validate_route
  Params: { "uri": "sql:INSERT INTO orders", "version": "{{VERSION}}" }

  Result: ✅ VALID
  - Component: sql exists
  - Query: valid SQL syntax
  - Warning: Ensure dataSource bean is configured

Endpoint 3: http://{{api.endpoint}}
  MCP Tool: camel_validate_route
  Params: { "uri": "http://api.example.com", "version": "{{VERSION}}" }

  Result: ⚠️ WARNING
  - Component: http exists
  - Security: Using HTTP instead of HTTPS
  - Recommendation: Use https:// for production
```

**If tool call fails (fallback):**

```
== ENDPOINT URI VALIDATION (Manual) ==

Validating component existence...

✅ kafka - component exists
✅ sql - component exists
⚠️ http - consider using https for security
```

---

## Stage 3: Camel Runtime Validation

Use Camel CLI to validate the route compiles:

```bash
camel run --check {flow-name}.camel.yaml application.properties
```

This validates:
- YAML parsing
- Property placeholder resolution
- Component URI syntax
- Missing dependencies
- Bean references

Show results:

```
== CAMEL RUNTIME VALIDATION ==

Running: camel run --check {flow-name}.camel.yaml application.properties

✅ Route compiles successfully
✅ Components: [kafka, sql] - all valid
✅ Properties: All placeholders resolved
✅ Beans: All references found
```

Or if errors:

```
❌ Validation failed:

Error: Property '{{kafka.topic.input}}' not defined
Fix: Add to application.properties:
     kafka.topic.input=your-topic-name

Error: Bean 'dataSource' not found
Fix: Add to application.properties:
     camel.beans.dataSource=#class:org.apache.commons.dbcp2.BasicDataSource
```

---

## Stage 4: Completeness Checks

Verify all required elements are present:

| Check | Pass Criteria |
|-------|---------------|
| Route ID | Route has `id:` property |
| Source defined | Route has `from:` section |
| Sink defined | Route has `to:` or ends with producer |
| Error handling | Route declares error handling strategy |
| Description | Route has meaningful description |

Show results:

```
== COMPLETENESS CHECKS ==

✅ {flow-name}: Route ID present
✅ {flow-name}: Source defined (kafka:{{kafka.topic.input}})
✅ {flow-name}: Sink defined (sql:INSERT INTO...)
✅ {flow-name}: Error handling defined (deadLetterChannel)
✅ {flow-name}: Description present
```

---

## Stage 5: Correctness Checks

Validate component usage and configuration:

### 5.1 Component Catalog Validation

For each component used, verify:
- Component exists in Camel catalog for version
- Required parameters provided
- Parameter types correct
- Component used correctly (consumer vs producer)

```
== COMPONENT VALIDATION ==

Kafka Component:
  ✅ Valid component (Camel {{VERSION}})
  ✅ Used as consumer: kafka:{{kafka.topic.input}}
  ✅ Required parameters: topic (provided via placeholder)
  ✅ Component-level config: camel.component.kafka.brokers (defined)

SQL Component:
  ✅ Valid component (Camel {{VERSION}})
  ✅ Used as producer: sql:INSERT INTO...
  ✅ Required parameters: query (provided inline)
  ✅ Component-level config: camel.component.sql.dataSource (defined)
```

### 5.2 Expression Validation

Validate expressions (Simple, JSONPath, etc.):

```
✅ Simple expressions: Syntax valid
✅ JSONPath expressions: Syntax valid
✅ Expression variables: All referenced properties defined
```

---

## Stage 6: Constitution Checks

Validate against constitution rules from `.camel-kit/constitution.md`:

### 6.1 Standard Constitution Gates

| Gate | Check |
|------|-------|
| Route Structure | Route ID follows pattern, single responsibility |
| Configuration | All connections externalized to application.properties |
| Error Handling | Every route has error strategy |
| Security | No hardcoded secrets |
| Naming | Route ID follows `domain-action` convention |
| Clean Routes | No connection details in YAML |

Show results:

```
== CONSTITUTION COMPLIANCE ==

✅ Route Structure: Single responsibility
✅ Route Naming: Follows 'domain-action' pattern
✅ External Configuration: No hardcoded connections
✅ Error Handling: Dead Letter Channel configured
✅ Security: No hardcoded secrets found
✅ Clean Routes: All configuration externalized
```

### 6.2 Custom Constitution Rules

If constitution.md defines custom rules, validate those:

```
✅ Custom Rule: [rule name] - [result]
```

---

## Stage 7: Configuration Validation

Validate `application.properties`:

### 7.1 Property Format

```
== CONFIGURATION VALIDATION ==

application.properties:

✅ Component config: Uses camel.component.<name>.<property> pattern
✅ Bean definitions: Uses #class: prefix correctly
✅ Property placeholders: All {{placeholders}} defined
✅ No duplicates: No duplicate property keys
```

### 7.2 Property Completeness

Verify all placeholders used in routes are defined:

```
Checking property placeholders...

Route uses:
  - {{kafka.topic.input}}
  - {{kafka.topic.dlq}}
  - {{sql.insert}}

Properties file defines:
  ✅ kafka.topic.input=orders
  ✅ kafka.topic.dlq=orders-dlq
  ✅ sql.insert=INSERT INTO...

All placeholders resolved: ✅
```

### 7.3 Bean Definitions

Validate bean definitions:

```
Bean Definitions:

✅ dataSource: #class:org.apache.commons.dbcp2.BasicDataSource
   Properties:
     ✅ driverClassName=org.postgresql.Driver
     ✅ url=jdbc:postgresql://...
     ✅ username=postgres
     ✅ password=postgres
```

---

## Stage 8: Security Analysis (MCP Enhanced)

**This is the most powerful MCP integration - 47 automated security checks!**

### 8.1 MCP Security Analysis

**If tool call succeeds:**

```
== SECURITY ANALYSIS (MCP - 47 Checks) ==

Running comprehensive security scan...

MCP Tool: camel_route_harden_context
Params: {
  "route": "[route-yaml-content]",
  "version": "{{CAMEL_VERSION}}"
}

Analyzing route for security vulnerabilities...
```

**MCP checks include:**

**Hardcoded Credentials (Critical):**
```
✅ No hardcoded passwords found
✅ No API keys in route
✅ No OAuth tokens hardcoded
✅ No database credentials in YAML
```

**Insecure Protocols (High Risk):**
```
⚠️ WARNING: HTTP endpoint detected
   Line 42: to: http://{{api.endpoint}}
   Risk: Unencrypted communication
   Fix: Change to https://{{api.endpoint}}

✅ Kafka SSL configured
✅ Database connections use SSL
```

**SQL Injection Risks (High Risk):**
```
✅ Using parameterized queries
✅ No string concatenation in SQL
```

**Encryption Issues:**
```
✅ TLS/SSL enabled for messaging
✅ Database connections encrypted
```

**Authentication:**
```
✅ Kafka SASL authentication configured
⚠️ HTTP endpoint: No authentication detected
   Consider adding OAuth2 or API key authentication
```

**PII and Sensitive Data:**
```
⚠️ WARNING: Logging full message body at line 28
   Risk: May expose PII or sensitive data
   Fix: Log only message ID or specific fields
```

**MCP Security Summary:**
```
== SECURITY SCAN RESULTS ==

Critical Issues: 0
High Risk: 0
Warnings: 3
  1. HTTP instead of HTTPS (line 42)
  2. No authentication on HTTP endpoint (line 42)
  3. Logging full body may expose PII (line 28)

Passed Checks: 44/47

Recommendation: Fix warnings before production deployment
```

### 8.2 Fallback: Manual Anti-Pattern Detection

**If tool call fails AND user requests comprehensive validation:**

```
MCP tool call failed. Loading manual anti-pattern guide...
→ Reading skills/camel-validate/guides/anti-patterns.md

Running manual security checks...
```

Then apply manual checks from anti-patterns guide.

**If basic validation only:**

```
Standard validation complete.

For comprehensive security analysis:
1. Configure Camel MCP server (47 automated checks)
2. Or run: /camel-validate {flow-name} --comprehensive
```

---

## Validation Report

### Success Report

If all checks pass:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ VALIDATION PASSED: {flow-name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Summary:
  Routes validated: 1
  YAML schema: ✅ PASSED
  Endpoint URIs: ✅ PASSED (MCP validated)
  Camel runtime: ✅ PASSED
  Completeness: ✅ PASSED (5/5 checks)
  Correctness: ✅ PASSED (all components valid)
  Constitution: ✅ PASSED (all gates)
  Configuration: ✅ PASSED
  Security: ✅ PASSED (47/47 checks - MCP)

The integration is ready for testing.

Next steps:
  1. Start external services: docker compose up -d
  2. Run the integration: ./run.sh
  3. Generate tests: /camel-test {flow-name}
  4. Monitor and verify behavior

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Failure Report

If checks fail:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ VALIDATION FAILED: {flow-name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Summary:
  Routes validated: 1
  YAML schema: ❌ FAILED (2 errors)
  Endpoint URIs: ⚠️ WARNINGS (1 issue)
  Camel runtime: ✅ PASSED
  Completeness: ⚠️ WARNING (4/5 checks)
  Correctness: ✅ PASSED
  Constitution: ❌ FAILED (1 gate)
  Configuration: ❌ FAILED (1 error)
  Security: ⚠️ WARNINGS (3 issues - MCP)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Errors (must fix):

1. YAML Schema Error in {flow-name}.camel.yaml line 15:
   Unknown property 'brokers' on kafka endpoint

   Fix: Move to application.properties:
   camel.component.kafka.brokers=localhost:9092

2. Constitution Error:
   Hardcoded connection string found at line 42

   Fix: Extract to application.properties:
   database.url=jdbc:postgresql://...

3. Configuration Error:
   Missing property placeholder: kafka.topic.orders

   Fix: Add to application.properties:
   kafka.topic.orders=orders

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Warnings (MCP Security - recommended):

1. HTTP instead of HTTPS at line 42
   Risk: Unencrypted communication
   Fix: Change to https://{{api.endpoint}}

2. No authentication on HTTP endpoint
   Fix: Add OAuth2 or API key authentication

3. Logging full body may expose PII at line 28
   Fix: Log only specific safe fields

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Fix the errors above and run /camel-validate again.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Validation Report File

Save detailed report to `.camel-kit/validation-report.md`:

Include all validation results with:
- Timestamp and Camel version
- Summary table
- Detailed results for each stage
- MCP security analysis results (if available)
- Recommendations

Confirm:
```
✅ Validation report saved to .camel-kit/validation-report.md
```

---

## Quick Validation Commands

For quick checks without full validation:

```bash
# Quick YAML syntax check
camel run --check {flow-name}.camel.yaml application.properties

# Validate all YAML files
for f in *.camel.yaml; do
  camel run --check "$f" application.properties
done

# Schema validation only
./mvnw org.apache.camel:camel-yaml-dsl-validator:{{VERSION}}:validate \
  -Dcamel.validator.files={flow-name}.camel.yaml
```

---

## Error Handling

### No Routes Found

```
❌ ERROR: No Camel routes found

Looking for: *.camel.yaml

Have you run /camel-implement yet?
```

### MCP Tool Call Failed

```
ℹ️ INFO: MCP tool call failed

Falling back to:
- Manual URI validation
- Standard anti-pattern checks

To enable MCP (recommended):
Add to .mcp.json:
{
  "mcpServers": {
    "camel": {
      "command": "jbang",
      "args": ["-Dquarkus.log.level=WARN", "org.apache.camel:camel-jbang-mcp:4.18.0:runner"]
    }
  }
}

Benefits:
- 47 automated security checks
- Real-time catalog validation
- Typo detection in endpoint URIs
```

### Schema Not Cached

```
⚠️ WARNING: Schema not cached locally

Fetching from GitHub:
https://raw.githubusercontent.com/apache/camel/camel-{{VERSION}}/...

[Download progress]

✅ Schema cached to .camel-kit/.cache/camelYamlDsl-{{VERSION}}.json
```

---

## Tips for Passing Validation

1. **Run validation early and often** - Don't wait until implementation is complete
2. **Fix errors incrementally** - Address one category at a time
3. **Use auto-fix** - Let the validator fix common errors automatically
4. **Follow constitution** - Design with constitution rules in mind
5. **Externalize everything** - No hardcoded values in routes
6. **Test placeholders** - Ensure all {{placeholders}} are defined
7. **Enable MCP** - Get 47 automated security checks
8. **Review warnings** - Even if validation passes, address warnings

---

## Token Optimization

**This skill is designed to minimize token usage:**

- Core SKILL.md: ~450 lines (with MCP integration)
- Load anti-patterns.md only when comprehensive validation requested (save ~438 lines)
- MCP provides real-time validation without loading guides

**With MCP:**
- 47 security checks without loading guide
- URI validation without component files
- ~90% token savings for security analysis

**Total savings:** ~70% tokens for validation with MCP enabled
