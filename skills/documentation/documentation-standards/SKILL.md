---
name: documentation-standards
description: Clear technical documentation with JSDoc, READMEs, Mermaid diagrams, ISMS policy references, and comprehensive code examples
license: MIT
---

# Documentation Standards Skill

## Context
This skill applies when:
- Writing or updating README files
- Documenting APIs, functions, or MCP tools
- Creating architecture or design documentation
- Adding JSDoc comments to TypeScript code
- Creating diagrams for system architecture or flows
- Documenting security policies and compliance
- Writing user guides or integration tutorials
- Creating contribution guidelines

## Rules

1. **Document Security Context**: All security-related code must reference ISMS policies in documentation
2. **Use JSDoc for Public APIs**: All exported functions, classes, and interfaces must have JSDoc comments
3. **Include Type Information**: JSDoc comments must include `@param` and `@returns` with TypeScript types
4. **Provide Examples**: All public APIs must include code examples showing correct usage
5. **Document Exceptions**: Use `@throws` to document all possible exceptions and error conditions
6. **Use Mermaid for Diagrams**: Create flowcharts, sequence diagrams, and architecture diagrams using Mermaid
7. **Keep READMEs Current**: Update README.md when adding features, changing setup, or modifying architecture
8. **Write for Beginners**: Assume readers are unfamiliar with the codebase - explain context and rationale
9. **Link to External Docs**: Reference official documentation for MCP protocol, European Parliament APIs, and standards
10. **Document Decisions**: Use Architecture Decision Records (ADRs) for significant technical decisions
11. **Security First**: Document threat models, security controls, and compliance requirements
12. **Show Anti-Patterns**: Include examples of incorrect usage to prevent common mistakes
13. **Maintain Changelog**: Keep CHANGELOG.md updated following Keep a Changelog format
14. **Version Documentation**: Clearly state which version of the code the documentation applies to
15. **Accessibility**: Use semantic markdown, descriptive link text, and alt text for images

## Examples

### ✅ Good Pattern: Comprehensive JSDoc for MCP Tool

```typescript
/**
 * Searches European Parliament documents by keyword and filters.
 * 
 * This function implements the MCP protocol search tool with comprehensive
 * security controls as required by Hack23 ISMS Policy AC-001 (Access Control Policy).
 * 
 * Security Controls:
 * - Input validation and sanitization to prevent injection attacks
 * - Rate limiting to prevent abuse (100 requests per 15 minutes)
 * - Audit logging of all search queries
 * - Data classification enforcement per European Parliament guidelines
 * 
 * Compliance: ISO 27001:2022 A.5.15, NIST CSF PR.AC-1, CIS Control 6.3
 * 
 * @param query - Search parameters
 * @param query.keywords - Search keywords (max 200 chars, alphanumeric + spaces)
 * @param query.documentType - Filter by document type (e.g., "REPORT", "RESOLUTION")
 * @param query.dateFrom - Start date for document filtering (ISO 8601 format)
 * @param query.dateTo - End date for document filtering (ISO 8601 format)
 * @param query.limit - Maximum results to return (1-100, default: 20)
 * @param options - Optional search options
 * @param options.language - Preferred language code (e.g., "en", "fr", "de")
 * @param options.sortBy - Sort field ("date", "relevance", "title")
 * 
 * @returns Search results with document metadata
 * 
 * @throws {ValidationError} When query parameters are invalid
 * @throws {RateLimitError} When rate limit is exceeded
 * @throws {AuthorizationError} When access is denied
 * @throws {ExternalAPIError} When European Parliament API is unavailable
 * 
 * @example
 * ```typescript
 * // Basic search
 * const results = await searchDocuments({
 *   keywords: 'climate change',
 *   limit: 10
 * });
 * console.log(`Found ${results.total} documents`);
 * 
 * // Advanced search with filters
 * const filtered = await searchDocuments(
 *   {
 *     keywords: 'digital markets',
 *     documentType: 'REPORT',
 *     dateFrom: '2024-01-01',
 *     dateTo: '2024-12-31'
 *   },
 *   {
 *     language: 'en',
 *     sortBy: 'date'
 *   }
 * );
 * 
 * // Error handling
 * try {
 *   const results = await searchDocuments({ keywords: 'test' });
 * } catch (error) {
 *   if (error instanceof RateLimitError) {
 *     console.error('Rate limit exceeded, retry after:', error.retryAfter);
 *   }
 * }
 * ```
 * 
 * @see {@link https://github.com/Hack23/ISMS-PUBLIC/blob/main/policies/AC-001.md | Access Control Policy}
 * @see {@link https://data.europarl.europa.eu/api/docs | European Parliament API Documentation}
 * @since 1.0.0
 * @public
 */
export async function searchDocuments(
  query: SearchQuery,
  options?: SearchOptions
): Promise<SearchResult> {
  // Implementation
}
```

### ✅ Good Pattern: README with Security Context

