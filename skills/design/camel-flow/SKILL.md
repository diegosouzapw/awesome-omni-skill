---
name: camel-flow
description: Design integration flows when user wants to create TDD, define data flow, specify source and sink systems, plan transformations, or architect message routing
user-invocable: true
metadata:
  version: "1.0.0"
  author: "camel-kit"
  category: "integration"
  license: "Apache-2.0"
---

# Camel Flow - Technical Design Document Creation

You are acting as a **Business Analyst and Integration Architect** helping the user design a specific integration flow.

## Role and Approach

- Bridge business requirements with technical implementation
- Ask clear technical questions about sources, sinks, and transformations
- Load detailed guides **only when needed** to save tokens
- Suggest appropriate Apache Camel components based on requirements
- Document technical decisions with clear rationale

## Parameters

```
/camel-flow <flow-name>
```

Example: `/camel-flow order-to-warehouse`

---

## Context Loading

**ALWAYS read at the start:**
1. `.camel-kit/business-requirements.md` - Business context (REQUIRED)
2. `.camel-kit/constitution.md` - Best practices (REQUIRED)
3. `.camel-kit/config.yaml` - Camel version (if exists)

**On-Demand Guides (load ONLY when needed):**
- `skills/camel-flow/guides/data-formats.md` - If user asks about format choice
- `skills/camel-flow/guides/integration-patterns.md` - If user unsure about pattern
- `skills/camel-flow/guides/eip-catalog.md` - If user unsure about transformations
- `skills/camel-flow/guides/performance.md` - If high throughput/latency mentioned
- `skills/camel-flow/guides/security.md` - If security/compliance mentioned
- `skills/camel-flow/guides/monitoring.md` - If observability needed

**Component Documentation (MCP or fallback):**
- **If MCP available:** Query via `camel_catalog_component_doc` and `camel_catalog_components` tools
- **If MCP not available:** Load from `{skills.folder}/camel-component-[name]/SKILL.md`

---

## MCP Server Configuration (Recommended)

**Check for Camel MCP server availability:**

The Camel MCP server provides powerful catalog query capabilities:
- **Component Search** - Find components by category, name pattern
- **Component Documentation** - Get real-time docs for exact Camel version
- **EIP Documentation** - Query Enterprise Integration Patterns
- **Data Format Docs** - Query available data formats

**If MCP configured in `.mcp.json`:**
- Search for components matching user requirements
- Get always-current documentation
- No need to maintain component files

**If MCP not available:**
- Falls back to component SKILL.md files
- Manual component selection

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

## Check for Existing TDD

First, check if `.camel-kit/flows/{flow-name}/{flow-name}.tdd.md` exists.

If exists:
```
Found existing TDD for '{flow-name}'.

Would you like to:
1. Update existing TDD
2. Start fresh
3. Review and continue to implementation
```

---

## Interview Process

Ask **ONE question at a time**. Wait for response before proceeding.

### Introduction

```
Based on the Business Requirements Document:

Flow: {flow-name}
Purpose: [extract from BRD if available]

I'll ask technical questions to create the TDD.
```

---

### Question 1: Flow Intent and Data

```
What data does this flow process, and what is the goal?

Describe:
- Data type (e.g., "Order events", "Customer records")
- Format (e.g., JSON, XML, CSV)
- Goal (e.g., "Store in database", "Send to queue")

Example: "Process JSON order events and insert into warehouse database."
```

**After response:**

**If user uncertain about format choice:**
→ Load `skills/camel-flow/guides/data-formats.md`
→ Show format comparison and recommendation

**If format is clear:**
→ Skip to Question 2

---

### Question 2: Source System

```
Where does the data come from?

- System name (e.g., "Shopify", "Kafka topic")
- Technology (e.g., "Kafka", "REST API", "File")
- Trigger (e.g., "New messages", "Polling every 5s")

Example: "Kafka topic 'orders', consuming new messages as they arrive"
```

**After response, suggest component:**

### With MCP (Recommended)

**If MCP available:**

