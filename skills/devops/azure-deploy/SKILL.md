---
name: azure-deploy
description: Deploy applications to Azure App Service, Azure Functions, and Static Web Apps. Analyzes projects to recommend services, provides local preview, and guides deployment. Use phrases like "what Azure service should I use", "analyze my project for Azure", "preview locally", "guide me through deployment".
---

# Azure Deploy Skill

Deploy applications to Azure with intelligent service selection, local preview, and guided deployment workflows.

## CRITICAL: When to Use This Skill

**USE azure-deploy for ANY Azure deployment question.** This skill detects your app type, recommends the best service, and guides you through deployment.

**ALWAYS use azure-deploy when the user asks about:**

| User Question Pattern | Example | Action |
|-----------------------|---------|--------|
| What Azure service to use | "What Azure service should I use for this?" | Run detection workflow |
| Analyze project for Azure | "Analyze my project for Azure deployment" | Scan files, recommend service |
| App type detection | "Detect my app type" / "Is this a static site?" | Check package.json, configs |
| Service comparison | "Should I use App Service or Functions?" | Compare based on app structure |
| Local preview/testing | "Preview this locally" / "Test before deploying" | Provide local dev commands |
| Step-by-step deployment | "Guide me through Azure deployment" | Run full deployment workflow |
| Package and deploy | "Package and deploy my app" | Build + deploy commands |
| Monorepo deployment | "Deploy my monorepo to Azure" | Recommend azd + IaC |
| Infrastructure as Code | "Set up IaC for this" / "Use azd" | azure.yaml + Bicep guidance |
| Framework-specific | "Deploy my Next.js/React/Flask app" | Detect framework, recommend service |

**Explicit Invocation (most reliable):**
```
"@azure-deploy analyze my project"
"Use the azure-deploy skill to deploy this"
```

**NOTE:** Avoid generic phrases like "deploy to Azure" - they may trigger Azure MCP tools instead.

---

## Quick Start Decision Tree

```
User wants to deploy → Run detection workflow below
```

---

## Phase 1: Application Detection

**ALWAYS start by scanning the user's project to detect the application type.**

### Step 1.1: Check for Existing Azure Configuration

Look for these files first (HIGH confidence signals):

| File Found | Recommendation | Action |
|------------|----------------|--------|
| `azure.yaml` | Already configured for azd | Use `azd up` to deploy |
| `function.json` or `host.json` | Azure Functions project | Deploy as Functions |
| `staticwebapp.config.json` | Static Web Apps project | Deploy as SWA |

If found, skip to the appropriate deployment section.

### Step 1.2: Detect Application Framework

Scan for configuration files and dependencies:

**Node.js / JavaScript / TypeScript:**
```
package.json exists →
├── next.config.js/mjs/ts → Next.js
│   ├── Has `output: 'export'` → Static Web Apps (SSG)
│   └── Has API routes or no export config → App Service (SSR)
├── nuxt.config.ts/js → Nuxt
│   ├── Has `ssr: false` or `target: 'static'` → Static Web Apps
│   └── Otherwise → App Service (SSR)
├── angular.json → Angular → Static Web Apps
├── vite.config.* → Vite-based (React/Vue/Svelte) → Static Web Apps
├── gatsby-config.js → Gatsby → Static Web Apps
├── astro.config.mjs → Astro → Static Web Apps
├── nest-cli.json → NestJS → App Service
├── Has express/fastify/koa/hapi dependency → App Service
└── No framework, just static build → Static Web Apps
```

**Python:**
```
requirements.txt or pyproject.toml exists →
├── function_app.py exists → Azure Functions (v2 programming model)
├── Has flask dependency → App Service
├── Has django dependency → App Service
├── Has fastapi dependency → App Service
└── Has azure-functions dependency → Azure Functions
```

**.NET:**
```
*.csproj or *.sln exists →
├── <AzureFunctionsVersion> in csproj → Azure Functions
├── Blazor WebAssembly project → Static Web Apps
├── ASP.NET Core web app → App Service
└── .NET API project → App Service
```

**Java:**
```
pom.xml or build.gradle exists →
├── Has azure-functions-* dependency → Azure Functions
├── Has spring-boot dependency → App Service
└── Standard web app → App Service
```

**Static Only:**
```
index.html exists + no package.json/requirements.txt →
└── Pure static site → Static Web Apps
```

### Step 1.3: Detect Multi-Service Architecture

Check for complexity indicators that suggest azd + IaC:

```
Multi-service triggers:
├── Monorepo structure (frontend/, backend/, api/, packages/, apps/)
├── docker-compose.yml with multiple services
├── Multiple package.json in different subdirectories
├── Database references in config (connection strings, .env files)
├── References to Redis, Service Bus, Event Hubs, Storage queues
├── User mentions "multiple environments", "staging", "production"
└── More than one deployable component detected
```

