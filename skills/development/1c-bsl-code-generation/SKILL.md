---
name: 1c-bsl-code-generation
description: Skill for generating 1C:Enterprise (BSL) code with mandatory validation through MCP tools to prevent hallucinations. Use when generating, editing, or validating 1C BSL code, working with 1C metadata, or answering questions about 1C platform API.
---

# 1C BSL Code Generation with Anti-Hallucination Validation

## Introduction & When to Use

This skill provides mandatory procedures for generating 1C:Enterprise (BSL) code with strict validation to prevent hallucinations and ensure code correctness.

**Trigger this skill when:**
- Generating, editing, or validating 1C BSL code
- Working with 1C metadata objects (Справочники, Документы, Регистры, etc.)
- Using 1C platform API
- Writing 1C Query Language code
- Answering questions about 1C platform capabilities

**Keywords for detection:**
- `1с`, `bsl`, `1c:enterprise`, `справочник`, `документ`, `регистр`, `язык запросов`
- `catalog`, `document`, `register`, `query language`

**Semantic triggers:**
- Creating/editing/validating BSL code
- Working with 1C configuration metadata
- Implementing 1C business logic
- Writing procedures/functions in BSL

---

## 🚨 CRITICAL: Absolute Compliance Requirements

**⚠️ THESE RULES ARE NON-NEGOTIABLE AND MUST BE FOLLOWED WITHOUT EXCEPTION ⚠️**

**Failure to follow these rules constitutes a CRITICAL ERROR and produces hallucinated, potentially broken code.**

### Core Anti-Hallucination Principles

**MANDATORY RULES - NO EXCEPTIONS ALLOWED:**

#### 1. 🚫 NEVER ASSUME 1C API EXISTS

- **PROHIBITION**: Writing ANY platform API call without validation
- **MANDATORY ACTION**: ALWAYS call `getMembers()` BEFORE using any method/property
- **CONSEQUENCE**: Hallucinated methods that don't exist = broken code
- **ENFORCEMENT**: If API not validated → STOP and validate first

#### 2. 🚫 NEVER ASSUME METADATA EXISTS

- **PROHIBITION**: Using metadata objects (Справочники.*, Документы.*, etc.) without verification
- **MANDATORY ACTION**: ALWAYS verify through `search_metadata` or `search_metadata_by_description`
- **CONSEQUENCE**: References to non-existent objects = runtime errors
- **ENFORCEMENT**: If metadata not verified → STOP and verify first

#### 3. 🚫 NEVER WRITE CODE FROM SCRATCH IF PATTERN EXISTS

- **PROHIBITION**: Writing new implementations without searching for existing patterns
- **MANDATORY ACTION**: ALWAYS call `search_code()` FIRST before writing new code
- **CONSEQUENCE**: Duplicate code, missed best practices, inconsistent patterns
- **ENFORCEMENT**: If pattern search not performed → STOP and search first

#### 4. ✅ ALWAYS VALIDATE BEFORE WRITE

- **REQUIREMENT**: Complete validation workflow BEFORE writing ANY .bsl file
- **MANDATORY CHECKS**: API validated + Metadata verified + Patterns searched
- **CONSEQUENCE**: Skip validation = high probability of errors
- **ENFORCEMENT**: Cannot proceed to file write without completed validation

#### 5. ✅ ALWAYS LINT AFTER WRITE

- **REQUIREMENT**: Run `read_lints()` IMMEDIATELY after writing ANY .bsl file
- **MANDATORY FIX**: Fix ALL critical errors before proceeding
- **CONSEQUENCE**: Linter errors left unfixed = broken code in repository
- **ENFORCEMENT**: 0 critical errors required, max 3 fix attempts, then escalate to user

### 🔴 CRITICAL ENFORCEMENT POLICY

- These rules are NOT suggestions or recommendations
- These rules are ABSOLUTE REQUIREMENTS
- Agent MUST NOT skip any validation step
- Agent MUST NOT proceed if validation fails
- Agent MUST STOP and report if unable to validate

