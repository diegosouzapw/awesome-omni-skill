---
name: new-product
description: Deep research and architecture design for new product development
args:
  input:
    type: string
    description: Path to documentation files or product description
    required: true
---

# New Product - Deep Research & Architecture Design

Given documentation or a product description, perform deep research and cross-reference technologies to determine the optimal architecture for building a new application.

## Your Task

You are orchestrating a deep research workflow to design product architecture. Follow these stages in order, using parallel agents for deep research, and present a final comprehensive review at the end.

---

## Stage 0: Initialize Product Workspace

**Parse input argument:**
- If the input is a file path (contains `/` or ends in `.md`, `.txt`, etc.), read the documentation
- If the input is plain text, treat it as a product description

**Extract product name:**
- From documentation: look for project title, product name
- From description: derive a short kebab-case name (e.g., "real-time chat app" → "realtime-chat")

**Create folder structure:**
```bash
mkdir -p /job-queue/product-{name}/research-notes
```

**Output:**
- Display: "🚀 Stage 0/6: Workspace created at /job-queue/product-{name}/"
- Store the product name for use in later stages

---

## Stage 1: Big Idea & High-Level Research

**Goal:** Generate a comprehensive high-level understanding of the product and initial technology landscape.

### Step 1.1: Analyze Input

**If documentation provided:**
- Read all provided files
- Extract key concepts: features, user flows, requirements
- Identify technical hints or constraints mentioned

**If product description provided:**
- Parse the description for:
  - Core functionality
  - Target users
  - Key features
  - Scale expectations

### Step 1.2: Ask Initial Clarifying Questions

Use `AskUserQuestion` to gather essential context:

**Questions to ask:**

1. **Product Vision**
   - Header: "Vision"
   - Question: "What is the primary goal of this product?"
   - Options:
     - "Solve a specific problem for users" - "Focus on targeted pain points"
     - "Innovate in an existing space" - "Bring new approach to established market"
     - "Integrate multiple services" - "Connect disparate systems"
     - "Explore new technology" - "Research/experimental project"

2. **Scale & Audience**
   - Header: "Scale"
   - Question: "What scale are you targeting?"
   - Options:
     - "Prototype/MVP (100s of users)" - "Proof of concept"
     - "Small-Medium (1,000s of users)" - "Early stage product"
     - "Medium-Large (100,000s of users)" - "Established product"
     - "Enterprise scale (1M+ users)" - "High-scale production"

3. **Deployment Context**
   - Header: "Deployment"
   - Question: "Where will this product run?"
   - Options:
     - "Cloud-hosted (AWS/GCP/Azure)" - "Managed infrastructure"
     - "Self-hosted/On-prem" - "Customer infrastructure"
     - "Edge/Distributed" - "CDN, edge compute"
     - "Hybrid" - "Mix of cloud and local"

### Step 1.3: High-Level Technology Research

Use `WebSearch` to research broad technology landscape:

**Search queries:**
- "[product type] architecture patterns 2026"
- "[product type] technology stack comparison 2026"
- "[product type] best practices 2026"
- "[scale level] architecture considerations 2026"

**For each search result:**
- Use `WebFetch` to read top 2-3 articles
- Extract: common patterns, recommended technologies, pitfalls to avoid

**Save research notes:**
```bash
Write to: /job-queue/product-{name}/research-notes/stage1-research.md
```

### Step 1.4: Generate big-idea.md

**Content structure:**

```markdown
# Big Idea: [Product Name]

## Product Overview
[2-3 paragraphs describing what the product does, who it's for, and why it matters]

## Core Value Proposition
[What makes this product unique or valuable]

## High-Level Architecture Approach
[Based on research, what architectural style makes sense?]
- Monolithic vs Microservices
- Serverless vs Server-based
- Event-driven vs Request-response
- [Any other key architectural decisions]

## Technology Landscape
[Summary of technology options researched]

### Frontend Options
- Option 1: [technology] - [pros/cons]
- Option 2: [technology] - [pros/cons]
- Option 3: [technology] - [pros/cons]

### Backend Options
- Option 1: [technology] - [pros/cons]
- Option 2: [technology] - [pros/cons]
- Option 3: [technology] - [pros/cons]

### Database Options
- Option 1: [technology] - [pros/cons]
- Option 2: [technology] - [pros/cons]

### Deployment Options
- Option 1: [platform] - [pros/cons]
- Option 2: [platform] - [pros/cons]

## Key Challenges Identified
1. [Challenge 1 and mitigation approach]
2. [Challenge 2 and mitigation approach]
3. [Challenge 3 and mitigation approach]

## Next Steps
- Deep dive into runtime execution patterns
- Research abstraction layer approaches
- Evaluate integration strategies
- Design output rendering pipeline

## Research Sources
[List of URLs and documentation consulted]
```