```markdown
# European Parliament MCP Server

> Model Context Protocol server providing access to European Parliament open data

[![CI/CD](https://github.com/Hack23/European-Parliament-MCP-Server/workflows/CI/badge.svg)](https://github.com/Hack23/European-Parliament-MCP-Server/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE.md)
[![Coverage](https://img.shields.io/codecov/c/github/Hack23/European-Parliament-MCP-Server)](https://codecov.io/gh/Hack23/European-Parliament-MCP-Server)

## 🔒 Security & Compliance

This project implements security controls aligned with [Hack23 AB's ISMS](https://github.com/Hack23/ISMS-PUBLIC):

- **ISO 27001:2022** - Information security management
- **NIST CSF 2.0** - Cybersecurity framework
- **CIS Controls v8.1** - Security best practices
- **OWASP API Security Top 10** - API security

For detailed security information, see [SECURITY.md](SECURITY.md).

## 🚀 Quick Start

### Prerequisites

- Node.js 20.x or later
- npm 10.x or later
- MCP-compatible client (Claude Desktop, VS Code with MCP extension)

### Installation

\`\`\`bash
# Clone repository
git clone https://github.com/Hack23/European-Parliament-MCP-Server.git
cd European-Parliament-MCP-Server

# Install dependencies
npm install

# Build the project
npm run build

# Run tests
npm test
\`\`\`

### Configuration

Create a configuration file for your MCP client:

\`\`\`json
{
  "mcpServers": {
    "european-parliament": {
      "command": "node",
      "args": ["path/to/build/index.js"],
      "env": {
        "LOG_LEVEL": "info"
      }
    }
  }
}
\`\`\`

## 📦 Available Scripts

| Script | Description | ISMS Policy |
|--------|-------------|-------------|
| `npm run build` | Production build | SC-001 |
| `npm run dev` | Development mode with watch | - |
| `npm run lint` | Run ESLint | SC-002 |
| `npm test` | Run unit tests | SC-002 |
| `npm run test:integration` | Run integration tests | SC-002 |
| `npm run coverage` | Generate coverage report | SC-002 |
| `npm audit` | Check dependencies for vulnerabilities | SC-003 |

## 🏗️ Architecture

\`\`\`mermaid
graph TB
    A[MCP Client] -->|Request| B[MCP Server]
    B -->|Parse| C[Request Handler]
    C -->|Validate| D[Input Validator]
    D -->|Query| E[European Parliament API]
    E -->|Response| F[Data Transformer]
    F -->|Format| G[MCP Response]
    G -->|Return| A
    
    C -->|Log| H[Audit Logger]
    C -->|Check| I[Rate Limiter]
    
    style B fill:#4A90E2
    style D fill:#E8A631
    style H fill:#50C878
    style I fill:#E85D75
\`\`\`

## 🔧 MCP Tools

### search_documents
Search European Parliament documents by keywords and filters.

\`\`\`typescript
const result = await client.callTool('search_documents', {
  keywords: 'climate change',
  documentType: 'REPORT',
  limit: 10
});
\`\`\`

### get_document
Retrieve a specific document by ID.

\`\`\`typescript
const document = await client.callTool('get_document', {
  documentId: 'EP-1234567'
});
\`\`\`

See [API Documentation](docs/API.md) for complete tool reference.

## 🧪 Testing

We maintain 80%+ code coverage with three testing layers:

1. **Unit Tests** (Vitest) - Individual function testing
2. **Integration Tests** (Vitest) - API integration testing
3. **E2E Tests** - Full MCP protocol flow testing

\`\`\`bash
# Run all tests with coverage
npm run coverage

# Run specific test file
npm test -- search.test.ts

# Run tests in watch mode
npm test -- --watch
\`\`\`

## 📊 Code Quality

| Metric | Target | Current |
|--------|--------|---------|
| Code Coverage | ≥ 80% | 85% |
| TypeScript Strict | ✅ Enabled | ✅ |
| Security Headers | ✅ All | ✅ |
| API Response Time | < 500ms | 350ms |

## 🔐 Security

### Reporting Vulnerabilities

See [SECURITY.md](SECURITY.md) for vulnerability reporting process.

### Security Features

- ✅ Input validation and sanitization
- ✅ Rate limiting (100 requests per 15 minutes)
- ✅ Audit logging for all operations
- ✅ Dependency scanning (npm audit)
- ✅ License compliance checking

## 📝 License

MIT License - see [LICENSE.md](LICENSE.md) file for details.

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.

## 📚 Documentation

- [API Reference](docs/API.md)
- [MCP Protocol Guide](docs/MCP_PROTOCOL.md)
- [Architecture Decision Records](docs/adr/)
- [Security Documentation](SECURITY.md)
```

### ✅ Good Pattern: Mermaid Diagram for MCP Flow

```markdown
## MCP Request Flow

\`\`\`mermaid
sequenceDiagram
    participant C as MCP Client
    participant S as MCP Server
    participant V as Validator
    participant A as EP API
    participant L as Logger
    
    C->>S: search_documents request
    S->>V: Validate input
    
    alt Invalid Input
        V-->>S: ValidationError
        S-->>C: Error response
    else Valid Input
        V-->>S: Validated query
        S->>A: HTTP GET /documents
        A-->>S: JSON response
        S->>L: Log request
        S-->>C: MCP response
    end
    
    Note over L: ISMS Policy AC-001<br/>Audit logging
    Note over A: Rate limit: 100/15min
\`\`\`
```