**Violation examples that are STRICTLY FORBIDDEN:**

```
❌ "I'll write the code quickly without checking API"
❌ "I assume this method exists based on similar patterns"
❌ "I'll skip search_code() because I know the pattern"
❌ "Linter check can be done later"
❌ "This is simple code, doesn't need validation"
```

**Correct approach - MANDATORY statements:**

```
✅ "BEFORE writing code, I MUST validate API via getMembers()"
✅ "BEFORE using metadata, I MUST verify via search_metadata()"
✅ "BEFORE implementing, I MUST search existing patterns via search_code()"
✅ "AFTER writing file, I MUST run read_lints() immediately"
✅ "IF validation fails, I MUST STOP and resolve before proceeding"
```

---

## MCP Tools Reference

### A. API Validation Tools (`mcp_bsl-platform-context_*`)

**Usage strategy: Proactive validation BEFORE code generation + Reactive fixes AFTER linter errors**

#### PROACTIVE Tools (use BEFORE writing code)

##### 1. getMembers - Get ALL members of a type

- **When**: BEFORE generating code that uses platform types
- **Purpose**: Understand available methods/properties of a type
- **Syntax**: `bsl-platform-context.getMembers(typeName: "TypeName")`
- **Example**: `getMembers("СправочникМенеджер")` → list all catalog manager methods
- **Use case**: "I need to work with Справочник → check what methods are available"
- **Critical**: Call this FIRST to avoid hallucinating non-existent methods

**Example call:**
```json
{
  "typeName": "СправочникОбъект"
}
```

**Expected result**: List of all methods and properties available for the type

##### 2. getConstructors - Get constructors for a type

- **When**: Need detailed description of class constructor
- **Purpose**: Understand how to create instances of a type
- **Syntax**: `bsl-platform-context.getConstructors(typeName: "TypeName")`
- **Example**: `getConstructors("ТаблицаЗначений")` → how to create ValueTable
- **Use case**: Creating new objects, need to know constructor signatures

**Example call:**
```json
{
  "typeName": "ТаблицаЗначений"
}
```

**Expected result**: List of available constructors with parameters

#### REACTIVE Tools (use AFTER linter errors)

##### 3. search - Search API by description

- **When**: Linter reports unknown method/property AND need to find alternatives
- **Purpose**: Find correct method name when linter says current one doesn't exist
- **Syntax**: `bsl-platform-context.search(query: "description", type: "method|property|type", limit: 10)`
- **Example**: Linter error "Method 'НайтиПоКоду' not found" → search with query "find catalog by code"

**Example call:**
```json
{
  "query": "find by code",
  "type": "method",
  "limit": 10
}
```

##### 4. getMember - Get specific member details

- **When**: Linter reports error with specific method/property
- **Purpose**: Verify exact signature and parameters of a member
- **Syntax**: `bsl-platform-context.getMember(typeName: "Type", memberName: "Member")`
- **Parameters**: typeName, memberName
- **Example**: Linter error on `Записать()` → `getMember("СправочникОбъект", "Записать")`

**Example call:**
```json
{
  "typeName": "СправочникОбъект",
  "memberName": "Записать"
}
```

##### 5. info - Detailed information about API element

- **When**: Linter error AND need full documentation
- **Purpose**: Get complete documentation for troubleshooting
- **Syntax**: `bsl-platform-context.info(name: "ElementName", type: "method|property|type")`
- **Parameters**: name (exact name), type (method/property/type)
- **Example**: Complex linter error → `info("НайтиПоСсылке", "method")` for full docs

**Example call:**
```json
{
  "name": "НайтиПоСсылке",
  "type": "method"
}
```

#### API Validation Workflow

**BEFORE code generation (proactive):**

