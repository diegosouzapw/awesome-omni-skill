---
name: analyze-m1-module-for-migration
description: Systematically analyze a Magento 1 module to determine its purpose, usage, and migration requirements for Magento 2. Use when you need to decide whether to migrate a M1 module, find alternatives, or skip it.
---

# Skill: Analyze M1 Module for Migration

**Purpose:** Systematically analyze a Magento 1 module to determine its purpose, usage, and migration requirements for Magento 2.

**When to use:** When you encounter a Magento 1 module during migration and need to decide whether to migrate it, find alternatives, or skip it.

## Overview

This skill provides a repeatable process to:
1. Understand what a module does
2. Verify if it's actively being used in production
3. Determine if it should be migrated to M2
4. Research available M2 alternatives
5. Make a migration recommendation with supporting data

## Process

### Step 1: Locate and Identify the Module

**Objective:** Find the module files and basic metadata.

**Tasks:**
1. Locate the module directory:
   ```bash
   # M1 modules are typically in:
   # - app/code/local/{Vendor}/{Module}/
   # - app/code/community/{Vendor}/{Module}/
   # - app/code/core/{Vendor}/{Module}/ (avoid modifying core)

   find /path/to/m1 -type d -name "{ModuleName}" 2>/dev/null
   ```

2. Read module metadata:
   - `app/code/{pool}/{Vendor}/{Module}/etc/config.xml` - Module version, dependencies, configuration
   - `app/etc/modules/{Vendor}_{Module}.xml` - Module activation status, code pool

3. Extract key information:
   - Module name and version
   - Vendor/author
   - Dependencies (other modules required)
   - Is it active? (check `<active>true</active>`)

**Output:** Basic module identification and status.

---

### Step 2: Analyze Module Structure and Functionality

**Objective:** Understand what the module does and how it works.

**Tasks:**

#### 2.1 Configuration Analysis
Read `etc/config.xml` to identify:
- **Admin configuration** (`<adminhtml>` section)
- **Events/Observers** (`<events>` section)
- **Rewrites** (`<rewrite>` sections) - what core classes are modified
- **Routes** (`<routers>` section) - custom controllers
- **Cron jobs** (`<crontab>` section)
- **Layout updates** (`<layout>` section)

#### 2.2 Database Schema Analysis
Check for database modifications:
```bash
# Look for SQL setup files
find app/code/{pool}/{Vendor}/{Module}/ -name "*install*.php" -o -name "*upgrade*.php"
ls -la app/code/{pool}/{Vendor}/{Module}/sql/
```

Read setup scripts to identify:
- Custom tables created
- Columns added to core tables
- Indexes and foreign keys

#### 2.3 Code Component Analysis

**Models** (`Model/` directory):
- What data structures does it manage?
- What are the main CRUD operations?

**Resource Models** (`Model/Resource/` or `Model/Mysql4/`):
- What database operations are performed?
- Are there custom queries or complex logic?

**Blocks** (`Block/` directory):
- What UI elements are added/modified?
- Admin blocks vs. frontend blocks

**Controllers** (`controllers/` directory):
- What admin actions are available?
- What frontend endpoints exist?

**Helpers** (`Helper/` directory):
- What utility functions are provided?

**Observers** (`Model/Observer.php`):
- What events are being listened to?
- What actions are triggered?

**Example analysis command:**
```bash
# Get overview of module structure
tree -L 3 app/code/{pool}/{Vendor}/{Module}/

# Count files by type
find app/code/{pool}/{Vendor}/{Module}/ -name "*.php" | grep -E "Model|Block|Controller|Helper|Observer" | sort
```

**Output:** Detailed functionality description - "This module does X by doing Y when Z happens."

---

### Step 3: Check Database Usage (Production Verification)

**Objective:** Verify if the module is actively being used with real data.

**Tasks:**

#### 3.1 Identify Custom Tables
From setup scripts, list all custom tables:
```sql
SHOW TABLES LIKE '%{module_prefix}%';
```

#### 3.2 Count Records
For each custom table:
```sql
SELECT COUNT(*) as total_records FROM {table_name};
```

