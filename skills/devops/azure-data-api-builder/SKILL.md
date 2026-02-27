---
name: azure-data-api-builder
description: Deploy Data API Builder (DAB) to Azure Container Apps with Azure SQL, Azure Container Registry (ACR), and Azure Developer CLI (azd). Produces Bicep templates, Dockerfile, and azure.yaml. Use when asked to deploy DAB to Azure, create Bicep for DAB, or set up cloud API hosting.
---

# Data API Builder — Azure Deployment

This skill covers deploying **Data API Builder (DAB)** to **Azure Container Apps** with **Azure SQL**, using **Azure Developer CLI (azd)** for orchestration.

For local development, see the `aspire-data-api-builder` or `docker-data-api-builder` skill.

---

## Core Mental Model

- **Always build a custom Docker image** with `dab-config.json` baked in — never use volume mounts in Azure.
- **azd** orchestrates the full deployment: Bicep provisions resources, a post-provision hook deploys content.
- **ACR** hosts all custom images (DAB, web, Inspector).
- **Connection strings use secrets** — never plain-text env vars for credentials.
- **`TrustServerCertificate=true`** is required in all Azure SQL connection strings.

---

## Anti-Patterns (NEVER DO)

- ❌ Azure Files share mount for `dab-config.json`
- ❌ Azure Storage Account for config files
- ❌ Volume mounts in Azure Container Apps for DAB config
- ❌ Any external file storage for DAB config in cloud
- ✅ Custom Docker image with config embedded via `COPY`

**Why custom image?** Config is versioned with the image — immutable, reproducible, no extra infrastructure, faster startup, fewer failure modes.

---

## Architecture

```
azd up
  ├── Bicep provisions:
  │     Azure SQL Server + Database
  │     Azure Container Registry (ACR)
  │     Container Apps Environment
  │     DAB Container App (port 5000)
  │     SQL Commander Container App (port 8080)
  │     Web Container App (port 80)
  │     MCP Inspector Container App (port 80)
  │
  └── post-provision.ps1:
        1. Add client IP to SQL firewall
        2. Build + deploy dacpac schema
        3. Build custom DAB image (with CORS) → push to ACR
        4. Update DAB container app
        5. Build web image → push to ACR
        6. Update web container app
        7. Build Inspector image → push to ACR
        8. Update Inspector container app
```

---

## File Structure

```
project/
  ├── azure.yaml              # azd entry point
  ├── azure/
  │   ├── main.bicep           # Subscription-scoped entry
  │   ├── resources.bicep      # All Azure resources
  │   └── post-provision.ps1   # Content deployment hook
  ├── api/
  │   ├── dab-config.json      # DAB configuration
  │   └── Dockerfile           # Custom DAB image
  └── database/
      ├── database.sqlproj
      ├── Tables/*.sql
      └── Scripts/PostDeployment.sql
```

---

## Templates

### Dockerfile (DAB Custom Image)

```dockerfile
FROM mcr.microsoft.com/azure-databases/data-api-builder:1.7.83-rc
COPY dab-config.json /App/dab-config.json
```

> Two lines. Config baked in. No volume mounts.

### azure.yaml

```yaml
name: my-project
metadata:
  template: my-project

infra:
  provider: bicep
  path: azure
  module: main

hooks:
  postprovision:
    shell: pwsh
    continueOnError: false
    interactive: true
    run: ./azure/post-provision.ps1
```

### main.bicep (Subscription Scope)

