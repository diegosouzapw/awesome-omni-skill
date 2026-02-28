---
name: c4-modeling-process
description: C4 modeling methodology - design system hierarchy top-to-bottom from Context to Components. Ensures consistent, stakeholder-focused architecture documentation.
---

# C4 Modeling Process

Use this skill to design system models following the C4 framework methodology.

**Core principle:** Always design **top-to-bottom** (C1 → C2 → C3) to ensure clarity and progressive detail.

---

## C4 Framework Overview

The C4 model organizes architecture documentation at four levels, each with specific purpose and audience:

| Level | Name | Purpose | Audience | Detail Level |
|-------|------|---------|----------|--------------|
| **C1** | **Context** | System boundary and external actors | Everyone (business, technical) | Highest overview |
| **C2** | **Container** | Major components and technologies | Architects, developers | System internals |
| **C3** | **Component** | Modules within a container | Developers | Low-level design |
| **C4** | **Code** | Classes and methods | Developers | Implementation (usually code, not diagrams) |

---

## Step 1: Define C1 Context (System Boundary)

**Goal:** Establish what the system is, who uses it, and what external systems it interacts with.

### Define the System

```likec4
model {
  mySystem = System 'My Product' {
    description 'Core business system serving customers'
  }
}
```

### Identify External Actors

Determine WHO interacts with the system:

```likec4
model {
  // Primary users
  customer = Actor_Person 'Customer' {
    description 'End user of the product'
  }
  
  admin = Actor_Person 'Administrator' {
    description 'System administrator managing configuration'
  }
  
  // Services/automated actors
  scanner = Actor_System 'Virus Scanner' {
    description 'External security scanning service'
  }
  
  // Infrastructure
  browser = Container_Browser 'Web Browser' {
    technology 'Chrome / Firefox / Safari'
    description 'User web browser for accessing the system'
  }
}
```

### Identify External Systems

Determine WHAT other systems your system integrates with:

```likec4
model {
  // Payment processing
  paymentGateway = System_Existing 'Payment Gateway' {
    #external
    description 'Third-party payment processing'
  }
  
  // Data integration
  warehouse = System_Existing 'Data Warehouse' {
    #external
    description 'Cloud data warehouse for analytics'
  }
  
  // Communication
  emailService = System_Existing 'Email Service' {
    #external
    description 'Third-party email delivery'
  }
}
```

### Create C1 Relationships

Connect actors to your system:

```likec4
model {
  customer -> mySystem 'Uses'
  admin -> mySystem 'Administers'
  mySystem -> paymentGateway 'Processes payments via'
  mySystem -> emailService 'Sends emails via'
  scanner -> mySystem 'Scans files in'
}
```

### C1 Checklist

- [ ] System is clearly defined with purpose
- [ ] All primary actors (users/roles) identified
- [ ] All external systems/services identified
- [ ] All major relationships documented
- [ ] One C1 Context View created showing the boundary

**Example from refactored project:**
```likec4
views 'C1' {
  view c1_context {
    title 'Secure Vault System'
    include customer
    include browser
    include vault
    include scanner
    
    rank source { customer }
    rank sink { scanner }
  }
}
```

---

## Step 2: Define C2 Containers (System Architecture)

**Goal:** Break down the system into major building blocks and show how they communicate.

### What is a Container?

A container represents a **runtime boundary** - something that must be running for the system to work. It encapsulates either:
- **Code execution:** An application, service, or function running in a process space
- **Data storage:** A database, file system, blob store, or any persistent data

**NOT a Docker container!** (confusing naming). A container is defined by its runtime deployment unit, not the infrastructure platform.

**Examples of containers:**
- Server-side web application (e.g., Node.js, Django, Spring Boot)
- Client-side web application (e.g., React SPA in browser)
- Desktop or mobile application
- Microservice or serverless function
- Database (a schema or entire DBMS instance)
- Message queue or event bus
- Object storage (S3, MinIO, Blob Storage)
- File system or cache
- Batch process or scheduled task

**Key distinction:** Multiple containers can run on the same physical/virtual machine or runtime platform, but they still remain separate containers because they have separate process spaces or deployment units.

### Container Design Strategy

Design containers around **independent deployment**:
- Can it be deployed independently from other parts?
- Does it have its own process space / runtime boundary?
- Can it evolve and scale independently?

