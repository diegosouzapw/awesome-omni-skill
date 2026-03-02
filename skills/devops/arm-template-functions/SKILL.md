---
name: ARM Template Functions
description: Expert knowledge for using Azure Resource Manager (ARM) template functions, especially reference(), listKeys(), and resourceId() in subscription-level and nested deployments. Use when working with ARM templates, encountering template validation errors, or implementing cross-scope resource references.
---

# ARM Template Functions Skill

## When to Use This Skill

- Working with Azure ARM templates (azuredeploy.json files)
- Encountering template validation errors related to functions
- Implementing subscription-level deployments
- Using nested deployments with cross-scope references
- Accessing resource properties in outputs sections

## Critical Rules for ARM Template Functions

### reference() Function

**Valid Usage Locations:**
- ✅ Outputs section
- ✅ Properties object of resource definitions
- ❌ Top-level resource properties (type, name, location)
- ❌ Count property in copy loops

**Scope Considerations:**
```json
// ✅ CORRECT: Simple reference in same scope
"outputs": {
  "myOutput": {
    "value": "[reference('myResourceName').someProperty]"
  }
}

// ❌ INCORRECT: Cannot nest reference() inside other functions in outputs
"outputs": {
  "myOutput": {
    "value": "[listKeys(resourceId('rg', 'type', reference('dep').outputs.name.value))]"
  }
}
```

**Nested Deployment References:**
- Use `reference(deploymentName).outputs.propertyName.value` for outputs
- Requires `expressionEvaluationOptions.scope: inner` in nested deployment
- Cannot use `reference()` inside `resourceId()` or `listKeys()` in outputs

**Conditional Deployment Warning:**
- `reference()` evaluates even if resource is conditionally not deployed
- This can cause deployment failures
- Always ensure referenced resources exist

### listKeys() Function

**Requirements:**
- Can only be used in outputs section or resource properties
- Requires fully qualified resource ID
- Resource must exist before evaluation
- Requires correct API version

**Valid Pattern:**
```json
"outputs": {
  "storageKey": {
    "value": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('name')), '2023-01-01').keys[0].value]"
  }
}
```

**Security Warning:**
- Never expose sensitive list functions (listKeys, listSecrets) in outputs
- Output values are stored in deployment history
- Anyone with read access to deployment can see outputs

### resourceId() Function Scope Variations

**Resource Group Scope (default):**
```json
"[resourceId('Microsoft.Storage/storageAccounts', parameters('name'))]"
// Returns: /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{name}
```

**Cross-Resource Group:**
```json
"[resourceId(parameters('resourceGroupName'), 'Microsoft.Storage/storageAccounts', parameters('name'))]"
```

**Subscription Scope:**
```json
"[subscriptionResourceId('Microsoft.EventHub/namespaces', parameters('name'))]"
// Returns: /subscriptions/{sub}/providers/Microsoft.EventHub/namespaces/{name}
```

**⚠️ CRITICAL: Cross-Scope from Subscription Template to Resource Group Resources**

When in a **subscription-level template** referencing resources IN a resource group:
```json
// ❌ WRONG - Will fail with "not valid subscription identifier"
"[resourceId(parameters('resourceGroupName'), 'Microsoft.Storage/storageAccounts', parameters('name'))]"

// ✅ CORRECT - Must include subscription ID
"[resourceId(subscription().subscriptionId, parameters('resourceGroupName'), 'Microsoft.Storage/storageAccounts', parameters('name'))]"
```

This applies to:
- listKeys() calls
- Any resourceId() in outputs or properties
- Diagnostic settings eventHubAuthorizationRuleId

**Management Group/Tenant:**
```json
"[tenantResourceId('Microsoft.Authorization/policyDefinitions', parameters('name'))]"
```

**⚠️ Critical:** In subscription-level templates, use `subscriptionResourceId()` NOT `resourceId()` for subscription-scoped resources.