```bicep
targetScope = 'subscription'

@minLength(1)
@maxLength(64)
param environmentName string

param location string

@secure()
param sqlAdminPassword string
param sqlAdminUser string = 'sqladmin'

var tags = { 'azd-env-name': environmentName }
param resourceToken string = ''
var effectiveToken = empty(resourceToken)
  ? uniqueString(subscription().id, environmentName, location)
  : resourceToken

resource rg 'Microsoft.Resources/resourceGroups@2024-03-01' = {
  name: 'rg-${environmentName}-${effectiveToken}'
  location: location
  tags: tags
}

module resources 'resources.bicep' = {
  scope: rg
  name: 'resources'
  params: {
    location: location
    tags: tags
    resourceToken: effectiveToken
    sqlAdminUser: sqlAdminUser
    sqlAdminPassword: sqlAdminPassword
  }
}

// Outputs become env vars for post-provision hook
output AZURE_RESOURCE_GROUP string = rg.name
output AZURE_SQL_SERVER_NAME string = resources.outputs.sqlServerName
output AZURE_SQL_SERVER_FQDN string = resources.outputs.sqlServerFqdn
output AZURE_SQL_DATABASE string = 'sql-db'
output AZURE_SQL_ADMIN_USER string = sqlAdminUser
output AZURE_ACR_NAME string = resources.outputs.acrName
output AZURE_CONTAINER_APP_API_NAME string = resources.outputs.dabAppName
output AZURE_CONTAINER_APP_API_FQDN string = resources.outputs.dabFqdn
```

### resources.bicep — DAB Container App

The DAB container app definition within `resources.bicep`:

```bicep
param location string
param tags object
param resourceToken string
param sqlAdminUser string
@secure()
param sqlAdminPassword string

// ── SQL Server + Database ──

resource sqlServer 'Microsoft.Sql/servers@2023-08-01-preview' = {
  name: 'sql-server-${resourceToken}'
  location: location
  tags: tags
  properties: {
    administratorLogin: sqlAdminUser
    administratorLoginPassword: sqlAdminPassword
  }
}

resource sqlFirewallAzure 'Microsoft.Sql/servers/firewallRules@2023-08-01-preview' = {
  parent: sqlServer
  name: 'AllowAzureServices'
  properties: {
    startIpAddress: '0.0.0.0'
    endIpAddress: '0.0.0.0'
  }
}

resource sqlDb 'Microsoft.Sql/servers/databases@2023-08-01-preview' = {
  parent: sqlServer
  name: 'sql-db'
  location: location
  tags: tags
  sku: { name: 'Basic', tier: 'Basic' }
}

// ── Container Registry ──

resource acr 'Microsoft.ContainerRegistry/registries@2023-11-01-preview' = {
  name: 'acr${resourceToken}'
  location: location
  tags: tags
  sku: { name: 'Basic' }
  properties: { adminUserEnabled: true }
}

// ── Container Apps Environment ──

resource cae 'Microsoft.App/managedEnvironments@2024-03-01' = {
  name: 'environment-${resourceToken}'
  location: location
  tags: tags
  properties: {}
}

// ── DAB Container App ──

var sqlConnString = 'Server=tcp:${sqlServer.properties.fullyQualifiedDomainName},1433;Database=sql-db;User Id=${sqlAdminUser};Password=${sqlAdminPassword};Encrypt=true;TrustServerCertificate=true'

resource dabApp 'Microsoft.App/containerApps@2024-03-01' = {
  name: 'data-api-${resourceToken}'
  location: location
  tags: tags
  properties: {
    managedEnvironmentId: cae.id
    configuration: {
      ingress: {
        external: true
        targetPort: 5000
      }
      registries: [
        {
          server: acr.properties.loginServer
          username: acr.listCredentials().username
          passwordSecretRef: 'acr-password'
        }
      ]
      secrets: [
        { name: 'db-conn', value: sqlConnString }
        { name: 'acr-password', value: acr.listCredentials().passwords[0].value }
      ]
    }
    template: {
      containers: [
        {
          name: 'dab-api'
          image: 'mcr.microsoft.com/azure-databases/data-api-builder:1.7.83-rc'
          resources: { cpu: json('0.5'), memory: '1Gi' }
          env: [
            { name: 'MSSQL_CONNECTION_STRING', secretRef: 'db-conn' }
          ]
        }
      ]
      scale: { minReplicas: 1, maxReplicas: 1 }
    }
  }
}

// ── Outputs ──

output sqlServerName string = sqlServer.name
output sqlServerFqdn string = sqlServer.properties.fullyQualifiedDomainName
output acrName string = acr.name
output dabAppName string = dabApp.name
output dabFqdn string = dabApp.properties.configuration.ingress.fqdn
```