### ✅ Good Pattern: Architecture Decision Record

```markdown
# ADR-001: Use MCP Protocol for European Parliament Data Access

## Status
Accepted

## Context
We need to provide structured access to European Parliament open data for AI applications. Options considered:
1. REST API wrapper
2. GraphQL API
3. Model Context Protocol (MCP) server

## Decision
We will use MCP protocol to expose European Parliament data.

## Rationale

### Pros
- **AI-First Design**: MCP is designed specifically for AI assistants
- **Standardization**: Industry-standard protocol (Anthropic, et al.)
- **Type Safety**: Built-in schema validation and TypeScript support
- **Extensibility**: Easy to add new tools and capabilities
- **Security**: Built-in authentication and rate limiting support

### Cons
- **Newer Protocol**: Less established than REST/GraphQL
- **Limited Tooling**: Fewer debugging tools available
- **Client Support**: Requires MCP-compatible clients

## Security Implications
- MCP protocol provides structured security boundaries
- Input validation enforced by protocol schema
- ISMS Policy SC-001 (Secure Configuration) compliance maintained
- Rate limiting prevents abuse per ISMS Policy AC-002

## Consequences
- Developers must learn MCP protocol patterns
- Testing strategy includes MCP-specific integration tests
- Documentation must cover both MCP concepts and European Parliament APIs
- Client applications must support MCP protocol

## References
- [MCP Protocol Specification](https://modelcontextprotocol.io/)
- [European Parliament Open Data](https://data.europarl.europa.eu/)
- [ISMS Policy SC-001](https://github.com/Hack23/ISMS-PUBLIC/blob/main/policies/SC-001.md)

## Date
2025-01-08

## Author
Development Team
```

### ❌ Bad Pattern: Missing Context

```typescript
/**
 * Searches documents
 * @param query Search query
 * @returns Results
 */
function search(query: any): any {
  // No security context, no examples, poor types
}
```

### ❌ Bad Pattern: No Examples

```typescript
/**
 * Complex European Parliament API query with multiple parameters
 * and edge cases, but no examples showing how to use it.
 */
export async function queryParliamentaryDocuments(
  keywords: string,
  documentTypes: string[],
  dateRange: DateRange,
  pagination: PaginationOptions,
  filters: DocumentFilters
): Promise<QueryResult> {
  // Complex implementation without usage examples
}
```

### ❌ Bad Pattern: Outdated README

```markdown
# European Parliament MCP Server

Run `npm start` to start. <!-- Wrong command! -->

## Features
- Document search <!-- Actually implemented -->
- Member voting records <!-- Not implemented yet -->
- Real-time plenary sessions <!-- Removed 6 months ago -->
```

### ❌ Bad Pattern: No Security Documentation

```typescript
// Bad: Security-critical code without ISMS reference
export function validateApiKey(key: string): boolean {
  // Implements API key validation, but no documentation about
  // which ISMS policy it implements or compliance mapping
  return checkKey(key);
}
```

## References

### Documentation Standards
- [Google Developer Documentation Style Guide](https://developers.google.com/style)
- [Write the Docs](https://www.writethedocs.org/)
- [JSDoc](https://jsdoc.app/)
- [TypeDoc](https://typedoc.org/)

### Diagram Tools
- [Mermaid](https://mermaid.js.org/)
- [Mermaid Live Editor](https://mermaid.live/)
- [Mermaid in GitHub](https://github.blog/2022-02-14-include-diagrams-markdown-files-mermaid/)

### Markdown
- [GitHub Flavored Markdown](https://github.github.com/gfm/)
- [Markdown Guide](https://www.markdownguide.org/)
- [Keep a Changelog](https://keepachangelog.com/)

### Architecture
- [Architecture Decision Records (ADR)](https://adr.github.io/)
- [C4 Model](https://c4model.com/)

### MCP Protocol
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [MCP Specification](https://spec.modelcontextprotocol.io/)

## Remember

- **Security Context**: Always reference ISMS policies in security-related documentation
- **Code Examples**: Every public API needs working code examples
- **Keep Current**: Documentation should be updated with code changes
- **Mermaid Diagrams**: Use Mermaid for visual documentation - it's version-controlled and maintainable
- **JSDoc Everything**: All exported functions, classes, and interfaces need JSDoc
- **Write for Beginners**: Assume readers are new to the codebase
- **Link to Policies**: Reference external ISMS policies and compliance frameworks
- **Show Anti-Patterns**: Document what NOT to do to prevent mistakes
- **Accessibility**: Use semantic markdown and descriptive text
- **ADRs for Decisions**: Document significant architectural decisions
- **Changelog**: Maintain CHANGELOG.md following Keep a Changelog format
- **Version Docs**: Clearly state which version documentation applies to
- **Examples Over Words**: Show don't tell - code examples are more valuable than prose
- **Test Documentation**: Verify code examples actually work
- **MCP Protocol**: Document MCP-specific patterns and tool schemas
- **European Parliament Context**: Reference official EP API documentation
