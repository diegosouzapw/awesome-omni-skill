---
name: frontend-deployment
description: Deploy frontend applications from aramb.toml. Creates frontend service, resolves backend references from deployment outputs, builds static files, and deploys with environment variables. Returns deployment URL. Use for all frontend deployments.
category: deployment
tags: [frontend, deployment, static, toml, react, vue, nextjs, angular, build, env]
license: MIT
---

# Frontend Deployment

Deploy frontend applications from aramb.toml configuration. Creates service, resolves backend references, and deploys with environment variables.

## Session Continuity

Your session persists. You may be started fresh OR resumed with new context.

### Trigger Types

| Trigger | Meaning |
|---------|---------|
| `start` | Normal task execution (first time) |
| `resume` | User provided additional context |
| `task_chat` | Direct message from task chat UI |

### Resume Trigger Format

When resumed, you receive:
```
## Task Resumed

The user has provided additional context:

<user's message>

Your previous status: <completed/failed>
You have full context of your previous work in this session.
```

### How to Handle Resume

| Previous Status | User Intent | Action |
|-----------------|-------------|--------|
| `failed` | Providing fix info | Retry with new context |
| `completed` | Wants redeployment | Redeploy or update |
| `completed` | Asking question | Answer from your context |
| `in_progress` | Adding context | Incorporate and continue |

### Q&A Mode

If resumed with `mode="qa"`:
- Only answer questions
- Do NOT perform new deployments
- Use `TaskChatResponse` to reply

## Role

You are a frontend deployment specialist that supports two deployment modes: TOML-based (when aramb.toml exists) and auto-create (when no TOML). Detects frameworks, builds static files, deploys services, and returns deployment ID and URL from aramb-cli.

## Deployment Flow (New)

**Standard Flow** (with aramb.toml):
1. Read aramb.toml configuration
2. Create frontend service: `aramb services create --name {name} --type frontend --application {app_id} --tags frontend,aramb`
3. Get backend deployment outputs (if references exist): `aramb deploy details --service {backend_slug} -o json`
4. Build static files if needed
5. Deploy with environment variables: `aramb deploy {static_dir}/ --service {slug} --env KEY=VALUE`

**Key Principles:**
- ✅ Create service first before deployment
- ✅ Resolve backend references from deployment outputs
- ✅ Set environment variables during deployment
- ❌ Any error → EXIT immediately (no recovery)

## Task Chat Communication

Send progress updates to the task chat so users can follow along. Use `TaskUserResponse` MCP tool for key milestones:

**When to send updates:**
- **Starting**: What you're deploying and the deployment mode
- **Key milestones**: Build complete, deployment started
- **Completion**: Final URL and deployment status

**Example:**
```
TaskUserResponse(message="🚀 Starting frontend deployment. Detected React project, building static files...")
```

```
TaskUserResponse(message="📦 Build complete! Deploying to aramb...")
```

```
TaskUserResponse(message="✅ Deployed! URL: https://my-app-web.aramb.dev - Service healthy and ready.")
```

```
TaskUserResponse(message="❌ Deployment failed: Build error in React project. See output for details.")
```

Keep messages concise. Focus on status and final URL.

## Responsibilities

- Read aramb.toml configuration file
- Create frontend service with proper metadata
- Extract frontend service details from TOML
- Resolve backend references from deployment outputs
- Detect if static files exist or need building
- Build static files if needed using appropriate framework
- Prepare environment variables with resolved references
- Deploy static files with environment variables
- Validate deployment successful
- Return deployment URL and status

## Constraints

### Strict Flow Requirements

- **MUST** install aramb-cli as FIRST and FOREMOST step (Step 0) - **CRITICAL**
- **MUST** exit immediately if aramb-cli installation fails
- **MUST NOT** attempt to debug or fix installation failures
- **MUST** have aramb.toml in project root
- **MUST** have APPLICATION_ID environment variable set
- **MUST** have ARAMB_API_TOKEN environment variable set
- **MUST** create frontend service before deployment (Step 2) - **CRITICAL**
- **MUST** resolve backend references from deployment outputs (if exist)
- **MUST** set environment variables during deployment
- **MUST** use service SLUG (not name) for all commands

### Exit Immediately If:

- aramb-cli installation fails (Step 0) - **EXIT, do NOT debug or fix**
- aramb.toml not found (Step 1) - **EXIT**
- Required environment variables not set - **EXIT**
- Frontend service creation fails (Step 2) - **EXIT**
- Static file build fails (Step 5) - **EXIT**
- Deployment fails (Step 7) - **EXIT**

### No Recovery Allowed

- **NO** retry logic
- **NO** debugging
- **NO** error recovery
- **NO** alternative flows
- **EXIT** with clear error message

## Inputs

- `project_path`: Frontend project directory (default: current directory)
- `service_name`: Service name to create (default: auto-generate from directory name)
- `port`: Service port (default: 8080)
- `force_build`: Force rebuild even if static files exist (default: false)

## Workflow

### 0. Install aramb-cli (CRITICAL FIRST STEP)

**IMPORTANT: This is the first and foremost step. If installation fails, EXIT immediately. Do NOT attempt to debug or fix.**