**Save to:**
```bash
Write to: /job-queue/product-{name}/big-idea.md
```

**Output:**
- Display: "✅ Stage 1/6: Big idea generated at product-{name}/big-idea.md"

---

## Stage 2-5: Parallel Deep Research for Architectural Documents

Launch **4 parallel research agents** using the `Task` tool with `subagent_type="general-purpose"`.

Each agent is assigned one architectural document and operates independently.

### Stage 2: Runtime Execution Research (Agent 1)

**Agent task:**
```
You are researching runtime execution architecture for: [Product Name]

CONTEXT:
[Include big-idea.md content]

YOUR GOAL:
Create a comprehensive `runtime-execution.md` document that explains how the system executes work.

RESEARCH APPROACH:
1. Based on the technology options in big-idea.md, research each option's runtime model
2. Use WebSearch to find:
   - "[technology] runtime architecture 2026"
   - "[technology] execution model 2026"
   - "[technology] process lifecycle 2026"
   - "[technology] concurrency model 2026"

3. Use WebFetch to read official documentation for top candidates

4. Ask user clarifying questions about runtime preferences using AskUserQuestion

QUESTIONS TO ASK USER:

1. **Execution Model**
   - Header: "Execution"
   - Question: "How should the system execute work?"
   - Options:
     - "Request-response" - "Traditional HTTP request/response"
     - "Event-driven" - "Async processing with queues"
     - "Stream processing" - "Continuous data flow"
     - "Batch processing" - "Scheduled periodic jobs"

2. **Concurrency Needs**
   - Header: "Concurrency"
   - Question: "What concurrency model do you need?"
   - Options:
     - "Single-threaded async" - "Node.js style, high I/O"
     - "Multi-threaded" - "Python/Java threads, CPU work"
     - "Actor model" - "Erlang/Akka style isolation"
     - "Function-based" - "Serverless auto-scaling"

3. **State Management**
   - Header: "State"
   - Question: "How should runtime state be managed?"
   - Options:
     - "Stateless" - "No server-side state, scale easily"
     - "In-memory cache" - "Redis/Memcached for sessions"
     - "Persistent state" - "Database-backed state"
     - "Distributed state" - "Multi-node coordination"

DOCUMENT STRUCTURE:

Create: /job-queue/product-{name}/runtime-execution.md

# Runtime Execution Architecture

## Executive Summary
[2-3 paragraphs: What runtime model was chosen and why]

## Core Execution Engine

### Selected Technology
- **Primary Runtime**: [chosen technology]
- **Rationale**: [why this choice]
- **Trade-offs**: [what we're giving up]

### Execution Paradigm
[Describe: request-response, event-driven, streaming, batch]

## Process Lifecycle

### Initialization
[How the system starts up, bootstraps, loads config]

### Request Handling
[How incoming work is received and routed]

### Execution Flow
[Step-by-step: what happens when work arrives]

### Teardown/Cleanup
[How resources are released, graceful shutdown]

## State Management

### State Architecture
[Where state lives: memory, cache, database]

### State Lifecycle
[How state is created, updated, invalidated]

### State Consistency
[How consistency is maintained across requests]

## Concurrency Model

### Concurrency Approach
[Threads, async, actors, serverless]

### Parallelism Strategy
[How work is distributed across cores/nodes]

### Synchronization
[Locks, queues, coordination mechanisms]

## Hot-Reload & Dynamic Updates

### Development Experience
[Hot reload during dev? How does it work?]

### Production Updates
[Zero-downtime deploys? Blue-green? Rolling?]

## Resource Constraints & Optimization

### Memory Management
[How memory is allocated, GC considerations]

### CPU Utilization
[How CPU is used, optimization strategies]

### I/O Patterns
[Network, disk, database access patterns]

### Scaling Strategy
[Vertical vs horizontal, auto-scaling triggers]

## Technology Comparison

### Evaluated Options
| Technology | Pros | Cons | Fit Score |
|------------|------|------|-----------|
| [Tech 1]   | ...  | ...  | 8/10      |
| [Tech 2]   | ...  | ...  | 6/10      |
| [Tech 3]   | ...  | ...  | 4/10      |

### Decision Rationale
[Why we chose what we chose]

## Implementation Considerations

### Development Setup
[How to run locally]

### Testing Strategy
[How to test runtime behavior]

### Monitoring Needs
[What metrics to track]

## Research Sources
[URLs consulted during research]

---

After completing research and asking user questions, write this document to:
/job-queue/product-{name}/runtime-execution.md

Also save your raw research notes to:
/job-queue/product-{name}/research-notes/runtime-research.md
```

