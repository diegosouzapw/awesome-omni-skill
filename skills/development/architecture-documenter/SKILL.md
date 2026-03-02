---
name: architecture-documenter
description: "Maintain and update architecture documentation in docs/architecture/. Uses tool-based inspection first, falling back to CLI only when needed."
---

# Architecture Documenter Skill

## General Guidance

This skill maintains architecture documentation under:

```text
docs/architecture/
```

Always use **tool-based inspection first** (MCP tools if configured).  
Only fall back to CLI when necessary.

Documentation must always reflect **current reality**, not assumptions.

When asked to update documentation:

1. Inspect existing docs (if any).
2. Inspect code/configs/infrastructure via tools.
3. Determine which documentation files make sense for this project.
4. Create/update only relevant files.
5. Never guess — ask for confirmation if unclear.
6. Use **Mermaid diagrams** where they add clarity.

---

## Documentation Structure

### Index File (MANDATORY)

**Always create** `docs/architecture/index.md` as the main entry point. This is the only mandatory file.

This file should:

- Provide a brief overview of the system
- Link to other architecture documents (if any exist)
- Explain the system architecture in a way that's readable on its own
- Include a high-level diagram of key components
- Be comprehensive enough to understand the system without other docs

### Optional Supporting Documentation Files

**Only create additional files if the system is complex enough to warrant them.** Use these as templates when relevant:

#### system-overview.md

High-level map of the entire system, major components, and external integrations.

**When to create:**

- Multi-service systems
- Systems with external integrations
- Complex architectures needing bird's-eye view

#### cloud-architecture.md

Cloud deployment topology, networking model, load balancers, compute layers, storage.

**When to create:**

- Cloud-deployed applications
- Infrastructure-heavy systems
- Multi-region or complex networking

#### service-architecture.md

Internal services, modules, queues, APIs, and data flows.

**When to create:**

- Microservices architectures
- Service-oriented systems
- Multiple internal APIs

#### cicd-architecture.md

Build, test, artifact creation, deployment flow, environments.

**When to create:**

- Complex CI/CD pipelines
- Multiple deployment environments
- Custom build/deploy processes

**You can also create custom documents** based on the project's specific needs (e.g., `data-architecture.md`, `security-architecture.md`, `api-architecture.md`).

---

## Templates

Template files are provided in this skill directory:

- `index.md` - Entry point template (always use this)
- `system-overview.md` - Optional system-level template
- `cloud-architecture.md` - Optional cloud infrastructure template
- `service-architecture.md` - Optional service/API template
- `cicd-architecture.md` - Optional CI/CD pipeline template

**Adapt these templates** to fit the project. Remove sections that don't apply, add sections that are needed.

## Behavioral Rules

- **MANDATORY**: Always create `docs/architecture/index.md` - it's the only required file.
- Use tool-based introspection before CLI.
- Ask for files or context when uncertain.
- Only create additional documents if the project complexity warrants them.
- The index.md should be comprehensive enough to stand alone for simple systems.
- Update diagrams to match reality.
- Keep diagrams readable and scoped.
- Maintain consistency across documents.
- Filenames: All filenames MUST be lowercase and use dashes (-) as word separators. Do not use spaces or underscores.
- If a suggested template doesn't fit, adapt or skip it.
- When in doubt about which files to create, ask the user.
- When in doubt about which files to create, ask the user.

- README maintenance: Keep the repository `README.md` up to date with the current state of the application. Prefer an overview-only README that links to the relevant `docs/architecture/` pages for detailed information.
