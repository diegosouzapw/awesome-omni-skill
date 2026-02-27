---
name: moqui-service-writer
description: This skill should be used when users need to create, validate, or modify Moqui framework services, entities, and queries. It provides comprehensive guidance for writing correct Moqui XML definitions, following framework patterns and conventions.
license: Complete terms in LICENSE.txt
---

# Moqui Service Writer

## Overview

This skill enables writing correct Moqui framework services, entities, and queries by providing comprehensive patterns, validation tools, and reference documentation. It helps developers follow Moqui conventions and avoid common pitfalls when working with the enterprise framework.

## Quick Start

### For New Moqui Development
1. **Generate service templates** using `scripts/generate_service.py --interactive`
2. **Create entity definitions** using `assets/entity_template.xml` as a starting point
3. **Validate XML files** with `scripts/validate_service.py` and `scripts/validate_entity.py`
4. **Format XML consistently** using `scripts/format_moqui_xml.py`

### For Existing Moqui Projects
1. **Validate current files** to identify issues and improvements
2. **Reference patterns** in `references/` for best practices
3. **Use templates** in `assets/` for new components
4. **Apply consistent formatting** across all XML files

## Writing Services

### Service Creation Workflow

#### Step 1: Choose Service Type
Determine service purpose and select appropriate verb:
- **create** - Create new records
- **update** - Modify existing records  
- **find** - Search/retrieve records
- **get** - Retrieve single record
- **delete** - Delete records
- **process** - Custom business logic

#### Step 2: Generate Template
Use service generator:
```bash
python3 scripts/generate_service.py --interactive
```

Or generate specific types:
```bash
python3 scripts/generate_service.py --verb create --noun Product --type create
```

#### Step 3: Customize Service
Edit the generated service following patterns in `references/service_patterns.md`:
- Add proper authentication levels
- Define input/output parameters
- Implement business logic in actions
- Add error handling and validation

#### Step 4: Validate Service
```bash
python3 scripts/validate_service.py path/to/your/service.xml
```

### Common Service Patterns

#### CRUD Services
Use entity-auto services for standard operations:
```xml
<service verb="create" noun="Product" type="entity-auto">
    <in-parameters>
        <auto-parameters entity-name="com.example.Product" include="nonpk"/>
    </in-parameters>
    <out-parameters>
        <parameter name="productId"/>
    </out-parameters>
</service>
```

#### Search Services
Implement find services with filtering:
```xml
<service verb="find" noun="Product" allow-remote="true">
    <in-parameters>
        <parameter name="productTypeEnumId"/>
        <parameter name="statusId"/>
        <parameter name="orderByField" default-value="productName"/>
    </in-parameters>
    <out-parameters>
        <parameter name="productList" type="List"/>
        <parameter name="totalCount" type="Integer"/>
    </out-parameters>
    <actions>
        <entity-find entity-name="com.example.Product" list="productList" count="totalCount">
            <econdition field-name="productTypeEnumId" ignore-if-empty="true"/>
            <econdition field-name="statusId" ignore-if-empty="true"/>
            <order-by field-name="${orderByField}"/>
        </entity-find>
    </actions>
</service>
```

#### Custom Business Logic Services
Implement complex operations with proper error handling:

**Follow Moqui Framework Transaction Patterns:**

- **Default**: No transaction attribute (98.5% of framework services)
- **Critical Operations**: Use `transaction="force-new"` for financial/bulk operations
- **Long-running**: Add `transaction-timeout` for operations > 5 minutes
- **Avoid**: `transaction="cache"` (causes locking issues in framework)

```xml
<!-- Standard service - use framework defaults -->
<service verb="process" noun="Order" authenticate="true">
    <in-parameters>
        <parameter name="orderId" type="id" required="true"/>
    </in-parameters>
    <actions>
        <!-- Validate order exists -->
        <entity-find-one entity-name="com.example.Order" value-field="order"/>
        <if condition="order == null">
            <return error="true" message="Order not found with ID: ${orderId}"/>
        </if>
        
        <!-- Check permissions -->
        <set field="hasPermission" from="ec.user.hasPermission('ORDER', 'PROCESS')"/>
        <if condition="!hasPermission">
            <return error="true" message="Permission denied"/>
        </if>
        
        <!-- Business logic -->
        <service-call name="update#OrderStatus" in-map="[orderId: orderId, statusId: 'OsProcessing']"/>
        <service-call name="send#OrderNotification" in-map="[orderId: orderId]"/>
        
        <message public="true" type="success">Order processed successfully</message>
    </actions>
</service>
```