### Stage 3: Abstraction Layer Research (Agent 2)

**Agent task:**
```
You are researching abstraction layer architecture for: [Product Name]

CONTEXT:
[Include big-idea.md content]

YOUR GOAL:
Create a comprehensive `abstraction-layer.md` document that explains how user intent translates to executable logic.

RESEARCH APPROACH:
1. Research how modern frameworks handle abstraction
2. Use WebSearch for:
   - "[technology] abstraction patterns 2026"
   - "[technology] DSL design 2026"
   - "no-code/low-code architecture 2026" (if applicable)
   - "[technology] configuration vs code 2026"

3. Use WebFetch to read architectural docs

4. Ask user clarifying questions

QUESTIONS TO ASK USER:

1. **Input Format**
   - Header: "Input"
   - Question: "How will users define what they want the system to do?"
   - Options:
     - "Code (APIs, SDKs)" - "Developer-focused, programmatic"
     - "Configuration (YAML, JSON)" - "Declarative configs"
     - "Visual interface (drag-drop)" - "No-code UI builder"
     - "Natural language" - "AI-powered interpretation"

2. **Translation Approach**
   - Header: "Translation"
   - Question: "How should user intent become executable code?"
   - Options:
     - "Direct interpretation" - "Runtime interpretation, flexible"
     - "Compilation to native" - "Compiled ahead-of-time, fast"
     - "IR/AST transformation" - "Intermediate representation"
     - "Template-based generation" - "Code generation from templates"

3. **Extensibility**
   - Header: "Extensibility"
   - Question: "How should users extend default behavior?"
   - Options:
     - "Plugin system" - "Loadable extensions"
     - "Hooks/callbacks" - "Event-based extension points"
     - "Subclassing/inheritance" - "OOP-style extension"
     - "Middleware/interceptors" - "Pipeline-based modification"

DOCUMENT STRUCTURE:

Create: /job-queue/product-{name}/abstraction-layer.md

# Abstraction Layer Architecture

## Executive Summary
[How user intent becomes executable logic]

## Input Formats

### Primary Input Method
[Code, config, visual, NLP]

### Input Schema
[Structure of input: JSON schema, API surface, grammar]

### Validation & Parsing
[How inputs are validated and parsed]

## Intermediate Representation

### IR Design
[AST, JSON schema, custom IR - what does it look like?]

### Why This IR
[Rationale for chosen representation]

### Example Transformation
[Show: input → IR → output]

## Translation Mechanisms

### Chosen Approach
[Interpretation vs compilation vs generation]

### Translation Pipeline
[Step-by-step: input → IR → executable]

### Optimization Passes
[Any optimization during translation?]

## Mapping Tables

### UI/Config → System Primitives
[How high-level concepts map to low-level operations]

### Example Mappings
| User Concept | System Primitive | Implementation |
|--------------|------------------|----------------|
| [Concept 1]  | [Primitive 1]    | [How it works] |

## Extension Points

### Extension Architecture
[Plugin system, hooks, inheritance]

### Extension Registration
[How extensions are discovered and loaded]

### Extension API
[What APIs are exposed to extensions]

### Example Extension
[Code sample showing how to extend]

## Trade-offs Analysis

### Flexibility vs Performance
[What we optimized for]

### Simplicity vs Power
[Where we drew the line]

### Developer Experience
[How easy is it to use?]

## Technology Comparison

### Evaluated Approaches
| Approach       | Flexibility | Performance | DX  | Chosen |
|----------------|-------------|-------------|-----|--------|
| [Approach 1]   | High        | Medium      | Low | ❌     |
| [Approach 2]   | Medium      | High        | High| ✅     |

## Implementation Considerations

### Schema Evolution
[How to version the abstraction layer]

### Backward Compatibility
[How to handle breaking changes]

### Documentation Strategy
[How users learn the abstraction]

## Research Sources
[URLs consulted]

---

Write to: /job-queue/product-{name}/abstraction-layer.md
Research notes: /job-queue/product-{name}/research-notes/abstraction-research.md
```