```
1. Identify platform types used in task (СправочникМенеджер, ДокументОбъект, etc.)
2. For EACH type:
   ├─ Call getMembers(type) → get available methods/properties
   └─ If creating instances → call getConstructors(type)
3. Select correct methods from validated list
4. Write code using ONLY validated API
```

**AFTER linter errors (reactive):**

```
If linter reports API errors:
1. Identify error type:
   ├─ Unknown method/property → use search() to find correct name
   ├─ Wrong signature → use getMember() to verify parameters
   └─ Complex error → use info() for full documentation
2. Fix code with correct API
3. Re-run linter
```

---

### B. Metadata Validation Tools (`mcp_1c-metacode_*`)

**Priority order of tool usage:**

1. **search_code** (PRIMARY) - ALWAYS search for existing patterns FIRST
2. **search_metadata_by_description** - when uncertain about metadata names
3. **search_metadata** - for structural queries and finding usage

#### 1. search_code - Finding existing code patterns (PRIMARY TOOL)

- **CRITICAL**: Use FIRST before writing any new code
- **Purpose**: Find existing implementations, procedures, functions to reuse
- **Query language**: RUSSIAN only (критично!)
- **Parameter** `is_ssl_api: true` for БСП/SSL (standard subsystems library) code
- **Operations**:
  - `find_routines_by_description` - search by description
  - `get_routine_body` - get full procedure/function body

**Template JSON format (mandatory):**
```json
{
  "op": "find_routines_by_description",
  "text": "запись справочника",
  "is_ssl_api": false,
  "limit": 50
}
```

**Examples:**
- "запись справочника" → find existing catalog write patterns
- "проведение документа" → find document posting patterns
- "формирование движений" → find register movement patterns
- "работа с файлами" with `is_ssl_api: true` → find SSL file handling routines

**When to use:**
- ALWAYS before writing new procedure/function
- When implementing common patterns (write, post, validate, etc.)
- When working with БСП/SSL functionality

#### 2. search_metadata_by_description - Semantic metadata search

- **When to use**: Agent is UNCERTAIN about exact metadata object names
- **Purpose**: Find objects by business description/purpose
- **Query**: Natural language description in Russian
- **Template JSON format (mandatory):**

```json
{
  "op": "search_metadata_by_description",
  "text": "справочники для хранения банковских счетов",
  "categories": ["Справочники"],
  "limit": 50
}
```

**Examples:**
- "справочники для хранения банковских счетов" → find bank account catalogs
- "документы поступления товаров" → find goods receipt documents
- "регистры с информацией о ценах" → find price information registers

**Use case**: When user mentions business entity but exact 1C object name is unknown

#### 3. search_metadata - Structural queries

**Purpose:**
- Get object structure (attributes, tabular parts, etc.)
- Find objects that USE specific attribute/resource/dimension

**Template JSON mode (MANDATORY!)**

**Key operations:**

##### object_structure - Get full structure
```json
{
  "op": "object_structure",
  "object": "Контрагенты"
}
```
Returns: attributes, tabular parts, resources, dimensions

##### list_attributes - Get object attributes
```json
{
  "op": "list_attributes",
  "object": "Контрагенты",
  "limit": 500
}
```

##### list_tabular_attributes - Get tabular section attributes
```json
{
  "op": "list_tabular_attributes",
  "object": "Счёт",
  "tabular": "Позиции",
  "limit": 500
}
```

##### find_objects_by_attribute - Find objects using specific attribute
```json
{
  "op": "find_objects_by_attribute",
  "attr": "ИНН",
  "match": "contains",
  "limit": 500
}
```

##### find_objects_by_resource - Find objects using specific resource
```json
{
  "op": "find_objects_by_resource",
  "resource": "Сумма",
  "match": "exact",
  "limit": 500
}
```

##### find_objects_by_dimension - Find objects using specific dimension
```json
{
  "op": "find_objects_by_dimension",
  "dimension": "Контрагент",
  "match": "exact",
  "limit": 500
}
```