```bash
# Check if aramb-cli is installed
if ! command -v aramb &> /dev/null; then
  echo "aramb-cli not found. Installing..."

  # Detect OS and architecture
  OS=$(uname -s | tr '[:upper:]' '[:lower:]')
  ARCH=$(uname -m)

  # Map architecture names
  if [ "$ARCH" = "x86_64" ]; then
    ARCH="amd64"
  elif [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "arm64" ]; then
    ARCH="arm64"
  fi

  # Construct binary name
  BINARY_NAME="aramb-${OS}-${ARCH}"

  # Download latest release
  echo "Downloading ${BINARY_NAME}..."
  curl -LO "https://github.com/aramb-ai/release-beta/releases/latest/download/${BINARY_NAME}"

  if [ $? -ne 0 ]; then
    echo "ERROR: Failed to download aramb-cli"
    exit 1
  fi

  # Make executable and install
  chmod +x "${BINARY_NAME}"
  sudo mv "${BINARY_NAME}" /usr/local/bin/aramb

  if [ $? -ne 0 ]; then
    echo "ERROR: Failed to install aramb-cli to /usr/local/bin/aramb"
    exit 1
  fi

  echo "✓ aramb-cli installed successfully"
else
  echo "✓ aramb-cli already installed ($(aramb --version 2>/dev/null || echo 'version unknown'))"
fi
```

**Exit if:** Download fails OR installation fails. Do NOT attempt to debug or fix.

### 1. Read aramb.toml

```bash
# Check aramb.toml exists
if [ ! -f "aramb.toml" ]; then
  echo "ERROR: aramb.toml not found in current directory"
  exit 1
fi

echo "✓ Found aramb.toml"

# Verify ARAMB_API_TOKEN
[ -n "$ARAMB_API_TOKEN" ] || { echo "ERROR: ARAMB_API_TOKEN not set"; exit 1; }

# Verify APPLICATION_ID
[ -n "$APPLICATION_ID" ] || { echo "ERROR: APPLICATION_ID not set"; exit 1; }
```

**Exit if:** aramb.toml doesn't exist OR required environment variables not set

---

### 2. Create Frontend Service

```bash
# Extract frontend service details from aramb.toml
FRONTEND_NAME=$(grep -A 10 'type = "frontend"' aramb.toml | grep 'name = ' | head -1 | cut -d'"' -f2)
FRONTEND_DESC=$(grep -A 10 'type = "frontend"' aramb.toml | grep 'description = ' | head -1 | cut -d'"' -f2)

if [ -z "$FRONTEND_NAME" ]; then
  echo "ERROR: No frontend service found in aramb.toml"
  exit 1
fi

echo "Creating frontend service: $FRONTEND_NAME"

# Create frontend service
aramb services create \
  --name "$FRONTEND_NAME" \
  --type frontend \
  --description "${FRONTEND_DESC:-Frontend service}" \
  --application "$APPLICATION_ID" \
  --tags frontend,aramb

if [ $? -ne 0 ]; then
  echo "ERROR: Failed to create frontend service"
  exit 1
fi

echo "✓ Frontend service created: $FRONTEND_NAME"

# Get the created service slug
FRONTEND_SLUG=$(grep -A 10 'type = "frontend"' aramb.toml | grep 'slug = ' | head -1 | cut -d'"' -f2)
```

**Exit if:** Service creation fails OR frontend service not found in TOML

---

### 3. Resolve Backend References

```bash
# Check if frontend configuration has references (e.g., backend_url)
# Extract environment variables that reference backend outputs
BACKEND_REF=$(grep -A 20 'type = "frontend"' aramb.toml | grep -E 'env.*backend' || echo "")

if [ -n "$BACKEND_REF" ]; then
  echo "Found backend references, resolving..."

  # Get backend service slug from TOML
  BACKEND_SLUG=$(grep -A 5 'type = "backend"' aramb.toml | grep 'slug = ' | head -1 | cut -d'"' -f2)

  if [ -n "$BACKEND_SLUG" ]; then
    # Get backend deployment outputs
    BACKEND_OUTPUTS=$(aramb deploy details --service "$BACKEND_SLUG" -o json)

    if [ $? -ne 0 ]; then
      echo "WARNING: Could not get backend deployment outputs"
    else
      # Extract PUBLIC_URL from backend outputs
      BACKEND_URL=$(echo "$BACKEND_OUTPUTS" | jq -r '.outputs.PUBLIC_URL // empty')

      if [ -n "$BACKEND_URL" ]; then
        echo "✓ Resolved backend URL: $BACKEND_URL"
        # Store for later use in deployment
        export BACKEND_PUBLIC_URL="$BACKEND_URL"
      fi
    fi
  fi
else
  echo "ℹ No backend references found"
fi
```

**Note:** Backend references are optional. If not found, deployment continues without them.

---

### 4. Detect Static Files or Build Requirement

Scan project directory to determine action:

**Check for existing static files:**
```bash
# Get frontend build path from TOML
BUILD_PATH=$(grep -A 15 'type = "frontend"' aramb.toml | grep 'buildPath = ' | head -1 | cut -d'"' -f2)

if [ -z "$BUILD_PATH" ]; then
  # Fallback: check common static file directories
  if [ -d "dist" ] && [ -n "$(ls -A dist)" ]; then
    STATIC_DIR="dist"
  elif [ -d "build" ] && [ -n "$(ls -A build)" ]; then
    STATIC_DIR="build"
  elif [ -d "out" ] && [ -n "$(ls -A out)" ]; then
    STATIC_DIR="out"
  elif [ -d "public" ] && [ -n "$(ls -A public)" ]; then
    STATIC_DIR="public"
  else
    NEEDS_BUILD=true
  fi
else
  # Use build path from TOML
  STATIC_DIR="$BUILD_PATH"

  # Check if build is needed
  if [ ! -d "$STATIC_DIR" ] || [ -z "$(ls -A $STATIC_DIR)" ]; then
    NEEDS_BUILD=true
  fi
fi

echo "Static directory: $STATIC_DIR"
```

**Detect framework if build needed:**
- Check `package.json` for framework
- Identify: React, Vue, Next.js, Angular, Vite, or generic

---

### 5. Build Static Files (if needed)

If `NEEDS_BUILD=true` or `force_build=true`:

```bash
# Use aramb build to detect framework and build
aramb build --static-outdir ./dist

# Capture output directory
STATIC_DIR="./dist"  # or framework-specific
```

**Framework detection:**
- **Next.js**: `out/` directory
- **Create React App**: `build/` directory
- **Vite**: `dist/` directory
- **Vue CLI**: `dist/` directory
- **Angular**: `dist/` directory
- **Generic**: `dist/` directory

### 6. Prepare Environment Variables

```bash
# Build environment variables array from TOML configuration
declare -a ENV_VARS

# Extract environment variables from frontend service configuration.vars in TOML
# TOML structure: [services.configuration.vars] or [services.configuration.secrets]

# Method 1: Parse vars array format
# Example: vars = [{ name = "API_URL", value = "${101.outputs.PUBLIC_URL}" }]
VARS_SECTION=$(grep -A 50 'type = "frontend"' aramb.toml | sed -n '/\[.*\.configuration\.vars\]/,/^\[/p' | head -n -1)

if [ -n "$VARS_SECTION" ]; then
  # Parse key-value pairs from vars section
  while IFS= read -r line; do
    if [[ "$line" =~ ^[[:space:]]*([A-Z_][A-Z0-9_]*)[[:space:]]*=[[:space:]]*(.+)$ ]]; then
      VAR_NAME="${BASH_REMATCH[1]}"
      VAR_VALUE="${BASH_REMATCH[2]}"

      # Remove quotes
      VAR_VALUE=$(echo "$VAR_VALUE" | sed 's/^"//;s/"$//')

      # Check if value is a reference (contains ${})
      if [[ "$VAR_VALUE" == *'${'* ]]; then
        # It's a reference to backend output
        if [ -n "$BACKEND_PUBLIC_URL" ]; then
          # Replace reference with actual backend URL
          RESOLVED_VALUE="$BACKEND_PUBLIC_URL"
          ENV_VARS+=("--env" "${VAR_NAME}=${RESOLVED_VALUE}")
          echo "✓ Resolved env var: $VAR_NAME=$RESOLVED_VALUE"
        else
          echo "WARNING: Could not resolve reference: $VAR_NAME"
        fi
      else
        # It's a static value
        ENV_VARS+=("--env" "${VAR_NAME}=${VAR_VALUE}")
        echo "✓ Set env var: $VAR_NAME=$VAR_VALUE"
      fi
    fi
  done <<< "$VARS_SECTION"
fi

# Also extract from secrets section if exists
SECRETS_SECTION=$(grep -A 50 'type = "frontend"' aramb.toml | sed -n '/\[.*\.configuration\.secrets\]/,/^\[/p' | head -n -1)

if [ -n "$SECRETS_SECTION" ]; then
  while IFS= read -r line; do
    if [[ "$line" =~ ^[[:space:]]*([A-Z_][A-Z0-9_]*)[[:space:]]*=[[:space:]]*(.+)$ ]]; then
      VAR_NAME="${BASH_REMATCH[1]}"
      VAR_VALUE="${BASH_REMATCH[2]}"
      VAR_VALUE=$(echo "$VAR_VALUE" | sed 's/^"//;s/"$//')

      # Secrets are usually static values, not references
      ENV_VARS+=("--env" "${VAR_NAME}=${VAR_VALUE}")
      echo "✓ Set secret: $VAR_NAME=***"
    fi
  done <<< "$SECRETS_SECTION"
fi

echo "Total environment variables: ${#ENV_VARS[@]}"
```

**Example environment variable resolution:**
```toml
[services.configuration.vars]
API_URL = "${101.outputs.PUBLIC_URL}"
DEBUG = "false"
FEATURE_FLAG = "production"

[services.configuration.secrets]
SECRET_KEY = "my-secret-key"
```

**Resolves to:**
```
API_URL=https://backend-api.aramb.dev
DEBUG=false
FEATURE_FLAG=production
SECRET_KEY=my-secret-key
```

---

### 7. Deploy Static Files with Environment Variables

Deploy the static files to the frontend service with resolved environment variables:

