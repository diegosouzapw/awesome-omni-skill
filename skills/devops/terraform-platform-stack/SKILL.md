---
name: terraform-platform-stack
description: Terraform specialist for Platform as a Service Stack v3.0.0+. Expert in deterministic naming (MD5), RBAC role assignments (uuidv5), feature flag orchestration, time-based RBAC propagation, and anti-pattern detection. Always validates with Terraform Registry MCP before ANY code generation to ensure latest provider schemas and avoid deprecated attributes.
---

# Terraform Platform Stack Specialist

## Overview

This skill provides expert guidance for Terraform infrastructure as code development in the **Platform as a Service Stack v3.0.0+** environment. It focuses on deterministic patterns (MD5 naming, uuidv5 RBAC), feature flag orchestration, root-level module coordination, and strict adherence to platform anti-pattern rules.

## When to Use This Skill

- Creating new Terraform modules for Platform Stack workloads
- Implementing feature flags with proper dependency validation
- Refactoring code to eliminate anti-patterns (random_string, null checks in count)
- Debugging "depends on resource attributes" errors
- Implementing RBAC with deterministic uuidv5() role assignments
- Adding time_sleep for RBAC propagation (180s delays)
- Orchestrating modules at root main.tf level (no inter-module dependencies)
- Validating Azure Provider 4.x compatibility (deprecated attributes)
- Troubleshooting RBAC propagation failures
- Optimizing Terraform state with boolean flags (not null checks)

## MCP Integration
> **Canonical source**: See agent's MCP Tool Usage Protocol. Always validate provider versions and resource schemas via MCP tools before any code generation.

## Critical Platform Stack Standards

### State Management (Azure Blob with Azure AD)

**ALL projects use Azure Blob Storage with Azure AD authentication:**

```hcl
# backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-paas"
    storage_account_name = "storagepaas"
    container_name       = "tfstate"
    key                  = "platform.terraform.tfstate"  # Override per platform
    use_azuread_auth     = true  # MANDATORY (no access keys)
  }
}
```

**Initialization (CLI)**:
```bash
terraform init \
  -backend-config="key=myplatform.terraform.tfstate" \
  -backend-config="use_azuread_auth=true"
```

### Module Structure (Single Concern)

**Standard module layout**:
```
terraform/modules/{domain}/{resource}/
├── main.tf         # Resource + RBAC + network rules
├── variables.tf    # name, location, tags, managed_identity_principal_id
├── outputs.tf      # IDs, URIs, names (NO secret values)
```

**Workloads include**:
```
└── workloads/
    ├── observability/      # Log Analytics + App Insights
    ├── storage-account/    # RBAC-only storage
    ├── service-bus/        # Premium namespace
    ├── event-grid/         # Domain with subscriptions
    ├── sql/                # SQL Server + Database
    └── container-apps/     # Container Apps Environment
        # container-registry → external: tfmodules-as-a-service-stack
```

**Root orchestration ONLY** (terraform/main.tf):
```hcl
module "storage" {
  count = var.enable_storage ? 1 : 0
  
  source                        = "./modules/workloads/storage-account"
  name                          = module.naming.storage_account
  location                      = var.location
  resource_group_name           = module.resource_group.name
  managed_identity_principal_id = var.enable_managed_identity ? module.managed_identity[0].principal_id : null
  vnet_subnet_ids               = var.enable_vnet ? [module.vnet_spoke[0].default_subnet_id] : []
  tags                          = var.tags
}
```

**CRITICAL**: Modules NEVER reference other modules - orchestration at root only!

### Anti-Patterns
> See `terraform-platform-instructions.md` — Anti-Pattern Detection section for the full grep validation workflow.

### Count Conditions (Boolean Flags ONLY)

```hcl
# ❌ WRONG - Causes "depends on resource attributes" error
count = var.log_analytics_workspace_id != null ? 1 : 0

# ✅ CORRECT - Use boolean flag
count = var.enable_observability ? 1 : 0

# Root main.tf passes workspace_id directly (not via count)
module "container_apps" {
  count = var.enable_container_apps ? 1 : 0
  
  log_analytics_workspace_id = var.enable_observability ? module.observability[0].workspace_id : null
}
```

**Rule**: Count conditions ONLY accept boolean variables. NO null checks, NO string comparisons.

### Feature Flag Table