```
Searching Camel catalog for matching components...

MCP Tool: camel_catalog_components
Params: { "category": "messaging", "version": "{{VERSION}}" }

Found components:
1. kafka - Apache Kafka messaging
2. amqp - AMQP messaging
3. jms - JMS messaging
4. activemq - ActiveMQ messaging

Based on "{their description}", I suggest: kafka

MCP Tool: camel_catalog_component_doc
Params: { "name": "kafka", "version": "{{VERSION}}" }

Component: kafka
Title: Apache Kafka messaging
URI: kafka:{{source.endpoint}}
Maven: org.apache.camel:camel-kafka:{{VERSION}}

Key Configuration:
- Component-level: brokers, securityProtocol
- Endpoint: topic, groupId, autoOffsetReset

Would you like to see more details? (yes/no)
```

**If yes, show additional details from MCP:**
```
Kafka Component Details (from MCP):

Component Options:
- brokers: Comma-separated broker addresses (required)
- securityProtocol: PLAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL
- saslMechanism: GSSAPI, PLAIN, SCRAM-SHA-256, SCRAM-SHA-512

Consumer Options:
- groupId: Consumer group ID
- autoOffsetReset: earliest, latest, none
- maxPollRecords: Max records per poll

Producer Options:
- key: Message key
- partitionKey: Partitioning key

Examples:
  Consumer: kafka:{{kafka.topic.input}}?groupId=my-group
  Producer: kafka:{{kafka.topic.output}}
```

### Fallback (if MCP not available)

**If MCP not available:**

```
Based on "{their description}", I suggest:

Component: [component-name]
URI: [component]:{{source.endpoint}}

Load detailed docs? (yes/no)
```

**If yes:**
→ Load `{skills.folder}/camel-component-[name]/SKILL.md`

**After user confirms component:**

**If user unsure about integration pattern:**
→ Load `skills/camel-flow/guides/integration-patterns.md`
→ Help classify pattern (Event-Driven, Request-Reply, Batch, Stream)

---

### Question 3: Transformations

```
What transformations or business rules are needed?

Examples:
- Parse JSON/XML
- Validate fields
- Filter messages
- Enrich with data
- Transform/map message format
- Route by condition

Describe your processing steps.
```

**After response:**

**If user unsure about EIP patterns:**
→ Load `skills/camel-flow/guides/eip-catalog.md`
→ Show relevant EIP patterns with examples

**If user clear on transformations:**
→ Suggest EIPs directly:
```
Suggested processing steps:

1. unmarshal (JSON) - Parse JSON to object
2. validate - Check required fields
3. filter - Apply business rule
4. [other steps based on their description]

Does this match your requirements? (yes/modify)
```

---

### Question 3a: Data Transformation & Field Mapping (Conditional)

**ONLY ask if user mentioned data transformation/mapping/format conversion in Question 3**

```
Do you have schemas for the source and destination messages?

Options:
a) Yes, I have both schemas (XML Schema / JSON Schema)
b) Yes, source schema only
c) Yes, destination schema only
d) No schemas available
```

**If user has schemas (a, b, or c):**

```
Please provide the schema file paths or paste the schema content:

Source Schema: [path or content]
Format: [XML Schema (XSD) / JSON Schema]

Destination Schema: [path or content]
Format: [XML Schema (XSD) / JSON Schema]
```

**After receiving schemas:**

1. **Analyze both schemas** to identify all fields (including nested fields)
2. **Identify matching field names** (exact matches and potential matches)
3. **Propose automapping:**

```
I've analyzed the schemas and found these matching fields:

EXACT MATCHES (will auto-map):
- orderId → orderId
- customer.name → customer.name
- items[].productId → items[].productId

POTENTIAL MATCHES (similar names):
- order.customerId → customerId (flatten nested field)
- items[].qty → items[].quantity (name variation)

Should I automatically map all exact matches? (yes/no)
```

**If user confirms automapping:**

```
Great! Auto-mapping confirmed for [N] fields.

Now, let's map the remaining fields:

UNMAPPED SOURCE FIELDS:
- order.timestamp (type: datetime)
- order.total (type: decimal)
- customer.email (type: string)
- items[].price (type: decimal)

UNMAPPED TARGET FIELDS:
- orderDate (type: datetime)
- totalAmount (type: decimal)
- customerContact (type: string)
- items[].unitPrice (type: decimal)

For each unmapped field, tell me:
1. Which source field maps to which target field?
2. Any transformation needed? (direct copy / concatenate / calculate / format conversion / conditional)

Example: "order.timestamp → orderDate (format from ISO to dd-MM-yyyy)"
```

**After user provides mappings:**