```xml
<!-- Critical financial operation - explicit transaction -->
<service verb="close" noun="FinancialPeriod" authenticate="true" 
         transaction="force-new" transaction-timeout="600">
    <in-parameters>
        <parameter name="financialPeriodId" type="id" required="true"/>
    </in-parameters>
    <actions>
        <!-- Critical financial closing logic -->
        <script>ec.transaction.commitBeginOnly();</script>
        <!-- ... closing operations ... -->
    </actions>
</service>
```

## Transaction Management Best Practices

### Framework Patterns (Based on Moqui Framework Analysis)

**Default Behavior (98.5% of framework services):**
```xml
<!-- Most services - no transaction attribute needed -->
<service verb="create" noun="Example" authenticate="true">
    <!-- Framework handles transactions automatically -->
</service>
```

**Critical Operations (1.5% of framework services):**
```xml
<!-- Financial/bulk operations requiring isolation -->
<service verb="close" noun="FinancialPeriod" authenticate="true" 
         transaction="force-new" transaction-timeout="600">
    <!-- Long-running critical operation -->
</service>
```

### Transaction Attribute Guidelines

| Scenario | Transaction Attribute | Timeout | When to Use |
|-----------|-------------------|----------|--------------|
| **Standard CRUD** | *(none)* | *(none)* | 98.5% of services |
| **Financial Operations** | `force-new` | 600-3600s | Period closing, payments |
| **Bulk Data Operations** | `force-new` | 600-1800s | Import/export, cleanup |
| **Long-running Tasks** | *(none)* | 600+ | Add timeout only if >5min |
| **Record Locking Issues** | `ignore` | *(none)* | Bulk operations only |

### Anti-Patterns to Avoid

❌ **Avoid `transaction="cache"`**
- Framework comments indicate locking and stale data issues
- Being removed from order processing in framework

❌ **Avoid `transaction="use-or-begin"`**
- Redundant (this is the framework default)
- Adds noise to service definitions

❌ **Avoid `transaction-timeout="60"`**
- Too short for most operations
- Framework uses 600-3600 seconds for long operations

### Recommended Approach

1. **Start with no transaction attribute** (framework default)
2. **Add `transaction="force-new"`** only for critical financial/bulk operations
3. **Add `transaction-timeout`** only for operations expected to run >5 minutes
4. **Test thoroughly** - transaction boundaries affect data consistency

## Creating Entities

### Entity Design Workflow

#### Step 1: Plan Entity Structure
Consider:
- Primary key design (usually `{entityName}Id`)
- Required fields and data types
- Relationships to other entities
- Audit trail requirements
- Status and type enumerations

#### Step 2: Use Entity Template
Start with `assets/entity_template.xml` and customize:
- Update entity name and package
- Define fields with appropriate types
- Add relationships with short aliases
- Include seed data for enumerations

#### Step 3: Validate Entity
```bash
python3 scripts/validate_entity.py path/to/your/entity.xml
```

### Field Type Selection

Use `references/field_types.md` to choose appropriate types:

#### Common Patterns
```xml
<!-- Primary Key -->
<field name="productId" type="id" is-pk="true"/>

<!-- Text Fields -->
<field name="productName" type="text-medium" enable-localization="true"/>
<field name="description" type="text-long" enable-localization="true"/>

<!-- Numeric Fields -->
<field name="price" type="currency-amount"/>
<field name="quantity" type="number-integer"/>

<!-- Date Fields -->
<field name="createdDate" type="date-time"/>
<field name="fromDate" type="date-time"/>
<field name="thruDate" type="date-time"/>

<!-- Status Fields -->
<field name="statusId" type="id" enable-audit-log="true"/>

<!-- Audit Fields -->
<field name="createdByUserAccountId" type="id" enable-audit-log="true"/>
<field name="lastUpdatedStamp" type="date-time"/>
```

### Relationship Design

#### One-to-Many
```xml
<!-- In parent entity -->
<relationship type="many" related="com.example.OrderItem" short-alias="items">
    <key-map field-name="orderId"/></relationship>

<!-- In child entity -->
<relationship type="one" related="com.example.OrderHeader" short-alias="order">
    <key-map field-name="orderId"/></relationship>
```

