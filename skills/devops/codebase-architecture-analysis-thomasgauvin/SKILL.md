---
name: codebase-architecture-analysis
description: Analyze a GitHub codebase to create comprehensive architecture documentation including ASCII diagrams, component relationships, data flow, hosting infrastructure, and file structure assessment.
compatibility: Requires git, GitHub authentication via PAT, and network access to GitHub
license: MIT
---

# Codebase Architecture Analysis

## When to use this skill

Use this skill when you need to understand the high-level architecture and structure of a codebase. Specifically, use it when you need to:
- Create architecture diagrams for documentation
- Understand component relationships and dependencies
- Assess hosting infrastructure and deployment architecture
- Generate comprehensive architectural overviews
- Document data flow between system components
- Create visual representations of code organization

## Overview

This skill guides a specialized agent through a comprehensive analysis of a GitHub repository to produce detailed architecture documentation. The analysis includes ASCII diagrams, component maps, infrastructure details, and file-level assessments.

## Workflow

### Step 1: Repository Setup

**Input required:**
- GitHub repository URL or `owner/repo` format
- Optional: Specific branch or commit to analyze

**Actions:**
1. Clone the repository using the GitHub PAT from environment variables
2. Verify the repository is cloned successfully
3. Document the repository metadata (language, size, structure)

YOU MUST CLONE THE REPOSITORY AND INSPECT THE FILES USING BASH TOOLS AND GIT CLI RATHER THAN MCP.

### Step 2: Codebase Discovery and Assessment

**Analyze the directory structure:**
1. Map the complete directory tree
2. Identify major components/modules/packages
3. Classify directories by purpose (src, config, tests, build, etc.)
4. Count files by type (TypeScript, Python, JSON, etc.)
5. Identify entry points and main application files

**For each file, assess:**
- File type and purpose
- Size and complexity
- Key imports and dependencies
- What module/component it belongs to
- Role in the overall system

### Step 3: Create Architecture Diagrams

**Create multiple ASCII diagrams:**

1. **System Architecture Diagram**
   - High-level components and their relationships
   - External systems and services
   - Data flow between components

2. **Deployment Architecture**
   - Hosting infrastructure (cloud platform, containers, etc.)
   - Service relationships
   - Network and database layers

3. **File Structure Diagram**
   - Directory hierarchy showing major components
   - Key files and their purposes
   - Organization by feature or layer

4. **Data Flow Diagram**
   - How data moves through the system
   - API endpoints and their interactions
   - Database access patterns

Example ASCII diagram structure:

```
┌─────────────────────────────────────────────────┐
│                  Client (React)                  │
├─────────────────────────────────────────────────┤
│  - Components                                    │
│  - Pages                                         │
│  - State Management                              │
└────────────────┬────────────────────────────────┘
                 │ HTTP/WebSocket
┌────────────────▼────────────────────────────────┐
│            Backend Server (Node.js)              │
├─────────────────────────────────────────────────┤
│  - API Routes                                    │
│  - Authentication                                │
│  - Business Logic                                │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────┐
│            Database & External APIs              │
├─────────────────────────────────────────────────┤
│  - PostgreSQL / MongoDB                          │
│  - Third-party Services                          │
└─────────────────────────────────────────────────┘
```

### Step 4: Component Analysis

**For each major component, document:**
- Purpose and responsibilities
- Key files and entry points
- External dependencies
- Interactions with other components
- API surface (if applicable)

**Document component categories:**
- **Frontend Components**: UI components, pages, layouts
- **Backend Services**: API endpoints, middleware, handlers
- **Business Logic**: Core algorithms, processing
- **Infrastructure**: Configuration, build, deployment
- **Testing**: Test utilities, test files
- **Documentation**: READMEs, specs, guides

### Step 5: Technology Stack Assessment

**Identify and document:**
- Programming languages used
- Key frameworks and libraries
- Database systems
- External services and APIs
- Development tools and build systems
- Container/deployment technologies
- Version numbers where significant

### Step 6: Hosting and Infrastructure Analysis

**Assess the deployment architecture:**
1. Identify hosting platform (AWS, GCP, Vercel, Cloudflare, etc.)
2. Document service configuration
   - Environment variables
   - Build processes
   - Deployment scripts
3. Identify infrastructure-as-code files (Terraform, Docker, etc.)
4. Document scaling considerations
5. Identify external service dependencies

### Step 7: Generate Final Documentation

**Create a comprehensive architecture document including:**

1. **Executive Summary**
   - Project purpose
   - High-level architecture overview
   - Technology stack
   - Hosting platform and deployment

2. **Architecture Diagrams** (multiple views as described in Step 3)

3. **Component Catalog**
   - List of major components
   - Purpose of each
   - Key files
   - Dependencies

4. **File Structure Overview**
   - Directory layout with purposes
   - Important files highlighted

5. **Data Flow Explanation**
   - How requests are processed
   - Database interactions
   - External API calls

6. **Technology Details**
   - Language versions
   - Framework versions
   - Key library versions
   - Database schema summary (if visible in code)

7. **Deployment and Hosting**
   - Hosting platform details
   - Build and deployment process
   - Environment configuration
   - Scaling considerations

8. **Dependencies and Integrations**
   - List of external services
   - API integrations
   - Authentication/authorization approach

## Common Patterns

### For Monolithic Applications
- Single codebase containing frontend, backend, and shared logic
- Clear separation between presentation, business logic, and data layers
- Review package.json/requirements.txt for all dependencies

### For Microservices
- Multiple services in separate directories or repositories
- Service communication documented in deployment config
- API contracts between services
- Separate databases per service (typically)

### For Full-Stack Web Applications
- Frontend framework (React, Vue, Angular, etc.)
- Backend framework (Node.js/Express, Python/Django, etc.)
- Database (SQL or NoSQL)
- API layer connecting frontend and backend

## Edge Cases

**Large Codebases:**
- Focus on major components first
- Group related files together
- Create summary before diving into details

**Polyglot Repositories:**
- Separate analysis by language when relevant
- Document language integration points
- Highlight cross-language dependencies

**Complex Infrastructure:**
- Document infrastructure-as-code separately
- Identify deployment stages (dev, staging, prod)
- Note auto-scaling or load balancing configurations

## Output Format

The final architecture analysis should be delivered as:
1. A comprehensive Markdown document with embedded ASCII diagrams
2. Clear section headers and navigation
3. Links between related sections
4. Visual hierarchy showing component relationships
5. Concise but complete descriptions

## Tools and Resources

The agent may use:
- Git commands to explore repository structure
- File reading tools to examine source code
- Text parsing to extract key information
- ASCII art libraries for diagram generation

## Example Use Case

**User Request:** "Analyze the architecture of the user-management microservice in our platform"

**Agent Process:**
1. Clones the user-management repository
2. Maps the directory structure (controllers, models, tests, config)
3. Creates ASCII diagrams showing:
   - Service components (auth handler, user DB access, role manager)
   - Data flow (API request → controller → service → database)
   - Deployment (Docker container → Kubernetes → PostgreSQL)
4. Documents all dependencies and integrations
5. Provides complete architecture documentation

## Success Criteria

The analysis is complete when:
- ✅ Repository successfully cloned and analyzed
- ✅ All major components identified
- ✅ Multiple ASCII diagrams created showing different views
- ✅ File structure documented and explained
- ✅ Technology stack clearly identified
- ✅ Hosting/deployment architecture understood
- ✅ Data flow between components visible
- ✅ Comprehensive documentation generated