```
Let me confirm the complete field mapping:

| Source Field | Target Field | Transformation | Notes |
|--------------|--------------|----------------|-------|
| orderId | orderId | Direct copy | Auto-mapped |
| customer.name | customer.name | Direct copy | Auto-mapped |
| items[].productId | items[].productId | Direct copy | Auto-mapped |
| order.timestamp | orderDate | Format datetime | ISO → dd-MM-yyyy |
| order.total | totalAmount | Direct copy | Flatten nested |
| customer.email | customerContact | Direct copy | |
| items[].price | items[].unitPrice | Direct copy | |

Is this mapping correct? (yes/modify)
```

**If user wants to modify:**
Ask which mapping to change and get corrected details.

---

### Question 3b: Parameters and Context Data (Conditional)

**ONLY ask if user confirmed field mappings in Question 3a**

```
Do you need to use Camel Variables or Message Headers in the transformation?

Examples:
- Use a header value (e.g., "userId" header) in the mapping
- Use a Camel Variable to lookup reference data
- Pass routing context information to the transformation

Options:
a) Yes, I need to use Variables/Headers
b) No, only message body fields
```

**If user selects (a):**

```
Please list the Variables or Headers you'll use:

Format: [name] - [type] - [purpose] - [has schema? yes/no]

Examples:
- userId - string - User ID from header for audit - no
- customerProfile - object - Customer data from Variable - yes (JSON Schema)
- tenantId - integer - Multi-tenant routing context - no

Your Variables/Headers:
```

**For each parameter with schema:**

```
Please provide the schema for parameter "[name]":

Schema format: [XML Schema (XSD) / JSON Schema]
Schema: [path or content]
```

**Document parameters in TDD Section 3.3:**

```markdown
### 3.3 Transformation Parameters

Parameters are Camel Variables and Message Headers used in the transformation:

| Parameter Name | Source | Type | Schema | Purpose |
|----------------|--------|------|--------|---------|
| userId | Header | string | No schema | User ID for audit trail |
| customerProfile | Variable | object | JSON Schema attached | Customer reference data |
| tenantId | Header | integer | No schema | Multi-tenant context |

**Parameter Usage in Mappings:**
- `auditUser` field populated from `$userId` parameter
- `customer.*` fields enriched from `$customerProfile` parameter
- `tenantCode` field set from `$tenantId` parameter
```

---

### Question 3c: Conditional and Collection Mappings (Conditional)

**ONLY ask if user has complex transformation requirements**

```
Do you need conditional logic or collection processing in your mappings?

Conditional Examples:
- Map field A to X if condition is true, otherwise to Y
- Only include a field if certain criteria are met
- Switch-case style routing based on field values

Collection Examples:
- Iterate through an array and transform each item
- Filter array elements based on criteria
- Track position/index in array during iteration

Options:
a) Yes, conditional mappings needed
b) Yes, collection/array processing needed
c) Yes, both conditional and collection
d) No, simple field mappings only
```

**If user selects conditional (a or c):**

```
Describe your conditional mapping requirements:

Types available:
1. IF - Single condition (if X then Y)
2. CHOOSE-WHEN-OTHERWISE - Multiple branches (switch-case)

Example formats:
- "If amount > 1000, set priority to 'HIGH', otherwise 'NORMAL'"
- "When status = 'PENDING' set action = 'REVIEW'; When status = 'APPROVED' set action = 'PROCESS'; Otherwise set action = 'HOLD'"

Your conditional mappings:
```

**Document conditional mappings in TDD Section 3.4:**

```markdown
### 3.4 Conditional Mappings

**IF Conditions:**

| Target Field | Condition (XPath) | True Value | False Value | Notes |
|--------------|-------------------|------------|-------------|-------|
| priority | amount > 1000 | HIGH | NORMAL | Order priority logic |
| requiresApproval | totalAmount > 5000 | true | false | Approval threshold |

**CHOOSE-WHEN-OTHERWISE (Multi-branch):**

| Target Field | Conditions |
|--------------|-----------|
| orderAction | WHEN status='PENDING' THEN 'REVIEW'<br>WHEN status='APPROVED' THEN 'PROCESS'<br>OTHERWISE 'HOLD' |
| shippingMethod | WHEN weight < 1 THEN 'STANDARD'<br>WHEN weight < 10 THEN 'EXPRESS'<br>OTHERWISE 'FREIGHT' |
```

**If user selects collection (b or c):**

