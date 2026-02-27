---
name: platform-detection
description: Detect project type and recommend deployment platform. Use when deploying projects, choosing hosting platforms, analyzing project structure, or when user mentions deployment, platform selection, MCP servers, APIs, frontend apps, static sites, FastMCP Cloud, DigitalOcean, Vercel, Hostinger, Netlify, or Cloudflare.
allowed-tools: Bash, Read, Glob, Grep
---

# Platform Detection

Automatically detect project type and recommend optimal deployment platforms based on project characteristics, frameworks, and hosting requirements.

## Overview

This skill provides comprehensive project detection and platform recommendation capabilities:

- **Project Type Detection**: Identifies MCP servers, REST APIs, GraphQL APIs, frontend apps, static sites, monorepos
- **Framework Detection**: Recognizes Next.js, React, Vue, Astro, FastAPI, Express, FastMCP, and more
- **Platform Recommendation**: Maps projects to optimal hosting platforms (FastMCP Cloud, DigitalOcean, Vercel, Hostinger, Netlify, Cloudflare)
- **Configuration Analysis**: Examines package.json, requirements.txt, Dockerfile, and other config files
- **Multi-Service Detection**: Handles monorepos and projects with multiple deployment targets

## Available Scripts

### Detection Scripts

**`scripts/detect-project-type.sh`**
- Analyzes project structure to identify project type
- Checks for package.json, requirements.txt, MCP manifest, Dockerfile
- Detects frameworks and dependencies
- Returns: `mcp-server`, `api`, `frontend`, `static-site`, or `monorepo`
- Usage: `bash scripts/detect-project-type.sh <project-path>`

**`scripts/detect-framework.sh`**
- Identifies specific framework used in project
- Scans dependencies and configuration files
- Detects: FastMCP, Next.js, React, Vue, Astro, FastAPI, Express, etc.
- Returns: Framework name and version
- Usage: `bash scripts/detect-framework.sh <project-path>`

**`scripts/recommend-platform.sh`**
- Recommends deployment platform based on project characteristics
- Considers project type, framework, scalability needs, budget
- Returns: Platform name with justification
- Usage: `bash scripts/recommend-platform.sh <project-path>`

### Validation Scripts

**`scripts/validate-platform-requirements.sh`**
- Validates project meets requirements for target platform
- Checks for required configuration files
- Verifies deployment scripts and build commands
- Reports missing dependencies or configuration
- Usage: `bash scripts/validate-platform-requirements.sh <project-path> <platform>`

**`scripts/analyze-deployment-config.sh`**
- Analyzes existing deployment configuration
- Detects Docker, CI/CD pipelines, deployment scripts
- Identifies optimization opportunities
- Usage: `bash scripts/analyze-deployment-config.sh <project-path>`

## Available Templates

### Detection Rules Templates

**`templates/detection-rules.json`**
- Complete ruleset for project type detection
- File patterns, dependency patterns, framework signatures
- Extensible JSON structure for adding new detection rules

**`templates/platform-routing-rules.json`**
- Platform recommendation logic
- Maps project characteristics to optimal platforms
- Includes scoring weights for multi-criteria decisions

**`templates/framework-signatures.json`**
- Framework detection patterns
- Dependencies, file structures, config patterns
- Supports TypeScript and Python frameworks

### Configuration Templates

**`templates/platform-config/fastmcp-cloud.json`**
- FastMCP Cloud deployment configuration template
- Server configuration, environment variables, scaling settings

**`templates/platform-config/digitalocean.json`**
- DigitalOcean App Platform configuration template
- Build settings, resource allocation, environment setup

**`templates/platform-config/vercel.json`**
- Vercel deployment configuration template
- Build commands, output directory, serverless functions

## Available Examples

**`examples/basic-detection.md`**
- Simple project type detection workflow
- Single-service project examples
- Common detection scenarios

**`examples/advanced-monorepo-detection.md`**
- Complex monorepo detection with multiple services
- Handling projects with API + frontend
- Multi-platform deployment recommendations

