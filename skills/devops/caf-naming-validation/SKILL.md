---
name: caf-naming-validation
description: Validate Azure Cloud Adoption Framework (CAF) naming conventions in Terraform modules and examples. Use this skill to ensure resource names follow Microsoft CAF standards and prevent naming conflicts.
---

# CAF Naming Validation for Azure Terraform Modules

Validate and enforce Azure Cloud Adoption Framework naming conventions across modules and examples.

## When to Use This Skill

Use this skill when:
- Creating a new module with named resources
- Creating examples for modules
- Reviewing existing modules for naming compliance
- User asks to "validate names" or "check naming conventions"
- Debugging naming-related errors (e.g., "name too long", "invalid characters")
- Implementing naming standards enforcement

## Azure CAF Naming Principles

The Azure Cloud Adoption Framework defines naming standards for consistency, governance, and automation.

### Core Principles

1. **Prefixes**: Resource type abbreviation (e.g., `rg-`, `st`, `kv-`)
2. **Base Name**: Descriptive identifier
3. **Suffixes**: Random string for uniqueness (optional)
4. **Constraints**: Length limits, allowed characters, uniqueness requirements

### Handled by azurecaf Provider

This framework uses the **aztfmodnew/azurecaf** Terraform provider to automatically:
- ✅ Add appropriate prefixes
- ✅ Enforce character constraints
- ✅ Apply length limits
- ✅ Generate random suffixes
- ✅ Ensure uniqueness

---

## Step-by-Step Validation

### Step 1: Identify Named Resources

Not all Azure resources have names. Check if the resource supports a `name` attribute.

**Named Resources** (require CAF naming):
- ✅ Resource Groups
- ✅ Storage Accounts
- ✅ Key Vaults
- ✅ Virtual Networks
- ✅ Subnets
- ✅ Virtual Machines
- ✅ AKS Clusters
- ✅ App Services
- ✅ SQL Databases
- ✅ Redis Cache

**Unnamed Resources** (no CAF naming needed):
- ❌ Role Assignments
- ❌ Policy Assignments
- ❌ Private Endpoints (optional name)
- ❌ Network Security Rules

---

### Step 2: Check Module Has azurecaf_name.tf

**Location**: `modules/<category>/<module_name>/azurecaf_name.tf`

**Verification**:
```bash
ls modules/<category>/<module_name>/azurecaf_name.tf
```

If file doesn't exist for a named resource, the module is missing CAF naming support.

---

### Step 3: Validate azurecaf_name.tf Pattern

**Standard Template**:

```hcl
#
# Naming convention
#

resource "azurecaf_name" "name" {
  name          = var.settings.name
  resource_type = "<azurecaf_resource_type>"
  prefixes      = var.global_settings.prefixes
  random_length = var.global_settings.random_length
  clean_input   = true
  passthrough   = var.global_settings.passthrough
  use_slug      = var.global_settings.use_slug
}
```

**Key Elements**:

1. **resource_type**: azurecaf resource type (e.g., `azurerm_resource_group`, `azurerm_storage_account`)
2. **name**: Base name from settings
3. **prefixes**: Global prefix array
4. **random_length**: Random suffix length (for uniqueness)
5. **clean_input**: Remove invalid characters
6. **passthrough**: Skip CAF naming if true
7. **use_slug**: Include resource type abbreviation

---

### Step 4: Map Azure Resource Type to azurecaf Type

The azurecaf provider uses specific resource type identifiers.

**Common Mappings**:

| Azure Resource | azurecaf Resource Type |
|----------------|------------------------|
| azurerm_resource_group | `azurerm_resource_group` |
| azurerm_storage_account | `azurerm_storage_account` |
| azurerm_key_vault | `azurerm_key_vault` |
| azurerm_virtual_network | `azurerm_virtual_network` |
| azurerm_subnet | `azurerm_subnet` |
| azurerm_kubernetes_cluster | `azurerm_kubernetes_cluster` |
| azurerm_container_registry | `azurerm_container_registry` |
| azurerm_redis_cache | `azurerm_redis_cache` |
| azurerm_app_service | `azurerm_app_service` |
| azurerm_mssql_database | `azurerm_mssql_database` |
| azurerm_application_gateway | `azurerm_application_gateway` |
| azurerm_cognitive_account | `azurerm_cognitive_account` |
| azurerm_container_app | `azurerm_container_app` |

