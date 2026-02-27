---
name: workshop-review-technical
description: Review workshop content for technical accuracy
disable-model-invocation: true
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

# Workshop Review: Technical Accuracy

**Purpose:** Review workshop content for technical accuracy, query correctness, and Neo4j best practices.

**When to use:** After pedagogy review is complete and content is ready for technical validation.

**Prerequisites:**
- Workshop content is complete
- Grammar review completed
- Pedagogy review completed
- Access to Neo4j instance for testing

---

## Overview

This skill performs a **technical accuracy review** that checks:
- Cypher query syntax and correctness
- Property naming consistency
- Neo4j best practices
- Verification files functionality
- SQL comparisons accuracy
- Tool references correctness
- Data model accuracy

**This review focuses on TECHNICAL CORRECTNESS, not pedagogy or grammar.**

---

## Phase 1: Pre-Review Setup (10 min)

### Checklist: Preparation
- [ ] Read WORKSHOP-PLAN.md for technical details
- [ ] Set up Neo4j instance for testing
- [ ] Load workshop data files
- [ ] Review data model
- [ ] Create review tracking document

### Setup Test Environment

**Required:**
1. Neo4j Aura Free instance (or local Neo4j)
2. Workshop data files downloaded
3. Data model imported
4. Access to Query tool

**Preparation steps:**
1. Create fresh Aura instance
2. Import workshop data using Data Importer
3. Verify all nodes/relationships exist
4. Ready to test queries

### Create Tracking Document

Create `TECHNICAL-REVIEW-PROGRESS.md`:

```markdown
# Technical Review Progress

**Workshop:** [Name]
**Reviewer:** Technical Review Skill
**Date:** [Date]
**Test Instance:** [URL or name]

## Data Model Verification

- [ ] Products: [N] nodes
- [ ] Customers: [N] nodes
- [ ] Orders: [N] nodes
- [ ] Relationships verified

## Review Status

### Module 1: [Name]
- [ ] Lesson 1 - Technical review
- [ ] Lesson 2 - Technical review
- [ ] Lesson 3 - Technical review

[Continue for all modules...]

## Issues Found
- Query error: [Description]
- Property naming: [Description]

## Review Complete
- All queries tested: Yes/No
- All verify.cypher files tested: Yes/No
- SQL comparisons verified: Yes/No
```

---

## Phase 2: Data Model Review (20 min)

### Checklist: Data Model
- [ ] All node labels documented
- [ ] All relationship types documented
- [ ] Property names consistent
- [ ] Property types correct
- [ ] Data constraints understood
- [ ] Sample data loaded

### Data Model Validation

**Verify the data model matches workshop:**

```cypher
// Check node counts
MATCH (n)
RETURN labels(n) AS label, count(n) AS count;

// Check relationship types
MATCH ()-[r]->()
RETURN type(r) AS type, count(r) AS count;

// Check properties on nodes
MATCH (n)
RETURN DISTINCT labels(n) AS label,
       [key IN keys(n) | key] AS properties
LIMIT 1;

// Check properties on relationships
MATCH ()-[r]->()
RETURN DISTINCT type(r) AS type,
       [key IN keys(r) | key] AS properties
LIMIT 1;
```

**Document findings:**

```markdown
## Data Model

### Nodes
- `Product`: [N] nodes
  - Properties: id, name, unitPrice, unitsInStock
- `Customer`: [N] nodes
  - Properties: id, name, city, country
- `Order`: [N] nodes
  - Properties: id, date, total

### Relationships
- `PLACED`: Customer → Order ([N] relationships)
- `ORDERS`: Order → Product ([N] relationships)
  - Properties: quantity, unitPrice
```

**Issues to flag:**
- Missing expected nodes/relationships
- Property names inconsistent
- Property types wrong
- Data not loaded

---

## Phase 3: Query Testing (Per Lesson)

### Checklist: Query Testing
- [ ] All Cypher queries run without errors
- [ ] Queries return expected results
- [ ] Property names match data model
- [ ] Query syntax follows best practices
- [ ] Performance is reasonable

### Query Testing Process

**For each Cypher query in lessons:**

1. **Copy query from lesson**
2. **Run in Query tool**
3. **Verify:**
   - No syntax errors
   - Returns results (or appropriate empty set)
   - Results match lesson description
   - Property names correct
4. **Check best practices:**
   - Uses pattern matching appropriately
   - Avoids anti-patterns
   - Performance is reasonable

### Common Query Issues

**Issue 1: Property naming**

❌ **Wrong property names:**
```cypher
MATCH (c:Customer {customerId: 'ALFKI'})
RETURN c.companyName;
```

✅ **Correct property names:**
```cypher
MATCH (c:Customer {id: 'ALFKI'})
RETURN c.name;
```

