---
name: research
description: This skill should be used when the user asks to "research documentation", "find framework docs", "search for best practices", "look up API documentation", "find implementation patterns", "research how to implement", or when gathering external technical knowledge to inform planning or design decisions.
version: 0.1.0
allowed-tools: WebSearch, WebFetch, Read, Write
---

# Research Skill

Guide WebSearch usage to find authoritative technology documentation and best practices.

## When to Use

Invoke research when encountering:
- Unknown API or framework features
- Implementation patterns for specific technology
- Version-specific documentation needs
- Best practices for technical decisions
- Integration requirements with external services

## Search Query Formulation

### Basic Pattern

Structure queries as:
```
[technology] [version] [specific topic] documentation [year]
```

**Examples:**
- "Next.js 14 app router data fetching documentation 2026"
- "PostgreSQL 16 JSON operators documentation"
- "Tailwind CSS v4 responsive design patterns"
- "React 19 server components patterns 2026"

### Version-Specific Searches

Always include version numbers when relevant:
- Framework versions: "Next.js 14", "React 19", "Vue 3"
- Language versions: "Python 3.12", "TypeScript 5.x", "Node.js 22"
- Database versions: "PostgreSQL 16", "MongoDB 7"

### Year-Aware Searches

Include current year (2026) for recent documentation:
- Ensures results reflect latest API changes
- Avoids outdated patterns and deprecated features
- Finds current best practices

## Targeting Authoritative Sources

### Official Documentation Domains

| Technology | Primary Domain | Notes |
|------------|----------------|-------|
| React | react.dev | Canonical React docs |
| Next.js | nextjs.org | App Router focus |
| TypeScript | typescriptlang.org | Type system reference |
| Node.js | nodejs.org | API documentation |
| Python | docs.python.org | Standard library |
| PostgreSQL | postgresql.org | SQL reference |
| Tailwind CSS | tailwindcss.com | Utility classes |
| Prisma | prisma.io | ORM documentation |
| Drizzle | orm.drizzle.team | ORM documentation |

### Site-Restricted Searches

Use site restriction for targeted results:
```
[query] site:react.dev
[query] site:github.com/[org]
```

**Examples:**
- "server components patterns site:react.dev"
- "authentication middleware site:nextjs.org"
- "drizzle postgres setup site:orm.drizzle.team"

### Source Priority

Prioritize sources in this order:

1. **Official documentation** (highest authority)
   - First-party documentation from framework/library creators
   - API references and guides

2. **GitHub repositories**
   - Source code and examples
   - README files and discussions
   - Issue resolutions

3. **Reputable technical blogs**
   - Vercel blog (Next.js patterns)
   - React team blog posts
   - Kent C. Dodds (React patterns)
   - Josh W. Comeau (CSS/React)

4. **Stack Overflow** (with recent answers)
   - Filter for answers from 2024+
   - Check answer acceptance and votes
   - Verify against official docs

## Search Strategies by Context

### API Discovery

When exploring unfamiliar APIs:
1. Search for official API reference: "[library] API reference"
2. Find usage examples: "[library] [feature] example"
3. Look for migration guides: "[library] migration from [version]"

### Implementation Patterns

When implementing features:
1. Search for patterns: "[framework] [feature] pattern"
2. Find best practices: "[framework] [feature] best practices 2026"
3. Look for tutorials: "[framework] [feature] tutorial"

### Troubleshooting

When encountering errors:
1. Search exact error message (in quotes)
2. Include framework and version
3. Add "solution" or "fix" to query

## WebFetch for Deep Reading

After identifying relevant URLs via WebSearch, use WebFetch to:
- Read full documentation pages
- Extract code examples
- Gather detailed implementation steps

**Prompts for WebFetch:**
- "Extract all code examples from this page"
- "Summarize the key steps for implementing [feature]"
- "List all configuration options mentioned"

## Integration with Planning Workflow

### During PRD Gathering

Research to inform requirements:
- Technical feasibility of proposed features
- Existing patterns for similar functionality
- Integration requirements with third-party services

### During Design Phase

Research to inform architecture:
- Recommended patterns for the tech stack
- Performance considerations
- Security best practices

### Output Format

Save research findings to planning documents:

```markdown
## Research Notes: [Topic]

### Sources Consulted
- [Source 1 title](url) - Brief summary
- [Source 2 title](url) - Brief summary

### Key Findings
1. [Finding 1]
2. [Finding 2]

### Recommendations
- [Recommendation based on research]

### Relevant Code Examples
[Code snippet from documentation]
```

## Quality Checklist

Before completing research:

- [ ] Checked official documentation first
- [ ] Verified information currency (2024+ preferred)
- [ ] Noted version-specific requirements
- [ ] Summarized key findings clearly
- [ ] Cited sources with URLs
- [ ] Extracted relevant code examples
- [ ] Identified any conflicting guidance

## Additional Resources

### Reference Files

For detailed search patterns by technology:
- **`references/search-patterns.md`** - Technology-specific query templates

## Common Pitfalls

### Avoid

- Relying solely on Stack Overflow without verification
- Using patterns from outdated documentation
- Ignoring version compatibility
- Missing breaking changes between versions

### Best Practices

- Cross-reference multiple authoritative sources
- Test code examples before recommending
- Note version requirements explicitly
- Update research when versions change