#### 3.3 Check Data Freshness
Determine if data is recent/active:
```sql
-- If table has timestamps
SELECT
    MIN(created_at) as oldest,
    MAX(created_at) as newest,
    COUNT(*) as total
FROM {table_name};

-- Check associations with active entities
SELECT COUNT(DISTINCT {foreign_key})
FROM {table_name}
WHERE {foreign_key} IS NOT NULL;
```

#### 3.4 Sample Data Analysis
Pull sample records to understand usage patterns:
```sql
SELECT * FROM {table_name} LIMIT 10;
```

#### 3.5 Show Products Using the Feature (Max 10)
**Objective:** Display real products that actively use the module's functionality for testing/verification purposes.

**Purpose:**
- Provides concrete test cases for M2 migration
- Shows real-world usage patterns
- Helps identify which products to verify after migration
- Useful for stakeholder review (they can see familiar products)

**Query Template:**
```sql
-- Generic template (adjust based on module's data structure)
SELECT
    module_table.id,
    module_table.{key_field},
    cpe.entity_id,
    cpe.sku,
    cpev.value as product_name,
    module_table.{relevant_data_column}
FROM {module_table_name} AS module_table
LEFT JOIN catalog_product_entity cpe
    ON module_table.product_id = cpe.entity_id
LEFT JOIN catalog_product_entity_varchar cpev
    ON cpe.entity_id = cpev.entity_id
    AND cpev.attribute_id = (
        SELECT attribute_id
        FROM eav_attribute
        WHERE attribute_code = 'name'
        AND entity_type_id = 4
    )
WHERE module_table.product_id IS NOT NULL
ORDER BY cpe.entity_id
LIMIT 10;
```

**Example (Custom Option Default Values):**
```sql
-- Show 10 products with default option values configured
SELECT
    dov.option_id,
    dov.option_type_id,
    dov.product_id,
    cpe.sku,
    cpev.value as product_name,
    cpot.title as option_name,
    cpotv.title as default_value_title
FROM default_option_value dov
LEFT JOIN catalog_product_entity cpe
    ON dov.product_id = cpe.entity_id
LEFT JOIN catalog_product_entity_varchar cpev
    ON cpe.entity_id = cpev.entity_id
    AND cpev.attribute_id = (
        SELECT attribute_id
        FROM eav_attribute
        WHERE attribute_code = 'name'
        AND entity_type_id = 4
    )
LEFT JOIN catalog_product_option cpo
    ON dov.option_id = cpo.option_id
LEFT JOIN catalog_product_option_title cpot
    ON cpo.option_id = cpot.option_id
    AND cpot.store_id = 0
LEFT JOIN catalog_product_option_type_value cpotv
    ON dov.option_type_id = cpotv.option_type_id
WHERE dov.product_id IS NOT NULL
ORDER BY cpe.entity_id
LIMIT 10;
```

**Example (Product Images):**
```sql
-- Show 10 products with custom images uploaded
SELECT
    cpe.entity_id,
    cpe.sku,
    cpev.value as product_name,
    cpotv.option_type_id,
    cpotv.image as image_path,
    cpotv.title as option_value_title
FROM catalog_product_option_type_value cpotv
INNER JOIN catalog_product_option cpo
    ON cpotv.option_id = cpo.option_id
INNER JOIN catalog_product_entity cpe
    ON cpo.product_id = cpe.entity_id
LEFT JOIN catalog_product_entity_varchar cpev
    ON cpe.entity_id = cpev.entity_id
    AND cpev.attribute_id = (
        SELECT attribute_id
        FROM eav_attribute
        WHERE attribute_code = 'name'
        AND entity_type_id = 4
    )
WHERE cpotv.image IS NOT NULL
ORDER BY cpe.entity_id
LIMIT 10;
```

**Output Format:**
Present the results in a clear, readable format:
```
Sample Products Using {Feature}:

1. SKU: X9458146 | Product: "500 Gallon Vertical Tank" | Option: "Tank Color" → Default: "White"
2. SKU: X2264184 | Product: "1000 Gallon Horizontal Tank" | Option: "FDA Compliant" → Default: "Yes"
3. SKU: X2197105 | Product: "1500 Gallon Vertical Tank" | Option: "Specific Gravity" → Default: "1.5"
...
10. SKU: X4297366 | Product: "2500 Gallon Vertical Tank" | Option: "Base Type" → Default: "Flat Bottom"

Total Products: 1,222 (showing 10 for reference)
```