##### find_objects_using_object - Find where object is used as attribute/resource/dimension
```json
{
  "op": "find_objects_using_object",
  "target": "Контрагенты",
  "match": "exact",
  "in_category": "Документы",
  "include_tabular": true,
  "limit": 500
}
```

##### find_usages_of_object - Find usage sites (full paths)
```json
{
  "op": "find_usages_of_object",
  "target": "Контрагенты",
  "match": "exact",
  "limit": 500
}
```
Returns 1C-style paths like:
- "Справочники.Номенклатура.Реквизиты.Владелец"
- "Документы.Счёт.ТабличныеЧасти.Позиции.Реквизиты.Номенклатура"

**Critical use case**: "Find all documents that use Справочник.Контрагенты" → use `find_objects_using_object`

#### Metadata Validation Workflow

```
Step 1: Pattern Search (PRIMARY - do this FIRST!)
└─ Call search_code() with task description in RUSSIAN
   └─ If patterns found → use as foundation for implementation

Step 2: If uncertain about metadata names
└─ Call search_metadata_by_description() in RUSSIAN
   └─ Identify correct object names

Step 3: Structural validation
├─ Call search_metadata(op: "object_structure") → get full structure
└─ Validate that required attributes/tabular parts exist

Step 4: Find usage (if needed)
├─ find_objects_by_attribute() → "What uses this attribute?"
├─ find_objects_by_resource() → "What uses this resource?"
├─ find_objects_by_dimension() → "What uses this dimension?"
└─ find_objects_using_object() → "What references this object?"

Step 5: Write code based on validated patterns and metadata
```

---

### C. Code Validation Tools

#### Error Severity Levels (for Cursor Linter)

**CRITICAL**: 
- **Definition**: Errors that MUST be fixed before code can be considered complete
- **Examples**: Syntax errors, undefined variables, type mismatches, missing required methods
- **Requirement**: 0 critical errors MANDATORY for completion

**WARNING**:
- **Definition**: Issues that should be addressed but don't break functionality
- **Examples**: Unused variables, deprecated methods, code style violations
- **Requirement**: Review warnings, fix if reasonable, document if left unfixed

**INFO**:
- **Definition**: Informational messages and suggestions
- **Examples**: Code optimization hints, alternative approaches
- **Requirement**: Optional to address

**For this skill**: "0 critical errors" means zero CRITICAL-level errors. Warnings and info messages should be reviewed but don't block completion.

#### 1. Cursor Linter (read_lints) - Primary validation

- **When**: AFTER writing EVERY file (PRIMARY CHECK - always run first)
- **Purpose**: Check for syntax and semantic errors via BSL Language Server
- **Success criteria**: 0 CRITICAL-level errors
- **Process**: If CRITICAL errors found → analyze → fix → write corrected file → re-run linter

**Example call:**
```python
read_lints(["path/to/file.bsl"])
```

**Auto-fix loop:**
```
1. Write BSL file
2. Call read_lints([file_path])
3. Analyze results by severity:
   ├─ CRITICAL errors found:
   │  ├─ Analyze error messages
   │  ├─ Determine fixes
   │  ├─ Apply fixes to code
   │  ├─ Write corrected file
   │  └─ Repeat from step 2 (max 3 attempts)
   ├─ Only WARNING/INFO found:
   │  └─ Proceed to additional validation (acceptable state)
   └─ No issues:
      └─ Proceed to additional validation
4. If CRITICAL errors remain after 3 attempts → escalate to user
```

#### 2. 1c-copilot-proxy.check_1c_code - Additional validation

- **When**: AFTER linter check passes (secondary check)
- **Purpose**: Additional syntax/logic validation through 1C AI assistant
- **Parameters**: 
  - `code` (text of code to check)
  - `check_type` (default: "syntax", options: "syntax", "logic", "performance")
- **Types of checks**:
  - `syntax` - syntax errors (additional to linter)
  - `logic` - logical errors and potential bugs
  - `performance` - performance issues