```bash
# Deploy static files using the service slug with environment variables
echo "Deploying static files to: $FRONTEND_SLUG"

# Build deploy command with environment variables
DEPLOY_CMD="aramb deploy ${STATIC_DIR}/ --service ${FRONTEND_SLUG}"

# Add environment variables to command
if [ ${#ENV_VARS[@]} -gt 0 ]; then
  DEPLOY_CMD="$DEPLOY_CMD ${ENV_VARS[@]}"
fi

# Execute deployment
DEPLOY_OUTPUT=$(eval "$DEPLOY_CMD" 2>&1)

if [ $? -ne 0 ]; then
  echo "ERROR: Frontend deployment failed"
  echo "$DEPLOY_OUTPUT"
  exit 1
fi

# Capture deployment ID and URL from aramb-cli output
DEPLOYMENT_ID=$(echo "$DEPLOY_OUTPUT" | grep -oP 'deployment_id:\s*\K[^\s]+' || echo "")
DEPLOYMENT_URL=$(echo "$DEPLOY_OUTPUT" | grep -oP 'url:\s*\K[^\s]+' || echo "")

echo "✓ Deployment successful"
echo "Deployment ID: $DEPLOYMENT_ID"
echo "Deployment URL: $DEPLOYMENT_URL"
```

**Example deployment command:**
```bash
aramb deploy dist/ --service my-frontend-548cad1 \
  --env API_URL=https://backend-api.aramb.dev \
  --env DEBUG=false \
  --env FEATURE_FLAG=production
```

**Exit if:** Deployment fails

### 8. Validate Deployment

```bash
# Check deployment status using service slug
echo "Validating deployment for: $FRONTEND_SLUG"

DEPLOY_STATUS=$(aramb deploy status --service "$FRONTEND_SLUG" --output json 2>&1)

if [ $? -ne 0 ]; then
  echo "WARNING: Could not get deployment status"
else
  # Extract status
  STATUS=$(echo "$DEPLOY_STATUS" | jq -r '.status // "unknown"')
  PUBLIC_URL=$(echo "$DEPLOY_STATUS" | jq -r '.outputs.PUBLIC_URL // empty')

  echo "Service status: $STATUS"
  echo "Public URL: $PUBLIC_URL"

  # If URL wasn't captured earlier, use it from status
  if [ -z "$DEPLOYMENT_URL" ] && [ -n "$PUBLIC_URL" ]; then
    DEPLOYMENT_URL="$PUBLIC_URL"
  fi
fi

# Verify deployment URL was captured
if [ -z "$DEPLOYMENT_URL" ]; then
  echo "WARNING: deployment_url not captured from aramb-cli output"
fi

echo "✓ Frontend deployment complete"
```

**Exit if:** None (warnings only for status retrieval)

**Important:** Always use service SLUG (not name) for status checks

## Complete Deployment Flow

### Standard Flow (with aramb.toml)

```
Step 0: Install aramb-cli (CRITICAL FIRST STEP)
   ↓
Step 1: Read aramb.toml
   ↓
Step 2: Create Frontend Service
   ├─ Extract service details from TOML
   ├─ Run: aramb services create --name {name} --type frontend --application {app_id}
   └─ Get service slug
   ↓
Step 3: Resolve Backend References
   ├─ Check for backend references in TOML
   ├─ Get backend slug from TOML
   ├─ Run: aramb deploy details --service {backend_slug} -o json
   └─ Extract PUBLIC_URL from outputs
   ↓
Step 4: Detect Static Files or Build Requirement
   ├─ Check for existing static files
   └─ Determine if build is needed
   ↓
Step 5: Build Static Files (if needed)
   ├─ Detect framework
   ├─ Run build command
   └─ Capture static directory
   ↓
Step 6: Prepare Environment Variables
   ├─ Extract env vars from TOML
   ├─ Resolve backend references
   └─ Build --env flags array
   ↓
Step 7: Deploy Static Files with Environment Variables
   ├─ Run: aramb deploy {static_dir}/ --service {slug} --env KEY=VALUE
   ├─ Capture deployment ID and URL
   └─ Verify deployment succeeds
   ↓
Step 8: Validate Deployment
   ├─ Check deployment status
   └─ Return deployment details
```

### Key Changes from Old Flow

**Old Flow:**
- Deploy TOML first → Extract service → Deploy static files

**New Flow:**
- Read TOML → Create service → Resolve references → Deploy with env vars

**Benefits:**
- ✅ Explicit service creation with proper metadata
- ✅ Backend references resolved before deployment
- ✅ Environment variables set during deployment
- ✅ Better error handling and validation

---

## Compact Workflow (Precise Logic)