**`examples/platform-recommendation-flow.md`**
- Complete platform recommendation workflow
- Decision tree examples
- Platform comparison matrices

**`examples/error-handling.md`**
- Edge cases and error scenarios
- Ambiguous project structures
- Missing configuration handling
- Fallback strategies

**`examples/integration-with-deploy-commands.md`**
- Using platform-detection in deployment workflows
- Integration with `/deployment:deploy` command
- Automated platform selection in CI/CD

## Usage Instructions

### 1. Detect Project Type

```bash
# Detect project type
PROJECT_TYPE=$(bash scripts/detect-project-type.sh /path/to/project)
echo "Detected: $PROJECT_TYPE"
```

### 2. Detect Framework

```bash
# Detect framework and version
FRAMEWORK=$(bash scripts/detect-framework.sh /path/to/project)
echo "Framework: $FRAMEWORK"
```

### 3. Recommend Platform

```bash
# Get platform recommendation
PLATFORM=$(bash scripts/recommend-platform.sh /path/to/project)
echo "Recommended: $PLATFORM"
```

### 4. Validate Platform Requirements

```bash
# Validate project for specific platform
bash scripts/validate-platform-requirements.sh /path/to/project vercel
```

### 5. Complete Detection Workflow

```bash
# Full detection and recommendation
cd /path/to/project

# Step 1: Detect type
PROJECT_TYPE=$(bash scripts/detect-project-type.sh .)
echo "Project Type: $PROJECT_TYPE"

# Step 2: Detect framework
FRAMEWORK=$(bash scripts/detect-framework.sh .)
echo "Framework: $FRAMEWORK"

# Step 3: Recommend platform
PLATFORM=$(bash scripts/recommend-platform.sh .)
echo "Recommended Platform: $PLATFORM"

# Step 4: Validate requirements
bash scripts/validate-platform-requirements.sh . "$PLATFORM"
```

## Detection Logic

### Project Type Detection Rules

1. **MCP Server**
   - Has `.mcp.json` or MCP manifest
   - Contains FastMCP dependencies
   - Has MCP server initialization code

2. **API**
   - Has REST/GraphQL endpoints
   - Contains FastAPI, Express, or similar framework
   - Has API routing configuration

3. **Frontend**
   - Has Next.js, React, Vue, or Astro
   - Contains client-side routing
   - Has build process for static assets

4. **Static Site**
   - No server-side code
   - Contains only HTML/CSS/JS
   - Has static site generator (Hugo, Jekyll, etc.)

5. **Monorepo**
   - Contains multiple services
   - Has workspace configuration (pnpm, yarn, lerna)
   - Multiple package.json files

### Platform Recommendation Logic

**FastMCP Cloud**
- Project Type: MCP Server
- Framework: FastMCP (Python or TypeScript)
- Best for: MCP server hosting with automatic scaling

**DigitalOcean App Platform**
- Project Type: API, MCP Server (non-FastMCP)
- Languages: Any (Docker support)
- Best for: Containerized applications, custom backends

**Vercel**
- Project Type: Frontend, Static Site, Full-stack (Next.js)
- Frameworks: Next.js, React, Vue, Astro
- Best for: Frontend apps, serverless functions, edge computing

**Hostinger / Netlify / Cloudflare Pages**
- Project Type: Static Site, Frontend
- Best for: Static hosting, CDN distribution, simple sites

## Requirements

- Project must have identifiable configuration files (package.json, requirements.txt, etc.)
- For accurate framework detection, dependencies must be installed or listed
- Platform recommendations assume standard project structures
- Validation requires target platform's deployment requirements

## Integration Points

This skill integrates with:
- `/deployment:deploy` - Automatic platform selection
- `/deployment:validate` - Pre-deployment validation
- `/deployment:configure` - Platform-specific configuration
- Deployment agents for automated decision-making

---

**Skill Location**: /home/gotime2022/.claude/plugins/marketplaces/ai-dev-marketplace/plugins/deployment/skills/platform-detection/
**Version**: 1.0.0
