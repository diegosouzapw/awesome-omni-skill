---
name: azure-deployment
description: Deploys applications to Azure using Azure Dev CLI, Bicep infrastructure as code, and GitHub Actions CI/CD. Use this skill when asked to deploy to Azure, create infrastructure, set up CI/CD, configure Azure resources, or create deployment pipelines.
---

# Azure Deployment Skill

This skill guides deployment of applications to Azure using modern best practices including Azure Dev CLI (azd), Infrastructure as Code with Bicep, and automated CI/CD pipelines.

## When to Use This Skill

- Deploying applications to Azure
- Creating infrastructure as code
- Setting up CI/CD pipelines
- Configuring Azure resources
- Implementing deployment automation

## Prerequisites

Before deploying, read and analyze:
1. `specs/prd.md` - Product requirements to understand application needs
2. `specs/features/*.md` - Feature requirements
3. `specs/adr/*.md` - Architecture decisions and technology stack
4. Codebase structure (`src/backend`, `src/frontend`, etc.)

## Deployment Workflow

### Step 1: Pre-Deployment Analysis

**Understand the application:**
- Review specifications to understand requirements
- Identify required Azure services
- Analyze codebase structure
- Determine compute, storage, and networking needs

### Step 2: Azure Dev CLI Setup

**Initialize azd:**
- Run `azd init` if not already initialized
- Configure `azure.yaml` with service definitions
- Define environments (dev, staging, prod)
- Set up environment-specific configurations

**azure.yaml structure:**
```yaml
name: your-app-name
services:
  api:
    project: ./src/backend
    language: dotnet|python|node
    host: containerapp|appservice
  web:
    project: ./src/frontend
    language: js
    host: staticwebapp
```

### Step 3: Infrastructure as Code (Bicep)

**Bicep best practices:**
- Create all infrastructure in `infra/` folder
- **Use Azure Verified Modules (AVM)** when available
- **Get Azure best practices** before writing Bicep code
- Create `main.bicep` as orchestrator
- Follow Azure naming conventions
- Configure resource tags for cost tracking

**Resource organization:**
```
infra/
├── main.bicep              # Main orchestrator
├── main.parameters.json    # Environment parameters
├── modules/
│   ├── app-service.bicep
│   ├── sql-database.bicep
│   └── storage.bicep
└── resources.bicep         # Shared resources
```

**Critical Bicep guidelines:**
- Use parameters for environment-specific values
- Output connection strings and endpoints
- Enable diagnostic settings on all resources
- Use managed identities for authentication
- Apply least-privilege access controls

### Step 4: GitHub Actions CI/CD

**Create `.github/workflows/deploy.yml`:**
- Build and test on push
- Deploy using `azd` commands
- Set up environment protection rules
- Configure approval gates for production
- Use GitHub secrets for sensitive values

**Workflow structure:**
```yaml
name: Deploy to Azure
on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build application
        run: # build commands
      
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Login to Azure
        uses: azure/login@v1
      - name: Deploy with azd
        run: azd deploy
```

### Step 5: Security & Monitoring

**Security hardening:**
- ✅ Enable **managed identities** for all Azure resources
- ✅ Store secrets in **Azure Key Vault**
- ✅ Use **RBAC** with least-privilege access
- ✅ Enable **private endpoints** where applicable
- ✅ Configure **network security groups**
- ✅ Never commit secrets to source control

**Monitoring setup:**
- ✅ Configure **Application Insights** for observability
- ✅ Enable **diagnostic logging** for all resources
- ✅ Set up **metric alerts** for critical resources
- ✅ Configure **log analytics workspace**
- ✅ Create **availability tests** for endpoints

### Step 6: Deploy & Verify

**Deployment commands:**
```bash
# Provision infrastructure
azd provision

# Deploy application
azd deploy

# Or do both
azd up
```

**Verification checklist:**
- ✅ Infrastructure provisioned successfully
- ✅ Application deployed without errors
- ✅ Health endpoints responding
- ✅ SSL certificates configured
- ✅ Monitoring data flowing to Application Insights
- ✅ DNS and custom domains configured (if applicable)

### Step 7: Documentation

Create `docs/deployment.md` with:
- **Prerequisites** - Azure subscription, permissions needed
- **Setup instructions** - How to run `azd` commands
- **Environment variables** - Required configuration
- **Azure resources** - List of all created resources
- **Monitoring** - How to access logs and metrics
- **Troubleshooting** - Common issues and solutions
- **Runbook** - Common operational procedures

## Tools Priority Order

1. **Azure Dev CLI (azd)** - Primary deployment tool
2. **Bicep Experimental MCP** - For Bicep best practices and Azure Verified Modules
3. **Azure MCP tools** - For Azure operations and best practices
4. **Microsoft Docs MCP** - For official Azure documentation
5. **GitHub MCP** - For creating workflows

## Azure Best Practices

### Always Do
✅ Use Azure Verified Modules when available
✅ Get Azure best practices before writing infrastructure code
✅ Use managed identities instead of connection strings
✅ Enable diagnostic logging from day one
✅ Tag all resources (environment, project, owner, cost-center)
✅ Plan for scalability even if starting small
✅ Use environment-specific parameter files
✅ Implement proper error handling and retries

### Never Do
❌ Hardcode secrets or connection strings
❌ Skip resource tagging
❌ Deploy without monitoring configured
❌ Use admin credentials in connection strings
❌ Create resources without diagnostic settings
❌ Deploy to production without approval gates

## Resource Naming Conventions

Follow Azure naming conventions:
- Use consistent prefixes: `rg-`, `app-`, `sql-`, `kv-`, `st-`
- Include environment: `-dev`, `-staging`, `-prod`
- Include region: `-eastus`, `-westeu`
- Example: `rg-myapp-prod-eastus`

## Cost Management

**Cost optimization strategies:**
- Use appropriate SKUs for environment (dev can use lower tiers)
- Enable autoscaling for production
- Implement resource cleanup for dev/test environments
- Tag resources for cost allocation
- Set up budget alerts
- Use Azure reservations for production workloads

## Common Azure Services Patterns

### Web Application
- **Compute**: Azure App Service or Container Apps
- **Database**: Azure SQL Database or Cosmos DB
- **Storage**: Azure Storage Account (Blob, Table, Queue)
- **Cache**: Azure Cache for Redis
- **CDN**: Azure Front Door or CDN
- **Auth**: Azure AD B2C or App Service Authentication

### API Backend
- **Compute**: Azure Container Apps or App Service
- **API Management**: Azure API Management
- **Database**: Azure SQL or Cosmos DB
- **Message Queue**: Azure Service Bus or Event Grid
- **Functions**: Azure Functions for event-driven tasks

### Static Website + API
- **Frontend**: Azure Static Web Apps
- **Backend**: Azure Functions or Container Apps
- **Database**: Cosmos DB or Azure SQL
- **CDN**: Built-in to Static Web Apps

## Troubleshooting

### Common Issues
- **Deployment failures**: Check `azd` logs and Azure Portal activity log
- **Resource conflicts**: Ensure unique resource names
- **Permission errors**: Verify RBAC assignments
- **Connection failures**: Check network security groups and firewalls
- **SSL issues**: Verify certificate configuration in Azure

## Next Steps

After deployment:
1. Verify application functionality
2. Configure monitoring alerts
3. Set up CI/CD for automatic deployments
4. Document deployment process
5. Plan for disaster recovery
6. Set up staging environment

## Templates

See `templates/bicep-template.md` and `templates/github-actions-template.md` for infrastructure templates.

## Sample Output

See `examples/sample-azure-yaml.md` for a complete deployment example.