Do NOT design containers by:
- ❌ Technology stack (orthogonal concern)
- ❌ Team ownership alone (too organizational)
- ❌ Code organization (JAR files, modules - that's code-level organization)

Focus on: **Runtime boundaries and independent deployment**

### Example: Container vs Code Organization

```likec4
// RIGHT: Design by runtime boundary
model {
  mySystem = System 'My Product' {
    frontend = Container_Spa 'Web UI' {
      description 'React SPA running in browser (separate process space)'
    }
    api = Container_API 'REST API' {
      description 'Node.js service (can deploy independently)'
    }
    database = Container_Database 'Database' {
      description 'PostgreSQL instance (independent data store)'
    }
  }
}

// WRONG: Design by code organization
// ❌ Don't create containers for:
// - UserService.js class
// - models/ folder
// - repositories/ module
// These are code-level organization, not runtime boundaries
```

### Create C2 Relationships

Connect containers showing:
- Synchronous calls (API requests, direct calls)
- Asynchronous messaging (queue, events)
- Data flows (reads/writes to storage)

```likec4
model {
  // Synchronous flows
  frontend -> api 'HTTP requests'
  api -> database 'Query/update'
  api -> storage 'Upload/download files'
  
  // Asynchronous flows
  api -> queue 'Publish async job'
  queue -> worker 'Deliver job'
  worker -> database 'Update status'
  worker -> storage 'Access files'
}
```

### C2 Checklist

- [ ] System broken into logical deployable units
- [ ] Each container has clear purpose and responsibility
- [ ] All technologies documented
- [ ] All major relationships between containers shown
- [ ] Synchronous vs. asynchronous patterns clear
- [ ] One C2 Container View showing all containers

**Example from refactored project:**
```likec4
views 'C2' {
  view c2_container {
    title 'Vault System Internals'
    include customer
    include browser
    include vault.*
    include scanner
    
    rank source { customer }
    rank sink { scanner }
  }
}
```

---

## Step 3: Define C3 Components (Container Internals)

**Goal:** Show internal structure of important containers by grouping code-level building blocks.

### What is a Component?

A component is a **grouping of related functionality behind a well-defined interface**.

**Critical distinction:**
- Components are NOT separately deployable
- All components within a container execute in the same process space
- Components are code-level organization, not runtime boundaries
- A component might span multiple code files, classes, modules, packages, etc.

**Examples:**
- A group of related classes implementing a service interface
- A JavaScript module with exported functions
- A microservice layer (controllers, services, repositories)
- A set of C files in a directory
- A functional programming module grouping related functions

### Components vs Code Organization

```likec4
// RIGHT: Component as logical grouping with responsibility
model {
  api = Container_API 'REST API' {
    router = Component_Service 'API Router' {
      description 'Express middleware - routes requests'
    }
    auth = Component_Service 'Authentication' {
      description 'JWT validation, user authentication'
    }
    business = Component_Service 'Business Logic' {
      description 'Domain logic and business rules'
    }
    dataAccess = Component_Service 'Data Repository' {
      description 'Abstracts database queries'
    }
  }
}

// WRONG: Don't create C3 components for:
// ❌ UserController.java class
// ❌ UserService.java class
// ❌ UserRepository.java class
// These are individual classes, not logical groupings

// ALSO WRONG: Don't use C3 for code organization:
// ❌ models/ folder as component
// ❌ utils/ folder as component
// These are organizational structure, not functional groupings
```

### Component Grouping Strategy

Group classes/code by **related functionality and responsibility**, not by:
- ❌ Code organization (folders, packages, modules)
- ❌ Technology framework (controllers, services, repositories as separate components)
- ❌ Architectural layers (don't create component per layer)

DO group by:
- ✅ Shared responsibility or feature
- ✅ Related business concepts
- ✅ Well-defined internal interface

**Example from refactored project - Upload Service:**
```likec4
views 'C3' {
  view c3_upload_service {
    title 'Upload Service'
    include customer
    include browser
    include vault.*
    include vault.uploadService.*
    include vault.minio.*
    include scanner
    
    rank source { customer }
    rank sink { scanner }
  }
}
```

---

## Step 4: Validate the Hierarchy

### Naming Consistency

```
C1: c1_<system_name>           → Title: <System Name>
C2: c2_<system_name>           → Title: <System Name> Internals
C3: c3_<container_name>        → Title: <Container Name>
```

### Completeness Check

```
✓ C1: 1 system + N actors + M external systems + relationships
✓ C2: 1 container view showing all major containers + all relationships
✓ C3: 1 view per complex/critical container showing internal components
✓ Views: C1, C2, and C3 views created in system-views.c4
```

### Consistency Validation

- [ ] Every C3 view has a corresponding C2 container
- [ ] C3 view ID matches its C2 container name
- [ ] Every container appears in both C2 and its C3 (if exists)
- [ ] All relationships consistent across levels
- [ ] Technology documented at all levels

---

## Design Workflow Summary

```
1. START: Define C1 Context
   ├─ Identify system boundary
   ├─ List all external actors
   ├─ List all external systems
   └─ Create 1 C1 Context view
   
2. EXPAND: Define C2 Containers
   ├─ Break system into deployable units
   ├─ Show container relationships
   └─ Create 1 C2 Container view (shows all)
   
3. DETAIL: Define C3 Components (for complex containers)
   ├─ Choose containers needing detail
   ├─ Show internal modules/components
   └─ Create 1 C3 view per important container
   
| | | |
| ✓ Step 4: Validate consistency | Naming, completeness, relationships | Manual review |
| ✓ Step 5: Deployment diagrams | Infrastructure and deployment | `model-deployment` + `design-view` |
| ✓ Step 6: Dynamic diagrams | Runtime behaviors and workflows | `create-sequence-view` + `design-view` |
| **NEW: Step 7: Validation** | Quality checklist and syntax | `test-model` + MCP `validate` |

---

## Step 7: Validate Model (Official C4 Checklist)

**Goal:** Review all diagrams against official C4 checklist and LikeC4 syntax validation.

### Official C4 Diagram Review Checklist

Based on https://c4model.com/diagrams/checklist

#### General (All Diagrams)

Every diagram must have:
- [ ] **Title** - Clear, descriptive title
- [ ] **Diagram type** - Clearly identifies C1, C2, C3, Deployment, Dynamic, etc.
- [ ] **Scope** - Clearly states what the diagram covers (system, container, component, use case, environment)
- [ ] **Key/Legend** - Explains colors, shapes, icons, line styles if used

#### Elements (All Diagrams)

Every element shown must have:
- [ ] **Name** - Clear, meaningful name
- [ ] **Type clarity** - You understand the element type (actor, system, container, component)
- [ ] **Purpose** - Clear what it does (from title + description)
- [ ] **Technology** - Where applicable, technology choices are explicit
- [ ] **No undefined acronyms** - All abbreviations are explained or clear from context
- [ ] **Colors explained** - If colors differ by type, it's documented
- [ ] **Shapes explained** - If shapes are used (person, rectangle, cylinder), meaning is clear
- [ ] **Icons explained** - Tech stack icons are identified in legend
- [ ] **Styling clear** - Border styles, sizing means something clear

#### Relationships (All Diagrams)

Every connection/arrow must have:
- [ ] **Label describing intent** - What is the relationship purpose?
- [ ] **Direction clear** - Arrow shows correct direction of flow/dependency
- [ ] **Technology documented** - Protocols, API types, message formats noted if significant
- [ ] **No undefined acronyms** - Communication method is clear (HTTP, gRPC, message queue, etc.)
- [ ] **Colors meaningful** - If different line colors, meaning is explained
- [ ] **Arrow styles clear** - Solid vs dashed, thick vs thin means something
- [ ] **Line styles meaningful** - Different line styles should indicate relationship type

### LikeC4 Syntax Validation

LikeC4 provides automated validation to catch errors:

```bash
# Validate entire project for syntax and layout errors
npx likec4 validate

# Validate specific directory
npx likec4 validate ./my-architecture

# Skip layout validation (if manual positioning intentional)
npx likec4 validate --ignore-layout
```

**What LikeC4 validates:**
- ✓ Element references are valid (no broken links)
- ✓ Relationship endpoints exist
- ✓ View includes/excludes are syntactically correct
- ✓ Kind definitions match specification
- ✓ No duplicate element IDs
- ✓ No circular relationships (where applicable)
- ✓ Manual layout consistency (elements in bounds, no overlap if enforced)

### LikeC4 Element & View Quality Checklist

For each element in model:
- [ ] **Element has description** - Not just title, but what is it and why
- [ ] **Technology documented** - `technology 'Node.js, Express'` where applicable
- [ ] **Tags applied appropriately** - Using consistent tags from specification
- [ ] **Icons set for clarity** - `icon tech:nodejs`, `icon aws:lambda` etc.
- [ ] **Metadata complete** - Links to documentation, ownership info if relevant

For each view:
- [ ] **Title is descriptive** - "Vault System Internals" not just "Architecture"
- [ ] **Description explains scope** - What does this view show? Why?
- [ ] **Include statements are scoped** - Not too broad (avoid `include **`), not too narrow
- [ ] **Exclude only if necessary** - Generally prefer carefully scoped includes
- [ ] **Relationships labeled** - Every line has descriptive label
- [ ] **View tags applied** - `#production`, `#critical`, `#external` etc.
- [ ] **External links present** - If this diagram relates to documentation, Jira, ADRs

**Example - Well-documented view:**
```likec4
views 'C2' {
  view c2_container {
    title 'Secure Vault System Internals'
    
    description """
      Shows major containers (services and data stores) and how they communicate.
      
      **Audience:** Architects and developers
      **Scope:** All containers within Vault system
    """
    
    #architecture, #logicallayer
    
    include customer, browser
    include vault.*
    include scanner
    
    link https://wiki.internal/vault 'Vault Documentation'
    link https://jira.internal/PROJ-123 'Architecture Decision'
    
    rank source { customer }
    rank sink { scanner }
  }
}
```

### Complete Validation Workflow

```
1. Syntax validation (automated)
   └─ Run: npx likec4 validate
   └─ Fix: Broken references, syntax errors

2. Element quality review (manual)
   └─ Check: All elements have description, technology, icons
   └─ Verify: Naming conventions consistent

3. Relationship review (manual)
   └─ Check: Every relationship labeled
   └─ Verify: Technology and direction clear

4. View completeness (manual)
   └─ Check: Title, description, scope all present
   └─ Verify: Includes/excludes appropriately scoped

5. C4 Diagram checklist (against official C4)
   └─ Check: General, Elements, Relationships sections
   └─ Verify: All "Yes" answers on C4 review checklist
```

### When Validation Fails

**Syntax errors** → Run `npx likec4 validate` to find issues

**Element quality issues:**
- ❌ Element with no description → Add: `description 'Purpose and responsibility'`
- ❌ Container without technology → Add: `technology 'Node.js, Express'`
- ❌ Missing icons → Add: `icon tech:nodejs`

**Relationship issues:**
- ❌ Unlabeled arrow → Add label: `api -> database 'Query data'`
- ❌ Unclear direction → Verify arrow points correct way
- ❌ No technology → Add: `api -> database 'SQL queries'`

**View issues:**
- ❌ No description → Add: `description 'Shows....'`
- ❌ Over-broad includes → Change `include **` to `include system.*`
- ❌ Missing titles → Add: `title 'System Context'`

### Validation Success Criteria

✅ All diagrams pass `npx likec4 validate` without errors
✅ Official C4 checklist: All boxes checked "Yes"
✅ LikeC4 quality:
   - Every element has description and technology (where applicable)
   - Every relationship has label
   - Every view has title, description, scope
   - All external links working
✅ Consistency:
   - Naming conventions followed (C1/C2/C3, view IDs)
   - Terminology consistent (no "API" vs "api" confusion)
   - Relationships match across all levels

---
   ├─ Model infrastructure (VMs, databases, services)
   ├─ Show how containers are deployed
   └─ Create views per environment (dev, staging, prod)

6. EXPLAIN: Create dynamic diagrams
   ├─ Choose 2-5 important use cases/workflows
   ├─ Show temporal sequence of interactions
   └─ Create dynamic view per significant use case
```

**Then use related skills:**
- `design-view` - Organize and visualize all diagrams
- `model-deployment` - Define infrastructure and deployment
- `create-sequence-view` - Document runtime behaviors
```

---

## Step 5: Deployment Diagrams (Infrastructure & Runtime)

**Goal:** Show how software systems and containers are deployed to infrastructure within a given environment.

### What is a Deployment Diagram?

A deployment diagram illustrates:
- **How containers are deployed** to infrastructure (VMs, servers, cloud, Docker, etc.)
- **Infrastructure nodes** (physical servers, VMs, cloud instances, databases, load balancers)
- **Relationships between containers and infrastructure** (which container runs where)
- **Multiple environments** (production, staging, development shown separately)

**Key distinction from C1/C2/C3:**
- C1/C2/C3 show **logical architecture** (design)
- Deployment shows **physical architecture** (runtime implementation)

### Deployment Diagram Elements

```
Deployment Node
├─ Physical infrastructure (data centers, computers)
├─ Virtualized infrastructure (VMs, cloud instances)
├─ Containerized infrastructure (Docker, Kubernetes)
├─ Execution environments (Java app servers, databases)
└─ Can be nested (VM contains Docker containers contains apps)

Container/System Instance
├─ Deployed instance of a container running on a node
└─ Multiple instances possible (scaling, replication)

Infrastructure Nodes
├─ DNS, load balancers, firewalls
├─ CDN, gateways
├─ Not deployed containers, but infrastructure services
└─ Shows support infrastructure
```

### Example Deployment: Production with Multiple Tiers

```likec4
deployment view production {
  title 'Production Deployment'
  
  include
    Internet.UserBrowser,
    Internet.CDN ->,
    Prod.DMZ,
    Prod.DMZ.** where tag is #Vm,
    Prod.AppTier.** where tag is #Vm,
    Prod.DataTier.** where tag is #Vm,
    Prod.AppTier.* ->,
    Prod.DataTier.* ->
}
```

### Deployment Diagrams Are Recommended

**Official C4 says:** Yes, deployment diagrams are **recommended** for:
- ✅ Understanding infrastructure organization
- ✅ Infrastructure/DevOps/SRE audiences
- ✅ Multiple environments (dev, staging, production)
- ✅ Deployment strategy and scaling

### When to Create Deployment Diagrams

- One diagram per environment (production, staging, development)
- Separate diagrams if environments differ significantly
- Can zoom in on specific tiers (app tier, data tier)
- Show infrastructure nodes (load balancers, firewalls, CDN)

**Skill reference:** Use `model-deployment` skill for detailed infrastructure modeling and `design-view` for organizing deployment diagrams

---

## Step 6: Dynamic Diagrams (Runtime Behavior & Interactions)

**Goal:** Show how containers/components interact at runtime to implement features, workflows, or use cases.

### What is a Dynamic Diagram?

A dynamic diagram illustrates:
- **How elements collaborate at runtime** to deliver a specific feature or use case
- **Temporal sequence** of interactions (ordering of messages)
- **Data flows** through the system during operation
- **Interesting or recurring patterns** in the system

**Two styles available:**
1. **Sequence style:** Vertical timeline showing order of interactions (UML sequence diagram style)
2. **Collaboration style:** Free-form layout with numbered interactions (UML communication diagram style)

LikeC4 uses **sequence style** (numbered arrows showing order).

### What Dynamic Diagrams Show

```
Dynamic diagrams can operate at different levels:
- System level: How external systems interact (like C1 but with flow)
- Container level: How containers talk during a use case (most common)
- Component level: How components inside a container interact
```

### Example Dynamic Diagram - File Upload Use Case

```likec4
views 'Use Cases' {
  dynamic view upload_flow {
    title 'File Upload'
    
    customer -> browser 'Upload file'
    browser -> api 'POST /upload with file'
    api -> database 'Check file quota'
    database -> api 'Quota OK'
    api -> storage 'Store encrypted file'
    storage -> api 'File stored'
    api -> queue 'Publish processing job'
    queue -> worker 'Deliver job'
    worker -> scanner 'Scan for malware'
    scanner -> worker 'Safe'
    worker -> database 'Update file status'
    worker -> browser 'Notify user'
  }
}
```

### Dynamic Diagram Audience

- **Technical people:** Software architects, developers, DevOps
- **Non-technical people:** Product managers, business analysts
- **Shows:** What happens when user does X (feature demonstration)

### Use Sparingly (Official C4 Guidance)

Dynamic diagrams should show:
- ✅ **Interesting patterns:** Complex interactions worth explaining
- ✅ **Recurring workflows:** Important features users care about
- ✅ **Non-obvious flows:** Where behavior isn't obvious from C1/C2/C3
- ❌ **NOT every interaction:** That creates too many diagrams

Recommendation: **Create 2-5 dynamic diagrams maximum**

### When to Create Dynamic Diagrams

Create dynamic diagrams for:
1. **Primary user workflows** (happy path, main features)
2. **Complex interactions** (async processing, error handling, orchestration)
3. **Integration patterns** (external system interactions, API flows)
4. **Disaster recovery** (failover, replication flows, recovery procedures)

**Skill reference:** Use `create-sequence-view` skill for detailed guidance. Place in `views 'Use Cases'` subfolder with titles like "File Upload", "User Login", "Backup Restore"

---

## Complete C4 Documentation Pyramid

```
STATIC ARCHITECTURE         BEHAVIORAL & INFRASTRUCTURE

C1 CONTEXT (What?)         Dynamic (How does it work?)
    ↓
C2 CONTAINERS (How built?)
    ↓                       Deployment (Where runs?)
C3 COMPONENTS (Internals)
```

### Views to Create (Recommended Set)

**View placement convention (mandatory):** Every view must live inside a category folder using `views 'FolderName'`, **except** the **index** view, which must be at the root:

```likec4
views {
  view index extends c1_context { }
}
```
- `C1`, `C2`, `C3` → static architecture views in `system-views.c4`
- `Use Cases` → dynamic views in `system-sequences.c4`
- `Deployment` → deployment views in `deployment-views.c4`
- `Operations` → security/monitoring/DR/CI/CD views in `operations-views.c4`

**Static Structure (Architecture Design):**
- [ ] 1 C1 Context view (always create)
- [ ] 1 C2 Container view (always create)
- [ ] 2-5 C3 Component views (for complex/critical containers)
- [ ] LikeC4 index view for navigation: `view index extends c1_context { }`
  - If a C0 landscape view exists, extend that instead (e.g., `view index extends c0_landscape { }`)
  - This is a **LikeC4 view convention**, not an additional C4 level

**Behavioral & Infrastructure:**
- [ ] 2-5 Dynamic diagrams (important use cases/workflows)
- [ ] 1-3 Deployment diagrams (per environment: dev, staging, prod)

**Quality Assurance:**
- [ ] Run `npx likec4 validate` - syntax and reference validation
- [ ] Review against C4 checklist - General, Elements, Relationships sections
- [ ] LikeC4 quality check - descriptions, technology, labels complete

**Optional Supporting (if needed):**
- [ ] System Landscape (if multiple systems exist)
- [ ] Operational views (monitoring, DR, CI/CD)

---

---

❌ **Bottom-up design:** Starting with classes/code and working up
- Problem: Loses sight of system boundaries and user needs
- Solution: Always start with C1 context

❌ **Too many containers:** Every class as a container
- Problem: C2 becomes unreadable, loses architectural overview
- Problem: Adds noise instead of clarity
- Solution: Group related classes into logical containers

❌ **Too many components:** Adding C3 for every container
- Problem: Creates documentation burden, most containers are simple
- Solution: Only C3 for complex/critical containers

❌ **Inconsistent naming:** `api_gateway`, `ApiGateway`, `api-gateway`
- Problem: Makes it hard to track relationships
- Solution: Use consistent naming: kind in PascalCase, variable in camelCase

❌ **Missing relationships:** Focusing only on structure, not communication
- Problem: Doesn't show how system actually works
- Solution: Document all significant relationships at each level

❌ **Unclear containers:** "Service", "Module", "Component"
- Problem: Doesn't describe actual purpose or responsibility
- Solution: Use descriptive names reflecting domain: `uploadService`, `authService`, `documentDB`

---

## Key Distinctions (Official C4 Model)

### Containers: Runtime Boundaries, Not Technology

❌ **Wrong:** "We use Node.js so it's one container, Java so it's another"
✅ **Correct:** "This runs in its own process space (separate deployment unit) so it's a container"

A single Tomcat server can run multiple containers (web apps), even though they share the same Java runtime. At deployment time, they might be on separate servers. Either way, they're separate containers.

### Components: Code-Level Grouping, NOT Separately Deployable

❌ **Wrong:** "Each class is a component"
❌ **Wrong:** "Each folder/package is a component"
❌ **Wrong:** "Components can be deployed independently"

✅ **Correct:** "Component is a logical grouping of related classes/code"
✅ **Correct:** "All components in a container execute in the same process space"
✅ **Correct:** "Only the container is deployable, not individual components"

### Your Definition Aligns with C4 Practice

Your view: "Containers are an aggregate of components, independently deployable, technically autonomous, functionally dependent"

This is **exactly right** and actually **more practical** than the official C4 terminology:
- **"Aggregate of components"** = Container contains multiple components (code groupings)
- **"Independently deployable"** = Container is the deployment unit
- **"Technically autonomous"** = Container has its own process space / runtime boundary
- **"Functionally dependent"** = System requires multiple containers working together

---

## When to Use This Skill

This skill covers the complete C4 modeling process from start to finish:

- **Starting new model:** Follow C1 → C2 → C3 → Deployment → Dynamic progression
- **Understanding existing model:** Learn what each level represents and how they connect
- **Adding to existing model:** Use this to understand where new elements fit in hierarchy
- **Refactoring architecture:** Validate C1/C2/C3 before changing C3/Deployment details
- **Reviewing someone's model:** Check if follows C4 principles and completeness

---

## Complete Workflow Summary

```
PHASE 1: DESIGN ARCHITECTURE
├─ Step 1: C1 Context (system boundary, actors, external systems)
├─ Step 2: C2 Container (deployable units and relationships)
├─ Step 3: C3 Components (internal modules for complex containers)
└─ Step 4: Validate hierarchy (naming, completeness, consistency)

PHASE 2: VISUALIZE DEPLOYMENT & BEHAVIOR
├─ Step 5: Deployment diagrams (infrastructure per environment)
├─ Step 6: Dynamic diagrams (runtime workflows and use cases)
└─ Step 7: VALIDATE (official C4 checklist + LikeC4 syntax)
```

### Skills Used at Each Step

| Step | Skill | Purpose |
|------|-------|---------|
| 1-3 | `c4-modeling-process` (this) | Design C1/C2/C3 hierarchy |
| 1-3 | `create-element` | Create system, containers, components |
| 2-3 | `create-relationship` | Define interactions between elements |
| 1-6 | `design-view` | Create and organize views in subfolders |
| 4 | `model-deployment` | Model infrastructure and deployment nodes |
| 5 | `design-view` | Create deployment views and organize |
| 6 | `create-sequence-view` | Create dynamic/sequence diagrams |
| **7** | **`test-model`** | **Validate syntax and consistency** |
| **7** | **LikeC4 CLI** | **Run `npx likec4 validate`** |

---

## Next Steps After Modeling

Once C4 model with deployment and dynamic diagrams is complete:

1. **Create relationships:** Use `create-relationship` skill to document all interactions
2. **Organize views:** Use `design-view` skill to create views in correct subfolders
3. **Model infrastructure:** Use `model-deployment` skill for detailed deployment definition
4. **Document use cases:** Use `create-sequence-view` skill for dynamic interactions
5. **Test model:** Use `test-model` skill to validate consistency and correctness
6. **Document decisions:** Use `document-decision` skill to record architecture choices

---

## Resources

### Official C4 Model
- **C4 Model Home:** https://c4model.com/ - Complete C4 documentation
- **System Context:** https://c4model.com/diagrams/system-context
- **Container:** https://c4model.com/diagrams/container
- **Component:** https://c4model.com/diagrams/component
- **Deployment:** https://c4model.com/diagrams/deployment
- **Dynamic:** https://c4model.com/diagrams/dynamic
- **Review Checklist:** https://c4model.com/diagrams/checklist (integrated in Step 7)

### LikeC4 Documentation
- **LikeC4 Official:** https://likec4.dev/
- **LikeC4 Docs:** https://docs.likec4.dev/
- **LikeC4 CLI Validate:** `npx likec4 validate` command for syntax validation

### Related Skills
- `create-element` - Creating individual elements
- `create-relationship` - Defining relationships
- `design-view` - Creating and organizing views
- `model-deployment` - Infrastructure definition
- `create-sequence-view` - Dynamic/workflow diagrams
- `test-model` - Testing and validating models