> **Note:** Bicep provisions with a placeholder image. The post-provision hook replaces it with the custom ACR image.

---

## Connection Strings

### Azure SQL (SQL Auth)

```
Server=tcp:<server>.database.windows.net,1433;Database=sql-db;User Id=sqladmin;Password=<password>;Encrypt=true;TrustServerCertificate=true
```

### Azure SQL (Managed Identity)

```
Server=tcp:<server>.database.windows.net,1433;Database=sql-db;Authentication=Active Directory Default;Encrypt=true;TrustServerCertificate=true
```

> **CRITICAL:** `TrustServerCertificate=true` is required — DAB and SQL Commander will not connect without it.

---

## CORS Configuration

DAB's `dab-config.json` needs the web app's Azure origin in its CORS config. Use a placeholder that gets replaced during image build:

### dab-config.json (with placeholder)

```json
"cors": {
  "origins": ["http://localhost:5173", "__WEB_URL_AZURE__"],
  "allow-credentials": false
}
```

### Replacement in post-provision.ps1

```powershell
$webOrigin = "https://$webFqdn"
$dabConfig = Get-Content -Path "api/dab-config.json" -Raw
$dabConfig = $dabConfig.Replace("__WEB_URL_AZURE__", $webOrigin)
$dabConfig | Out-File -FilePath "api/dab-config.json" -Encoding utf8 -Force
```

> This replacement happens before `az acr build`, so the correct origin is baked into the image.

---

## Post-Provision Hook Pattern

The `post-provision.ps1` script runs after Bicep provisions all resources. It handles content that can't be done declaratively:

```powershell
$ErrorActionPreference = "Stop"

# Variables from Bicep outputs (set by azd as env vars)
$resourceGroup    = $env:AZURE_RESOURCE_GROUP
$sqlServerName    = $env:AZURE_SQL_SERVER_NAME
$sqlServerFqdn    = $env:AZURE_SQL_SERVER_FQDN
$sqlDb            = $env:AZURE_SQL_DATABASE
$sqlAdminUser     = $env:AZURE_SQL_ADMIN_USER
$sqlAdminPassword = $env:AZURE_SQL_ADMIN_PASSWORD
$acrName          = $env:AZURE_ACR_NAME
$dabAppName       = $env:AZURE_CONTAINER_APP_API_NAME
$dabFqdn          = $env:AZURE_CONTAINER_APP_API_FQDN

$sqlConn = "Server=tcp:$sqlServerFqdn,1433;Database=$sqlDb;User Id=$sqlAdminUser;Password=$sqlAdminPassword;Encrypt=true;TrustServerCertificate=true"

# ── 1. Open SQL firewall for local machine ──
$myIp = (Invoke-WebRequest -Uri "https://api.ipify.org" -UseBasicParsing).Content
az sql server firewall-rule create `
    --resource-group $resourceGroup `
    --server $sqlServerName `
    --name "azd-deploy-client" `
    --start-ip-address $myIp `
    --end-ip-address $myIp 2>$null | Out-Null

# ── 2. Deploy database schema (dacpac) ──
dotnet build database/database.sqlproj -c Release
sqlpackage /Action:Publish `
    /SourceFile:database/bin/Release/database.dacpac `
    /TargetConnectionString:"$sqlConn" `
    /p:BlockOnPossibleDataLoss=false

# ── 3. Build custom DAB image with CORS → push to ACR ──
$apiDeployDir = "api-deploy-temp"
Copy-Item -Path "api" -Destination $apiDeployDir -Recurse -Force
$webOrigin = "https://$env:AZURE_WEB_APP_FQDN"
$dabConfig = Get-Content -Path "$apiDeployDir/dab-config.json" -Raw
$dabConfig = $dabConfig.Replace("__WEB_URL_AZURE__", $webOrigin)
$dabConfig | Out-File -FilePath "$apiDeployDir/dab-config.json" -Encoding utf8 -Force
az acr build --registry $acrName --image dab-api:latest --file "$apiDeployDir/Dockerfile" $apiDeployDir/
Remove-Item $apiDeployDir -Recurse -Force

# ── 4. Update DAB container app with custom image ──
az containerapp update `
    --name $dabAppName `
    --resource-group $resourceGroup `
    --image "$acrName.azurecr.io/dab-api:latest"