**Issue 2: Case sensitivity**

❌ **Wrong case:**
```cypher
match (c:customer) return c.Name;
```

✅ **Correct case:**
```cypher
MATCH (c:Customer) RETURN c.name;
```

**Issue 3: Syntax errors**

❌ **Missing semicolon or wrong syntax:**
```cypher
MATCH (c:Customer)
WHERE c.id = ALFKI  // Missing quotes
RETURN c.name
```

✅ **Correct syntax:**
```cypher
MATCH (c:Customer)
WHERE c.id = 'ALFKI'
RETURN c.name;
```

**Issue 4: Anti-patterns**

❌ **OPTIONAL MATCH for counts:**
```cypher
MATCH (c:Customer)
OPTIONAL MATCH (c)-[:PLACED]->(o:Order)
RETURN c.name, count(o);
```

✅ **COUNT subquery:**
```cypher
MATCH (c:Customer)
RETURN c.name, COUNT { (c)-[:PLACED]->(:Order) } AS orderCount;
```

---

## Phase 4: Verification Files Testing (Per Challenge)

### Checklist: Verification Files
- [ ] verify.cypher exists
- [ ] verify.cypher runs without errors
- [ ] verify.cypher returns `outcome` and `reason`
- [ ] verify.cypher handles edge cases
- [ ] solution.cypher exists
- [ ] solution.cypher runs without errors
- [ ] solution.cypher passes verification

### Verification File Testing Process

**For each challenge lesson:**

1. **Test verify.cypher**
   - Run in Query tool
   - Returns exactly 2 columns: `outcome`, `reason`
   - `outcome` is boolean
   - `reason` is helpful message

2. **Test with no data**
   - Delete nodes: `MATCH (n:NodeLabel) DETACH DELETE n`
   - Run verify.cypher
   - `outcome` should be false
   - `reason` should explain what's missing

3. **Test with partial data**
   - Create some but not all expected nodes
   - Run verify.cypher
   - `outcome` should be false if incomplete
   - `reason` should indicate partial import

4. **Test with complete data**
   - Import all expected data
   - Run verify.cypher
   - `outcome` should be true
   - `reason` should confirm success

5. **Test solution.cypher**
   - Start with empty database
   - Run solution.cypher
   - Run verify.cypher
   - `outcome` should be true

### Example Verification Testing

```cypher
// Test 1: No data
MATCH (n:Product) DETACH DELETE n;

// Run verify.cypher - should return:
// outcome: false
// reason: "No Product nodes found. Make sure you ran the import..."

// Test 2: Partial data
CREATE (:Product {id: "1", name: "Test"});

// Run verify.cypher - should return:
// outcome: false
// reason: "Only 1 Product nodes found. Expected 77."

// Test 3: Complete data
// (Import full dataset)

// Run verify.cypher - should return:
// outcome: true
// reason: "Success! You imported 77 Product nodes."

// Test 4: Solution passes
MATCH (n:Product) DETACH DELETE n;

// Run solution.cypher
// Run verify.cypher - should return:
// outcome: true
```

**Issues to flag:**
- verify.cypher doesn't return `outcome` and `reason`
- verify.cypher crashes with errors
- Error messages not helpful
- solution.cypher doesn't pass verification
- verification doesn't handle edge cases

---

## Phase 5: SQL Comparison Validation (Per Validation Lesson)

### Checklist: SQL Comparisons
- [ ] SQL queries are syntactically correct
- [ ] SQL queries return same results as Cypher
- [ ] SQL queries are fair (not intentionally bad)
- [ ] Comparisons are accurate (lines, JOINs counted correctly)
- [ ] Metrics are defensible

### SQL Comparison Testing Process

**For each SQL comparison:**

1. **Verify SQL syntax**
   - SQL query is valid
   - Would run in a relational database
   - Uses appropriate dialect (PostgreSQL, MySQL, etc.)

2. **Verify equivalence**
   - SQL produces same results as Cypher
   - Row counts match
   - Columns match
   - Data values match

3. **Verify fairness**
   - SQL is reasonable (not intentionally verbose)
   - Uses appropriate JOINs and indexes
   - Represents what a competent SQL developer would write

4. **Verify metrics**
   - Line count accurate
   - JOIN count correct
   - Complexity claims defensible

### Example SQL Validation

**Cypher query:**
```cypher
MATCH (c:Customer {id: 'ALFKI'})
      -[:PLACED]->(:Order)
      -[:ORDERS]->(p:Product)
RETURN DISTINCT p.name;
```