#### Many-to-One
```xml
<relationship type="one" related="moqui.basic.StatusItem" short-alias="status">
    <key-map field-name="statusId"/></relationship>
```

#### Self-Referencing
```xml
<relationship type="one-nofk" title="Parent" related="com.example.Category" short-alias="parent">
    <key-map field-name="parentCategoryId" related="categoryId"/></relationship>
```

## Writing Queries

### Entity-Find Patterns

#### Basic Queries
```xml
<!-- Find single record -->
<entity-find-one entity-name="com.example.Product" value-field="product"/>

<!-- Find multiple records -->
<entity-find entity-name="com.example.Product" list="productList">
    <econdition field-name="statusId" value="PsActive"/>
    <order-by field-name="productName"/>
</entity-find>
```

#### Filtered Queries
```xml
<entity-find entity-name="com.example.Product" list="productList">
    <econdition field-name="productTypeEnumId" from="productTypeEnumId" ignore-if-empty="true"/>
    <econdition field-name="statusId" from="statusId" ignore-if-empty="true"/>
    <econdition field-name="productName" operator="like" value="${searchText}%"/>
    <order-by field-name="productName"/>
</entity-find>
```

#### Date Range Queries
```xml
<entity-find entity-name="com.example.Order" list="orderList">
    <date-filter valid-date="orderDate"/>
    <order-by field-name="orderDate" descending="true"/>
</entity-find>
```

#### Pagination
```xml
<entity-find entity-name="com.example.Product" list="productList" count="totalCount">
    <econdition field-name="statusId" value="PsActive"/>
    <order-by field-name="productName"/>
</entity-find>

<!-- Manual pagination -->
<script>
def startIdx = pageIndex * pageSize
def endIdx = startIdx + pageSize
productList = productList.subList(startIdx, Math.min(endIdx, productList.size()))
</script>
```

### Complex Query Patterns

#### Join Queries
```xml
<entity-find entity-name="com.example.Product" list="productList">
    <econdition field-name="status.description" operator="like" value="${statusFilter}%"/>
    <relationship-join relationship="status"/>
    <order-by field-name="productName"/>
</entity-find>
```

#### View Entity Queries
```xml
<entity-find entity-name="com.example.ProductAndStatus" list="productList">
    <econdition field-name="statusId" value="PsActive"/>
    <econdition field-name="statusDescription" operator="like" value="${statusFilter}%"/>
    <order-by field-name="productName"/>
</entity-find>
```

## Security and Permissions

### Authentication Levels
Choose appropriate authentication based on sensitivity:
- `anonymous-all` - Public services
- `anonymous-view` - Read-only public data
- `true` - User must be logged in
- Add `require-all-roles` for restricted access

### Permission Checking
```xml
<set field="hasPermission" from="ec.user.hasPermission('PRODUCT', 'CREATE')"/>
<if condition="!hasPermission">
    <return error="true" message="Permission denied"/>
</if>
```

### Row-Level Security
```xml
<if condition="!ec.user.isUserInRole('ADMIN')">
    <script>
        productList = productList.findAll { it.createdByUserAccountId == ec.user.userId }
    </script>
</if>
```

## Validation and Quality Assurance

### Service Validation
```bash
# Validate single service file
python3 scripts/validate_service.py service/ExampleServices.xml

# Validate entire directory
python3 scripts/validate_service.py --directory service/
```

### Entity Validation
```bash
# Validate single entity file
python3 scripts/validate_entity.py entity/ExampleEntities.xml

# Validate entire directory
python3 scripts/validate_entity.py --directory entity/
```

### XML Formatting
```bash
# Format single file with backup
python3 scripts/format_moqui_xml.py service/ExampleServices.xml

# Format directory without backup
python3 scripts/format_moqui_xml.py --directory entity/ --no-backup
```

## Resources

### scripts/
Executable tools for validation and generation:

- **validate_service.py** - Validate service XML files for patterns and structure
- **validate_entity.py** - Validate entity XML files for patterns and conventions  
- **generate_service.py** - Generate service templates for common patterns
- **format_moqui_xml.py** - Format and pretty-print Moqui XML files

### references/
Comprehensive documentation for reference while working:

- **service_patterns.md** - Common service patterns and examples
- **entity_patterns.md** - Entity design patterns and relationships
- **query_examples.md** - Entity-find query patterns and examples
- **security_patterns.md** - Authentication, authorization, and security patterns
- **field_types.md** - Complete field type reference with usage examples

### assets/
Template files for starting new components:

- **service_template.xml** - Empty service template with proper structure
- **entity_template.xml** - Complete entity template with common patterns
- **component_template.xml** - Component configuration template

## Best Practices

### Service Design
1. **Use descriptive verb-noun combinations** following Moqui conventions
2. **Include proper authentication** based on service sensitivity
3. **Validate input parameters** before processing
4. **Return meaningful error messages** for debugging
5. **Use entity-auto services** for simple CRUD operations
6. **Implement permission checks** for sensitive operations
7. **Use transactions** - Follow framework defaults (98.5% use no attribute), force-new for critical operations

### Entity Design
1. **Include audit fields** on transactional entities
2. **Use appropriate field types** for data
3. **Define relationships** with meaningful short-aliases
4. **Use view entities** for complex queries
5. **Add seed data** for enumerations and reference data
6. **Follow naming conventions** consistently

### Query Design
1. **Use ignore-if-empty** for optional filter conditions
2. **Add proper ordering** for consistent results
3. **Use pagination** for large result sets
4. **Select specific fields** when possible for performance
5. **Use date-filter** for time-sensitive data

### Security
1. **Always authenticate** sensitive services
2. **Use principle of least privilege** for permissions
3. **Validate all input** parameters
4. **Implement audit logging** for important operations
5. **Use parameterized queries** to prevent SQL injection

## Entity Field Validation

### Always Validate Field Names
Before using field names in code, verify against entity definitions:
```bash
# Find entity definition
find . -name "*.xml" -path "*/entity/*" | xargs grep -l "entity-name=\"YourEntity\""

# Check available fields
grep -A 5 -B 5 "<field name="fieldName"*/entity/YourEntity.xml
```

### Common Field Name Confusions
| Intended Field | Correct Field | Entity | Context |
|---------------|---------------|---------|---------|
| trackingUrl | masterTrackingUrl | ShipmentRouteSegment | Overall shipment tracking |
| trackingUrl | trackingUrl | ShipmentPackageRouteSeg | Package-level tracking |
| trackingIdNumber | masterTrackingCode | ShipmentRouteSegment | Overall tracking number |
| trackingIdNumber | trackingCode | ShipmentPackageRouteSeg | Package tracking number |
| externalId | otherPartyOrderId | OrderPart | External system reference |

### Field Validation Workflow
1. **Check entity XML** for correct field names
2. **Use field mapping tables** when similar fields exist
3. **Validate with entity-find** before using in code
4. **Test field access** in development environment

## Service Behavior Understanding

### Critical Service Distinctions
Services can have misleading names - always verify actual behavior:

| Service | What It Actually Does | When to Use |
|----------|---------------------|-------------|
| ship#OrderPart | Creates NEW shipment + packs items + marks shipped | No existing shipment |
| create#OrderPartShipment | Creates shipment record only | Need custom packing logic |
| ship#Shipment | Marks existing shipment as shipped | Shipment exists, needs status change |
| update#OrderStatus | Changes order status only | Status transition needed |

### Service Verification Workflow
1. **Read service implementation** in XML files
2. **Check service description** and parameters
3. **Look for entity-auto** vs custom implementations
4. **Test service behavior** in development
5. **Document actual behavior** for future reference

### Common Service Usage Patterns
```xml
<!-- WRONG: Assumes ship#OrderPart updates existing shipment -->
<service-call name="ship#OrderPart" in-map="[orderId: orderId]"/>

<!-- CORRECT: Different approaches for existing vs new -->
<if condition="existingShipment">
    <service-call name="ship#Shipment" in-map="[shipmentId: shipmentId]"/>
<else>
    <service-call name="ship#OrderPart" in-map="[orderId: orderId]"/>
</if>
```

## Groovy Best Practices in Moqui

### Closure Scope Management
Groovy closures have different variable scope rules than traditional blocks:

#### Problem Pattern
```groovy
// BROKEN: logger not accessible in closure
existingShipments.each { shipment ->
    logger.info("Shipment: ${shipment.shipmentId}") // NullPointerException!
}
```