### Stage 4: Integration Layer Research (Agent 3)

**Agent task:**
```
You are researching integration layer architecture for: [Product Name]

CONTEXT:
[Include big-idea.md content]

YOUR GOAL:
Create a comprehensive `integration-layer.md` document that explains how the system connects to external resources.

RESEARCH APPROACH:
1. Research connector patterns and integration architectures
2. Use WebSearch for:
   - "[technology] connector architecture 2026"
   - "API integration best practices 2026"
   - "[technology] authentication patterns 2026"
   - "service mesh architecture 2026" (if microservices)

3. Use WebFetch for vendor-specific docs (if integrating with known services)

4. Ask user clarifying questions

QUESTIONS TO ASK USER:

1. **Integration Scope**
   - Header: "Integrations"
   - Question: "What external systems will this product integrate with?"
   - MultiSelect: true
   - Options:
     - "Databases" - "PostgreSQL, MySQL, MongoDB, etc."
     - "APIs (REST/GraphQL)" - "External HTTP APIs"
     - "Message queues" - "RabbitMQ, Kafka, SQS, etc."
     - "Storage services" - "S3, GCS, Azure Storage"

2. **Authentication Strategy**
   - Header: "Auth"
   - Question: "How should the system authenticate to external services?"
   - Options:
     - "API keys" - "Simple token-based auth"
     - "OAuth 2.0" - "Delegated authorization"
     - "Mutual TLS" - "Certificate-based auth"
     - "Service accounts" - "Cloud provider IAM"

3. **Discovery Mechanism**
   - Header: "Discovery"
   - Question: "How should the system discover external services?"
   - Options:
     - "Static configuration" - "Hardcoded endpoints"
     - "Environment variables" - "Deploy-time config"
     - "Service discovery" - "Consul, Eureka, k8s DNS"
     - "Dynamic registry" - "Runtime service registration"

DOCUMENT STRUCTURE:

Create: /job-queue/product-{name}/integration-layer.md

# Integration Layer Architecture

## Executive Summary
[How the system connects to external resources]

## Connector Architecture

### Connector Pattern
[Plugin-based, driver-based, adapter pattern]

### Supported Protocols
- REST API
- GraphQL
- gRPC
- Message queues
- Database protocols
- Custom protocols

### Connector Lifecycle
[How connectors are initialized, used, closed]

## Authentication & Credential Management

### Authentication Methods
[API keys, OAuth, mTLS, service accounts]

### Credential Storage
[Where credentials live: env vars, secrets manager, vault]

### Rotation Strategy
[How credentials are rotated without downtime]

### Security Boundaries
[How credentials are isolated per service]

## Service Discovery

### Discovery Mechanism
[Static, env-based, service discovery, dynamic]

### Configuration Format
```json
{
  "services": {
    "database": {
      "type": "postgresql",
      "discovery": "static",
      "endpoint": "..."
    }
  }
}
```

### Health Checking
[How to detect unhealthy services]

## Data Flow Patterns

### Pull Pattern
[Request-response, polling]

### Push Pattern
[Webhooks, callbacks]

### Streaming Pattern
[WebSockets, SSE, gRPC streams]

### Chosen Approach per Integration
| Integration | Pattern | Rationale |
|-------------|---------|-----------|
| [Service 1] | Pull    | [Why]     |
| [Service 2] | Stream  | [Why]     |

## Error Handling & Retry Logic

### Error Classification
- Transient errors (retry)
- Permanent errors (fail fast)
- Rate limit errors (backoff)

### Retry Strategy
- Exponential backoff
- Circuit breaker pattern
- Fallback mechanisms

### Example: Retry Configuration
```json
{
  "retry": {
    "maxAttempts": 3,
    "backoff": "exponential",
    "initialDelay": "100ms"
  }
}
```

## Security & Isolation

### Network Segmentation
[How integrations are network-isolated]

### Data Encryption
[TLS for transit, encryption at rest]

### Least Privilege
[IAM policies, scoped credentials]

## Technology Comparison

### Evaluated Integration Patterns
| Pattern           | Complexity | Reliability | Chosen |
|-------------------|------------|-------------|--------|
| Direct API calls  | Low        | Medium      | ✅     |
| Message queue     | Medium     | High        | ❌     |
| Service mesh      | High       | High        | ❌     |

## Implementation Considerations

### Local Development
[How to run external dependencies locally]

### Testing Strategy
[Mocking, test containers, integration tests]

### Monitoring & Observability
[Metrics, logs, traces for integrations]

## Research Sources
[URLs consulted]

---

Write to: /job-queue/product-{name}/integration-layer.md
Research notes: /job-queue/product-{name}/research-notes/integration-research.md
```