**SQL equivalent:**
```sql
SELECT DISTINCT p.productName
FROM customers c
JOIN orders o ON c.customerId = o.customerId
JOIN order_details od ON o.orderId = od.orderId
JOIN products p ON od.productId = p.productId
WHERE c.customerId = 'ALFKI';
```

**Validation:**
- [ ] SQL syntax correct: ✅
- [ ] Returns same products: ✅
- [ ] Fair representation: ✅
- [ ] Metrics: 4 lines vs 7 lines ✅, 0 JOINs vs 3 JOINs ✅

**Issues to flag:**
- SQL syntax errors
- SQL doesn't return same results
- SQL is intentionally bad/verbose
- Metrics are inaccurate (wrong JOIN count, wrong line count)
- Complexity claims not defensible

---

## Phase 6: Property Naming Consistency (Cross-Lesson)

### Checklist: Property Names
- [ ] All queries use `id` (not `customerId`, `productId`)
- [ ] All queries use `name` (not `companyName`, `productName`)
- [ ] All queries use `date` (not `orderDate`)
- [ ] Property names match import configuration
- [ ] Consistent across all lessons

### Property Naming Validation

**Standard property names:**
```
id - for all entity identifiers
name - for all entity names
date - for order dates
total - for order totals
city - for customer city
country - for customer country
unitPrice - for product prices
quantity - for order quantities
```

**Search all lessons for:**
```bash
# Bad patterns to find
customerId
productId
orderId
companyName
productName
orderDate
```

**Should be:**
```
id
id
id
name
name
date
```

**Issues to flag:**
- Inconsistent property names across lessons
- Using old property names (customerId instead of id)
- Property names don't match data model

---

## Phase 7: Neo4j Best Practices Review

### Checklist: Best Practices
- [ ] Uses COUNT subqueries (not OPTIONAL MATCH for counts)
- [ ] Uses pattern comprehensions for nested values
- [ ] Appropriate use of indexes (if mentioned)
- [ ] No Cartesian products
- [ ] LIMIT used appropriately
- [ ] No anti-patterns

### Best Practice Validation

**Pattern 1: COUNT subqueries**

❌ **Anti-pattern:**
```cypher
MATCH (c:Customer)
OPTIONAL MATCH (c)-[:PLACED]->(o:Order)
RETURN c.name, count(o);
```

✅ **Best practice:**
```cypher
MATCH (c:Customer)
RETURN c.name, COUNT { (c)-[:PLACED]->(:Order) } AS orderCount;
```

**Pattern 2: Pattern comprehensions**

❌ **Verbose:**
```cypher
MATCH (c:Customer)-[:PLACED]->(o:Order)
WITH c, collect(o.id) AS orderIds
RETURN c.name, orderIds;
```

✅ **Concise:**
```cypher
MATCH (c:Customer)
RETURN c.name,
       [(c)-[:PLACED]->(o:Order) | o.id] AS orderIds;
```

**Pattern 3: Cartesian products**

❌ **Accidental Cartesian:**
```cypher
MATCH (c:Customer), (p:Product)
WHERE c.country = 'USA'
RETURN c.name, p.name;
```

✅ **Intentional or avoided:**
```cypher
MATCH (c:Customer)-[:PLACED]->(:Order)-[:ORDERS]->(p:Product)
WHERE c.country = 'USA'
RETURN c.name, p.name;
```

**Issues to flag:**
- Use of OPTIONAL MATCH for counts
- Accidental Cartesian products
- Missing LIMIT on queries that could return thousands
- Anti-patterns that should be updated

---

## Phase 8: Tool Reference Accuracy

### Checklist: Tool References
- [ ] Module 1 Lesson 3 is tool mechanics source of truth
- [ ] Later lessons reference Module 1 Lesson 3 (not repeat mechanics)
- [ ] Tool names correct (Data Importer, Query tool, Explore)
- [ ] Screenshots match current UI
- [ ] Button text accurate

### Tool Reference Validation

**Module 1 Lesson 3 should have:**
- Comprehensive Data Importer walkthrough
- Query tool introduction
- Explore tool introduction
- Screenshots of each tool

**All other lessons should:**
- Reference Module 1 Lesson 3 for mechanics
- NOT repeat step-by-step UI instructions
- Focus on WHAT to do, not HOW to click

**Example good reference:**

```asciidoc
[.slide]
== Using Data Importer

**Reference:** See Module 1 Lesson 3 for Data Importer mechanics.

**Steps for this challenge:**
1. Upload products.csv
2. Map to Product node
3. Configure properties (table below)
4. Run import
```

**Example bad reference (repeating mechanics):**

```asciidoc
[.slide]
== Using Data Importer

1. Click "Import" in the Aura console
2. Click "Add Data"
3. Click "Upload File"
4. Navigate to products.csv
5. Click "Open"
[... 20 more steps ...]
```

