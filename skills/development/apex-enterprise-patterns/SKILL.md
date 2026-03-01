---
name: Apex Enterprise Patterns
description: This skill should be used when the user asks to "create apex class", "write apex code", "implement service layer", "domain pattern", "selector pattern", "apex security", "bulkification", or mentions DRY, SOLID, or TDD principles for Apex. Provides framework-agnostic enterprise patterns for Apex development.
version: 0.1.0
---

# Apex Enterprise Patterns

Enterprise patterns provide scalable, maintainable architecture for Salesforce Apex development. This skill provides framework-agnostic guidance for building robust, secure, and performant Apex code using separation of concerns, SOLID principles, and bulkification patterns.

## Architecture Overview

### Three-Layer Pattern

**Selector Layer**: Encapsulates SOQL queries and database read operations. Centralizes query logic, enforces security, and prevents query duplication.

**Domain Layer**: Contains business logic specific to a single SObject type. Enforces validation rules, maintains data integrity, and orchestrates record-level operations.

**Service Layer**: Coordinates cross-object business processes. Orchestrates domain classes, handles transactions, and implements use case logic.

### Pattern Benefits

**Separation of Concerns**: Each layer has a single responsibility. Selectors handle queries, domains handle SObject logic, services orchestrate processes.

**Testability**: Mock interfaces enable unit testing without database operations. Test service logic independently from query implementations.

**Reusability**: Centralized query logic eliminates duplicate SOQL. Domain methods provide consistent validation across triggers and services.

**Maintainability**: Changes to query logic occur in one location. Business rules live in domain classes, not scattered across triggers.

**Governor Limit Optimization**: Bulkified patterns handle collections efficiently. Queries execute once per transaction, not per record.

## Selector Pattern

### Purpose

Encapsulate all SOQL queries for a single SObject type. Provide consistent query methods with security enforcement.

### Implementation

**Query Methods**: Name methods descriptively: `selectById`, `selectByAccountId`, `selectRecentlyModified`. Accept collections for bulkification: `Set<Id>`, `List<Id>`.

**Field Selection**: Define field sets as constants or methods. Include commonly queried fields in a base field set. Add relationship fields when needed.

**Security Enforcement**: Use `WITH SECURITY_ENFORCED` for user-mode queries. Implement separate methods for system-mode when required. Document security mode clearly.

**Query Limits**: Accept optional `Integer limitCount` parameter. Default to reasonable limits (e.g., 200) to prevent heap issues. Use `LIMIT` clause to control result size.

**Example Pattern**:
```apex
public with sharing class AccountSelector {
    public List<Account> selectById(Set<Id> accountIds) {
        return [
            SELECT Id, Name, Type, Industry, OwnerId
            FROM Account
            WHERE Id IN :accountIds
            WITH SECURITY_ENFORCED
        ];
    }
}
```

See `examples/SelectorTemplate.cls` for complete implementation.

### Best Practices

**Return Types**: Return `List<SObject>` for multiple records, `SObject` for single record or null. Avoid returning empty lists when expecting single record.

**Parameter Types**: Use `Set<Id>` for ID parameters (automatic deduplication). Use `List<Id>` when order matters or duplicates are meaningful.

**Naming**: Prefix with SObject name: `AccountSelector`, `OpportunitySelector`. Use verb `select` for query methods: `selectById`, `selectByStatus`.

**Caching**: Cache query results when appropriate using static variables. Clear cache in test methods using `@TestVisible` methods.

## Domain Pattern

### Purpose

Encapsulate business logic for a single SObject type. Handle validation, defaulting, and record-level operations.

### Implementation

**Constructor**: Accept `List<SObject>` to support bulkification. Store records in instance variable for processing.

**Validation Methods**: Validate input data before DML. Add errors using `addError()` method. Check required fields, format, and business rules.

**Calculation Methods**: Perform field calculations and derivations. Update related fields based on business rules. Execute in bulk for all records in collection.

**DML Operations**: Centralize insert/update/delete logic. Enforce CRUD/FLS security before DML. Handle partial success scenarios appropriately.

**Example Pattern**:
```apex
public with sharing class AccountDomain {
    private List<Account> accounts;

    public AccountDomain(List<Account> accounts) {
        this.accounts = accounts;
    }

    public void validateRecords() {
        for (Account acc : accounts) {
            if (String.isBlank(acc.Name)) {
                acc.addError('Name is required');
            }
        }
    }
}
```

See `examples/DomainTemplate.cls` for complete implementation.

### Best Practices

**Bulkification**: Process entire collection in single method. Avoid per-record method calls from triggers or services.