### Stage 5: Output Rendering Research (Agent 4)

**Agent task:**
```
You are researching output rendering architecture for: [Product Name]

CONTEXT:
[Include big-idea.md content]

YOUR GOAL:
Create a comprehensive `output-rendering.md` document that explains how results are delivered to consumers.

RESEARCH APPROACH:
1. Research rendering strategies for the chosen tech stack
2. Use WebSearch for:
   - "[technology] rendering strategies 2026"
   - "SSR vs CSR vs SSG comparison 2026"
   - "[technology] streaming rendering 2026"
   - "[technology] caching strategies 2026"

3. Use WebFetch for framework-specific rendering docs

4. Ask user clarifying questions

QUESTIONS TO ASK USER:

1. **Rendering Strategy**
   - Header: "Rendering"
   - Question: "How should content be rendered?"
   - Options:
     - "Server-side (SSR)" - "Render on server, send HTML"
     - "Client-side (CSR)" - "Render in browser with JS"
     - "Static generation (SSG)" - "Pre-render at build time"
     - "Hybrid" - "Mix of SSR/CSR/SSG per route"

2. **Real-time Requirements**
   - Header: "Real-time"
   - Question: "Do outputs need real-time updates?"
   - Options:
     - "No real-time needed" - "Traditional request-response"
     - "Polling" - "Client polls for updates"
     - "WebSockets" - "Bidirectional real-time"
     - "Server-sent events (SSE)" - "Unidirectional streaming"

3. **Output Formats**
   - Header: "Formats"
   - Question: "What output formats do you need?"
   - MultiSelect: true
   - Options:
     - "HTML" - "Web pages"
     - "JSON/XML" - "API responses"
     - "Binary (images, PDFs)" - "Generated files"
     - "Streams" - "Continuous data streams"

DOCUMENT STRUCTURE:

Create: /job-queue/product-{name}/output-rendering.md

# Output Rendering Architecture

## Executive Summary
[How results are delivered to consumers]

## Output Formats

### Supported Formats
- HTML (web UI)
- JSON (API responses)
- Binary (files, images)
- Streams (continuous data)

### Serialization Strategy
[How data is serialized for each format]

### Content Negotiation
[How clients request specific formats]

## Rendering Pipeline

### Chosen Rendering Strategy
[SSR, CSR, SSG, hybrid]

### Rendering Flow
[Step-by-step: data → rendered output]

### Server-Side Rendering (if applicable)
- Template engine: [e.g., React Server Components, Jinja2]
- Hydration strategy
- Performance characteristics

### Client-Side Rendering (if applicable)
- JavaScript framework: [React, Vue, Svelte]
- Bundle strategy
- Initial load performance

### Static Generation (if applicable)
- Build-time generation
- Incremental static regeneration (ISR)
- When to use vs SSR

## Streaming & Real-Time Updates

### Real-Time Strategy
[None, polling, WebSockets, SSE]

### Streaming Architecture
[How data streams to client]

### Example: Real-Time Flow
```
1. Client subscribes to updates
2. Server pushes incremental changes
3. Client updates UI reactively
```

## Template/Component System

### Component Architecture
[How UI is composed: components, templates, views]

### Component Library
[Design system, reusable components]

### Composition Patterns
[How components are nested and composed]

### Example Component Structure
```
<Layout>
  <Header />
  <Main>
    <DataGrid data={...} />
  </Main>
  <Footer />