**If multi-service detected → Recommend azd + Infrastructure as Code**
See [Multi-Service Deployment Guide](./reference/multi-service.md)

### Step 1.4: Confidence Assessment

After detection, assess confidence:

| Confidence | Criteria | Action |
|------------|----------|--------|
| **HIGH** | Azure config file found (azure.yaml, function.json, staticwebapp.config.json) | Proceed with detected service |
| **MEDIUM** | Framework detected from dependencies | Explain recommendation, ask for confirmation |
| **LOW** | Ambiguous or no clear signals | Ask clarifying questions |

**Clarifying questions for LOW confidence:**
1. "What type of application is this? (static website, API, full-stack, serverless functions)"
2. "Does your app need server-side rendering or is it purely client-side?"
3. "Will you need a database, caching, or other Azure services?"

---

## Phase 2: Local Preview (No Azure Auth Required)

Before deploying, help users test locally.

### ⛔ CRITICAL: macOS Compatibility

**NEVER use `detach: true` on macOS** - it WILL FAIL with "setsid: command not found".

| ❌ WRONG (fails on macOS) | ✅ CORRECT |
|---------------------------|------------|
| `mode: "async", detach: true` | Keep in same session + open browser |
| Backgrounding with `&` alone | Start server, wait, then `open` URL |
| `nohup` (session dies anyway) | All in one command chain |

### The Working Pattern for Local Preview

**The server dies when the session ends.** The solution is to start the server and open the browser in ONE session before it can close:

```bash
# Start server, wait for init, open browser - ALL IN ONE COMMAND
cd /path/to/project && npm run dev &
sleep 2
open http://localhost:5173/
```

**Why this works:**
1. Server starts in background (`&`)
2. `sleep 2` gives it time to bind to port
3. `open` launches browser immediately
4. Browser connection keeps the server alive

**Why previous approaches fail:**
| Approach | Problem |
|----------|---------|
| `detach: true` | `setsid` doesn't exist on macOS - silent failure |
| `&` backgrounding alone | Server killed when bash session ends |
| `nohup` | Session still terminates, server dies |

### Running Dev Servers in Copilot CLI

**Correct approach - all in one session:**

```bash
# Vite/React/Vue
cd /path/to/project && npm run dev & sleep 2 && open http://localhost:5173/

# Next.js
cd /path/to/project && npm run dev & sleep 3 && open http://localhost:3000/

# Flask
cd /path/to/project && flask run & sleep 2 && open http://localhost:5000/
```

**Server verification (if not opening browser):**
```bash
# Check if responding (returns HTTP status code)
curl -s -o /dev/null -w "%{http_code}" http://localhost:5173/
# 200 = success, 000 = not responding
```

### Local Preview Strategy

**IMPORTANT:** SWA CLI can have issues with session management, especially on macOS. Always have fallback options ready.

| Method | Best For | Reliability |
|--------|----------|-------------|
| `npm run dev` | Development with HMR | ✅ Most reliable |
| `npm run preview` | Test production build | ✅ Very reliable |
| `npx serve dist` | Any static build | ✅ Very reliable |
| `swa start` | SWA with API integration | ⚠️ Session issues on macOS |

### Recommended: Framework Dev Server (Most Reliable)

For SPAs, use the framework's dev server:

```bash
# Vite (React, Vue, Svelte)
npm run dev
# Default: http://localhost:5173 (auto-increments if occupied)

# Next.js
npm run dev
# Default: http://localhost:3000

# Angular
ng serve
# Default: http://localhost:4200
```

### Production Build Preview

To test the actual build output:

```bash
# Build first
npm run build

# Preview the production build
npm run preview
# or with host flag for network access
npm run preview -- --host
```

**Fallback for any static site:**
```bash
# Simple static server (works everywhere)
npx serve dist

# Or with Python (no npm needed)
cd dist && python3 -m http.server 8080
```

### SWA CLI - Local Preview (ALWAYS Run Interactively)

> ✅ **Best Practice:** Always run SWA CLI interactively (async mode, not detached). This ensures routing rules from `staticwebapp.config.json` are applied.

```bash
# Start SWA CLI interactively (recommended)
npx --yes @azure/static-web-apps-cli start ./dist --port 4280

# Use mode: "async" in Copilot CLI, NOT detach: true
```

**When to use SWA CLI instead of `npm run dev`:**
- Need `staticwebapp.config.json` routing rules (navigation fallback, headers)
- Testing authentication/authorization
- Testing with API integration (`--api-location`)

**Stopping SWA CLI:**
```bash
# Stop the server when done
pkill -f "swa"
```

**If SWA CLI fails:**
1. Use `npm run dev` for development (no routing rules)
2. Use `npm run preview` for production build testing
3. Use `npx serve dist` as universal fallback
4. For full SWA features, deploy to Azure and test there