**Error Handling**: Use `addError()` for validation failures in triggers. Throw custom exceptions in service layer context. Provide clear, actionable error messages.

**Separation**: Keep domain logic focused on single SObject. Delegate cross-object logic to service layer. Use selectors for querying related records.

**Sharing**: Declare `with sharing` for user-mode enforcement. Use `without sharing` only when explicitly required. Document sharing decisions in class comments.

## Service Pattern

### Purpose

Orchestrate complex business processes spanning multiple SObjects. Coordinate domain classes and manage transactions.

### Implementation

**Public Methods**: Expose use case methods: `createAccountWithContacts`, `processOpportunityApproval`. Accept DTOs or parameter objects for complex inputs. Return results or throw exceptions.

**Transaction Management**: Begin transactions in service methods. Use savepoints for partial rollback scenarios. Handle DML limits and order of operations.

**Orchestration**: Instantiate domain classes with query results. Call domain validation and calculation methods. Execute DML operations in correct sequence.

**Error Handling**: Catch and wrap exceptions with business context. Log errors for debugging and monitoring. Rollback transactions when appropriate.

**Example Pattern**:
```apex
public with sharing class AccountService {
    public static void createAccountWithContacts(
        Account newAccount,
        List<Contact> newContacts
    ) {
        // Validate and insert account
        AccountDomain accountDomain = new AccountDomain(
            new List<Account>{ newAccount }
        );
        accountDomain.validateRecords();
        insert newAccount;

        // Link contacts to account
        for (Contact con : newContacts) {
            con.AccountId = newAccount.Id;
        }
        insert newContacts;
    }
}
```

See `examples/ServiceTemplate.cls` for complete implementation.

### Best Practices

**Static Methods**: Use static methods for stateless services. Avoid instance variables unless necessary for state management.

**Parameter Objects**: Create wrapper classes for methods with 3+ parameters. Use inner classes for service-specific DTOs.

**Return Types**: Return created/updated records when needed by caller. Return success/failure indicators for async operations. Consider result wrapper classes for complex outcomes.

## Security Implementation

### CRUD Security

**Check Before DML**: Use `Schema.DescribeSObjectResult` to check create, read, update, delete access. Throw `NoAccessException` or display error when access denied.

**Example**:
```apex
if (!Schema.sObjectType.Account.isCreateable()) {
    throw new NoAccessException('Insufficient privileges to create Account');
}
```

**Strip Inaccessible**: Use `Security.stripInaccessible()` for queries and DML. Specify `AccessType.READABLE`, `CREATABLE`, `UPDATABLE`. Handle stripped fields in business logic.

```apex
SObjectAccessDecision decision = Security.stripInaccessible(
    AccessType.READABLE,
    accounts
);
return decision.getRecords();
```

### FLS Security

**Field Access Checks**: Verify field-level security before reading or updating. Use `Schema.DescribeFieldResult.isAccessible()`, `isCreateable()`, `isUpdateable()`.

**Example**:
```apex
if (!Schema.Account.Industry.getDescribe().isAccessible()) {
    throw new NoAccessException('Cannot read Industry field');
}
```

**User Mode Queries**: Use `WITH SECURITY_ENFORCED` in SOQL to enforce FLS automatically. System will filter inaccessible fields from query results.

**System Mode**: Use `WITH USER_MODE` in SOQL to enforce user permissions in system context. Required when using without sharing but need user security.

### Sharing Security

**Sharing Keywords**: Declare `with sharing` for user-mode sharing enforcement. Use `without sharing` for system-mode when business requires. Use `inherited sharing` to respect caller's context.

**Manual Sharing**: Check sharing rules before queries when needed. Use `UserRecordAccess` object to validate access. Query sharing reason fields for explicit shares.

**Example**:
```apex
public inherited sharing class AccountService {
    // Inherits sharing context from caller
}
```

### Input Validation

**Sanitize Inputs**: Validate all user inputs before processing. Check for null values, empty strings, invalid formats. Reject malicious patterns (SQL injection attempts).

**Escape Dynamic SOQL**: Use `String.escapeSingleQuotes()` for user input in dynamic SOQL. Better: Use static SOQL with bind variables instead of dynamic queries.

**Whitelist Validation**: Validate against allowed values using sets or picklist values. Reject unknown inputs rather than sanitizing.