**Why Limit to 10?**
- Keeps output concise and readable
- Provides sufficient examples without overwhelming the analysis
- Allows for quick manual verification in M2 after migration
- Can be easily included in reports and documentation

**Usage in Report:**
Include this list in the "Usage Analysis" section of the migration report under a "Sample Products for Testing" heading.

#### 3.6 Check Core Table Modifications
If module adds columns to core tables:
```sql
-- Check if custom columns exist
DESCRIBE {core_table};

-- Count records using custom columns
SELECT COUNT(*) FROM {core_table} WHERE {custom_column} IS NOT NULL;
```

**Output:**
- Record counts for all tables
- Active vs. inactive data
- Sample data showing real usage
- Percentage of entities using the feature

---

### Step 4: Check File System Usage

**Objective:** Verify if the module uses external files (images, uploads, generated files).

**Tasks:**

```bash
# Check for custom media directories
ls -la pub/media/ | grep -i {module_name}

# Count files if directory exists
find pub/media/{module_dir}/ -type f | wc -l

# Check file sizes
du -sh pub/media/{module_dir}/

# List recent files
ls -lt pub/media/{module_dir}/ | head -20
```

**Output:** File counts, storage usage, recent activity.

---

### Step 5: Research M2 Alternatives

**Objective:** Find existing M2 solutions before considering custom migration.

**Tasks:**

#### 5.1 Official Module Check
```
Search: "{VendorName} {ModuleName} Magento 2"
Check: Official vendor website, GitHub repositories
```

#### 5.2 Marketplace Search
```
Search Adobe Commerce Marketplace: https://commercemarketplace.adobe.com/
Keywords: Module functionality (not just vendor name)
```

#### 5.3 Open Source Alternatives
```
GitHub search: "{functionality} magento 2"
Example: "custom option default value magento 2"
```

#### 5.4 Free Alternatives
Check common sources:
- MagePal (often has free modules)
- Mageworx (has both free and paid)
- Mageplaza (extensive free module catalog)
- GitHub repositories

#### 5.5 Document Findings
For each alternative found, record:
- Name and vendor
- Cost (free, one-time, subscription)
- Features (matches M1 module?)
- Reviews/ratings
- Maintenance status (last update, active development)
- Repository URL or marketplace link

**Output:** Comparison table of M2 alternatives with recommendations.

---

### Step 6: Migration Decision Matrix

**Objective:** Make an informed decision on migration approach.

**Decision Criteria:**

| Criterion | Weight | Assessment Questions |
|-----------|--------|---------------------|
| **Is it in use?** | Critical | Record counts > 0? Active data? |
| **Is it business-critical?** | High | Affects customer experience? Revenue? Orders? |
| **Data volume** | Medium | How many records need migration? |
| **Functionality complexity** | Medium | Simple data storage or complex logic? |
| **M2 alternatives exist?** | High | Free? Paid? Feature-complete? |
| **Migration effort** | High | Data only? Code + data? Testing complexity? |
| **Cost consideration** | Medium | Free module vs. paid vs. custom development? |

#### Decision Tree:

```
Is module in use? (records > 0)
├─ NO → Skip migration, disable module
└─ YES → Is it business-critical?
    ├─ NO → Low priority, consider alternatives first
    └─ YES → Are M2 alternatives available?
        ├─ YES → Compare:
        │   ├─ Free alternative with all features → Use alternative
        │   ├─ Paid alternative ($) < Custom dev ($$$$) → Use alternative
        │   └─ No suitable alternative → Custom migration
        └─ NO → Custom migration required
```

**Output:** Clear recommendation with justification.

---

### Step 7: Generate Migration Analysis Report

**Objective:** Document findings for stakeholder review.

**Report Template:**