**Issues to flag:**
- Lessons repeat tool mechanics
- Don't reference Module 1 Lesson 3
- Tool names incorrect
- Screenshots outdated

---

## Phase 9: Building Block Technical Validation

### Checklist: Building Block Verification
- [ ] Each challenge actually completes what it claims
- [ ] Verification confirms building block
- [ ] Data model matches building block description
- [ ] Building blocks can be tested independently

### Building Block Testing

**For each building block:**

1. **Read the claim:**
   - "Building Block 1: Products exist in the graph ✓"

2. **Verify technically:**
   ```cypher
   // Test: Do products exist?
   MATCH (p:Product)
   RETURN count(p) AS productCount;
   // Should return 77
   ```

3. **Verify completeness:**
   ```cypher
   // Test: Do products have required properties?
   MATCH (p:Product)
   RETURN p.id, p.name, p.unitPrice, p.unitsInStock
   LIMIT 1;
   // Should return all properties
   ```

4. **Verify usability:**
   - Can this building block be used in next module?
   - Are the right relationships/properties in place?

**Example building block tests:**

```cypher
// Building Block 2: "Customer→Order path complete"
MATCH path = (c:Customer)-[:PLACED]->(o:Order)
RETURN count(path) AS pathCount;
// Should return number of orders

// Building Block 3: "Customer→Order→Product path complete"
MATCH path = (c:Customer)-[:PLACED]->(:Order)-[:ORDERS]->(p:Product)
RETURN count(path) AS pathCount;
// Should return number of order line items
```

**Issues to flag:**
- Building block claim doesn't match reality
- Data model incomplete for claimed building block
- Building block can't be used as described

---

## Phase 10: Final Technical Report

### Create Review Summary

Update `TECHNICAL-REVIEW-PROGRESS.md` with results:

```markdown
# Technical Review Complete

**Workshop:** [Name]
**Date Completed:** [Date]
**Test Instance:** [URL]

## Summary Assessment

### Data Model: ✅ Pass / ❌ Fail
- All nodes present: [Yes/No]
- All relationships present: [Yes/No]
- Property names consistent: [Yes/No]

### Query Accuracy: ✅ Pass / ❌ Fail
- All Cypher queries tested: [N/N]
- All queries run without errors: [Yes/No]
- Queries return expected results: [Yes/No]

### Verification Files: ✅ Pass / ❌ Fail
- All verify.cypher tested: [N/N]
- All return outcome + reason: [Yes/No]
- All solution.cypher pass verification: [Yes/No]

### SQL Comparisons: ✅ Pass / ❌ Fail
- All SQL queries validated: [N/N]
- All comparisons accurate: [Yes/No]
- All metrics defensible: [Yes/No]

### Best Practices: ✅ Pass / ❌ Fail
- Uses COUNT subqueries: [Yes/No]
- No anti-patterns found: [Yes/No]
- Property naming consistent: [Yes/No]

### Tool References: ✅ Pass / ❌ Fail
- Module 1 Lesson 3 is source of truth: [Yes/No]
- Later lessons reference correctly: [Yes/No]
- No repeated mechanics: [Yes/No]

## Issues Found

### Critical Issues (Must Fix)
1. [Query syntax error in Module X Lesson Y]
2. [verify.cypher doesn't work in Module X Lesson Y]

### Moderate Issues (Should Fix)
1. [Property naming inconsistency in Module X]
2. [SQL comparison metric inaccurate in Module Y]

### Minor Issues (Nice to Fix)
1. [Could use COUNT subquery in Module X Lesson Y]
2. [LIMIT recommended for query in Module Z]

## Test Results

### Queries Tested
- Total queries: [N]
- Passed: [N]
- Failed: [N]

### Verification Files Tested
- Total challenges: [N]
- verify.cypher working: [N/N]
- solution.cypher working: [N/N]

### SQL Comparisons Validated
- Total comparisons: [N]
- Accurate: [N]
- Issues found: [N]

## Recommendations

1. [Recommendation for improvement]
2. [Recommendation for improvement]

## Ready for Publication

[✅ / ❌] Workshop is technically accurate and ready for publication.

**Issues to resolve before publication:**
- [Issue 1]
- [Issue 2]
```

---

## References

- [CONTENT_GUIDELINES.md](../../asciidoc/courses/workshop-importing/CONTENT_GUIDELINES.md) - Property naming, SQL comparisons
- [.cursor/rules/neo4j-cypher.mdc](../neo4j-cypher.mdc) - Cypher best practices
- [.cursor/review-lesson-content.mdc](../review-lesson-content.mdc) - Technical review section
- [Neo4j Cypher Manual](https://neo4j.com/docs/cypher-manual/current/) - Official documentation
