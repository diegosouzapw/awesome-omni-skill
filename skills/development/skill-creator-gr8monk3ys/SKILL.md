---
name: skill-creator
description: Use this skill when creating new Claude Code skills, defining skill structures, or helping users build custom auto-invoked skills.
---

# Skill Creator Skill

You have expertise in creating Claude Code skills - auto-invoked capabilities that activate based on context.

## When to Use

This skill activates for:
- Creating new skills for Claude Code
- Defining skill structures and frontmatter
- Converting expertise into skill format
- Helping users build custom skills
- Organizing skill bundles

## Skill Structure

### Basic SKILL.md Template
```markdown
---
name: skill-name
description: When Claude should auto-invoke this skill. Be specific about triggers.
---

# Skill Name

Brief description of your expertise.

## When to Use

Bullet points of scenarios that trigger this skill:
- Scenario 1
- Scenario 2
- Scenario 3

## Core Capabilities

### Capability 1
Explanation and code examples...

### Capability 2
Explanation and code examples...

## Patterns & Templates

Code templates that users commonly need...

## Best Practices

1. **Practice 1** - Explanation
2. **Practice 2** - Explanation

## Common Pitfalls

- Pitfall 1: How to avoid
- Pitfall 2: How to avoid
```

### Frontmatter Fields
```yaml
---
name: kebab-case-name           # Required: unique identifier
description: |                   # Required: triggers auto-invocation
  Detailed description of when Claude should use this skill.
  Include keywords that users might mention.
disable-model-invocation: false  # Optional: set true for manual-only
---
```

## Skill Categories

### Development Skills
```
.claude/skills/
├── api-development/      # REST API, GraphQL, webhooks
├── frontend-development/ # React, Vue, components
├── database-operations/  # SQL, ORMs, migrations
├── testing/             # Unit, E2E, integration
└── devops/              # CI/CD, Docker, deployment
```

### Domain Skills
```
.claude/skills/
├── fintech/             # Payments, compliance, security
├── healthcare/          # HIPAA, HL7, medical systems
├── ecommerce/           # Carts, checkout, inventory
└── analytics/           # Metrics, dashboards, tracking
```

### Tool Skills
```
.claude/skills/
├── mcp-builder/         # Creating MCP servers
├── playwright/          # Browser automation
├── prisma/              # ORM operations
└── supabase/           # BaaS integration
```

## Writing Effective Descriptions

### Good Descriptions (Specific Triggers)
```yaml
description: |
  Use this skill when creating REST API endpoints, handling HTTP requests,
  implementing authentication/authorization, or adding rate limiting.
  Activates for Next.js API routes, Express handlers, and Fastify routes.
```

### Bad Descriptions (Too Vague)
```yaml
description: Helps with backend development
```

### Keyword-Rich Descriptions
Include keywords users might say:
- Action words: "create", "build", "implement", "add"
- Technology names: "React", "Next.js", "Prisma"
- Concepts: "authentication", "pagination", "caching"

## Skill Content Guidelines

### 1. Lead with Code
```markdown
## Quick Start

\`\`\`typescript
// Immediately useful code snippet
export function example() {
  // Implementation
}
\`\`\`
```

### 2. Provide Templates
```markdown
## Templates

### Basic Template
\`\`\`typescript
// Copy-paste ready code
\`\`\`

### Advanced Template
\`\`\`typescript
// More complex example
\`\`\`
```

### 3. Include Configuration
```markdown
## Configuration

### package.json
\`\`\`json
{
  "dependencies": { ... }
}
\`\`\`

### Environment Variables
- `API_KEY` - Your API key
- `DATABASE_URL` - Connection string
```

### 4. Document Patterns
```markdown
## Common Patterns

### Pattern 1: Name
Use when: [scenario]
\`\`\`typescript
// Implementation
\`\`\`

### Pattern 2: Name
Use when: [scenario]
\`\`\`typescript
// Implementation
\`\`\`
```

## Creating Composite Skills

Bundle related capabilities:
```
.claude/skills/fullstack-nextjs/
├── SKILL.md              # Main skill file
├── api-patterns.md       # Additional context
├── component-library.md  # Reference material
└── deployment.md         # Deployment patterns
```

Main SKILL.md references others:
```markdown
## Related Resources

See also:
- [API Patterns](./api-patterns.md)
- [Component Library](./component-library.md)
```

## Testing Skills

### Manual Testing
1. Create skill in `.claude/skills/skill-name/SKILL.md`
2. Start new Claude Code session
3. Ask questions that should trigger the skill
4. Verify skill activates and provides relevant help

### Checklist
- [ ] Description triggers on expected keywords
- [ ] Code examples are copy-paste ready
- [ ] Templates compile without errors
- [ ] Best practices are actionable
- [ ] Common pitfalls are addressed

## Skill Distribution

### In Plugin
Add to plugin structure:
```
.claude/skills/
└── my-skill/
    └── SKILL.md
```

### Standalone
Share skill file directly - users place in their `.claude/skills/` directory.

## Examples of Well-Designed Skills

### Focused Skill
```yaml
name: react-forms
description: |
  Use when building forms in React: form validation with react-hook-form,
  Zod schemas, error handling, form submission, and field components.
```

### Broad Skill
```yaml
name: fullstack-development
description: |
  Use for full-stack web development: Next.js pages and API routes,
  database integration, authentication, deployment, and DevOps.
```