```markdown
# Migration Analysis: {Vendor}_{Module}

## Executive Summary
- **Module Name:** {Vendor}_{Module}
- **Purpose:** {1-2 sentence description}
- **In Use:** {Yes/No} - {X records, Y products, Z files}
- **Recommendation:** {Migrate / Use Alternative / Skip}
- **Estimated Effort:** {Low/Medium/High}

## Module Overview
### Functionality
{Detailed description of what the module does}

### Technical Components
- Database Tables: {list tables}
- Custom Fields: {list modified core tables}
- Admin Features: {list admin functionality}
- Frontend Features: {list customer-facing features}
- Events/Observers: {list hooks}

## Usage Analysis
### Database Statistics
- **Total Records:** {X,XXX}
- **Active Products:** {X,XXX} ({XX%} of catalog)
- **Date Range:** {oldest} to {newest}
- **Sample Data:** {show 3-5 examples}

### Sample Products for Testing
List of 10 products actively using this feature (for M2 migration verification):

1. **SKU:** {sku} | **Product:** "{name}" | **Feature Data:** {relevant detail}
2. **SKU:** {sku} | **Product:** "{name}" | **Feature Data:** {relevant detail}
3. **SKU:** {sku} | **Product:** "{name}" | **Feature Data:** {relevant detail}
4. **SKU:** {sku} | **Product:** "{name}" | **Feature Data:** {relevant detail}
5. **SKU:** {sku} | **Product:** "{name}" | **Feature Data:** {relevant detail}
6. **SKU:** {sku} | **Product:** "{name}" | **Feature Data:** {relevant detail}
7. **SKU:** {sku} | **Product:** "{name}" | **Feature Data:** {relevant detail}
8. **SKU:** {sku} | **Product:** "{name}" | **Feature Data:** {relevant detail}
9. **SKU:** {sku} | **Product:** "{name}" | **Feature Data:** {relevant detail}
10. **SKU:** {sku} | **Product:** "{name}" | **Feature Data:** {relevant detail}

*Total: {X,XXX} products (showing 10 representative samples)*

### File System Usage
- **Directory:** pub/media/{path}/
- **File Count:** {XXX} files
- **Storage:** {XX MB/GB}

## M2 Alternatives Research
| Alternative | Type | Cost | Features | Status | Rating |
|-------------|------|------|----------|--------|--------|
| {Name} | {Free/Paid} | ${XX} | {✓/✗ feature match} | {Active/Stale} | {X/5 stars} |

### Recommended Alternative
{Name of alternative or "Custom migration required"}

**Rationale:**
{Why this alternative was chosen or why custom migration is needed}

## Migration Approach
### Option 1: {Recommended approach}
- **Effort:** {X hours/days}
- **Cost:** ${X,XXX}
- **Risks:** {list risks}
- **Steps:**
  1. {Step 1}
  2. {Step 2}
  3. {Step 3}

### Option 2: {Alternative approach}
{Same structure as Option 1}

## Data Migration Strategy
### Tables to Migrate
| M1 Table | M2 Table | Records | Migration Method |
|----------|----------|---------|------------------|
| {m1_table} | {m2_table} | {X,XXX} | {Direct/Transform/Manual} |

### Migration SQL Script
```sql
-- Add to migration_from_m1/run-data-migration.step6.sh
# Step #XX: Migrate {module} data
{SQL migration script}
```

## Testing Requirements
- [ ] Admin UI: {what to test}
- [ ] Frontend: {what to test}
- [ ] Data integrity: {what to verify}
- [ ] Performance: {what to benchmark}

## Rollback Plan
{How to undo migration if issues arise}

## Timeline
- Analysis: {completed}
- Development: {X days}
- Testing: {X days}
- Deployment: {X days}
- **Total:** {X days}

## Appendix
### M1 Module Files
```
{tree structure of module}
```

### Database Schema
```sql
{CREATE TABLE statements}
```

### Configuration Samples
```xml
{Relevant config.xml excerpts}
```
```

---

## Example Usage