**Verification**:
```bash
# Check azurecaf provider documentation
terraform providers schema -json | jq '.provider_schemas."registry.terraform.io/aztfmodnew/azurecaf"'
```

---

### Step 5: Validate Resource Uses Generated Name

**In main resource file** (e.g., `managed_redis.tf`):

```hcl
resource "azurerm_redis_cache" "redis" {
  name                = azurecaf_name.name.result  # ✅ CORRECT
  # NOT: name = var.settings.name  # ❌ WRONG
  
  resource_group_name = local.resource_group_name
  location            = local.location
  # ... other attributes
}
```

**Validation**:
- Resource MUST use `azurecaf_name.name.result`
- NEVER use `var.settings.name` directly

---

### Step 6: Check CAF Naming Prefixes

**Standard Prefixes by Resource Type**:

| Resource | Prefix | Example |
|----------|--------|---------|
| Resource Group | `rg-` | `rg-my-app-test-xxxxx` |
| Storage Account | `st` | `stmyapptestxxxxx` (no dashes) |
| Key Vault | `kv-` | `kv-my-app-test-xxxxx` |
| Virtual Network | `vnet-` | `vnet-my-app-test-xxxxx` |
| Subnet | `snet-` | `snet-my-app-test-xxxxx` |
| NSG | `nsg-` | `nsg-my-app-test-xxxxx` |
| AKS Cluster | `aks-` | `aks-my-app-test-xxxxx` |
| Container App | `ca-` | `ca-my-app-test-xxxxx` |
| Redis Cache | `redis-` | `redis-my-app-test-xxxxx` |
| Managed Grafana | `grafana-` | `grafana-my-app-test-xxxxx` |

**Note**: azurecaf provider adds these automatically based on `resource_type`.

---

### Step 7: Validate Examples Don't Include Prefixes

**CRITICAL**: Examples should NOT include CAF prefixes in names.

**❌ WRONG**:
```hcl
resource_groups = {
  rg1 = {
    name = "rg-my-app-test"  # ❌ Prefix will be duplicated
  }
}
```

**✅ CORRECT**:
```hcl
resource_groups = {
  rg1 = {
    name = "my-app-test"  # ✅ azurecaf adds "rg-" prefix
  }
}
```

**Generated Name**: `rg-my-app-test-ab12c` (with random suffix)

---

### Step 8: Check Length Constraints

Different Azure resources have different length limits.

**Common Length Limits**:

| Resource | Min | Max | Notes |
|----------|-----|-----|-------|
| Resource Group | 1 | 90 | Alphanumerics, underscores, hyphens, periods |
| Storage Account | 3 | 24 | Lowercase alphanumerics only, globally unique |
| Key Vault | 3 | 24 | Alphanumerics and hyphens, globally unique |
| Virtual Network | 2 | 64 | Alphanumerics, underscores, hyphens, periods |
| AKS Cluster | 1 | 63 | Alphanumerics and hyphens |
| Redis Cache | 1 | 63 | Alphanumerics and hyphens |

**azurecaf Handles This**:
- Automatically truncates names if too long
- Respects minimum length requirements
- Removes invalid characters

**Validation**:
```bash
# Test name generation
echo 'resource "azurecaf_name" "test" {
  name          = "my-very-long-resource-name-that-might-exceed-limits"
  resource_type = "azurerm_storage_account"
  random_length = 5
}' | terraform console
```

---

### Step 9: Validate Uniqueness Strategy

Some resources require globally unique names.

**Globally Unique Resources**:
- Storage Accounts
- Key Vaults
- Container Registries
- Cognitive Services
- App Services

**Uniqueness Strategy**:

```hcl
global_settings = {
  random_length = 5  # Adds 5-character random suffix
}
```

**Generated**: `st<name><random>` → `stmyappab12c`

**For Non-Unique Resources**:
```hcl
global_settings = {
  random_length = 0  # No random suffix needed
}
```

---

### Step 10: Validate Special Cases

#### Storage Accounts

**Special Constraints**:
- No hyphens or special characters
- Lowercase only
- 3-24 characters