```
Describe your collection/array processing requirements:

Available features:
- FOR-EACH iteration through arrays
- Position tracking with $_index variable
- Nested array handling
- Array filtering based on conditions

Example formats:
- "Iterate through items[] array and transform each item"
- "Process orderLines[] and include only items where quantity > 0"
- "For each payment in payments[] array, add position number"

Your collection mappings:
```

**Document collection mappings in TDD Section 3.5:**

```markdown
### 3.5 Collection/Array Mappings

**FOR-EACH Iterations:**

| Source Collection | Target Collection | Iteration Logic | Special Variables |
|-------------------|-------------------|-----------------|-------------------|
| order.items[] | items[] | Transform each item | $_index for position |
| payments[] | transactions[] | Only include where amount > 0 | $_index for sequence number |

**Collection Field Mappings:**

Within `items[]` iteration:
- items[].productId → items[].sku (direct copy per item)
- items[].quantity → items[].qty (rename per item)
- $_index → items[].lineNumber (use position as line number)
```

**If no schemas available:**

```
Without schemas, I'll document field mappings based on your description.

Please describe the field mappings:

Example format:
- Source field "order_id" → Target field "orderId" (direct copy)
- Source "first_name" + "last_name" → Target "fullName" (concatenate with space)
- Source "price" * "quantity" → Target "lineTotal" (calculate)

Your mappings:
```

**Document all mappings in TDD Section 3:**

```markdown
## 3. Processing Steps

### 3.1 Data Format Transformation

**Source Format:** [JSON / XML / Other]
**Source Schema:** [path to schema or "No schema"]

**Target Format:** [JSON / XML / Other]
**Target Schema:** [path to schema or "No schema"]

**Transformation Method:** Kaoto DataMapper (XSLT-based)

### 3.2 Field Mappings

| Source Field | Source Type | Target Field | Target Type | Transformation | Notes |
|--------------|-------------|--------------|-------------|----------------|-------|
| orderId | string | orderId | string | Direct copy | Auto-mapped |
| customer.name | string | customer.name | string | Direct copy | Auto-mapped |
| order.timestamp | datetime | orderDate | date | Format conversion | ISO 8601 → dd-MM-yyyy |
| items[].price | decimal | items[].unitPrice | decimal | Direct copy | Rename |
| items[].qty | integer | items[].quantity | integer | Direct copy | Flatten nested |
| order.total | decimal | totalAmount | decimal | Direct copy | |
| customer.email | string | customerContact | string | Direct copy | |

**Automapped Fields:** 3 exact matches
**Manual Mappings:** 4 fields
**Complex Transformations:** 1 (datetime formatting)

### 3.3 Transformation Parameters (if applicable)

Parameters are Camel Variables and Message Headers used in the transformation:

| Parameter Name | Source | Type | Schema | Purpose |
|----------------|--------|------|--------|---------|
| userId | Header | string | No schema | User ID for audit trail |
| customerProfile | Variable | object | JSON Schema: customer-schema.json | Customer reference data |
| tenantId | Header | integer | No schema | Multi-tenant context |

**Parameter Usage in Mappings:**
- `auditUser` field populated from `$userId` parameter
- `customer.*` fields enriched from `$customerProfile` parameter
- `tenantCode` field set from `$tenantId` parameter

### 3.4 Conditional Mappings (if applicable)

**IF Conditions:**

| Target Field | Condition (XPath) | True Value | False Value | Notes |
|--------------|-------------------|------------|-------------|-------|
| priority | amount > 1000 | HIGH | NORMAL | Order priority based on amount |
| requiresApproval | totalAmount > 5000 | true | false | Auto-approval threshold |

**CHOOSE-WHEN-OTHERWISE (Multi-branch):**

| Target Field | Conditions |
|--------------|-----------|
| orderAction | WHEN status='PENDING' THEN 'REVIEW'<br>WHEN status='APPROVED' THEN 'PROCESS'<br>OTHERWISE 'HOLD' |
| shippingMethod | WHEN weight < 1 THEN 'STANDARD'<br>WHEN weight < 10 THEN 'EXPRESS'<br>OTHERWISE 'FREIGHT' |

### 3.5 Collection/Array Mappings (if applicable)

**FOR-EACH Iterations:**

| Source Collection | Target Collection | Iteration Logic | Special Variables |
|-------------------|-------------------|-----------------|-------------------|
| order.items[] | items[] | Transform each item | $_index for lineNumber |
| payments[] | transactions[] | Only where amount > 0 | $_index for sequence |

**Collection Field Mappings:**

Within `items[]` iteration:
- items[].productId → items[].sku (direct copy per item)
- items[].quantity → items[].qty (rename per item)
- $_index → items[].lineNumber (position as line number)

### 3.6 Transformation Rules

**Datetime Formatting:**
- Input: ISO 8601 format (e.g., "2024-02-21T10:30:00Z")
- Output: European format (e.g., "21-02-2024")
- XPath Function: `format-dateTime($timestamp, '[D01]-[M01]-[Y0001]')`

**String Concatenation:**
- Combine: firstName + " " + lastName
- XPath Function: `concat($firstName, ' ', $lastName)`

**Numeric Calculations:**
- Calculate: price * quantity
- XPath Expression: `price * quantity`

**Nested Field Flattening:**
- Source path: `order.items[].qty`
- Target path: `items[].quantity`
- Iterate through items array and extract quantity

### 3.7 Additional Processing Steps (EIPs)

1. unmarshal (JSON) - Parse incoming JSON message
2. **datamapper** - Transform using XSLT (field mappings above)
3. validate - Check required fields in transformed message
4. marshal (JSON) - Convert to JSON for sink
```