```bash
# 1. Identify module
ls -la /home/lucas/workspace/uptactics/ntotankM1/app/code/local/Magebuzz/Customoption

# 2. Analyze structure
tree -L 3 /home/lucas/workspace/uptactics/ntotankM1/app/code/local/Magebuzz/Customoption

# 3. Check database usage
ddev exec mysql ntosource -e "SELECT COUNT(*) FROM default_option_value;"
ddev exec mysql ntosource -e "SELECT COUNT(DISTINCT product_id) FROM default_option_value WHERE product_id IS NOT NULL;"

# 4. Get 10 sample products using the feature
ddev exec mysql ntosource -e "
SELECT
    dov.product_id,
    cpe.sku,
    cpev.value as product_name,
    cpot.title as option_name,
    cpotv.title as default_value
FROM default_option_value dov
LEFT JOIN catalog_product_entity cpe ON dov.product_id = cpe.entity_id
LEFT JOIN catalog_product_entity_varchar cpev ON cpe.entity_id = cpev.entity_id AND cpev.attribute_id = (SELECT attribute_id FROM eav_attribute WHERE attribute_code = 'name' AND entity_type_id = 4)
LEFT JOIN catalog_product_option cpo ON dov.option_id = cpo.option_id
LEFT JOIN catalog_product_option_title cpot ON cpo.option_id = cpot.option_id AND cpot.store_id = 0
LEFT JOIN catalog_product_option_type_value cpotv ON dov.option_type_id = cpotv.option_type_id
WHERE dov.product_id IS NOT NULL
LIMIT 10;
"

# 5. Research alternatives
# Use WebSearch for "magento 2 custom option default value"

# 6. Make recommendation based on findings
```

## Tips and Best Practices

### Do's ✅
- **Always check production data** - Module may be installed but unused
- **Look at sample data** - Understand real-world usage patterns
- **Research thoroughly** - Free alternative might exist, saving $$$$
- **Consider consolidation** - Can functionality be merged into existing M2 module?
- **Document everything** - Future you will thank present you
- **Check file system** - Modules often have media files not in database

### Don'ts ❌
- **Don't assume active** - Installed ≠ in use
- **Don't skip alternatives** - Custom dev is expensive
- **Don't migrate blindly** - Understand what you're migrating first
- **Don't forget dependencies** - Other modules may depend on this one
- **Don't ignore edge cases** - Sample data reveals real usage
- **Don't overlook observers** - They may affect core functionality

## Common Pitfalls

1. **False positives on usage**
   - Old test data still in database
   - Module was used historically but no longer
   - Check date ranges and active associations

2. **Missing dependencies**
   - Module depends on another M1 module
   - Check `<depends>` in config.xml
   - Some dependencies are implicit (not declared)

3. **Underestimating data migration**
   - IDs change between M1 and M2
   - Need mapping tables for foreign keys
   - Data transformation may be required

4. **Overlooking configuration**
   - Module may have admin configuration that needs migration
   - Check `core_config_data` table for module settings

5. **Missing file migrations**
   - Images, PDFs, uploads need to be copied
   - File paths may change in M2
   - Verify permissions after copy

## Success Criteria

A thorough analysis should answer:
- ✅ What does this module do? (clear explanation)
- ✅ Is it being used? (concrete data)
- ✅ How much is it used? (record counts, percentages)
- ✅ Which products use it? (10 sample products for testing)
- ✅ Should we migrate it? (justified recommendation)
- ✅ What are the alternatives? (researched options)
- ✅ How do we migrate it? (step-by-step plan)
- ✅ How long will it take? (time estimate)
- ✅ What are the risks? (identified and mitigated)

## Output Deliverables

After completing this skill, you should have:

1. **Analysis Report** (markdown document)
2. **Migration SQL Scripts** (if migrating)
3. **Testing Checklist** (verification steps)
4. **Sample Data** (for testing)
5. **Alternative Comparison** (if applicable)
6. **Effort Estimate** (hours/days)
7. **Cost Analysis** (custom vs. alternative)

## Related Skills

- `create-backend-controller` - If custom M2 module needed
- `magento-controller-refactor` - If migrating M1 controllers to M2

## Version History

- **v1.1** (2025-01-05) - Added Step 3.5: Show 10 sample products using the feature for testing/verification
- **v1.0** (2025-01-05) - Initial skill creation based on Magebuzz_Customoption analysis