| Flag | Resource | Hard Dependency | Recommended Dependency |
|------|----------|----------------|------------------------|
| `enable_managed_identity` | Managed Identity | - | **Used by all workloads for RBAC** |
| `enable_vnet` | VNet Spoke | - | Used by Storage, SQL, Container Apps |
| `enable_observability` | Log Analytics + App Insights | - | **REQUIRED by Container Apps** |
| `enable_storage` | Storage Account | - | Managed Identity (RBAC), VNet |
| `enable_service_bus` | Service Bus | - | Managed Identity |
| `enable_event_grid` | Event Grid | - | Managed Identity, Service Bus |
| `enable_sql` | SQL Server | - | Managed Identity, VNet |
| `enable_key_vault` | Key Vault | SQL (for password) | Managed Identity |
| `enable_container_registry` | Container Registry (ACR) | - | Managed Identity (AcrPush + AcrPull) |
| `enable_container_apps` | Container Apps | **Observability** | VNet, Container Registry + MI |

**Container Registry Details**:
- **Source**: External module from `tfmodules-as-a-service-stack` (`git::https://github.com/orafaelferreiraa/tfmodules-as-a-service-stack.git//modules/azurerm_container_registry?ref=1.0.3`)
- **SKU**: Controlled by `container_registry_sku` (string, default `"Basic"`)
- **RBAC**: When `enable_managed_identity = true`, AcrPush + AcrPull roles are auto-assigned to the Managed Identity
- **Container Apps Integration**: When both `enable_container_registry` and `enable_managed_identity` are true, Container Apps receives MI pre-attached + ACR `login_server`
- **Output**: `container_app_ready_config` — composite zero-config output bundling MI + ACR login server for Container Apps

### Deterministic Patterns
> See `terraform-platform-instructions.md` for MD5 naming, uuidv5 RBAC, and 180s time_sleep patterns with ❌/✅ examples.

### State Management (CRITICAL)

**ALL projects use Azure Blob Storage remote state - NEVER local state**

```terraform
# backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-paas"
    storage_account_name = "storagepaas"
    container_name       = "tfstate"
    key                  = "<project>-<tenant>-<environment>.tfstate"

    # Credentials via environment variables:
    # ARM_SUBSCRIPTION_ID
    # ARM_TENANT_ID
    # ARM_CLIENT_ID
    # ARM_CLIENT_SECRET
  }
}
```

**State file naming convention:** `<project>-<tenant>-<environment>.tfstate`

**Examples:**
- `aks-na-prod.tfstate`
- `network-stack-prod.tfstate`
- `workers-backend-sophie-dev.tfstate`

**State initialization:**
```bash
terraform init -backend-config="key=<project>-<tenant>-<environment>.tfstate"
```

### Multi-Tenant Configuration Architecture

**Configuration layering (hub-and-spoke pattern):**

```
project-root/
├── backend.tf
├── providers.tf
├── main.tf
├── variables.tf
├── cluster-config/
│   ├── common/
│   │   └── main.tfvars          # Base configuration for ALL tenants
│   └── specific/
│       ├── na/
│       │   ├── dev.tfvars       # NA dev overrides
│       │   ├── qa.tfvars        # NA qa overrides
│       │   └── prod.tfvars      # NA prod overrides
│       ├── sophie/
│       │   ├── dev.tfvars
│       │   └── prod.tfvars
│       └── woopi/
│           └── prod.tfvars
└── modules/
    └── <custom-modules>/
```

**Usage pattern:**
```bash
# Apply with layered configs
terraform plan \
  -var-file="cluster-config/common/main.tfvars" \
  -var-file="cluster-config/specific/na/prod.tfvars"

terraform apply \
  -var-file="cluster-config/common/main.tfvars" \
  -var-file="cluster-config/specific/na/prod.tfvars"
```

**Variable naming for multi-tenant:**
```terraform
# Prefix tenant-specific variables
variable "na_subscription_id" {
  description = "North America subscription ID"
  type        = string
  sensitive   = true
}

variable "sophie_subscription_id" {
  description = "Sophie tenant subscription ID"
  type        = string
  sensitive   = true
}

# OR use maps for shared structure
variable "subscription_ids" {
  description = "Subscription IDs per tenant"
  type        = map(string)
  sensitive   = true

  default = {
    na     = ""
    sophie = ""
    woopi  = ""
  }
}
```

### Version Constraints (CRITICAL)

**ALWAYS pin versions with `~>` constraint:**

```terraform
# versions.tf
terraform {
  required_version = ">= 1.9.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.57.0"    # Allows patch updates (4.57.x)
    }
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 3.1.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.31.0"
    }
  }
}
```

**Version constraint rules:**
- ✅ Use `~> 4.57.0` - allows patch updates (4.57.1, 4.57.2)
- ❌ NEVER use `>= 3.0` - unpinned, breaks reproducibility
- ❌ NEVER use `= 4.57.0` - exact pin, too restrictive
- ❌ NEVER omit version - breaks state compatibility