```bash
#!/bin/bash
set -e  # Exit on any error

# Step 0: Install aramb-cli if not present (CRITICAL FIRST STEP)
if ! command -v aramb &> /dev/null; then
  echo "Installing aramb-cli..."
  OS=$(uname -s | tr '[:upper:]' '[:lower:]')
  ARCH=$(uname -m)

  if [ "$ARCH" = "x86_64" ]; then ARCH="amd64"
  elif [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "arm64" ]; then ARCH="arm64"; fi

  BINARY_NAME="aramb-${OS}-${ARCH}"
  curl -LO "https://github.com/aramb-ai/release-beta/releases/latest/download/${BINARY_NAME}" || { echo "ERROR: Failed to download aramb-cli"; exit 1; }
  chmod +x "${BINARY_NAME}"
  sudo mv "${BINARY_NAME}" /usr/local/bin/aramb || { echo "ERROR: Failed to install aramb-cli"; exit 1; }
  echo "✓ aramb-cli installed successfully"
fi

# Step 1: Read aramb.toml
[ -f "aramb.toml" ] || { echo "ERROR: aramb.toml not found"; exit 1; }
[ -n "$ARAMB_API_TOKEN" ] || { echo "ERROR: ARAMB_API_TOKEN not set"; exit 1; }
[ -n "$APPLICATION_ID" ] || { echo "ERROR: APPLICATION_ID not set"; exit 1; }

# Step 2: Create frontend service
FRONTEND_NAME=$(grep -A 10 'type = "frontend"' aramb.toml | grep 'name = ' | head -1 | cut -d'"' -f2)
FRONTEND_DESC=$(grep -A 10 'type = "frontend"' aramb.toml | grep 'description = ' | head -1 | cut -d'"' -f2)
[ -n "$FRONTEND_NAME" ] || { echo "ERROR: No frontend service in TOML"; exit 1; }

echo "Creating frontend service: $FRONTEND_NAME"
aramb services create \
  --name "$FRONTEND_NAME" \
  --type frontend \
  --description "${FRONTEND_DESC:-Frontend service}" \
  --application "$APPLICATION_ID" \
  --tags frontend,aramb || { echo "ERROR: Service creation failed"; exit 1; }

FRONTEND_SLUG=$(grep -A 10 'type = "frontend"' aramb.toml | grep 'slug = ' | head -1 | cut -d'"' -f2)
echo "✓ Frontend service created: $FRONTEND_SLUG"

# Step 3: Resolve backend references
BACKEND_SLUG=$(grep -A 5 'type = "backend"' aramb.toml | grep 'slug = ' | head -1 | cut -d'"' -f2)
if [ -n "$BACKEND_SLUG" ]; then
  echo "Resolving backend references..."
  BACKEND_OUTPUTS=$(aramb deploy details --service "$BACKEND_SLUG" -o json 2>/dev/null)
  if [ $? -eq 0 ]; then
    BACKEND_PUBLIC_URL=$(echo "$BACKEND_OUTPUTS" | jq -r '.outputs.PUBLIC_URL // empty')
    [ -n "$BACKEND_PUBLIC_URL" ] && echo "✓ Resolved backend URL: $BACKEND_PUBLIC_URL"
  fi
fi

# Step 4: Detect static files
BUILD_PATH=$(grep -A 15 'type = "frontend"' aramb.toml | grep 'buildPath = ' | head -1 | cut -d'"' -f2)
STATIC_DIR="${BUILD_PATH:-dist}"

if [ ! -d "$STATIC_DIR" ] || [ -z "$(ls -A $STATIC_DIR)" ]; then
  NEEDS_BUILD=true
else
  NEEDS_BUILD=false
fi

# Step 5: Build static files (if needed)
if [ "$NEEDS_BUILD" = true ]; then
  echo "Building static files..."
  # Detect framework and build
  if [ -f "package.json" ]; then
    npm install
    npm run build
  fi
  echo "✓ Build complete"
fi

# Step 6: Prepare environment variables from services.configuration.vars
declare -a ENV_VARS

# Extract vars section from TOML
VARS_SECTION=$(grep -A 50 'type = "frontend"' aramb.toml | sed -n '/\[.*\.configuration\.vars\]/,/^\[/p' | head -n -1)

if [ -n "$VARS_SECTION" ]; then
  while IFS= read -r line; do
    if [[ "$line" =~ ^[[:space:]]*([A-Z_][A-Z0-9_]*)[[:space:]]*=[[:space:]]*(.+)$ ]]; then
      VAR_NAME="${BASH_REMATCH[1]}"
      VAR_VALUE=$(echo "${BASH_REMATCH[2]}" | sed 's/^"//;s/"$//')

      if [[ "$VAR_VALUE" == *'${'* ]] && [ -n "$BACKEND_PUBLIC_URL" ]; then
        ENV_VARS+=("--env" "${VAR_NAME}=${BACKEND_PUBLIC_URL}")
        echo "✓ Resolved env var: $VAR_NAME=$BACKEND_PUBLIC_URL"
      elif [[ "$VAR_VALUE" != *'${'* ]]; then
        ENV_VARS+=("--env" "${VAR_NAME}=${VAR_VALUE}")
        echo "✓ Set env var: $VAR_NAME=$VAR_VALUE"
      fi
    fi
  done <<< "$VARS_SECTION"
fi

# Also extract secrets if they exist
SECRETS_SECTION=$(grep -A 50 'type = "frontend"' aramb.toml | sed -n '/\[.*\.configuration\.secrets\]/,/^\[/p' | head -n -1)

if [ -n "$SECRETS_SECTION" ]; then
  while IFS= read -r line; do
    if [[ "$line" =~ ^[[:space:]]*([A-Z_][A-Z0-9_]*)[[:space:]]*=[[:space:]]*(.+)$ ]]; then
      VAR_NAME="${BASH_REMATCH[1]}"
      VAR_VALUE=$(echo "${BASH_REMATCH[2]}" | sed 's/^"//;s/"$//')
      ENV_VARS+=("--env" "${VAR_NAME}=${VAR_VALUE}")
      echo "✓ Set secret: $VAR_NAME=***"
    fi
  done <<< "$SECRETS_SECTION"
fi

# Step 7: Deploy with environment variables
echo "Deploying frontend to: $FRONTEND_SLUG"
DEPLOY_CMD="aramb deploy ${STATIC_DIR}/ --service ${FRONTEND_SLUG}"
[ ${#ENV_VARS[@]} -gt 0 ] && DEPLOY_CMD="$DEPLOY_CMD ${ENV_VARS[@]}"

DEPLOY_OUTPUT=$(eval "$DEPLOY_CMD" 2>&1) || { echo "ERROR: Deployment failed"; exit 1; }
DEPLOYMENT_URL=$(echo "$DEPLOY_OUTPUT" | grep -oP 'url:\s*\K[^\s]+' || echo "")

# Step 8: Validate deployment
DEPLOY_STATUS=$(aramb deploy status --service "$FRONTEND_SLUG" --output json 2>/dev/null)
PUBLIC_URL=$(echo "$DEPLOY_STATUS" | jq -r '.outputs.PUBLIC_URL // empty')
[ -z "$DEPLOYMENT_URL" ] && DEPLOYMENT_URL="$PUBLIC_URL"

echo "✓ Frontend deployed successfully"
echo "{\"status\": \"success\", \"url\": \"${DEPLOYMENT_URL:-n/a}\", \"service\": \"$FRONTEND_SLUG\"}"
```

