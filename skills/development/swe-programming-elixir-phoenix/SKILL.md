---
name: swe-programming-elixir-phoenix
description: Phoenix Framework coding standards from authoritative docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ documentation
---

# Phoenix Framework Coding Standards

## Purpose

Progressive disclosure of Phoenix Framework standards for agents writing Phoenix applications.

**Authoritative Source**: [docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/README.md](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/README.md)

**Usage**: Auto-loaded for agents when writing Phoenix Framework code. Provides quick reference to Phoenix contexts, channels, REST APIs, and web patterns.

**Foundation**: Requires Elixir skill (swe-programming-elixir). Phoenix Framework is built on Elixir/OTP foundations.

## Prerequisite Knowledge

**CRITICAL**: This skill provides **OSE Platform Phoenix Framework standards**, not Phoenix tutorials.

**You MUST complete both Elixir AND Phoenix learning paths**:

**1. Elixir Foundation** (prerequisite for Phoenix):

- [Elixir Learning Path](../../../apps/ayokoding-web/content/en/learn/software-engineering/programming-languages/elixir/) - Initial setup, language overview (0-95% coverage)
- [Elixir By Example](../../../apps/ayokoding-web/content/en/learn/software-engineering/programming-languages/elixir/by-example/) - 75+ annotated code examples
- [Elixir In the Field](../../../apps/ayokoding-web/content/en/learn/software-engineering/programming-languages/elixir/in-the-field/) - Production patterns
- [Elixir Release Highlights](../../../apps/ayokoding-web/content/en/learn/software-engineering/programming-languages/elixir/release-highlights/) - Elixir 1.12-1.18 features

**2. Phoenix Framework Learning Path**:

- [Phoenix Initial Setup](../../../apps/ayokoding-web/content/en/learn/software-engineering/platform-web/tools/elixir-phoenix/initial-setup.md) - Environment setup
- [Phoenix Overview](../../../apps/ayokoding-web/content/en/learn/software-engineering/platform-web/tools/elixir-phoenix/overview.md) - Contexts, controllers, views
- [Phoenix By Example](../../../apps/ayokoding-web/content/en/learn/software-engineering/platform-web/tools/elixir-phoenix/by-example/) - 75+ examples
- [Phoenix In-the-Field](../../../apps/ayokoding-web/content/en/learn/software-engineering/platform-web/tools/elixir-phoenix/in-the-field/) - 30 production guides

**Documentation Separation**:

- **AyoKoding Phoenix** - "How to use Phoenix Framework" (educational, universal patterns)
- **docs/explanation/elixir-phoenix** - "How to use Phoenix in OSE Platform" (repository conventions)

**What this skill covers**: OSE Platform Phoenix Framework configuration, context patterns, channel usage, REST API design, data access patterns.

**What this skill does NOT cover**: Phoenix Framework basics, OTP fundamentals (those are in ayokoding-web).

## Quick Standards Reference

### Context Patterns

- Use contexts to group related functionality
- Keep contexts focused and bounded
- Contexts expose public API, hide implementation
- Use schemas within contexts for data structures

### Controllers and Views

- Keep controllers thin, delegate to contexts
- Use action fallback for error handling
- Render JSON with Jason for APIs
- Use Phoenix.HTML helpers for templates

### Routing

- Use resources for RESTful routes
- Scope routes by authentication requirements
- Use plugs for request pipeline customization
- Apply rate limiting at router level

### Channels and PubSub

- Use channels for real-time bidirectional communication
- Leverage Phoenix.PubSub for process communication
- Implement presence tracking for user activity
- Handle channel errors gracefully

### Data Access with Ecto

- Use Ecto schemas for data modeling
- Apply changesets for data validation
- Write composable Ecto queries
- Use Repo for database operations
- Apply database transactions for consistency

### REST API Design

- Use JSON:API or GraphQL conventions
- Implement proper HTTP status codes
- Apply authentication with Guardian or Pow
- Version APIs with URL prefixes or headers
- Document APIs with ExDoc

## Comprehensive Documentation

For detailed guidance, refer to the 15 Phoenix Framework standards files:

**Core Patterns**:

- [Idioms](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__idioms.md) - Phoenix-specific patterns
- [Best Practices](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__best-practices.md) - Framework standards
- [Anti-Patterns](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__anti-patterns.md) - Common mistakes

**Architecture & Configuration**:

- [Configuration](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__configuration.md)
- [Contexts](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__contexts.md)
- [Channels](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__channels.md)

**Data & Web**:

- [Data Access](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__data-access.md)
- [REST APIs](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__rest-apis.md)

**Quality & Operations**:

- [Security](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__security.md)
- [Testing](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__testing.md)
- [Performance](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__performance.md)
- [Observability](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__observability.md)
- [Deployment](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__deployment.md)

**Maintenance**:

- [Version Migration](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/ex-soen-plwe-to-elph__version-migration.md)

## Related Skills

- swe-programming-elixir - Elixir language fundamentals (prerequisite)
- docs-applying-content-quality - Content quality standards
- repo-practicing-trunk-based-development - Git workflow

## References

- [Phoenix Framework README](../../../docs/explanation/software-engineering/platform-web/tools/elixir-phoenix/README.md)
- [Elixir README](../../../docs/explanation/software-engineering/programming-languages/elixir/README.md)
- [Functional Programming](../../../governance/development/pattern/functional-programming.md)