</Layout>
```

## Caching & Persistence

### Caching Layers
1. **CDN caching**: [CloudFlare, CloudFront]
2. **Server caching**: [Redis, in-memory]
3. **Client caching**: [Service worker, browser cache]

### Cache Invalidation
[How caches are invalidated when data changes]

### Persistence Strategy
[How outputs are stored: database, file system, object storage]

## Performance Optimization

### Bundle Optimization
- Code splitting
- Tree shaking
- Lazy loading

### Image Optimization
- Responsive images
- Format selection (WebP, AVIF)
- CDN delivery

### Data Fetching
- Parallel fetching
- Request deduplication
- Prefetching strategies

## Technology Comparison

### Evaluated Rendering Approaches
| Approach | SEO | Performance | Complexity | Chosen |
|----------|-----|-------------|------------|--------|
| SSR      | ✅  | Medium      | Medium     | ✅     |
| CSR      | ❌  | Slow        | Low        | ❌     |
| SSG      | ✅  | Fast        | Low        | ✅     |

## Implementation Considerations

### Development Experience
[Hot reload, dev server, debugging]

### Testing Strategy
[Component tests, visual regression, E2E]

### Monitoring
[Performance metrics, Core Web Vitals]

## Research Sources
[URLs consulted]

---

Write to: /job-queue/product-{name}/output-rendering.md
Research notes: /job-queue/product-{name}/research-notes/output-research.md
```

### Parallel Execution

**Launch all 4 agents at once:**

```bash
# Agent 1: Runtime Execution
Task tool:
  subagent_type: "general-purpose"
  description: "Runtime execution research"
  prompt: [Stage 2 agent task]

# Agent 2: Abstraction Layer
Task tool:
  subagent_type: "general-purpose"
  description: "Abstraction layer research"
  prompt: [Stage 3 agent task]

# Agent 3: Integration Layer
Task tool:
  subagent_type: "general-purpose"
  description: "Integration layer research"
  prompt: [Stage 4 agent task]

# Agent 4: Output Rendering
Task tool:
  subagent_type: "general-purpose"
  description: "Output rendering research"
  prompt: [Stage 5 agent task]
```

**Wait for all agents to complete.**

**Output:**
- Display: "✅ Stage 2/6: Runtime execution research complete"
- Display: "✅ Stage 3/6: Abstraction layer research complete"
- Display: "✅ Stage 4/6: Integration layer research complete"
- Display: "✅ Stage 5/6: Output rendering research complete"

---

## Stage 6: Final Review & Presentation

**Goal:** Present all generated documents to the user for review and approval.

### Step 6.1: Read All Generated Documents

```bash
Read:
- /job-queue/product-{name}/big-idea.md
- /job-queue/product-{name}/runtime-execution.md
- /job-queue/product-{name}/abstraction-layer.md
- /job-queue/product-{name}/integration-layer.md
- /job-queue/product-{name}/output-rendering.md
```

### Step 6.2: Generate Executive Summary

Create a final summary document:

**File:** `/job-queue/product-{name}/ARCHITECTURE-SUMMARY.md`