**Example call:**
```json
{
  "code": "Процедура ПередЗаписью(Отказ)\n  // code here\nКонецПроцедуры",
  "check_type": "syntax"
}
```

---

## Mandatory Validation Workflow

### BEFORE Writing Any BSL Code

**Complete this workflow BEFORE file write - NO EXCEPTIONS:**

```
┌─────────────────────────────────────────────────────────────┐
│ Step 1: Pattern Search (PRIMARY - MUST do FIRST!)          │
├─────────────────────────────────────────────────────────────┤
│ IF task involves common pattern:                            │
│   1. Call search_code() with description in RUSSIAN        │
│   2. Review found patterns                                  │
│   3. IF suitable pattern exists:                            │
│      └─ Use as foundation (modify as needed)               │
│   4. IF no pattern found:                                   │
│      └─ Proceed to Step 2 (create new implementation)      │
│                                                              │
│ COMMON PATTERNS (always search first):                      │
│  • Catalog write/read (запись/чтение справочника)          │
│  • Document posting (проведение документа)                  │
│  • Register movements (формирование движений)               │
│  • Data validation (проверка данных)                        │
│  • Query construction (построение запроса)                  │
│  • File operations (работа с файлами)                       │
│  • Report generation (формирование отчёта)                  │
│  • Exchange/integration (обмен данными)                     │
│  • Scheduled jobs (регламентные задания)                    │
│  • Form event handlers (обработчики событий форм)           │
│                                                              │
│ IF task NOT in common patterns list:                        │
│  └─ Still run search_code() with task description          │
│     (may find relevant reusable code)                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ Step 2: API Validation (for EVERY platform API call)       │
├─────────────────────────────────────────────────────────────┤
│ FOR EACH platform API usage:                                │
│   1. Identify object type (СправочникМенеджер,             │
│      ДокументОбъект, etc.)                                  │
│   2. Call getMembers(type)                                  │
│   3. Verify method/property exists in returned list         │
│   4. IF creating instances:                                 │
│      └─ Call getConstructors(type)                         │
│   5. IF method/property NOT found:                          │
│      ├─ STOP code generation                               │
│      └─ Report to user with available alternatives         │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ Step 3: Metadata Validation (for EVERY metadata usage)     │
├─────────────────────────────────────────────────────────────┤
│ FOR EACH metadata object (Справочники.*, Документы.*, etc.):│
│   1. IF exact name unknown:                                 │
│      └─ Call search_metadata_by_description() in RUSSIAN   │
│   2. Call search_metadata(op: "object_structure")          │
│   3. Verify required attributes/tabular parts exist         │
│   4. IF attribute/tabular part NOT found:                   │
│      ├─ STOP code generation                               │
│      └─ Report to user with actual structure               │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ Step 4: Write Code (ONLY after validation complete)        │
├─────────────────────────────────────────────────────────────┤
│   1. Write file with validated code                         │
│   2. Use ONLY validated API                                 │
│   3. Use ONLY verified metadata                             │
│   4. Follow pattern if found in Step 1                      │
└─────────────────────────────────────────────────────────────┘
```

**ENFORCEMENT:** Agent MUST NOT write file until Steps 1-3 are complete. If validation fails → STOP and report to user.

### AFTER Writing BSL File

**Complete this workflow IMMEDIATELY after file write - NO EXCEPTIONS:**