```

> Steps 5-8 handle web app, Inspector — see `azure-mcp-inspector` and `azure-sql-commander` skills.

---

## Deployment Commands

### Full deployment (recommended)

```powershell
azd up
```

This runs Bicep provisioning + post-provision hook in one command.

### Re-deploy DAB only (after config change)

```powershell
# Replace CORS placeholder
$dabConfig = Get-Content -Path "api/dab-config.json" -Raw
$dabConfig = $dabConfig.Replace("__WEB_URL_AZURE__", $webOrigin)
$dabConfig | Out-File -FilePath "api-deploy-temp/dab-config.json" -Encoding utf8

# Build and push
az acr build --registry <acr-name> --image dab-api:latest --file api/Dockerfile api/

# Update container
az containerapp update --name <app-name> --resource-group <rg> --image <acr>.azurecr.io/dab-api:latest
```

### Re-deploy schema only

```powershell
dotnet build database/database.sqlproj -c Release
sqlpackage /Action:Publish `
    /SourceFile:database/bin/Release/database.dacpac `
    /TargetConnectionString:"Server=tcp:<fqdn>,1433;Database=sql-db;User Id=sqladmin;Password=<pw>;Encrypt=true;TrustServerCertificate=true" `
    /p:BlockOnPossibleDataLoss=false
```

---

## Verification

After deployment, verify DAB is healthy:

```powershell
$r = Invoke-WebRequest -Uri "https://<dab-fqdn>/health" -UseBasicParsing
$r.StatusCode  # Should be 200
```

Also check:
- REST: `https://<dab-fqdn>/api/<entity>`
- GraphQL: `https://<dab-fqdn>/graphql`
- MCP: `https://<dab-fqdn>/mcp`
- Swagger: `https://<dab-fqdn>/swagger`

---

## Common Issues

### DAB container fails to start
- Check connection string secret is correct in Container Apps
- Verify Azure SQL firewall allows Azure services (`0.0.0.0` rule)
- Check `TrustServerCertificate=true` is in the connection string

### Schema not found (404 on entities)
- Database schema must be deployed **before** the DAB image is built
- Post-provision runs dacpac first, then builds the DAB image — this order matters

### CORS errors from web app
- The `__WEB_URL_AZURE__` placeholder must be replaced before `az acr build`
- Verify the replacement happened: check post-provision output for "CORS origin set to..."
- After adding a new web origin, rebuild and push the DAB image

### ACR build fails
- Verify ACR admin is enabled: `az acr show --name <acr> --query adminUserEnabled`
- Check Dockerfile path is correct relative to build context

### Container app shows old image
- Azure caches images. After pushing, update the container:
  ```powershell
  az containerapp update --name <app> --resource-group <rg> --image <acr>.azurecr.io/dab-api:latest
  ```

---

## Key Differences: Local vs Azure

| Aspect | Local (Docker/Aspire) | Azure |
|--------|----------------------|-------|
| **Config mount** | Bind mount `:ro` | Baked into image |
| **Connection** | `Server=sql-2025` (service name) | `Server=tcp:<fqdn>,1433` |
| **CORS** | `*` or `localhost` | Specific `https://` origin |
| **Secrets** | `.env` file | Container Apps secrets |
| **Schema deploy** | `sqlpackage` to `localhost` | `sqlpackage` to Azure SQL FQDN |
| **Image source** | Docker Hub / MCR direct | ACR (custom image) |

---

## Related Skills

- **`aspire-data-api-builder`** — Local dev with .NET Aspire
- **`docker-data-api-builder`** — Local dev with Docker Compose
- **`data-api-builder-config`** — DAB config file reference
- **`data-api-builder-cli`** — DAB CLI commands
- **`azure-sql-commander`** — Deploy SQL Commander to Azure
- **`azure-mcp-inspector`** — Deploy MCP Inspector to Azure