### Azure Functions - Local Preview
```bash
# Install Azure Functions Core Tools (one-time)
npm install -g azure-functions-core-tools@4

# Start local Functions runtime
func start

# With specific port
func start --port 7071
```

### App Service Apps - Local Preview
Use the framework's built-in dev server:

```bash
# Node.js
npm run dev
# or
npm start

# Python Flask
flask run

# Python FastAPI
uvicorn main:app --reload

# .NET
dotnet run

# Java Spring Boot
./mvnw spring-boot:run
```

See [Local Preview Guide](./reference/local-preview.md) for detailed setup and troubleshooting.

---

## Phase 3: Prerequisites & Dependency Management

**ALWAYS check and install missing dependencies before proceeding.**

### 3.1 Azure Authentication (Auto-Login)

**Check login status and automatically login if needed:**
```bash
# Check if logged in, auto-login if not
if ! az account show &>/dev/null; then
    echo "Not logged in to Azure. Starting login..."
    az login
fi

# Verify and show current subscription
az account show --query "{name:name, id:id}" -o table
```

If the user has multiple subscriptions, help them select the correct one:
```bash
# List all subscriptions
az account list --query "[].{Name:name, ID:id, Default:isDefault}" -o table

# Set subscription
az account set --subscription "<name-or-id>"
```

### 3.2 Dependency Detection & Auto-Install

Run this check first and install any missing tools:

```bash
# Check all dependencies at once
check_deps() {
  local missing=()
  command -v az &>/dev/null || missing+=("azure-cli")
  command -v func &>/dev/null || missing+=("azure-functions-core-tools")
  command -v swa &>/dev/null || missing+=("@azure/static-web-apps-cli")
  command -v azd &>/dev/null || missing+=("azd")
  echo "${missing[@]}"
}
```

### 3.3 Install Missing Tools

**Azure CLI** (required for all deployments):
```bash
# macOS
brew install azure-cli

# Windows (PowerShell)
winget install Microsoft.AzureCLI

# Linux (Ubuntu/Debian)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

**Azure Functions Core Tools** (for Functions projects):
```bash
# npm (all platforms)
npm install -g azure-functions-core-tools@4

# macOS
brew tap azure/functions && brew install azure-functions-core-tools@4

# Windows
winget install Microsoft.AzureFunctionsCoreTools
```

**Static Web Apps CLI** (for SWA projects):
```bash
npm install -g @azure/static-web-apps-cli
```

**Azure Developer CLI** (for multi-service/IaC):
```bash
# macOS
brew install azd

# Windows
winget install Microsoft.Azd

# Linux
curl -fsSL https://aka.ms/install-azd.sh | bash
```

### 3.4 Project Dependencies

Detect and install project-level dependencies:

```bash
# Node.js - install if node_modules missing
[ -f "package.json" ] && [ ! -d "node_modules" ] && npm install

# Python - create venv and install if missing  
[ -f "requirements.txt" ] && [ ! -d ".venv" ] && python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt

# .NET - restore packages
[ -f "*.csproj" ] && dotnet restore

# Java - install dependencies
[ -f "pom.xml" ] && mvn dependency:resolve
```

---

## Phase 4: Single-Service Deployment (Azure CLI)

### 4.1 Static Web Apps Deployment

**Create resource and deploy:**
```bash
# Create resource group (if needed)
az group create --name <resource-group> --location <location>

# Create Static Web App
az staticwebapp create \
  --name <app-name> \
  --resource-group <resource-group> \
  --location <location> \
  --sku Free

# Get deployment token
az staticwebapp secrets list \
  --name <app-name> \
  --resource-group <resource-group> \
  --query "properties.apiKey" -o tsv

# Deploy with SWA CLI
swa deploy ./dist \
  --deployment-token <token> \
  --env production
```

**Plain HTML Site (Single File or No Build Step):**

For plain HTML sites without a build process, SWA CLI requires content in a dedicated folder:
```bash
# Create output directory and copy files
mkdir -p dist && cp -r *.html *.css *.js *.png *.jpg *.svg dist/ 2>/dev/null || true

# Get deployment token
TOKEN=$(az staticwebapp secrets list \
  --name <app-name> \
  --resource-group <resource-group> \
  --query "properties.apiKey" -o tsv)

# Deploy from dist folder
swa deploy ./dist --deployment-token "$TOKEN" --env production

# Clean up temp folder (optional)
rm -rf dist
```

**Smart defaults:**
- SKU: `Free` for dev/test, `Standard` for production
- Location: SWA has limited regions - use `centralus`, `eastus2`, `westus2`, `westeurope`, or `eastasia`

See [Static Web Apps Guide](./reference/static-web-apps.md) for detailed configuration.

### 4.2 Azure Functions Deployment

**Create and deploy:**
```bash
# Create resource group
az group create --name <resource-group> --location <location>

