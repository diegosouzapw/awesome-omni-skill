---
name: tmdd-threat-modeling
description: Create and manage TMDD threat models grounded in actual codebase architecture. Use when the user wants to threat-model a system, add a feature, create security threat mappings, run tmdd commands, or work with .tmdd/ YAML files.
globs: .tmdd/**/*.yaml, .tmdd/**/*.yml
alwaysApply: false
---

# TMDD - Threat Modeling Driven Development

## When to Use

Activate when the user asks to:
- Create, initialize, or scaffold a threat model
- Add a feature to an existing threat model
- Threat-model a codebase, service, or feature
- Run or fix `tmdd lint` errors
- Generate implementation prompts from a threat model

Also auto-activates when editing `.tmdd/**/*.yaml` files.

## Core Principle: Architecture-First Threat Modeling

**Every threat model MUST be grounded in the actual codebase.** Never produce
generic/textbook threats. Before writing any YAML, analyze the code to discover
real components, data flows, technologies, and attack surface.

---

## Phase 1 — Codebase Architecture Analysis

Before touching any `.tmdd/` file, perform these steps:

### 1.1 Discover Project Structure

Scan the repository to identify:
- **Language & framework** (package.json, requirements.txt, go.mod, Cargo.toml, pom.xml, etc.)
- **Entry points** (main files, route definitions, CLI commands)
- **Directory layout** (src/, api/, lib/, services/, models/, controllers/, etc.)

### 1.2 Identify Architectural Components

Search the codebase for real building blocks. Map each to a TMDD component:

| Look for | Maps to TMDD component type |
|----------|----------------------------|
| Route handlers, controllers, API endpoints | `api` |
| Frontend pages, React/Vue/Angular components | `frontend` |
| Database models, ORM schemas, migrations | `database` |
| Background workers, cron jobs, queue consumers | `service` |
| Redis/Memcached usage | `cache` |
| Message broker publishers/consumers (Kafka, RabbitMQ, SQS) | `queue` |
| Third-party API calls, SDK integrations | `external` |
| Auth middleware, session management, token handling | `service` (auth) |

**For each component, record:**
- The actual technology (e.g., "Express.js", "PostgreSQL via Prisma", "Redis")
- The trust boundary (`public` if internet-facing, `internal` if behind auth/VPN, `external` if third-party)
- Key source files/directories it lives in

### 1.3 Trace Real Data Flows

Follow how data actually moves through the code:
- HTTP requests from clients to API handlers
- Database reads/writes from handlers to ORM/query layer
- Inter-service calls (REST, gRPC, message queues)
- External API calls (payment providers, email services, OAuth providers)
- File uploads, websocket connections, SSE streams

**For each flow, note:**
- What data is transmitted (credentials, PII, tokens, user content)
- Authentication mechanism (JWT, session cookie, API key, mTLS, none)
- Protocol (HTTPS, gRPC, WebSocket, AMQP)

### 1.4 Identify Security-Relevant Code Patterns

Scan for patterns that inform threats directly:
- **Authentication**: How are users authenticated? (JWT, sessions, OAuth, API keys)
- **Authorization**: Is there RBAC/ABAC? Where are permission checks?
- **Input validation**: Is there schema validation (Zod, Joi, Pydantic)? Where?
- **SQL/ORM usage**: Raw queries vs parameterized? Which ORM?
- **File handling**: Uploads, path traversal risks, temp files?
- **Secrets management**: Env vars, vault, hardcoded?
- **Serialization**: JSON parsing, XML, YAML (deserialization attacks)?
- **Cryptography**: Hashing algorithms, encryption at rest/in transit?
- **Error handling**: Do errors leak stack traces or internal details?
- **Logging**: What is logged? Are secrets filtered?
- **Rate limiting**: Is there any? Where?
- **CORS/CSP**: What's the policy?

---

## Phase 2 — Threat Model Creation

**IMPORTANT — Before editing any YAML file:**
1. Check if `.tmdd/` already exists and contains populated YAML files
2. If YES: you are in **incremental mode** — read each file first, then append new
   entries or edit existing ones. NEVER rewrite a file from scratch. Skip `tmdd init`.
   Go directly to Phase 3 if adding a feature, or follow Phase 2.2 in append mode.
3. If NO: you are in **creation mode** — run `tmdd init`, then populate files per Phase 2.2

### 2.1 New Project (no `.tmdd/` directory)

```bash
tmdd init .tmdd --template <template> -n "System Name" -d "Description"
```

Templates: `minimal` (blank), `web-app` (7 web threats), `api` (OWASP API Top 10).

After init, replace the template content with architecture-specific data from Phase 1.

### 2.2 Populate YAML Files (in order)

**YOU MUST EDIT THE FILES DIRECTLY. DO NOT JUST OUTPUT YAML.**

