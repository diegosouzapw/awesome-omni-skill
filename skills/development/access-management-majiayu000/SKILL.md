---
name: access-management
description: RBAC/ABAC implementation patterns, least privilege access, row-level security, column masking, and access review workflows.
triggers:
  - access management
  - rbac patterns
  - access control
  - permission management
category: access-control
---

# Access Management Skill

## Overview

Comprehensive access management patterns for Unity Catalog including RBAC, ABAC, row-level security, column masking, and automated access reviews.

## RBAC (Role-Based Access Control)

### Standard Role Framework
```python
STANDARD_ROLES = {
    "data_consumer": {
        "privileges": ["USE CATALOG", "USE SCHEMA", "SELECT"],
        "scope": "gold layer",
        "description": "Read-only access to curated data"
    },
    "data_producer": {
        "privileges": ["USE CATALOG", "USE SCHEMA", "SELECT", "MODIFY", "CREATE TABLE"],
        "scope": "bronze, silver layers",
        "description": "Transform and load data"
    },
    "data_steward": {
        "privileges": ["USE CATALOG", "USE SCHEMA", "SELECT", "MODIFY", "ALTER", "GRANT"],
        "scope": "all layers",
        "description": "Manage schema and metadata"
    },
    "platform_admin": {
        "privileges": ["ALL PRIVILEGES"],
        "scope": "all catalogs",
        "description": "Full administrative access (minimal assignments)"
    }
}
```

### Role Assignment
```python
def assign_role(principal: str, role: str, catalog: str):
    """Assign standard role to user or group."""
    role_def = STANDARD_ROLES[role]

    for privilege in role_def["privileges"]:
        if role_def["scope"] == "all catalogs":
            grant_sql = f"GRANT {privilege} ON CATALOG {catalog} TO `{principal}`"
        elif role_def["scope"] == "gold layer":
            grant_sql = f"GRANT {privilege} ON SCHEMA {catalog}.gold TO `{principal}`"

        spark.sql(grant_sql)

    log_role_assignment(principal, role, catalog)
```

## ABAC (Attribute-Based Access Control)

### Attribute-Based Policies
```python
def create_abac_policy(resource: str, attributes: dict):
    """Create attribute-based access policy."""
    policy_conditions = []

    # Department-based access
    if "department" in attributes:
        policy_conditions.append(
            f"department = '{attributes['department']}' OR IS_ACCOUNT_GROUP_MEMBER('admin')"
        )

    # Clearance level-based
    if "clearance_level" in attributes:
        policy_conditions.append(
            f"user_clearance >= {attributes['clearance_level']}"
        )

    # Time-based access
    if "business_hours_only" in attributes:
        policy_conditions.append(
            "HOUR(NOW()) BETWEEN 8 AND 18"
        )

    # Generate policy function
    policy_sql = f"""
    CREATE FUNCTION abac_{resource}_policy()
    RETURNS BOOLEAN
    RETURN {' AND '.join(policy_conditions)};
    """

    spark.sql(policy_sql)
```

## Row-Level Security

### Pattern 1: Regional Access Control
```sql
CREATE FUNCTION regional_access(region STRING)
RETURNS BOOLEAN
RETURN CASE
  WHEN IS_ACCOUNT_GROUP_MEMBER('global_access') THEN TRUE
  WHEN IS_ACCOUNT_GROUP_MEMBER('us_team') AND region = 'US' THEN TRUE
  WHEN IS_ACCOUNT_GROUP_MEMBER('eu_team') AND region = 'EU' THEN TRUE
  ELSE FALSE
END;

ALTER TABLE sales_data SET ROW FILTER regional_access ON (region);
```

### Pattern 2: Multi-Tenant Isolation
```sql
CREATE FUNCTION tenant_filter(tenant_id STRING)
RETURNS BOOLEAN
RETURN current_user() LIKE CONCAT(tenant_id, '@%')
    OR IS_ACCOUNT_GROUP_MEMBER('support_admin');

ALTER TABLE saas.customer_data SET ROW FILTER tenant_filter ON (tenant_id);
```

### Pattern 3: Department-Based Access
```sql
CREATE FUNCTION department_access(dept STRING)
RETURNS BOOLEAN
RETURN IS_ACCOUNT_GROUP_MEMBER(CONCAT('dept_', LOWER(dept)))
    OR IS_ACCOUNT_GROUP_MEMBER('hr_admin');

ALTER TABLE employees SET ROW FILTER department_access ON (department);
```

## Column Masking

### Pattern 1: Conditional Masking
```sql
CREATE FUNCTION mask_email(email STRING)
RETURNS STRING
RETURN CASE
  WHEN IS_ACCOUNT_GROUP_MEMBER('pii_admin') THEN email
  WHEN IS_ACCOUNT_GROUP_MEMBER('pii_viewer') THEN
    CONCAT(LEFT(email, 3), '***@', SPLIT(email, '@')[1])
  ELSE 'REDACTED'
END;

ALTER TABLE customers ALTER COLUMN email SET MASK mask_email;
```

### Pattern 2: SSN Masking
```sql
CREATE FUNCTION mask_ssn(ssn STRING)
RETURNS STRING
RETURN CASE
  WHEN IS_ACCOUNT_GROUP_MEMBER('hr_full') THEN ssn
  WHEN IS_ACCOUNT_GROUP_MEMBER('hr_partial') THEN CONCAT('XXX-XX-', RIGHT(ssn, 4))
  ELSE 'XXX-XX-XXXX'
END;

ALTER TABLE employees ALTER COLUMN ssn SET MASK mask_ssn;
```

## Least Privilege Enforcement

```python
def enforce_least_privilege(catalog: str):
    """Remove excessive permissions and enforce least privilege."""
    # Find overly permissive grants
    all_grants = spark.sql(f"SHOW GRANTS ON CATALOG {catalog}").collect()

    for grant in all_grants:
        # Replace ALL PRIVILEGES with specific grants
        if grant.privilege == "ALL PRIVILEGES" and grant.principal not in ['platform_admin']:
            revoke_excessive_grant(grant)
            apply_minimal_grants(grant.principal, catalog)

        # Remove MODIFY from read-only users
        if grant.privilege == "MODIFY" and grant.principal in readonly_users:
            spark.sql(f"REVOKE MODIFY ON CATALOG {catalog} FROM `{grant.principal}`")
```

## Access Reviews

### Quarterly Access Review
```python
def conduct_quarterly_review(catalog: str):
    """Quarterly access recertification."""
    grants = get_all_grants(catalog)

    review_items = {
        "excessive_access": find_excessive_permissions(grants),
        "unused_access": find_unused_permissions(grants),
        "stale_accounts": find_stale_users(grants),
        "orphaned_grants": find_orphaned_grants(grants)
    }

    # Generate review report
    report = generate_review_report(review_items)

    # Send to data stewards for approval
    send_for_recertification(report, data_stewards)

    return report
```

## Best Practices

1. **Least Privilege**: Grant minimum required permissions
2. **Separation of Duties**: No single user has complete control
3. **Regular Reviews**: Quarterly access recertification
4. **Time-Bound Access**: Temporary grants for contractors
5. **Service Principal Ownership**: Use SPs not individuals
6. **Audit Trail**: Log all permission changes

## Templates

- **rbac-framework.sql**: Complete RBAC setup
- **row-filter-patterns.sql**: Row-level security patterns
- **masking-functions.sql**: Column masking library
- **access-review.py**: Automated review workflow

## Examples

- **regional-isolation**: Multi-region access control
- **multi-tenant-security**: Tenant isolation patterns
- **conditional-masking**: Dynamic data masking