```
┌─────────────────────────────────────────────────────────────┐
│ Step 5: Linter Check (MANDATORY PRIMARY CHECK)             │
├─────────────────────────────────────────────────────────────┤
│   1. Call read_lints([file_path])                           │
│   2. IF critical errors found:                              │
│      ├─ Analyze error messages                             │
│      ├─ IF API error:                                       │
│      │  ├─ Call search() or getMember() or info()          │
│      │  └─ Find correct API                                 │
│      ├─ Apply fixes to code                                 │
│      ├─ Write corrected file                                │
│      └─ Repeat from step 1 (max 3 iterations)              │
│   3. IF errors remain after 3 attempts:                     │
│      ├─ STOP processing                                     │
│      └─ Escalate to user with error details                │
│   4. SUCCESS: 0 critical errors → proceed to Step 6         │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ Step 6: Additional Validation (via 1C Copilot)             │
├─────────────────────────────────────────────────────────────┤
│   1. Read final file content                                │
│   2. Call 1c-copilot-proxy.check_1c_code(code, "syntax")     │
│   3. IF issues reported:                                    │
│      └─ Apply fixes and repeat Steps 5-6                   │
│   4. SUCCESS: No issues → proceed to Step 7                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ Step 7: Report to User                                      │
├─────────────────────────────────────────────────────────────┤
│   Report: "✅ Code validated successfully via:             │
│   - search_code() [pattern search]                          │
│   - getMembers() [API validation]                           │
│   - search_metadata() [metadata validation]                 │
│   - read_lints() [linter check: 0 errors]                  │
│   - check_1c_code() [additional validation: OK]"           │
└─────────────────────────────────────────────────────────────┘
```

**ENFORCEMENT:** Agent MUST NOT consider task complete until all steps pass. Zero critical errors is MANDATORY requirement.

---

## Special Cases & Advanced Validation

### 1C Query Language Validation

When writing 1C Query Language code:

```
1. Parse query text
2. Extract ALL object names (tables)
3. FOR EACH object name:
   └─ Call search_metadata(op: "object_structure", object: name)
4. Extract ALL field names
5. FOR EACH field:
   └─ Verify exists in object structure (from step 3)
6. IF any object/field NOT found:
   ├─ STOP code generation
   └─ Report missing elements to user
7. Write query with validated object and field names
```

**Example:**
```
Query: "ВЫБРАТЬ Контрагенты.Наименование, Контрагенты.ИНН ИЗ Справочник.Контрагенты"

Validation steps:
1. Extract object: "Контрагенты"
2. Call search_metadata(op: "object_structure", object: "Контрагенты")
3. Verify fields: "Наименование", "ИНН" exist in structure
4. IF all verified → write query
```

### БСП/SSL (Standard Subsystems Library) Code

When working with standard subsystems:

```
1. ALWAYS set is_ssl_api: true in search_code()
2. Search for standard procedures BEFORE writing custom ones
3. Example: File operations → search "работа с файлами" with is_ssl_api: true
4. Prefer standard procedures over custom implementation
```

**Example call:**
```json
{
  "op": "find_routines_by_description",
  "text": "работа с файлами",
  "is_ssl_api": true,
  "limit": 50
}
```

### Query Strings in Code

When code contains query as string variable:

```
1. Extract query text from string
2. Parse query (identify SELECT, FROM, WHERE clauses)
3. Extract table and field names
4. Validate each name via search_metadata
5. IF validation fails:
   ├─ STOP
   └─ Report invalid names to user
6. Write code with validated query
```

---

## Integration with Existing Rules

This skill complements existing project rules:

### Integration Order

```
1. Skills Index Detection
   └─ Keywords detected → load 1C_BSL_SKILL

2. Apply Anti-Hallucination Rules (this skill)
   ├─ NEVER assume API/metadata
   ├─ ALWAYS validate before write
   └─ ALWAYS lint after write

3. Follow TDD Workflow (if AgentMode.rules.md exists)
   ├─ Write tests first
   ├─ Implement with validation
   └─ Run tests

4. Apply Project-Specific Rules (if project_bsl_rules.mdc exists)
   ├─ Add required comment blocks
   ├─ Update registry.md
   └─ Follow DSSL UT conventions
```

### Rule Files Reference

- **project_bsl_rules.mdc** - Change commenting rules (DSSL UT specific)
- **AgentMode.rules.md** - TDD workflow and gate system
- **PlanMode.rules.md** - Planning and specifications
- **1C_BSL_SKILL.md** (this file) - Anti-hallucination validation