**Before upgrading providers:**
1. Check latest version: `mcp_hashicorp_ter_get_latest_provider_version`
2. Review CHANGELOG for breaking changes
3. Test in dev environment first
4. Update all modules to compatible versions

### File Structure Standards

**Standard project organization:**
```
project-root/
├── backend.tf        # Remote state configuration
├── providers.tf      # Provider configurations with aliases
├── versions.tf       # Terraform and provider version constraints
├── variables.tf      # Input variable declarations
├── locals.tf         # Local values and computed variables
├── main.tf           # Primary resource definitions
├── network.tf        # Networking resources (optional)
├── outputs.tf        # Output declarations
├── *.tfvars          # Variable values (NOT in git if contains secrets)
└── modules/          # Local modules
    └── <module-name>/
        ├── main.tf
        ├── variables.tf
        ├── outputs.tf
        └── versions.tf
```

**File-specific guidelines:**

**backend.tf:**
```terraform
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-paas"
    storage_account_name = "storagepaas"
    container_name       = "tfstate"
    key                  = "will-be-overridden-at-runtime.tfstate"
  }
}
```

**providers.tf:**
```terraform
terraform {
  required_version = ">= 1.9.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.57.0"
    }
  }
}

# Multiple provider configurations with aliases
provider "azurerm" {
  alias           = "stefanininam"
  subscription_id = var.stefanininam_subscription_id
  tenant_id       = var.stefanininam_tenant_id
  client_id       = var.stefanininam_client_id
  client_secret   = var.stefanininam_client_secret
  features {}
}

provider "azurerm" {
  alias           = "devops"
  subscription_id = var.devops_subscription_id
  tenant_id       = var.devops_tenant_id
  client_id       = var.devops_client_id
  client_secret   = var.devops_client_secret
  features {}
}
```

**variables.tf:**
```terraform
variable "environment" {
  description = "Deployment environment"
  type        = string

  validation {
    condition     = contains(["dev", "qa", "prod"], var.environment)
    error_message = "Environment must be dev, qa, or prod."
  }
}

variable "tenant" {
  description = "Tenant identifier"
  type        = string

  validation {
    condition     = contains(["na", "sophie", "woopi", "dex"], var.tenant)
    error_message = "Tenant must be na, sophie, woopi, or dex."
  }
}

variable "client_secret" {
  description = "Azure service principal client secret"
  type        = string
  sensitive   = true  # ALWAYS mark secrets as sensitive
}
```

**locals.tf:**
```terraform
locals {
  resource_prefix = "${var.tenant}-${var.environment}"

  common_tags = merge(
    var.tags,
    {
      Environment = var.environment
      Tenant      = var.tenant
      ManagedBy   = "Terraform"
      Project     = var.project_name
    }
  )

  location_short = {
    "East US"     = "eus"
    "West Europe" = "weu"
  }
}
```

**outputs.tf:**
```terraform
# Output complete resource object
output "resource_group" {
  description = "Complete resource group object"
  value       = azurerm_resource_group.main
}

# Output specific attributes
output "resource_group_id" {
  description = "Resource group ID"
  value       = azurerm_resource_group.main.id
}

# Output JSON summary
output "resource_summary" {
  description = "JSON summary of key attributes"
  value = jsonencode({
    id       = azurerm_resource_group.main.id
    name     = azurerm_resource_group.main.name
    location = azurerm_resource_group.main.location
  })
}

# Sensitive outputs
output "connection_string" {
  description = "Database connection string"
  value       = azurerm_storage_account.main.primary_connection_string
  sensitive   = true  # ALWAYS mark secrets as sensitive
}
```

## Module Development
> See `terraform-platform-instructions.md` — Module Structure section. Standard layout: `main.tf`, `variables.tf`, `outputs.tf` with types, validation, and dual outputs (object + JSON).

## Common Implementation Tasks

### 1. Creating New Resource Configuration

**MCP workflow:**
1. Get latest provider version: `mcp_hashicorp_ter_get_latest_provider_version`
2. Search for resource: `mcp_hashicorp_ter_search_providers`
3. Get resource details: `mcp_hashicorp_ter_get_provider_details`
4. Review workspace patterns: `grep_search` or `semantic_search`