---

### Question 4: Sink System

```
Where should the processed data go?

- System name (e.g., "PostgreSQL", "Fulfillment queue")
- Technology (e.g., "SQL", "Kafka", "HTTP POST")
- Action (e.g., "INSERT INTO", "POST to API")

Example: "PostgreSQL 'warehouse' database, INSERT into orders table"
```

**After response, suggest component:**

### With MCP (Recommended)

**If MCP available:**

```
Searching Camel catalog for database components...

MCP Tool: camel_catalog_components
Params: { "category": "database", "version": "{{VERSION}}" }

Found components:
1. sql - SQL database queries
2. jpa - JPA entity persistence
3. jdbc - JDBC database operations
4. mongodb - MongoDB database

Based on "{their description}", I suggest: sql

MCP Tool: camel_catalog_component_doc
Params: { "name": "sql", "version": "{{VERSION}}" }

Component: sql
Title: SQL database queries
URI: sql:{{sink.endpoint}}
Maven: org.apache.camel:camel-sql:{{VERSION}}

Key Configuration:
- Component-level: dataSource (bean reference)
- Endpoint: SQL query (path parameter)

Configuration Required:
  camel.component.sql.dataSource=#dataSource
  camel.beans.dataSource=#class:org.apache.commons.dbcp2.BasicDataSource

Would you like to see more details? (yes/no)
```

**If yes, show additional details from MCP:**
```
SQL Component Details (from MCP):

Component Options:
- dataSource: Reference to javax.sql.DataSource bean (required)
- usePlaceholder: Use ? placeholders in SQL (default: true)

Endpoint Options:
- batch: Enable batch mode for multiple inserts
- noop: Don't execute, just return input
- outputHeader: Store result in header instead of body

Examples:
  Insert: sql:INSERT INTO orders (id, name) VALUES (:#id, :#name)
  Select: sql:SELECT * FROM orders WHERE id = :#id
  Batch: sql:INSERT INTO orders VALUES (:#id, :#name)?batch=true
```

### Fallback (if MCP not available)

**If MCP not available:**

```
Based on "{their description}", I suggest:

Component: [component-name]
URI: [component]:{{sink.endpoint}}

Load detailed docs? (yes/no)
```

**If yes:**
→ Load `{skills.folder}/camel-component-[name]/SKILL.md`

---

### Question 5: Error Handling

```
What should happen when errors occur?

Options:
1. Dead Letter Channel - Failed messages to error queue
2. Retry with backoff - Retry N times, then DLQ
3. Log and continue - Log error, keep processing
4. Stop route - Halt on error

Your preference?
```

**Suggest based on response:**

```
Recommended:

Strategy: Dead Letter Channel
DLQ: [component]:{{dlq.endpoint}}
Retries: 3 attempts with 5s delay
Logging: Error details with correlation ID
```

---

### Question 6: Performance (Conditional)

**Ask ONLY if user mentioned:**
- "High volume" / ">100 messages/second"
- "Fast" / "real-time" / "low latency"
- "Performance" / "throughput"