**Priority:** Anti-hallucination rules (this skill) are MANDATORY and apply BEFORE any other rules.

---

## Examples

### Example 1: Creating Catalog Write Function

**User request:** "Create function to write Catalog.Контрагенты"

**Agent workflow (MUST follow exactly):**

```
Step 1: Pattern Search
✅ Call: search_code(op: "find_routines_by_description", 
                     text: "запись справочника контрагенты",
                     is_ssl_api: false, limit: 50)
   Result: Found 3 existing patterns
   Action: Review patterns, select most suitable as foundation

Step 2: Metadata Validation
✅ Call: search_metadata_by_description(text: "справочник контрагенты")
   Result: Found "Справочники.Контрагенты"
✅ Call: search_metadata(op: "object_structure", object: "Контрагенты")
   Result: Structure with attributes: Наименование, ИНН, КПП, etc.

Step 3: API Validation
✅ Call: getMembers("СправочникОбъект.Контрагенты")
   Result: Methods confirmed: Записать(), УстановитьСсылкуНового(), etc.

Step 4: Write Code
✅ Write function based on:
   - Found pattern (Step 1)
   - Validated metadata structure (Step 2)
   - Confirmed API methods (Step 3)

Step 5: Linter Check
✅ Call: read_lints(["path/to/file.bsl"])
   Result: 0 errors

Step 6: Additional Validation
✅ Call: check_1c_code(code, "syntax")
   Result: OK

Step 7: Report
✅ Report to user: "✅ Code validated via search_code(), search_metadata(),
   getMembers(), read_lints() (0 errors), check_1c_code() (OK)"
```

### Example 2: Writing Query for Catalogs

**User request:** "Write query to get Контрагенты with ИНН"

**Agent workflow (MUST follow exactly):**

```
Step 1: Pattern Search (query patterns)
✅ Call: search_code(op: "find_routines_by_description",
                     text: "запрос справочника контрагенты",
                     is_ssl_api: false, limit: 50)
   Result: Found example query patterns

Step 2: Metadata Validation
✅ Call: search_metadata_by_description(text: "справочник контрагенты")
   Result: "Справочники.Контрагенты"
✅ Call: search_metadata(op: "list_attributes", object: "Контрагенты", limit: 500)
   Result: Attributes list including "ИНН"
   Action: Verify "ИНН" exists → ✅ Confirmed

Step 3: Write Query
✅ Write query with validated names:
   "ВЫБРАТЬ
   |  Контрагенты.Ссылка,
   |  Контрагенты.Наименование,
   |  Контрагенты.ИНН
   |ИЗ
   |  Справочник.Контрагенты КАК Контрагенты
   |ГДЕ
   |  Контрагенты.ИНН <> """

Step 4: Validation
✅ Call: read_lints() → 0 errors
✅ Call: check_1c_code() → OK

Step 5: Report
✅ Report successful validation
```

### Example 3: Linter Error Recovery

**Scenario:** Linter reports error on unknown method

**Agent workflow (MUST follow exactly):**

```
Initial code written:
Процедура Записать()
  СправочникОбъект.СохранитьДанные(); // ❌ Method doesn't exist!
КонецПроцедуры

Step 1: Linter detects error
✅ Call: read_lints(["file.bsl"])
   Result: Error "Method 'СохранитьДанные' not found"

Step 2: Reactive API validation
✅ Call: search(query: "save catalog data", type: "method", limit: 10)
   Result: Suggests "Записать()"
✅ Call: getMember("СправочникОбъект", "Записать")
   Result: Confirmed signature: Записать()

Step 3: Fix code
✅ Replace: СохранитьДанные() → Записать()

Step 4: Re-validate
✅ Call: read_lints() → 0 errors

Step 5: Report
✅ "Error fixed using search() and getMember(). Code now validated."
```

---

## Error Handling & Fallbacks

### If API Not Found