```markdown
# Architecture Summary: [Product Name]

## Research Completion

**Date:** [timestamp]
**Input:** [original input]
**Product Name:** [derived name]

## Recommended Architecture

### Technology Stack
- **Frontend**: [chosen technology]
- **Backend**: [chosen technology]
- **Database**: [chosen technology]
- **Deployment**: [chosen platform]

### Architectural Style
- [Monolithic/Microservices/Serverless]
- [Event-driven/Request-response]
- [SSR/CSR/SSG]

### Key Design Decisions

#### Runtime Execution
[1-2 sentence summary from runtime-execution.md]

#### Abstraction Layer
[1-2 sentence summary from abstraction-layer.md]

#### Integration Layer
[1-2 sentence summary from integration-layer.md]

#### Output Rendering
[1-2 sentence summary from output-rendering.md]

## Trade-offs & Risks

### Major Trade-offs
1. [Trade-off 1]: Chose X over Y because [reason]
2. [Trade-off 2]: Chose X over Y because [reason]

### Identified Risks
1. [Risk 1]: [Mitigation strategy]
2. [Risk 2]: [Mitigation strategy]

## Next Steps

To proceed with implementation:

1. Review detailed architecture documents:
   - `runtime-execution.md`
   - `abstraction-layer.md`
   - `integration-layer.md`
   - `output-rendering.md`

2. Set up development environment

3. Create initial project structure

4. Begin implementation with chosen tech stack

## Research Artifacts

All research is saved in:
- `/job-queue/product-{name}/`
- `/job-queue/product-{name}/research-notes/`

Total research sources consulted: [count]
```

### Step 6.3: Present to User

**Display:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎉 PRODUCT RESEARCH COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Product: [Product Name]
Location: /job-queue/product-{name}/

Generated Documents:
  ✅ big-idea.md              - High-level vision and approach
  ✅ runtime-execution.md     - How the system executes work
  ✅ abstraction-layer.md     - How user intent becomes logic
  ✅ integration-layer.md     - How system connects externally
  ✅ output-rendering.md      - How results are delivered
  ✅ ARCHITECTURE-SUMMARY.md  - Executive summary

Recommended Stack:
  Frontend:   [technology]
  Backend:    [technology]
  Database:   [technology]
  Deployment: [platform]

Research Sources: [count] URLs consulted

Next Steps:
  1. Review detailed architecture docs in /job-queue/product-{name}/
  2. Approve or request revisions
  3. Begin implementation

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Ask for feedback:**

Use `AskUserQuestion`:
- **Question**: "Would you like to revise any architectural decisions?"
- **Options**:
  - "Approve - looks good" - "Proceed with this architecture"
  - "Revise runtime execution" - "Re-research execution model"
  - "Revise abstraction layer" - "Re-research abstraction approach"
  - "Revise integration layer" - "Re-research integrations"
  - "Revise output rendering" - "Re-research rendering strategy"

**If user approves:**
- Display: "✅ Architecture approved! Ready for implementation."

**If user requests revision:**
- Re-run the specific stage's research agent with feedback
- Regenerate that document
- Re-present for approval

---

## Error Handling

**At any stage failure:**

1. Display which stage failed
2. Show what was completed
3. Provide recovery instructions

**Example:**
```
❌ Research Failed at Stage 3/6: Abstraction Layer

Completed:
  ✓ [Stage 0] Workspace created
  ✓ [Stage 1] Big idea generated
  ✓ [Stage 2] Runtime execution researched
  ✗ [Stage 3] Abstraction layer FAILED

Error: [error message]

Partial work saved at: /job-queue/product-{name}/

Recovery:
  - Fix the issue and resume with: /new-product [input]
  - Or manually continue research
```

---

## Tools Used

- **WebSearch** - Technology research and comparisons
- **WebFetch** - Reading official documentation
- **Task Tool** - Parallel research agents
- **AskUserQuestion** - Iterative clarification
- **Read/Write** - Document generation

---

## Expected Outcomes

After completion:

1. ✅ Comprehensive architecture research
2. ✅ 5 detailed architecture documents
3. ✅ Technology stack recommendations
4. ✅ Trade-offs and risks identified
5. ✅ User approval obtained
6. ✅ Ready for implementation

---

## Notes

- **Deep research**: Use WebSearch + WebFetch extensively
- **Parallel execution**: Launch all 4 document agents at once
- **Iterative questions**: Ask questions during each stage, not all upfront
- **Cross-referencing**: Each agent reads big-idea.md for context
- **Documentation quality**: Comprehensive, well-structured, actionable

---

**Total Expected Duration:**

- Stage 0: ~10 seconds (folder creation)
- Stage 1: ~5-10 minutes (high-level research + big-idea.md)
- Stages 2-5: ~15-25 minutes (parallel deep research for 4 docs)
- Stage 6: ~2-3 minutes (summary generation + review)

**Total: ~20-40 minutes** (depending on research depth and user interaction speed)
