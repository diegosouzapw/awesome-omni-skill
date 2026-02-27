---
name: architectural-pattern-discovery
description: "Discovers architectural and design patterns across all abstraction levels. Analyzes structural patterns, component relationships, recurring solution approaches, and design principles. Works with any technology stack without prior framework knowledge to provide comprehensive pattern understanding from code-level to system-level architecture."

# When to Use This Skill
**Primary Use Cases:**
- **Project Onboarding**: Quickly understand the architectural foundation and design philosophy when joining a new project
- **Pattern Documentation**: Create comprehensive pattern documentation for knowledge sharing and team alignment
- **Refactoring Planning**: Identify existing patterns before large-scale refactoring to ensure consistency
- **Architecture Reviews**: Analyze architectural decisions and identify improvement opportunities
- **Technical Debt Assessment**: Understand how patterns have evolved and identify inconsistent applications

# What This Skill Provides

## Comprehensive Pattern Analysis
**Low-Level Design Patterns**
- Creational patterns: Factory, Singleton, Builder, Prototype implementations
- Structural patterns: Adapter, Decorator, Facade, Proxy, Bridge usage
- Behavioral patterns: Observer, Strategy, Command, State, Iterator patterns
- Code-level idioms and framework-specific patterns

**High-Level Architectural Patterns**
- Architectural styles: Monolithic, Microservices, Event-Driven, Layered, Service-Oriented
- Data management patterns: Centralized vs Distributed databases, CQRS, Event Sourcing
- Communication patterns: RESTful APIs, gRPC, Message Queues, WebSockets
- Integration patterns: API Gateway, Service Mesh, Circuit Breaker, Retry patterns

**Project-Specific Solution Patterns**
- Recurring feature implementation approaches and templates
- Common problem-solving patterns unique to the business domain
- Data transformation and processing workflows
- Error handling and recovery strategies
- Configuration and deployment patterns

## Pattern Relationship Mapping
**Component Interaction Analysis**
- Service-to-service communication patterns and protocols
- UI-backend synchronization approaches
- Authentication and authorization flows
- State management and propagation patterns
- Event-driven communication and message passing

**Data Flow and Processing Patterns**
- Data transformation pipelines and validation approaches
- State change propagation and consistency patterns
- Caching strategies and data synchronization
- Batch vs real-time processing patterns
- Integration with external data sources and APIs

## Technology-Aware Pattern Discovery
**Framework-Specific Patterns**
- How patterns manifest in different frameworks (React, Angular, Spring, Django)
- Framework-specific idioms and conventions
- Integration patterns for different database systems
- Cloud deployment patterns (AWS, GCP, Azure)

**Modern vs Legacy Patterns**
- Identification of outdated patterns and modernization opportunities
- Migration strategies from legacy to modern patterns
- Hybrid approaches combining multiple pattern generations

# Discovery Value for Agents
**For Architecture Specialists**: Provides comprehensive pattern landscape for architectural decision-making and system evolution planning.

**For Development Teams**: Documents established patterns ensuring consistency and enabling effective knowledge transfer between team members.

**For DevOps Engineers**: Reveals deployment, scaling, and monitoring patterns embedded in the architecture for operational optimization.

**For Security Specialists**: Identifies security-related patterns and potential vulnerabilities in architectural choices.

**For Technical Leads**: Offers complete pattern overview for strategic planning, technology selection, and team coordination.

---

## Pattern Description

**What**: Systematically analyzes a codebase to identify recurring architectural, design, and coding patterns across all abstraction levels, from low-level implementation idioms to high-level system architecture, including project-specific solution approaches and technology integration patterns.

**When**: Use this skill to understand the complete pattern landscape of a project, during architectural assessments, for refactoring efforts, when onboarding to complex systems, or when planning technology evolution and modernization strategies.

**Context**: Applicable across all technology stacks and project types, focusing on observable patterns rather than theoretical frameworks, with emphasis on understanding how patterns actually manifest in practice.

## Project-Specific Conventions

### Architectural Patterns
- **Overall Structure**: [e.g., Monolithic, Microservices, Layered, Event-Driven, Serverless]
- **Data Management**: [e.g., Centralized Database, Distributed Databases, CQRS, Event Sourcing]
- **Communication**: [e.g., RESTful APIs, gRPC, Message Queues, WebSockets, GraphQL]
- **Deployment**: [e.g., Container-based, Serverless, Traditional hosting, Multi-cloud]

### Design Patterns
- **Structural**: [e.g., Adapter, Decorator, Facade, Proxy, Bridge, Composite]
- **Behavioral**: [e.g., Observer, Strategy, Command, State, Iterator, Mediator]
- **Creational**: [e.g., Factory, Singleton, Builder, Prototype, Abstract Factory]

### Project-Specific Solution Patterns
- **Feature Templates**: [e.g., how new features are consistently structured and integrated]
- **Data Processing**: [e.g., standard data transformation and validation pipelines]
- **Error Handling**: [e.g., consistent error response patterns and recovery strategies]
- **Configuration**: [e.g., how configuration is managed and propagated across systems]

### Technology Integration Patterns
- **Framework Integration**: [e.g., how external libraries and frameworks are consistently integrated]
- **Database Patterns**: [e.g., ORM usage patterns, query optimization approaches, transaction handling]
- **API Design**: [e.g., consistent API structures, versioning strategies, response formats]
- **Security Patterns**: [e.g., authentication flows, authorization checks, data protection approaches]

## Common Pitfalls

### ❌ Implicit Patterns
**Problem**: Important architectural and design patterns exist but are not explicitly documented or understood by the team.
**Why It Fails**: Leads to inconsistent implementations, makes onboarding difficult, and creates technical debt through pattern drift.
**Better Approach**: Document discovered patterns using architectural decision records (ADRs) and maintain a living pattern catalog.

### ❌ Pattern Inconsistency
**Problem**: Similar problems are solved using different patterns without clear rationale, leading to architectural confusion.
**Why It Fails**: Increases cognitive load for developers, makes maintenance harder, and reduces system predictability.
**Better Approach**: Establish clear pattern selection guidelines and conduct regular architectural reviews to ensure consistency.

### ❌ Over-Engineering Patterns
**Problem**: Applying complex architectural patterns where simpler solutions would suffice.
**Why It Fails**: Increases unnecessary complexity, development time, and maintenance overhead without proportional benefits.
**Better Approach**: Choose patterns that match the problem's complexity; prioritize simplicity and evolve patterns as complexity grows.

## Related Resources

### Related Skills
- **convention-extraction**: For lower-level coding style and naming conventions
- **universal-technology-discovery**: To identify the technologies upon which patterns are built
- **integration-mapping-discovery**: For detailed analysis of component connections and data flow
- **operational-intelligence-discovery**: For understanding deployment and operational patterns

### Related Agents
- **architecture-specialist**: For in-depth analysis and recommendation of architectural patterns
- **lead-architect-agent**: For defining and evolving project-specific patterns and standards

### External Resources
- [Refactoring Guru - Design Patterns](https://refactoring.guru/design-patterns)
- [Martin Fowler - Architectural Patterns](https://martinfowler.com/architecture/)
- [Architectural Decision Records (ADR)](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [Domain-Driven Design (DDD)](https://martinfowler.com/bliki/DomainDrivenDesign.html)
- [Patterns of Enterprise Application Architecture](https://martinfowler.com/books/eaa.html)