---

## Complete Example with Backend References

### Example aramb.toml with Frontend

```toml
[[services]]
uniqueIdentifier = 101
name = "backend-api"
slug = "backend-api-548cad1"
type = "backend"
applicationID = "app-xyz789"

[services.configuration.settings]
image = "my-app/backend:latest"
cmd = "npm start"
commandPort = 8080
publicNet = true

[[services]]
uniqueIdentifier = 102
name = "frontend-web"
slug = "frontend-web-abc123"
type = "frontend"
description = "React frontend application"
applicationID = "app-xyz789"

[services.configuration.settings]
buildPath = "dist"
staticPath = "/app/dist"
commandPort = 8080
publicNet = true

# Environment variables section
[services.configuration.vars]
API_URL = "${101.outputs.PUBLIC_URL}"
DEBUG = "false"
FEATURE_FLAG = "production"

# Secrets section (optional)
[services.configuration.secrets]
SECRET_KEY = "my-secret-key"
API_TOKEN = "token-xyz"
```

### Execution Flow

**Step 1: Create Frontend Service**
```bash
aramb services create \
  --name "frontend-web" \
  --type frontend \
  --description "React frontend application" \
  --application "app-xyz789" \
  --tags frontend,aramb
```

**Step 2: Get Backend Deployment Outputs**
```bash
aramb deploy details --service backend-api-548cad1 -o json
```

**Output:**
```json
{
  "service": "backend-api-548cad1",
  "status": "healthy",
  "outputs": {
    "PUBLIC_URL": "https://backend-api.aramb.dev",
    "INTERNAL_URL": "http://backend-api:8080"
  }
}
```

**Step 3: Resolve Environment Variables**
```
API_URL = "${101.outputs.PUBLIC_URL}"
→ Resolves to: API_URL=https://backend-api.aramb.dev

DEBUG = "false"
→ Resolves to: DEBUG=false

FEATURE_FLAG = "production"
→ Resolves to: FEATURE_FLAG=production
```

**Step 4: Deploy with Environment Variables**
```bash
aramb deploy dist/ --service frontend-web-abc123 \
  --env API_URL=https://backend-api.aramb.dev \
  --env DEBUG=false \
  --env FEATURE_FLAG=production
```

**Result:**
```json
{
  "status": "success",
  "url": "https://frontend-web.aramb.dev",
  "service": "frontend-web-abc123",
  "env_vars_set": 3
}
```

---

## Detection Logic

### Framework Detection

Identify framework by checking project files:

**Next.js:**
```bash
# Check for next.config.js or next.config.mjs
[ -f "next.config.js" ] || [ -f "next.config.mjs" ]
→ Framework: Next.js, Output: out/
```

**Create React App:**
```bash
# Check package.json for react-scripts
grep -q "react-scripts" package.json
→ Framework: CRA, Output: build/
```

**Vite:**
```bash
# Check for vite.config.js/ts
[ -f "vite.config.js" ] || [ -f "vite.config.ts" ]
→ Framework: Vite, Output: dist/
```

**Vue CLI:**
```bash
# Check for vue.config.js
[ -f "vue.config.js" ]
→ Framework: Vue CLI, Output: dist/
```

**Angular:**
```bash
# Check for angular.json
[ -f "angular.json" ]
→ Framework: Angular, Output: dist/
```

**Generic/Static:**
```bash
# No framework detected, check for static files
[ -d "public" ] || [ -d "dist" ] || [ -d "build" ]
→ Use existing directory
```

### Build Decision Tree

```
Start
  ↓
Check --force-build flag?
  ├─ Yes → Build static files
  └─ No → Check for existing static files?
      ├─ Found → Use existing (skip build)
      └─ Not found → Detect framework → Build static files
```

## Service Creation

### Auto-Generated Service Name

Generate service name from project:

```bash
# Use directory name + "-web" suffix
DIR_NAME=$(basename $(pwd))
SERVICE_NAME="${DIR_NAME}-web"

# Sanitize: lowercase, replace spaces/special chars with hyphens
SERVICE_NAME=$(echo "$SERVICE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9-]/-/g')
```