**Example**:
```apex
public static List<Account> searchAccounts(String searchTerm) {
    if (String.isBlank(searchTerm)) {
        throw new IllegalArgumentException('Search term required');
    }

    String safeTerm = String.escapeSingleQuotes(searchTerm);
    // Use bind variable instead of string concatenation
    String query = 'SELECT Id, Name FROM Account WHERE Name LIKE :searchTerm';
    return Database.query(query);
}
```

## Bulkification Patterns

### Collection-Based Processing

**Accept Collections**: Method parameters should accept `List<SObject>` or `Set<Id>`. Process entire collection in single operation. Never loop to call single-record methods.

**Collect Related IDs**: Use loops to gather IDs for related queries. Query related records once with `IN` clause. Map results for processing.

**Example**:
```apex
// Gather all account IDs
Set<Id> accountIds = new Set<Id>();
for (Contact con : contacts) {
    accountIds.add(con.AccountId);
}

// Single query for all accounts
Map<Id, Account> accountMap = new Map<Id, Account>(
    [SELECT Id, Name FROM Account WHERE Id IN :accountIds]
);

// Process with map lookup
for (Contact con : contacts) {
    Account acc = accountMap.get(con.AccountId);
    // Process relationship
}
```

### Query Optimization

**Minimize Queries**: Query once per transaction, not per record. Store results in maps for fast lookup. Reuse query results across methods.

**Query Relationships**: Use relationship queries to reduce query count. Select parent fields: `Account.Name`, `Owner.Email`. Query child relationships: `(SELECT Id FROM Contacts)`.

**Selective Queries**: Use indexed fields in WHERE clauses. Filter on Id, Name, RecordTypeId, External ID fields. Avoid queries filtering on formula or text fields.

### DML Optimization

**Batch DML**: Collect all records for DML in collections. Execute single insert/update/delete per SObject type. Handle partial success with `Database.insert(records, false)`.

**Example**:
```apex
List<Account> accountsToUpdate = new List<Account>();
List<Contact> contactsToInsert = new List<Contact>();

// Collect records in loops
for (Opportunity opp : opportunities) {
    accountsToUpdate.add(opp.Account);
    contactsToInsert.addAll(opp.Contacts);
}

// Single DML per SObject
update accountsToUpdate;
insert contactsToInsert;
```

**DML Order**: Insert parents before children. Update records before deleting. Consider foreign key constraints and validation rules.

## Governor Limit Optimization

### SOQL Limits

**100 SOQL Limit**: Execute maximum 100 queries per transaction. Use relationship queries to reduce query count. Query related records in single SOQL with subqueries.

**50,000 Row Limit**: Maximum 50,000 rows returned across all queries. Filter queries to essential records only. Use LIMIT clause to cap result size.

**Query Consolidation**: Combine multiple queries with OR conditions. Use `IN` clause with collections instead of multiple queries. Select all needed fields in single query.

### DML Limits

**150 DML Limit**: Maximum 150 DML statements per transaction. Bulk all insert/update/delete operations. Process collections, not individual records.

**10,000 Row Limit**: Maximum 10,000 rows modified per transaction. Handle larger datasets with Batch Apex. Split processing across multiple batches if needed.

**DML Ordering**: Insert parent records first to obtain IDs. Update children with parent IDs. Delete in reverse order (children before parents).

### CPU Time Limits

**10,000ms Synchronous**: Optimize loops and calculations. Avoid nested loops over large collections. Use maps for O(1) lookup instead of O(n) loops.

**60,000ms Asynchronous**: Move complex processing to Queueable or Batch. Use async for API callouts and heavy computation.

**Algorithm Efficiency**: Replace nested loops with map-based lookups. Pre-calculate values outside loops. Use efficient data structures (Map vs List iteration).

**Example**:
```apex
// Inefficient: O(n²)
for (Contact con : contacts) {
    for (Account acc : accounts) {
        if (con.AccountId == acc.Id) {
            // Process
        }
    }
}

// Efficient: O(n)
Map<Id, Account> accountMap = new Map<Id, Account>(accounts);
for (Contact con : contacts) {
    Account acc = accountMap.get(con.AccountId);
    // Process
}
```

### Heap Size Limits

**6MB Synchronous, 12MB Asynchronous**: Limit collection sizes in memory. Process large datasets in batches. Clear collections when no longer needed.

**Query Field Selection**: Select only needed fields to reduce memory. Avoid selecting large text/blob fields unnecessarily.

**Collection Management**: Remove processed records from collections. Use iterator pattern for sequential processing. Avoid loading entire result set into memory.

## SOLID Principles in Apex

### Single Responsibility

**One Reason to Change**: Each class should have one responsibility. AccountSelector handles queries only. AccountDomain handles Account business logic only.