```
1. Report to user: "Method X not found for type Y"
2. Call getMembers(Y) → show available alternatives
3. Ask user: "Available methods: [list]. Which should I use?"
4. Wait for user confirmation
5. Use confirmed method
```

**Example:**
```
Agent: "Method 'НайтиПоКоду' not found for 'СправочникМенеджер'.
       Available methods: НайтиПоНаименованию(), НайтиПоРеквизиту().
       Which method should I use for finding by code?"
```

### If Metadata Not Found

```
1. Report to user: "Object X not found in configuration"
2. Call search_metadata_by_description() with similar terms
3. Show similar objects found
4. Ask user: "Did you mean: [list of similar]? Or provide exact name?"
5. Wait for user clarification
6. Use confirmed object name
```

**Example:**
```
Agent: "Object 'Контрагент' not found.
       Similar objects found:
       - Справочники.Контрагенты
       - Справочники.КонтрагентыПоставщики
       Which one did you mean?"
```

### If Linter Finds Errors (Auto-Fix)

```
1. Display errors to user
2. Attempt automatic fix (max 3 attempts):
   ├─ Iteration 1: Use search()/getMember() for API errors
   ├─ Iteration 2: Re-validate metadata if object not found
   └─ Iteration 3: Try alternative approach from search_code()
3. IF fixed → report success
4. IF not fixed after 3 attempts:
   ├─ Show remaining errors to user
   ├─ Explain what was tried
   └─ Request user guidance
```

**Example:**
```
Agent: "Linter found 2 errors. Attempting auto-fix...
       Iteration 1: Fixed API call using getMember()
       Iteration 2: Fixed metadata reference using search_metadata()
       ✅ All errors resolved. Code validated."
```

### If Validation Tools Fail

```
IF MCP tool returns error:
1. Log the error
2. Report to user: "Validation tool X failed: [error message]"
3. Explain impact: "Cannot validate Y without tool X"
4. Ask user: "Proceed without validation? (NOT recommended)"
5. IF user approves:
   └─ Add comment in code: "// WARNING: Not validated due to tool failure"
6. IF user declines:
   └─ STOP and wait for tool to be available
```

---

## Agent Checklist

**Before completing BSL code generation, verify ALL items:**

### Pre-Write Validation
- [ ] Pattern search performed via `search_code()` (if applicable)
- [ ] ALL platform API calls validated via `getMembers()` or `getMember()`
- [ ] ALL metadata objects verified via `search_metadata()` or `search_metadata_by_description()`
- [ ] Object structures validated (attributes, tabular parts confirmed to exist)
- [ ] Query object/field names validated (if writing queries)

### Post-Write Validation
- [ ] File written with validated code
- [ ] `read_lints()` executed → 0 critical errors achieved
- [ ] `1c-copilot-proxy.check_1c_code()` executed → OK
- [ ] All linter errors fixed (max 3 attempts used if needed)

### Reporting
- [ ] User notified of successful validation
- [ ] List of validation tools used provided to user
- [ ] Any warnings or caveats communicated clearly

### STOP Conditions (DO NOT PROCEED if any true)
- [ ] API validation failed and user has not provided guidance
- [ ] Metadata not found and user has not clarified
- [ ] Linter errors remain after 3 fix attempts
- [ ] Validation tool unavailable and user has not approved proceeding

---

## Summary: The Absolute Rules

1. **NEVER** write code without validating API via `getMembers()`
2. **NEVER** use metadata without verifying via `search_metadata()`
3. **ALWAYS** search for existing patterns via `search_code()` first
4. **ALWAYS** run `read_lints()` immediately after writing file
5. **ALWAYS** achieve 0 critical linter errors before completion
6. **STOP** immediately if validation fails - escalate to user
7. **REPORT** all validation steps performed to user

**These rules are ABSOLUTE. Violations produce hallucinated, broken code.**

---

*This skill ensures code quality and prevents hallucinations through systematic validation. Follow every step without exception.*