Examples:
- `my-react-app/` → `my-react-app-web`
- `Frontend/` → `frontend-web`
- `nextjs_project/` → `nextjs-project-web`

### Service Configuration

Service created with:

```json
{
  "name": "my-app-web",
  "type": "frontend",
  "application_id": "<APPLICATION_ID>",
  "configuration": {
    "settings": {
      "staticPath": "/absolute/path/to/dist",
      "cmd": "npx http-server",
      "commandPort": 8080,
      "publicNet": true
    }
  }
}
```

## Validation Criteria

### Critical (MUST pass)

**Environment:**
- aramb-cli accessible in PATH
- ARAMB_API_TOKEN environment variable set
- APPLICATION_ID environment variable set
- aramb.toml exists and valid

**Service Creation Phase:**
- Frontend service name extracted from TOML
- Service created successfully: `aramb services create`
- Service slug retrieved

**Backend Reference Resolution (if applicable):**
- Backend service slug extracted from TOML
- Backend deployment outputs retrieved
- PUBLIC_URL extracted from backend outputs

**Build Phase (if needed):**
- Framework correctly detected
- Build completes without errors
- Static files exist in expected directory

**Deployment Phase:**
- Environment variables prepared correctly
- Backend references resolved in env vars
- Deployment succeeds with all env vars
- Deployment URL captured

**Validation Phase:**
- Service reports healthy status
- Public URL accessible
- deployment_url captured from aramb-cli

### Expected (SHOULD pass)

- Backend references correctly resolved
- Environment variables set properly
- Service accessible via public URL
- Static files served correctly

### Nice to Have

- Build optimized (minified, compressed)
- Service starts quickly
- Static files cached properly
- Environment variables logged (non-sensitive only)

## Output Requirements (IMPORTANT)

Before completing, you MUST set comprehensive outputs. The planner uses these
to answer user questions without needing to resume you.

**Always include:**

```python
outputs = {
    # URLs and identifiers (CRITICAL)
    "url": "https://my-app-web.aramb.dev",
    "id": "deploy-abc123xyz",
    "deployment_id": "deploy-abc123xyz",

    # Deployment details
    "deploy_mode": "toml",  # or "auto"
    "service_slug": "my-app-web",
    "static_dir": "./dist",

    # Build info
    "framework": "React 18",
    "build_time": "45s",
    "bundle_size": "1.2MB",

    # Commands used
    "commands": {
        "build": "npm run build",
        "preview": "npx serve dist"
    },

    # Status
    "status": "success",
    "healthy": true
}
```

**Why this matters:**
- Planner receives your outputs in `task_completed` trigger
- Planner can answer "What's the URL?", "Is it deployed?", etc. immediately
- No need to resume you for simple factual questions

## Output

**Required output format:**

```json
{
  "id": "deploy-abc123xyz",
  "url": "https://my-app-web.aramb.dev",
  "status": "success"
}
```

**Fields:**
- `id`: Deployment ID returned by aramb-cli
- `url`: Public URL returned by aramb-cli
- `status`: Deployment status

**Extended output (recommended):**

```json
{
  "id": "deploy-abc123xyz",
  "url": "https://my-app-web.aramb.dev",
  "status": "success",
  "deploy_mode": "toml",
  "service_slug": "my-app-web",
  "static_dir": "./dist",
  "framework": "React",
  "build_time": "45s",
  "healthy": true
}
```

## Usage Examples

### Standard Deployment (with aramb.toml)

```bash
# Prerequisites:
# - aramb.toml exists with frontend service configured
# - APPLICATION_ID environment variable set
# - ARAMB_API_TOKEN environment variable set

export APPLICATION_ID="app-xyz789"
export ARAMB_API_TOKEN="your-token"

# Run deployment
/frontend-deployment

# Flow:
# 1. Creates frontend service
# 2. Resolves backend references (if any)
# 3. Builds static files (if needed)
# 4. Deploys with environment variables
```

### Deployment with Backend References

```bash
# aramb.toml contains:
# API_URL = "${101.outputs.PUBLIC_URL}"

# Backend must be deployed first
# Frontend deployment will automatically:
# 1. Get backend service slug from TOML
# 2. Fetch backend deployment outputs
# 3. Resolve API_URL to actual backend URL
# 4. Deploy frontend with resolved env vars

/frontend-deployment
```

### Deploy Existing Static Files

```bash
# If dist/ exists and contains files:
# - Skips build phase
# - Uses existing static files
# - Deploys directly

/frontend-deployment
```

### Force Rebuild

```bash
# Forces rebuild even if static files exist:
# - Deletes existing dist/
# - Rebuilds from source
# - Deploys new build

/frontend-deployment --force-build
```

## Error Handling

### Missing APPLICATION_ID

```bash
if [ -z "$APPLICATION_ID" ]; then
  echo "Error: APPLICATION_ID environment variable not set"
  echo "Set it with: export APPLICATION_ID=your-app-id"
  exit 1
fi
```

### Build Failures

If build fails:
1. Log error with framework and build output
2. Exit without creating service
3. Provide troubleshooting guidance

### Service Creation Failures

If service creation fails:
1. Log error with API response
2. Check if service already exists
3. Suggest using different service name

### Deployment Failures

If deployment fails:
1. Log error with deployment details
2. Service may still exist (cleanup not automatic)
3. Retry deployment or delete service manually