**Implementation:**
```terraform
# Variable declarations
variable "storage_account_replication" {
  description = "Storage account replication type"
  type        = string
  default     = "LRS"

  validation {
    condition     = contains(["LRS", "GRS", "RAGRS", "ZRS"], var.storage_account_replication)
    error_message = "Must be a valid replication type."
  }
}

# Resource with environment-specific logic
resource "azurerm_storage_account" "data" {
  provider                 = azurerm.stefanininam
  name                     = "st${var.tenant}${var.app_name}${var.environment}"
  resource_group_name      = azurerm_resource_group.main.name
  location                 = var.location
  account_tier             = "Standard"
  account_replication_type = var.environment == "prod" ? "GRS" : "LRS"

  # Dynamic configuration based on environment
  min_tls_version               = "TLS1_2"
  enable_https_traffic_only     = true
  allow_nested_items_to_be_public = false

  network_rules {
    default_action = var.environment == "prod" ? "Deny" : "Allow"
    ip_rules       = var.allowed_ips
    bypass         = ["AzureServices"]
  }

  tags = local.common_tags
}

# Outputs
output "storage_account_id" {
  description = "Storage account resource ID"
  value       = azurerm_storage_account.data.id
}
```

### 2. Debugging Terraform Issues

**State Lock Issues:**

**Symptoms:**
- "Error: Error locking state"
- "Error: state blob is already locked"

**MCP workflow:**
1. Search: "Terraform Azure blob storage state lock troubleshooting"

**Resolution:**
```powershell
# Check blob lease status
az storage blob show `
  --account-name storagepaas `
  --container-name tfstate `
  --name <project>-<tenant>-<environment>.tfstate `
  --query "properties.lease" `
  --auth-mode login

# If lease is "locked", verify no one is running terraform
# Contact team to confirm

# Break lease (CAUTION)
az storage blob lease break `
  --blob-name <project>-<tenant>-<environment>.tfstate `
  --container-name tfstate `
  --account-name storagepaas `
  --auth-mode login
```

**Plan Failures:**

**Symptoms:**
- "Error: Unsupported argument"
- "Error: Invalid value for variable"
- "Error: Cycle" (circular dependency)

**MCP workflow:**
1. Get provider details for resource: `mcp_hashicorp_ter_get_provider_details`
2. Search docs: "Terraform <error-message>"

**Debugging steps:**
```bash
# Enable detailed logging
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform-debug.log
terraform plan

# Review log for specific error
grep -i "error" terraform-debug.log

# Visualize dependencies (for cycle errors)
terraform graph | dot -Tpng > graph.png

# Validate syntax
terraform validate

# Check provider versions
terraform version
terraform providers
```

**Resource Already Exists:**

**Symptoms:**
- "Error: A resource with the ID /subscriptions/... already exists"

**Options:**

**Option 1: Import existing resource**
```bash
# Get resource ID from Azure
az resource show --resource-group <rg> --name <name> --resource-type <type> --query id -o tsv

# Import into Terraform state
terraform import azurerm_resource_group.main "/subscriptions/<sub-id>/resourceGroups/<rg-name>"

# Verify import successful
terraform plan  # Should show no changes or only minor updates
```

**Option 2: Remove from state (if recreating)**
```bash
terraform state rm azurerm_resource_group.main
```

**State Drift:**

**Symptoms:**
- Terraform plan shows unexpected changes
- Manual changes were made in Azure Portal

**Detection:**
```bash
# Refresh state from actual resources
terraform refresh

# Detect drift
terraform plan -detailed-exitcode
# Exit code 0 = no changes
# Exit code 1 = error
# Exit code 2 = changes detected (drift)

# Show specific drift
terraform show

# For detailed diff
terraform plan -out=plan.tfplan
terraform show -json plan.tfplan | jq '.resource_changes'
```

**Resolution:**
```bash
# Option 1: Accept drift (update code to match reality)
# Modify .tf files to match current state

# Option 2: Force compliance (apply Terraform configuration)
terraform apply

# Option 3: Ignore specific attributes
resource "azurerm_resource_group" "main" {
  # ...

  lifecycle {
    ignore_changes = [
      tags["LastModified"],  # Ignore auto-updated tags
    ]
  }
}
```

### 3. State Management Operations

**Migrating resources between states:**
```bash
# Step 1: Remove from current state
terraform state rm azurerm_resource_group.example

# Step 2: Initialize new backend
terraform init -backend-config="key=new-project-na-prod.tfstate"

# Step 3: Import into new state
terraform import azurerm_resource_group.example "/subscriptions/<sub-id>/resourceGroups/<rg-name>"

# Step 4: Verify
terraform plan  # Should show no changes
```

**Renaming resources in code:**
```bash
# Move resource within same state (no Azure changes)
terraform state mv azurerm_resource_group.old_name azurerm_resource_group.new_name

# Update code to match
# In main.tf: change resource name from "old_name" to "new_name"

# Verify no changes planned
terraform plan
```

### 4. Testing and Validation

**Pre-commit validation:**
```bash
# Format all Terraform files
terraform fmt -recursive

# Validate syntax
terraform validate

# Check for security issues (if tfsec is installed)
tfsec .

# Validate provider versions
terraform version
terraform providers

# Generate documentation
terraform-docs markdown . > README.md
```