**azurecaf handles this**:
```hcl
resource "azurecaf_name" "storage" {
  name          = "My-App-Storage"  # Input
  resource_type = "azurerm_storage_account"
  clean_input   = true  # Removes hyphens, lowercases
}
# Result: "stmyappstorage12345"
```

#### Key Vaults

**Special Constraints**:
- Globally unique
- 3-24 characters
- DNS name format

**Validation**:
```hcl
# Name must be valid DNS
# azurecaf ensures this automatically
```

#### Subnets

**Special Case**:
- Usually don't need random suffix
- Parent resource provides uniqueness scope

```hcl
resource "azurecaf_name" "subnet" {
  name          = "private-endpoints"
  resource_type = "azurerm_subnet"
  random_length = 0  # No random suffix needed
}
# Result: "snet-private-endpoints"
```

---

## Validation Checklist

### Module Validation

- [ ] Named resource has `azurecaf_name.tf` file
- [ ] Correct `resource_type` specified
- [ ] Resource uses `azurecaf_name.name.result`
- [ ] Not using `var.settings.name` directly
- [ ] `clean_input = true` for resources with character constraints
- [ ] `passthrough` and `use_slug` configured correctly

### Example Validation

- [ ] Names do NOT include CAF prefixes
- [ ] Names are descriptive and meaningful
- [ ] `global_settings.random_length` is set (typically 5)
- [ ] Storage account names don't have hyphens
- [ ] Names are appropriate length before prefix/suffix

### Output Validation

- [ ] Generated names meet length requirements
- [ ] Prefixes are correct for resource type
- [ ] Random suffix added (if configured)
- [ ] No duplicate names in same scope
- [ ] Special characters handled correctly

---

## Common Naming Errors and Fixes

### Error: "Name is too long"

**Cause**: Base name + prefix + suffix exceeds limit

**Fix**:
```hcl
# Shorten base name
resource_groups = {
  rg1 = {
    name = "app"  # Instead of "my-very-long-application-name"
  }
}
```

### Error: "Name contains invalid characters"

**Cause**: Special characters not allowed for resource type

**Fix**:
```hcl
# azurecaf_name.tf should have:
clean_input = true  # Automatically removes invalid characters
```

### Error: "Storage account name not available"

**Cause**: Name not globally unique

**Fix**:
```hcl
# Increase random_length
global_settings = {
  random_length = 8  # More entropy = less collision
}
```

### Error: "Name duplicated: rg-rg-myapp"

**Cause**: Prefix included in example name

**Fix**:
```hcl
# ❌ WRONG
name = "rg-myapp"

# ✅ CORRECT
name = "myapp"  # azurecaf adds "rg-"
```

### Error: "Name must be lowercase"

**Cause**: Storage account name has uppercase

**Fix**:
```hcl
# azurecaf handles this automatically if:
clean_input = true  # In azurecaf_name.tf
```

---

## Naming Best Practices

### 1. Descriptive Base Names

```hcl
# ❌ BAD: Generic
name = "app1"

# ✅ GOOD: Descriptive
name = "customer-portal-api"
```

### 2. Environment Indication (Optional)

```hcl
# Can be included in base name or via prefixes
name = "customer-portal-dev"

# Or use global prefixes
global_settings = {
  prefixes = ["dev"]
}
```

### 3. Consistent Naming Across Resources

```hcl
resource_groups = {
  rg1 = { name = "customer-portal" }
}

storage_accounts = {
  st1 = { name = "customerportal" }  # Matches (no hyphens)
}

key_vaults = {
  kv1 = { name = "customer-portal" }
}
```

### 4. Use random_length Appropriately

```hcl
# Globally unique resources
random_length = 5-8

# Scoped resources (RGs, VNets)
random_length = 3-5

# Subnets (parent provides scope)
random_length = 0
```

### 5. Avoid Reserved Keywords

```hcl
# ❌ AVOID
name = "default"
name = "admin"
name = "root"

# ✅ BETTER
name = "app-default"
name = "app-admin"
```

---

## Validation Tools

### Manual Validation

```bash
# Check module has CAF naming
find modules/<category>/<module_name> -name "azurecaf_name.tf"

# Verify resource uses CAF name
grep "azurecaf_name.name.result" modules/<category>/<module_name>/*.tf

# Check examples don't have prefixes
grep -E "rg-|st|kv-" examples/<category>/<service>/*/*.tfvars
```