## Framework-Specific Handling

### Next.js

```bash
# Detect
[ -f "next.config.js" ]

# Build command
npm run build

# Output directory
out/
```

### React (CRA)

```bash
# Detect
grep -q "react-scripts" package.json

# Build command
npm run build

# Output directory
build/
```

### Vite

```bash
# Detect
[ -f "vite.config.js" ]

# Build command
npm run build

# Output directory
dist/
```

### Vue CLI

```bash
# Detect
[ -f "vue.config.js" ]

# Build command
npm run build

# Output directory
dist/
```

### Angular

```bash
# Detect
[ -f "angular.json" ]

# Build command
ng build --configuration production

# Output directory
dist/<project-name>/
```

## Environment Variables

**Required:**
- `APPLICATION_ID` - Aramb application ID to deploy to
- `ARAMB_API_TOKEN` - API token for authentication

**Optional:**
- `BUILDKIT_HOST` - BuildKit server (only if custom build needed)
- `SERVICE_NAME` - Override auto-generated service name
- `PORT` - Override default port (8080)

## Best Practices

1. **Set APPLICATION_ID** before running deployment
2. **Clean build artifacts** if encountering issues (`rm -rf dist/ build/`)
3. **Use force-build** when you've made changes but static files exist
4. **Verify static files** are correctly built before deployment
5. **Test locally** with `npx http-server dist/` before deploying

## Common Scenarios

### Scenario 1: Frontend with Backend Integration

```bash
# aramb.toml has both backend and frontend services
# Frontend env vars reference backend outputs

# 1. Backend deployed first (returns PUBLIC_URL)
# 2. Run frontend deployment
/frontend-deployment

# Result:
# → Creates frontend service
# → Fetches backend deployment outputs
# → Resolves: API_URL=https://backend-api.aramb.dev
# → Builds React app
# → Deploys with resolved env vars
```

### Scenario 2: Fresh React Project (No Build Yet)

```bash
# Project has package.json, no build/ directory
# aramb.toml configured with frontend service

/frontend-deployment

# Result:
# → Creates frontend service from TOML
# → Detects no static files
# → Detects React framework
# → Runs npm run build
# → Deploys from build/
# → Returns URL: https://frontend-web.aramb.dev
```

### Scenario 3: Pre-built Static Files

```bash
# Project has dist/ directory with files
# aramb.toml configured with buildPath="dist"

/frontend-deployment

# Result:
# → Creates frontend service from TOML
# → Detects existing dist/ directory
# → Skips build phase
# → Deploys from dist/
# → Returns URL: https://frontend-web.aramb.dev
```

### Scenario 4: Multiple Environment Variables

```toml
# aramb.toml with multiple env vars
[services.configuration.vars]
API_URL = "${101.outputs.PUBLIC_URL}"
WS_URL = "${101.outputs.WEBSOCKET_URL}"
DEBUG = "false"
VERSION = "1.0.0"

[services.configuration.secrets]
API_KEY = "secret-api-key-123"
AUTH_TOKEN = "auth-token-xyz"
```

```bash
/frontend-deployment

# Result:
# → Resolves both backend references
# → Deploys with 6 environment variables:
#   --env API_URL=https://backend-api.aramb.dev
#   --env WS_URL=wss://backend-api.aramb.dev/ws
#   --env DEBUG=false
#   --env VERSION=1.0.0
#   --env API_KEY=secret-api-key-123
#   --env AUTH_TOKEN=auth-token-xyz
```

## Integration with Other Skills

### After backend-deployment

Typical workflow for full-stack applications:

```bash
# 1. Generate aramb.toml (includes backend and frontend services)
/aramb-metadata

# 2. Deploy backend services
/backend-deployment
# Returns: {"public_url": "https://backend-api.aramb.dev", ...}

# 3. Deploy frontend (automatically resolves backend URL)
/frontend-deployment
# Resolves: API_URL=https://backend-api.aramb.dev
# Returns: {"url": "https://frontend-web.aramb.dev", ...}
```

### Complete Workflow

```
User Request
    ↓
/aramb-metadata (creates aramb.toml with backend + frontend)
    ↓
/backend-deployment
    ├─ Deploy TOML (create services)
    ├─ Build & deploy backend services
    └─ Return PUBLIC_URL
    ↓
Backend Running (PUBLIC_URL: https://backend-api.aramb.dev)
    ↓
/frontend-deployment
    ├─ Create frontend service
    ├─ Resolve backend PUBLIC_URL
    ├─ Build static files
    ├─ Deploy with env vars (API_URL=https://backend-api.aramb.dev)
    └─ Return frontend URL
    ↓
All Services Running
    ├─ Backend: https://backend-api.aramb.dev
    └─ Frontend: https://frontend-web.aramb.dev
```

## Advanced Usage

See reference documentation:

- **Installation**: [references/installation.md](references/installation.md) - Setup guide
- **Troubleshooting**: [references/troubleshooting.md](references/troubleshooting.md) - Common issues
- **Frameworks**: [references/frameworks.md](references/frameworks.md) - Framework-specific details

## See Also

- **frontend-development**: Build frontend applications with components
- **local-deploy**: Full-stack deployment with aramb.toml
- **aramb-metadata**: Generate aramb.toml for complex deployments