**Cohesion**: Methods in a class should relate to its purpose. Separate validation logic from calculation logic when they serve different purposes.

### Open/Closed

**Extension via Inheritance**: Use virtual methods for extensibility. Override methods in subclasses for specialized behavior. Avoid modifying existing methods for new requirements.

**Strategy Pattern**: Define interfaces for varying behavior. Inject implementations via dependency injection. Swap implementations without changing client code.

### Liskov Substitution

**Substitutability**: Subclasses must be usable in place of parent classes. Override methods should maintain parent contract. Avoid strengthening preconditions or weakening postconditions.

### Interface Segregation

**Focused Interfaces**: Define small, specific interfaces. Clients should not depend on methods they don't use. Create role-specific interfaces rather than monolithic ones.

**Example**:
```apex
public interface IAccountSelector {
    List<Account> selectById(Set<Id> accountIds);
}

public interface IAccountDomain {
    void validateRecords();
}
```

### Dependency Inversion

**Depend on Abstractions**: High-level modules should not depend on low-level modules. Both should depend on interfaces. Inject dependencies rather than instantiating directly.

**Example**:
```apex
public with sharing class AccountService {
    private IAccountSelector selector;

    public AccountService(IAccountSelector selector) {
        this.selector = selector;
    }

    public List<Account> getAccounts(Set<Id> ids) {
        return selector.selectById(ids);
    }
}
```

## Test-Driven Development

### TDD Approach

**Red-Green-Refactor**: Write failing test first (Red). Implement minimum code to pass (Green). Refactor for quality and patterns (Refactor).

**Test First**: Define expected behavior in test. Forces consideration of API design. Ensures testable code structure.

**Small Increments**: Write one test at a time. Implement one feature at a time. Build complexity gradually.

### Test Structure

**Arrange-Act-Assert**: Set up test data (Arrange). Execute method under test (Act). Verify results (Assert).

**Test Methods**: One test per scenario. Name clearly: `testCreateAccount_WithValidData_Success`. Test positive and negative cases.

**Test Data**: Use `@TestSetup` for shared data. Create minimal data for each test. Use utility classes for test data generation.

### Code Coverage

**75% Minimum**: Salesforce requires 75% coverage for deployment. Aim for 100% coverage of business logic. Focus on meaningful assertions, not just coverage.

**Test Quality**: Assert expected outcomes, not implementation details. Test edge cases and error conditions. Verify security enforcement and validation rules.

**Example Test**:
```apex
@IsTest
private class AccountServiceTest {
    @TestSetup
    static void setup() {
        // Create test data
    }

    @IsTest
    static void testCreateAccount_WithValidData_Success() {
        // Arrange
        Account acc = new Account(Name = 'Test Account');

        // Act
        Test.startTest();
        AccountService.createAccount(acc);
        Test.stopTest();

        // Assert
        List<Account> results = [SELECT Id, Name FROM Account];
        System.assertEquals(1, results.size());
        System.assertEquals('Test Account', results[0].Name);
    }
}
```

See `examples/ServiceTemplateTest.cls` for complete test implementation.

### Mocking

**Interface Mocking**: Implement test doubles for interfaces. Return predefined values without database operations. Verify method calls and parameters.

**Stub Pattern**: Create stub implementations of selectors and domains. Inject stubs into services for unit testing. Test service logic independently from query implementation.

## Example Code

Complete template implementations are available in the `examples/` directory:

- `ServiceTemplate.cls` - Service layer pattern with transaction management
- `DomainTemplate.cls` - Domain layer pattern with validation and calculation
- `SelectorTemplate.cls` - Selector layer pattern with security enforcement
- `ServiceTemplateTest.cls` - Test class with mocking and coverage patterns

Reference these templates when implementing enterprise patterns in your codebase.

## Quick Reference

**Selector Layer**: Encapsulate SOQL, enforce security, accept collections, return typed results.

**Domain Layer**: Validate data, calculate fields, process collections, enforce business rules.

**Service Layer**: Orchestrate processes, manage transactions, coordinate domains, handle errors.

**Security**: Check CRUD/FLS before operations, use `WITH SECURITY_ENFORCED`, validate inputs, document sharing mode.

**Bulkification**: Accept collections, query once, use maps for lookup, batch DML operations.

**Governor Limits**: Minimize queries (100 max), batch DML (150 max), optimize CPU time, manage heap size.

**SOLID**: Single responsibility per class, extend via interfaces, depend on abstractions, inject dependencies.

**TDD**: Write test first, red-green-refactor, aim for 100% coverage, use mocks for unit tests.