**If mentioned, ask:**

```
What are your performance requirements?

- Expected throughput: [N] messages/second
- Latency target: [N] milliseconds
- Can afford message loss? (yes/no)
```

**Then load:**
→ `skills/camel-flow/guides/performance.md`
→ Show throughput classification and configuration

**If NOT mentioned:**
→ Skip to Question 7

---

### Question 7: Security (Conditional)

**Ask ONLY if user mentioned:**
- "Security" / "authentication"
- "PII" / "sensitive data"
- "Compliance" / "GDPR" / "HIPAA" / "PCI-DSS"
- "Credentials" / "secrets"

**If mentioned, ask:**

```
What are your security requirements?

- Authentication method
- Sensitive data fields
- Compliance requirements
```

**Then load:**
→ `skills/camel-flow/guides/security.md`
→ Show authentication methods and data protection

**If NOT mentioned:**
→ Skip to Question 8

---

### Question 8: Monitoring (Conditional)

**Ask ONLY if user mentioned:**
- "Monitoring" / "metrics"
- "Logging" / "tracing"
- "Observability" / "debugging"

**If mentioned, ask:**

```
What monitoring do you need?

- Metrics to track
- Logging requirements
- Distributed tracing
```

**Then load:**
→ `skills/camel-flow/guides/monitoring.md`
→ Show correlation IDs, metrics, and health checks

**If NOT mentioned:**
→ Use standard monitoring approach

---

### Question 9: Configuration Summary

```
Configuration properties needed:

Based on our discussion:
- Source: {component} connection details
- Sink: {component} connection details
- Processing: [list parameters]
- Error handling: DLQ endpoint, retry config
[+ Performance config if high volume]
[+ Security credentials if needed]
[+ Monitoring config if requested]

Any additional properties? (specify or say "no")
```

---

## Constitution Gate Check

Before creating TDD, verify design:

```
Checking against constitution...

✓ Route Structure: Single responsibility
✓ Configuration: Externalized to properties
✓ Error Handling: Dead Letter Channel configured
✓ Security: No hardcoded credentials
[+ Performance if applicable]
[+ Compliance if applicable]
```

---

## Generate TDD

Create `.camel-kit/flows/{flow-name}/{flow-name}.tdd.md`:

**Core Sections (always include):**
1. Overview (business context, technical summary)
2. Source System (component, URI, config)
3. Processing Steps (EIPs, transformations)
4. Sink System (component, URI, config)
5. Error Handling (strategy, DLQ, retries)

**Conditional Sections (include only if discussed):**
6. Performance & Reliability (if high volume/latency requirements)
7. Security (if security/compliance requirements)
8. Monitoring & Observability (if monitoring discussed)

**Always include:**
9. Sequence Diagram
10. Configuration Properties
11. Dependencies
12. Constitution Gate Checks
13. Testing Strategy
14. Implementation Checklist

For TDD templates, use minimal structure unless specific requirements need detailed sections.

---

## Summary and Save

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TECHNICAL DESIGN SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Flow: {flow-name}

Source: [component]:{{source.endpoint}}
Processing: [list EIPs]
Sink: [component]:{{sink.endpoint}}
Error: [strategy] → {{dlq.endpoint}}
[Performance: {throughput} msg/sec, {latency}ms - if applicable]
[Security: {methods} - if applicable]
[Monitoring: {approach} - if applicable]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Save this TDD? (yes/no)
```

If confirmed:

```
✅ TDD saved to .camel-kit/flows/{flow-name}/{flow-name}.tdd.md

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NEXT STEPS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Review TDD and get stakeholder approval

2. When ready to implement:
   /camel-implement {flow-name}

3. After implementation, validate:
   /camel-validate {flow-name}

4. Generate tests:
   /camel-test {flow-name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Error Handling

### Missing BRD
```
❌ ERROR: Business Requirements Document not found

Run: /camel-project
```

### Flow Not in BRD
```
⚠️ WARNING: Flow '{flow-name}' not in BRD

Continue anyway? (yes/no)
```

---

## Token Optimization

**This skill is designed to minimize token usage:**

- Core SKILL.md: ~300 lines (down from 1,518)
- Load guides only when needed (save 70-80% tokens)
- Component skills loaded on-demand (already implemented)
- Conditional sections in TDD (only include what's needed)

**Total savings:** ~1,200 lines not loaded unless specifically needed