# Create storage account (required for Functions)
az storage account create \
  --name <storage-name> \
  --resource-group <resource-group> \
  --location <location> \
  --sku Standard_LRS

# Create Function App
az functionapp create \
  --name <app-name> \
  --resource-group <resource-group> \
  --storage-account <storage-name> \
  --consumption-plan-location <location> \
  --runtime <node|python|dotnet|java> \
  --runtime-version <version> \
  --functions-version 4

# Deploy with func CLI
func azure functionapp publish <app-name>
```

**Smart defaults:**
- Plan: Consumption (pay-per-execution) for most cases
- Runtime version: Latest LTS for the detected language

See [Azure Functions Guide](./reference/functions.md) for advanced scenarios.

### 4.3 App Service Deployment

**Create and deploy:**
```bash
# Create resource group
az group create --name <resource-group> --location <location>

# Create App Service plan
az appservice plan create \
  --name <plan-name> \
  --resource-group <resource-group> \
  --location <location> \
  --sku B1 \
  --is-linux

# Create Web App
az webapp create \
  --name <app-name> \
  --resource-group <resource-group> \
  --plan <plan-name> \
  --runtime "<runtime>"

# Deploy code (zip deploy)
az webapp deploy \
  --name <app-name> \
  --resource-group <resource-group> \
  --src-path <path-to-zip> \
  --type zip
```

**Runtime values by language:**
- Node.js: `"NODE:18-lts"`, `"NODE:20-lts"`
- Python: `"PYTHON:3.11"`, `"PYTHON:3.12"`
- .NET: `"DOTNETCORE:8.0"`
- Java: `"JAVA:17-java17"`

**Smart defaults:**
- Plan SKU: `B1` for dev/test, `P1v3` for production
- Always use Linux (`--is-linux`) unless .NET Framework required

See [App Service Guide](./reference/app-service.md) for configuration options.

---

## Phase 5: Multi-Service Deployment (azd + IaC)

When multiple services or infrastructure dependencies are detected, recommend Azure Developer CLI with Infrastructure as Code.

### When to Use azd
- Multiple deployable components (frontend + API + functions)
- Needs database, cache, storage, or messaging services
- Requires consistent environments (dev, staging, production)
- Team collaboration with reproducible infrastructure

### Initialize azd Project
```bash
# Initialize from scratch
azd init

# Or use a template
azd init --template <template-name>
```

### Project Structure
```
project/
├── azure.yaml              # azd configuration
├── infra/
│   ├── main.bicep          # Main infrastructure
│   ├── main.parameters.json
│   └── modules/            # Reusable modules
├── src/
│   ├── web/                # Frontend
│   └── api/                # Backend
```

### Deploy with azd
```bash
# Provision infrastructure + deploy code
azd up

# Or separately:
azd provision  # Create infrastructure
azd deploy     # Deploy application code

# Manage environments
azd env new staging
azd env select staging
azd up
```

See [Multi-Service Guide](./reference/multi-service.md) for azure.yaml configuration.
See [Azure Verified Modules](./reference/azure-verified-modules.md) for Bicep module reference.

---

## Troubleshooting Quick Reference

### Common Issues

**"Not logged in" errors:**
```bash
az login
az account set --subscription "<name>"
```

**"Resource group not found":**
```bash
az group create --name <name> --location <location>
```

**SWA deployment fails:**
- Check build output directory is correct
- Verify deployment token is valid
- Ensure `staticwebapp.config.json` is properly formatted

**Functions deployment fails:**
- Verify `host.json` exists
- Check runtime version matches function app configuration
- Ensure storage account is accessible

**App Service deployment fails:**
- Verify runtime matches application
- Check startup command if using custom entry point
- Review deployment logs: `az webapp log tail --name <app> --resource-group <rg>`

See [Troubleshooting Guide](./reference/troubleshooting.md) for detailed solutions.

---

## Reference Files

Load these guides as needed for detailed information:

- [📦 App Service Guide](./reference/app-service.md) - Full App Service deployment reference
- [⚡ Azure Functions Guide](./reference/functions.md) - Functions deployment and configuration
- [🌐 Static Web Apps Guide](./reference/static-web-apps.md) - SWA deployment and configuration
- [🖥️ Local Preview Guide](./reference/local-preview.md) - Local development setup
- [🏗️ Multi-Service Guide](./reference/multi-service.md) - azd and IaC patterns
- [📚 Azure Verified Modules](./reference/azure-verified-modules.md) - Bicep module reference
- [🔧 Troubleshooting Guide](./reference/troubleshooting.md) - Common issues and fixes
- [📋 Common Patterns](./reference/common-patterns.md) - Shared commands (DRY reference)