### Automated Testing

```bash
# Run mock test to validate names
cd examples
terraform test -test-directory=./tests/mock -var-file=... -verbose

# Check generated names
terraform plan | grep "name.*will be created"
```

### Name Generation Preview

```bash
# Test name generation
terraform console <<EOF
resource "azurecaf_name" "test" {
  name          = "my-app"
  resource_type = "azurerm_storage_account"
  random_length = 5
  clean_input   = true
}
output "result" {
  value = azurecaf_name.test.result
}
EOF
```

---

## Special Resource Types

### Container Apps

```hcl
resource "azurecaf_name" "container_app" {
  name          = var.settings.name
  resource_type = "azurerm_container_app"  # Generates "ca-<name>-<random>"
  # ...
}
```

### Azure Managed Grafana

```hcl
resource "azurecaf_name" "grafana" {
  name          = var.settings.name
  resource_type = "azurerm_dashboard"  # Generic dashboard type
  # Generates "dash-<name>-<random>"
}
```

**Note**: Some services may not have dedicated azurecaf types. Use generic types or passthrough.

### Chaos Studio Resources

```hcl
# Targets and experiments don't have names
# No azurecaf_name.tf needed
```

---

## Passthrough Mode

Sometimes you need exact control over names (e.g., DNS requirements).

**Enable Passthrough**:

```hcl
# In azurecaf_name.tf
resource "azurecaf_name" "name" {
  name        = var.settings.name
  passthrough = true  # Use name exactly as provided
  # OR from global setting:
  passthrough = var.global_settings.passthrough
}
```

**In Example**:
```hcl
global_settings = {
  passthrough = true  # Disable CAF naming globally
}

resource_groups = {
  rg1 = {
    name = "my-exact-name"  # Used as-is
  }
}
```

**Use Cases**:
- Migration from existing infrastructure
- Specific DNS requirements
- Integration with external systems

---

## azurecaf Provider Configuration

**In module providers.tf**:

```hcl
terraform {
  required_providers {
    azurecaf = {
      source  = "aztfmodnew/azurecaf"
      version = ">= 1.2.0"
    }
  }
}
```

**Global Settings Structure**:

```hcl
# In examples
global_settings = {
  default_region = "region1"
  regions = {
    region1 = "westeurope"
  }
  
  # Naming settings
  random_length = 5
  prefixes      = []  # Optional prefix array
  passthrough   = false
  use_slug      = true  # Include resource type abbreviation
}
```

---

## Integration Checklist

- [ ] Module has `azurecaf_name.tf` for named resources
- [ ] Correct `resource_type` specified
- [ ] Resource uses `azurecaf_name.name.result`
- [ ] Examples don't include CAF prefixes
- [ ] `global_settings.random_length` configured in examples
- [ ] Length constraints respected
- [ ] Special character constraints handled
- [ ] Uniqueness strategy appropriate for resource type
- [ ] Names are descriptive and meaningful
- [ ] Tested name generation with examples

---

## Quick Reference

### Standard azurecaf_name.tf

```hcl
resource "azurecaf_name" "name" {
  name          = var.settings.name
  resource_type = "<azurecaf_resource_type>"
  prefixes      = var.global_settings.prefixes
  random_length = var.global_settings.random_length
  clean_input   = true
  passthrough   = var.global_settings.passthrough
  use_slug      = var.global_settings.use_slug
}
```

### Resource Reference

```hcl
name = azurecaf_name.name.result
```

### Example Configuration

```hcl
global_settings = {
  random_length = 5
}

resource_groups = {
  rg1 = {
    name = "my-app"  # NO prefix
  }
}
# Generated: "rg-my-app-ab12c"
```

---

## Related Skills

- **module-creation** - Include CAF naming from start
- **mock-testing** - Test name generation
- **azure-schema-validation** - Verify naming constraints

---

## Summary

CAF naming validation ensures:
1. ✅ Consistent naming across all resources
2. ✅ Automatic prefix/suffix handling
3. ✅ Length and character constraint compliance
4. ✅ Global uniqueness for resources that require it
5. ✅ Clean, descriptive names in examples
6. ✅ Automated name generation via azurecaf provider

This provides governance, consistency, and reduces naming errors across Azure deployments.