**NEVER overwrite existing content.** Before editing any YAML file:
1. Read the file first to see what entries already exist
2. Append new entries — do not remove or rewrite existing ones unless
   the user explicitly asks for changes to specific entries
3. When adding threats/mitigations, continue the existing ID sequence
   (e.g., if T005 exists, start new threats at T006)

Edit these files using the analysis from Phase 1:

#### 1. `components.yaml` — Map real code to components

```yaml
components:
  - id: api_backend            # REQUIRED, ^[a-z][a-z0-9_]*$
    description: "Express.js REST API handling user and order endpoints"
    type: api                  # frontend|api|service|database|queue|external|cache|other
    technology: "Node.js / Express"
    trust_boundary: public     # public|internal|external
    source_paths:              # OPTIONAL - glob patterns mapping to source files
      - "src/routes/**"
      - "src/middleware/**"
      - "src/server.ts"
```

**Rules:**
- One component per distinct architectural unit discovered in Phase 1
- `description` must mention the actual technology and what it does in this project
- `trust_boundary` must reflect the real deployment (not assumed)
- `source_paths` (optional) should list glob patterns for source files that belong to this
  component. This enables deterministic PR-to-component mapping for threat review workflows.
  Prefer specific globs over overly broad ones (e.g., `src/routes/**` over `src/**`).

#### 2. `actors.yaml` — Real users and external systems

```yaml
actors:
  - id: end_user               # REQUIRED, ^[a-z][a-z0-9_]*$
    description: "Authenticated user accessing the web dashboard"
```

#### 3. `data_flows.yaml` — Traced from actual code paths

```yaml
data_flows:
  - id: df_user_to_api             # REQUIRED
    source: end_user               # must exist in actors or components
    destination: api_backend       # must exist in actors or components
    data_description: "Login credentials (email + password) and session tokens"
    protocol: HTTPS
    authentication: JWT
```

**Rules:**
- Every flow must correspond to a real code path you found in Phase 1.3
- `data_description` must name the actual data types (not just "API calls")
- Include protocol and auth method from the code

#### 4. `threats/catalog.yaml` — Threats specific to THIS codebase

```yaml
threats:
  T001:                              # ^T\d+$
    name: "SQL Injection via raw query in search endpoint"
    description: "The /api/search endpoint in src/routes/search.ts uses string concatenation for the WHERE clause instead of parameterized queries"
    severity: high                   # low|medium|high|critical
    stride: T                        # S|T|R|I|D|E
    cwe: CWE-89
    suggested_mitigations: [M001]    # each must exist in mitigations.yaml
```

**CRITICAL — Threat Quality Rules:**
- `name` must reference the specific component, endpoint, or module affected
- `description` must describe the concrete vulnerability in this codebase, not a textbook definition. Reference file paths when possible.
- `severity` must be based on actual exploitability and impact in this system
- Every threat must be traceable to a component or data flow from Phase 1

**STRIDE analysis — apply to each component and data flow:**
- **S**poofing: Can identities be faked? (check auth implementation)
- **T**ampering: Can data be modified? (check input validation, CSRF protection)
- **R**epudiation: Can actions be denied? (check audit logging)
- **I**nformation Disclosure: Can data leak? (check error handling, logging, CORS)
- **D**enial of Service: Can availability be impacted? (check rate limiting, resource limits)
- **E**levation of Privilege: Can permissions be bypassed? (check authorization checks)

#### 5. `threats/mitigations.yaml` — Actionable controls with code references

```yaml
mitigations:
  # Simple format
  M001: "Use parameterized queries via Prisma ORM for all database access"

  # Rich format with code references (preferred — ties mitigation to implementation)
  M002:
    description: "Zod schema validation on all API request bodies"
    references:
      - file: "src/middleware/validate.ts"
        lines: "12-35"
      - file: "src/schemas/user.ts"
```

**Rules:**
- Reference actual files/lines where the mitigation is (or should be) implemented
- If the mitigation doesn't exist yet, describe it concretely enough to implement
- Use the rich format with `references` whenever a file location is known

#### 6. `threats/threat_actors.yaml`

```yaml
threat_actors:
  TA001: "External attacker"    # ^TA\d+$
```

#### 7. `features.yaml` — Features with threat-to-mitigation mapping

```yaml
features:
  - name: "User Login"            # REQUIRED
    goal: "Authenticate users"    # REQUIRED
    data_flows: [df_user_to_api]  # must exist in data_flows.yaml
    threat_actors: [TA001]        # must exist in threat_actors.yaml
    threats:                      # MUST be a dict, NOT a list
      T001: default               # inherit suggested_mitigations from catalog
      T002: [M003, M005]          # explicit mitigation override
      T003: accepted              # risk deliberately accepted
    last_updated: "2026-02-22"    # set by agent to today's date
    reviewed_at: "2000-01-01"     # SENTINEL — forces stale-review lint warning
    # reviewed_by: — DO NOT SET. Only a human adds this after manual review.
```

