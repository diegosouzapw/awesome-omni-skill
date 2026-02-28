---
name: architecture-design
description: Generate comprehensive software architecture documentation (ARCHITECTURE.md) with C4 diagrams, OpenAPI specs, and technology-specific guidance. This skill should be used when creating architecture documentation for new projects, documenting existing systems, or updating architectural specifications.
---

# Architecture Design

Generate professional software architecture documentation with visual diagrams and API specifications through structured interviews. Create complete ARCHITECTURE.md files covering all required sections with technology-specific patterns, C4 model diagrams, and OpenAPI 3.0 specifications.

## Core Workflow

### Step 1: Assess Project Type

Determine the project type by asking the user:

- **New project?** Gather requirements and design architecture
- **Existing system?** Document current state
- **Update needed?** Identify changed components

### Step 2: Gather Essential Information

Conduct a brief interview (5-7 questions maximum) to collect:

1. **Architecture pattern:** Monolith, microservices, or serverless?
2. **Primary technology:** Node.js, Python, Java, or other?
3. **Key components:** Main services or modules?
4. **Data stores:** Databases and caches?
5. **Cloud provider:** AWS, GCP, Azure, or on-premise?
6. **Deployment:** How is it deployed?
7. **External integrations:** Third-party services?

### Step 3: Select Template and Load References

**Choose template based on architecture pattern:**

- Monolith → Use `assets/ARCHITECTURE-monolith.md`
- Microservices → Use `assets/ARCHITECTURE-microservices.md`
- Uncertain → Use `assets/ARCHITECTURE.md`

**Load technology-specific references:**

- Node.js projects → Load `references/nodejs.md` for Express, NestJS, Fastify patterns
- Python projects → Load `references/python.md` for Django, Flask, FastAPI patterns
- Java projects → Load `references/java.md` for Spring Boot, Jakarta EE patterns

**Load pattern-specific references when applicable:**

- Microservices → Load `references/microservices.md` for service boundaries and communication patterns
- Serverless → Load `references/serverless.md` for function organization and event sources

### Step 4: Generate System Diagram

Generate appropriate diagrams using `scripts/generate_diagram.py`:

**For monolithic architectures:**

```bash
python scripts/generate_diagram.py layered
```

**For microservices architectures:**

```bash
python scripts/generate_diagram.py flow
```

**For simple systems:**

```bash
python scripts/generate_diagram.py simple
```

Customize diagrams with JSON configuration for specific components.

### Step 5: Populate Template

Complete the template sections in the specified order:

1. **Project Identification (Section 10)** - Add project name, repository, contact, date
2. **Project Structure (Section 1)** - Define directory layout
3. **System Diagram (Section 2)** - Insert generated diagram
4. **Core Components (Section 3)** - Document components from interview
5. **Data Stores (Section 4)** - Describe databases and caches
6. **External Integrations (Section 5)** - List third-party services
7. **Deployment (Section 6)** - Detail infrastructure setup
8. **Security (Section 7)** - Specify authentication and encryption
9. **Development (Section 8)** - Outline setup and testing procedures
10. **Future Considerations (Section 9)** - Document roadmap items
11. **Glossary (Section 11)** - Define domain terminology

Apply technology-specific patterns from loaded references to enhance each section.

### Step 6: Validate

Execute the validation script to ensure quality:

```bash
python scripts/validate_architecture.py ARCHITECTURE.md
```

Address any issues or warnings before delivering the documentation.

## Interview Best Practices

**Maintain focused questioning:**

- Ask 2-3 questions at a time
- Build upon previous answers
- Skip redundant questions

**Adapt communication style:**

- Technical users: Use precise terminology
- Non-technical users: Simplify language
- Uncertain users: Offer defaults or placeholders

**Handle information gaps:**

- Mark uncertain items for review
- Add [TODO] for missing information
- Suggest reasonable defaults based on context

## Technology-Specific Guidance

### Reference Loading Guidelines

**Load `references/nodejs.md` for:**

- Express, NestJS, Fastify projects
- Node.js microservices
- Serverless Node functions

**Load `references/python.md` for:**

- Django, Flask, FastAPI projects
- Python microservices
- Data pipelines and ML systems

**Load `references/java.md` for:**

- Spring Boot applications
- Jakarta EE systems
- Java microservices

**Load `references/workflows.md` for:**

- Complex interview scenarios
- Detailed process guidance
- Documentation update workflows

### Applying Technology Patterns

After loading references, apply the patterns to enhance:

- Project structure recommendations
- Deployment configurations
- Framework-specific best practices
- Common library suggestions
- Testing strategies and approaches

## Pattern-Specific Guidance

### Microservices Architecture

**Load `references/microservices.md` and include these elements:**

- Service boundaries and responsibilities
- Communication patterns (synchronous vs asynchronous)
- API gateway configuration
- Service discovery mechanism
- Data management approach
- Observability and monitoring setup

**Use the microservices template** (`assets/ARCHITECTURE-microservices.md`) for proper structure.

### Serverless Architecture

**Load `references/serverless.md` and include these elements:**

- Function organization and boundaries
- Event sources and triggers
- State management approach
- Cold start mitigation techniques
- Cost optimization strategies

### Monolithic Architecture

**Use the monolith template** (`assets/ARCHITECTURE-monolith.md`) and emphasize:

- Layered architecture patterns
- Module organization principles
- Potential future refactoring paths
- Scaling strategy and approaches

## Diagram Generation Examples

### Simple Architecture Diagram

```bash
python scripts/generate_diagram.py simple '{"components": ["User", "API", "DB"], "connections": [["User", "API"], ["API", "DB"]]}'
```

### Layered Architecture Diagram

```bash
python scripts/generate_diagram.py layered '{"Presentation": ["Web UI"], "Business": ["API"], "Data": ["PostgreSQL"]}'
```