## Common Errors and Solutions

### Error: "The template function 'reference' is not expected at this location"

**Cause:** Using `reference()` inside another function in outputs section

**Solution:** Store the referenced value in variables, or restructure to avoid nested reference calls:

```json
// ❌ INCORRECT
"outputs": {
  "connectionString": {
    "value": "[listKeys(resourceId('rg', 'type', reference('deployment').outputs.name.value))]"
  }
}

// ✅ CORRECT: Use parameters or variables for the name
"outputs": {
  "connectionString": {
    "value": "[listKeys(resourceId(parameters('resourceGroupName'), 'Microsoft.EventHub/namespaces/eventhubs/authorizationRules', parameters('namespace'), parameters('hub'), parameters('ruleName')), '2022-10-01-preview').primaryConnectionString]"
  }
}

// ✅ ALTERNATIVE: Use nested deployment outputs for simple values only
"outputs": {
  "simpleName": {
    "value": "[reference('deploymentName').outputs.nameOutput.value]"
  }
}
```

### Error: "The content for this response was already consumed"

**Cause:** Azure CLI bug in versions 2.74.0 and earlier with subscription-level template validation

**Solution:**
```bash
# Update Azure CLI
brew upgrade azure-cli  # macOS
# or
apt-get update && apt-get upgrade azure-cli  # Linux
```

### Error: Resource not found in outputs

**Cause:** Incorrect scope function or missing resource group parameter

**Solution:** Match the scope function to deployment level:
- Resource group deployment → `resourceId()`
- Subscription deployment → `subscriptionResourceId()` or `resourceId(resourceGroupName, ...)`
- Management group deployment → `managementGroupResourceId()`

## Best Practices for Nested Deployments

### Subscription-Level Template with Resource Group Nested Deployment

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "name": "nestedDeployment",
      "resourceGroup": "[parameters('resourceGroupName')]",
      "properties": {
        "mode": "Incremental",
        "expressionEvaluationOptions": {
          "scope": "inner"  // Critical for reference() in nested outputs
        },
        "template": {
          // Inner template deploys to resource group
          "outputs": {
            "resourceName": {
              "value": "[parameters('name')]"  // Simple values work
            }
          }
        }
      }
    }
  ],
  "outputs": {
    // Access nested outputs
    "name": {
      "value": "[reference('nestedDeployment').outputs.resourceName.value]"
    },

    // Use outputs in other functions - pass as parameters, not nested reference
    "connectionString": {
      "value": "[listKeys(resourceId(parameters('resourceGroupName'), 'Microsoft.Storage/storageAccounts', parameters('storageName')), '2023-01-01').keys[0].value]"
    }
  }
}
```

### Key Pattern: Avoid Nested Function Calls

**The Golden Rule:** In outputs, DO NOT nest `reference()` inside `listKeys()` or `resourceId()`.

**Instead:**
1. Pass resource names as parameters
2. Store complex expressions in variables (where allowed)
3. Use nested deployment outputs only for simple string/number values
4. Build resource IDs with parameters, not with reference() results

## Validation Commands

```bash
# Validate subscription-level template
az deployment sub validate \
  --location eastus \
  --template-file azuredeploy.json \
  --parameters @parameters.json

# Validate resource group template
az deployment group validate \
  --resource-group myResourceGroup \
  --template-file azuredeploy.json \
  --parameters @parameters.json

# What-if preview (subscription level)
az deployment sub what-if \
  --location eastus \
  --template-file azuredeploy.json \
  --parameters @parameters.json
```

## Additional Resources

- [ARM Template Functions Reference](https://learn.microsoft.com/azure/azure-resource-manager/templates/template-functions)
- [Resource Functions](https://learn.microsoft.com/azure/azure-resource-manager/templates/template-functions-resource)
- [Deployment Scopes](https://learn.microsoft.com/azure/azure-resource-manager/templates/deploy-to-subscription)