**Threat mapping values:**
- `default` — inherit `suggested_mitigations` from `threats/catalog.yaml` (preferred when suggestions fit)
- `[M001, M002]` — explicit mitigation list (override when you need different controls)
- `accepted` — risk deliberately accepted without mitigation

**Review fields (human-only attestation):**
- `reviewed_by` — name/username of the human analyst who verified the threat mappings.
  **AI agents MUST NOT set this field.** Only a human adds it after manual review.
- `reviewed_at` — date of last review (YYYY-MM-DD). AI agents MUST set this to `"2000-01-01"`
  as a sentinel so that `last_updated > reviewed_at` always triggers a stale-review lint warning.
  The human updates this to the real date when they review.
- `last_updated` — date the feature was last created or modified (YYYY-MM-DD).
  AI agents SHOULD set this to today's date.
- Features with `accepted` threats and no `reviewed_by` trigger a lint warning

```yaml
# CORRECT
threats:
  T001: default
  T002: [M001, M002]
  T003: accepted

# WRONG - will fail lint
threats: [T001, T002, T003]
```

---

## Phase 3 — Adding a Feature (existing `.tmdd/` project)

When adding a feature to an existing threat model:

### 3.1 Analyze the feature's code impact

Before editing YAML, answer:
1. **What new code paths does this feature introduce?** (new endpoints, new DB tables, new external calls)
2. **What existing components does it touch?**
3. **What sensitive data does it handle?** (PII, credentials, financial data, tokens)
4. **What new attack surface does it create?**

### 3.2 Use the `tmdd feature` workflow

```bash
# Step 1: Generate threat modeling prompt (new feature)
tmdd feature "Feature Name" -d "What it does"

# Step 2: Read the generated prompt
# .tmdd/out/<feature_name>.threatmodel.txt

# Step 3: Edit YAML files using findings from 3.1 (follow order in 3.3 below)

# Step 4: Validate
tmdd lint .tmdd

# Step 5: Generate implementation prompt (feature now exists)
tmdd feature "Feature Name"
```

### 3.3 Edit files in order

1. `components.yaml` — Add new components if the feature introduces new architectural units
2. `actors.yaml` — Add new actors if the feature serves new user types
3. `data_flows.yaml` — Add flows for new data paths the feature creates
4. `threats/catalog.yaml` — Add threats specific to the feature's code (not generic threats)
5. `threats/mitigations.yaml` — Add mitigations referencing actual or planned implementation files
6. `features.yaml` — Add the feature with full threat->mitigation mapping

---

## Phase 4 — Validation & Compilation

```bash
# Validate all cross-references
tmdd lint .tmdd

# Generate consolidated output
tmdd compile .tmdd                        # Full system
tmdd compile .tmdd --feature "Login"      # Single feature
```

---

## ID Conventions

| Type       | Pattern              | Example          |
|------------|----------------------|------------------|
| Entity     | `^[a-z][a-z0-9_]*$` | `api_backend`    |
| Threat     | `^T\d+$`            | `T001`           |
| Mitigation | `^M\d+$`            | `M001`           |
| Threat Actor | `^TA\d+$`         | `TA001`          |
| Data Flow  | `df_{src}_to_{dst}` | `df_user_to_api` |

## File Structure

```
.tmdd/
  system.yaml            # System metadata
  actors.yaml            # Who interacts with the system
  components.yaml        # Architecture building blocks
  data_flows.yaml        # Data movement between actors/components
  features.yaml          # Features with threat->mitigation mappings
  threats/
    catalog.yaml         # Threat definitions (T001, T002...)
    mitigations.yaml     # Security controls (M001, M002...)
    threat_actors.yaml   # Adversary profiles (TA001, TA002...)
```

## Cross-Reference Rules (enforced by lint)

1. `data_flows[].source/destination` must exist in actors or components
2. `features[].data_flows[]` must exist in data_flows.yaml
3. `features[].threat_actors[]` must exist in threat_actors.yaml
4. `features[].threats` keys must exist in threats/catalog.yaml
5. `features[].threats` mitigation values must exist in threats/mitigations.yaml
6. `catalog[].suggested_mitigations[]` must exist in mitigations.yaml

## Self-Validation Checklist

Before finishing edits, verify:
- [ ] Phase 1 analysis was performed — components and data flows reflect actual code
- [ ] All IDs follow naming conventions
- [ ] Every ID referenced in features.yaml exists in its source file
- [ ] features.yaml threats is a dict (not a list)
- [ ] Threat names/descriptions reference specific components, endpoints, or files
- [ ] Mitigations reference actual or planned implementation files where possible
- [ ] data_flows source/destination exist in actors or components
- [ ] Existing entries in all YAML files were preserved (no accidental overwrites)
- [ ] Run `tmdd lint .tmdd` and fix all errors