#### Solution Patterns
```groovy
// CORRECT: Use traditional for loop for outer variable access
for (EntityValue shipment in existingShipments) {
    logger.info("Shipment: ${shipment.shipmentId}") // Works!
}

// CORRECT: Explicit variable passing
existingShipments.each { shipment ->
    def log = logger // Explicit reference
    log.info("Shipment: ${shipment.shipmentId}")
}
```

### Scope Rules
- **.each { } closures** have limited outer variable access
- **Traditional for loops** maintain full scope access
- **Explicit variable passing** works but is verbose
- **Prefer for-in loops** when accessing outer variables

## Systematic Debugging Approach

### Debugging Priority Order
1. **Syntax Errors First** - Must compile before logic testing
2. **Entity Field Names** - Validate against definitions
3. **Service Behavior** - Verify what services actually do
4. **Logic Flow** - Check business logic after basics work
5. **Performance Issues** - Optimize after functionality works

### Incremental Testing Workflow
```bash
# 1. Fix syntax errors
groovy -cp . service/YourService.groovy

# 2. Test basic functionality
# Run service with minimal parameters

# 3. Add complex logic incrementally
# Test each addition separately

# 4. Full integration testing
# Test complete workflow
```

### Brace Balance Validation
```bash
# Quick brace check
open_count=$(grep -o '{' service/YourService.groovy | wc -l)
close_count=$(grep -o '}' service/YourService.groovy | wc -l)
echo "Open: $open_count, Close: $close_count"

# Should be equal for valid syntax
```

## Refactoring Mindset

### Before Writing Custom Logic
Always ask these questions first:

1. **Is there a Moqui service that does this?**
   - Check existing services in mantle-* components
   - Look for entity-auto services
   - Prefer built-in over custom implementation

2. **Am I duplicating existing functionality?**
   - Review service patterns in references/
   - Check if custom logic is necessary
   - Consider extending vs replacing

3. **Is this getting too complex?**
   - More than 20 lines = consider refactoring
   - Multiple nested conditions = simplify
   - Custom entity creation = use services

### Refactoring Example
```groovy
// COMPLEX: Manual shipment creation (48 lines)
if (existingShipment) {
    // Manual tracking updates
    // Manual route segment creation  
    // Manual shipment status changes
} else {
    // Manual shipment creation
    // Manual package creation
    // Manual status updates
}

// SIMPLE: Use Moqui services (15 lines)
if (existingShipment) {
    ec.service.sync().name("ship#Shipment")(shipmentId: shipmentId)
} else {
    ec.service.sync().name("ship#OrderPart")(orderId: orderId)
}
```

## Common Pitfalls to Avoid

1. **Missing Dependencies** - Always declare component dependencies in component.xml
2. **Incorrect Field Types** - Use appropriate field types (id, text-medium, number-decimal, etc.)
3. **Missing Relationships** - Define relationships for foreign keys with proper key-map
4. **No Authentication** - Set proper authenticate attribute on services
5. **Hardcoded Values** - Use enumerations instead of hardcoded strings
6. **SQL Injection** - Use entity-find with econdition, not raw SQL
7. **Transaction Issues** - Follow framework patterns: default (no attribute), force-new for critical operations, timeout for long-running tasks
8. **Missing Localization** - Use enable-localization for user-facing text
9. **Missing Audit Fields** - Include createdDate, createdByUserAccountId, lastUpdatedStamp
10. **Incorrect Service Names** - Follow verb-noun convention
11. **Invalid Field Names** - Always validate against entity definitions
12. **Service Misunderstanding** - Verify what services actually do before using
13. **Closure Scope Issues** - Use traditional loops for outer variable access
14. **Over-engineering** - Prefer existing Moqui services over custom logic

## Troubleshooting

### Validation Errors
- **XML Parse Errors** - Check for malformed XML, missing closing tags
- **Missing Attributes** - Add required attributes like entity-name, package
- **Naming Issues** - Follow Moqui naming conventions
- **Type Errors** - Use valid field types from reference

### Service Issues
- **Permission Denied** - Check authentication and role requirements
- **Parameter Errors** - Verify parameter names and types
- **Transaction Failures** - Check transaction attributes and error handling

### Entity Problems
- **Relationship Errors** - Verify key-map fields and related entities
- **Missing Primary Keys** - Ensure at least one is-pk field
- **Seed Data Issues** - Check enumeration and status item definitions