### Flow Architecture Diagram

```bash
python scripts/generate_diagram.py flow '[{"from": "Client", "to": "Gateway", "label": "HTTP"}, {"from": "Gateway", "to": "Service", "label": "route"}]'
```

### C4 Context Diagram

```bash
python scripts/generate_diagram.py c4 '{"system": "E-commerce Platform", "actors": ["Customer", "Admin"], "external_systems": ["Payment Gateway", "Email Service"]}'
```

Integrate generated diagrams into Section 2 (System Diagram) of the ARCHITECTURE.md template.

## Validation

**Execute validation before delivering documentation:**

```bash
python scripts/validate_architecture.py ARCHITECTURE.md
```

**Validation checks performed:**

- Verify all 11 sections are present
- Confirm required fields in Project Identification section
- Ensure minimal content in each section
- Count and report placeholder usage

**Address any warnings** about missing content or excessive placeholders.

## Documentation Update Workflow

**For incremental updates:**

1. Identify what has changed
2. Update only affected sections
3. Update the date in Section 10 (Project Identification)
4. Re-run validation to ensure quality

**For major updates:**

1. Review the entire document
2. Regenerate diagrams if structure has changed
3. Update multiple sections as needed
4. Consider adding version notes

## Mermaid Diagram Generation

After creating ARCHITECTURE.md, generate the complete set of 5 Mermaid diagrams.

### Load Mermaid Instructions

When users request diagrams or complete documentation packages:

```
Load references/mermaid-diagrams.md
```

### Generate Complete Diagram Set

Create all 5 diagrams following the guidance in mermaid-diagrams.md:

1. **C4 Context** (Level 1) - System in its broader context
2. **C4 Container** (Level 2) - Main application containers
3. **C4 Component** (Level 3) - Internal component structure
4. **Data Flow** - How data moves through the system
5. **C4 Deployment** - Infrastructure topology and deployment

Use `scripts/generate_mermaid.py` with system JSON configuration.

**Save diagrams as separate .mmd files:**

- `01-context.mmd`
- `02-container.mmd`
- `03-component.mmd`
- `04-dataflow.mmd`
- `05-deployment.mmd`

**Embed diagrams in ARCHITECTURE.md Section 2** as code blocks for easy viewing and editing.

## OpenAPI Specification Generation

For systems with REST APIs, generate comprehensive OpenAPI 3.0 specifications.

### Generate API Specifications

Use `scripts/generate_openapi.py` with appropriate parameters:

**For simple CRUD operations:**

```bash
python scripts/generate_openapi.py "ResourceName"
```

**For custom API specifications:**

```bash
python scripts/generate_openapi.py '{"system_name": "...", "endpoints": [...]}'
```

Save the generated specification as `openapi.json` in the project directory.

## Complete Documentation Workflow

Follow this end-to-end workflow for comprehensive architecture documentation:

1. **Conduct structured interview** (5-7 questions maximum)
2. **Select appropriate template** and **load relevant references**
3. **Setup work directory**: Use current working directory `$(pwd)`
4. **Generate ARCHITECTURE.md** with all 11 sections completed
5. **Generate Mermaid diagrams** (5 separate .mmd files) in work directory root
6. **Generate OpenAPI specification** (if applicable) in work directory
7. **Deliver all generated artifacts** to the user

## Deliverable Organization

Organize generated documentation files as follows:

```
├── ARCHITECTURE.md              # Main architecture document
├── openapi.json                 # API specification (if applicable)
├── *.mmd (5 files)             # Mermaid diagram sources
│   ├── 01-context.mmd
│   ├── 02-container.mmd
│   ├── 03-component.mmd
│   ├── 04-dataflow.mmd
│   └── 05-deployment.mmd
└── diagrams/ (created during packaging)
    └── *.png (5 files, if rendered to images)
```

## Example Usage

**User request:** "Create architecture documentation for my Node.js microservices project"

**Execution approach:**

1. Select microservices template (`assets/ARCHITECTURE-microservices.md`)
2. Load `references/nodejs.md` and `references/microservices.md`
3. Conduct focused interview: services, databases, communication patterns, deployment
4. Generate flow diagram using `scripts/generate_diagram.py flow`
5. Populate all sections with Node.js and microservices-specific patterns
6. Validate using `scripts/validate_architecture.py`
7. Create comprehensive ARCHITECTURE.md in work directory

## Common Usage Scenarios

### New Greenfield Projects

- Use base template (`assets/ARCHITECTURE.md`)
- Focus on design decisions and architecture rationale
- Include justification for technology choices
- Emphasize planned architecture and future scalability

### Existing System Documentation

- Ask about current pain points and challenges
- Document the as-is state accurately
- Note planned improvements in Section 9 (Future Considerations)
- Capture current technology stack and limitations

### Legacy System Analysis

- Identify undocumented or poorly understood areas
- Mark uncertain items for further investigation
- Suggest areas requiring clarification from stakeholders
- Document assumptions and risks

### Architecture Reviews and Updates

- Update only sections that have changed
- Preserve information that remains accurate
- Add new components and relationships
- Update date in Section 10 and note changes

## Best Practices

**Efficient documentation creation:**

- Begin with known information and build incrementally
- Use placeholders for unknown items to maintain momentum
- Leverage technology references to save time and ensure accuracy
- Validate frequently to catch issues early

**High-quality output standards:**

- Provide specific, concrete details rather than generic descriptions
- Include actual technology stack versions and configurations
- Use real service names, purposes, and data flows
- Document concrete deployment infrastructure and environments

**Positive user experience:**

- Avoid overwhelming users with excessive questions
- Explain the documentation process and next steps
- Show progress through the 11 sections clearly
- Offer refinement and improvement after initial delivery
