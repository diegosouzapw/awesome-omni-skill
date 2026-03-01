---
name: senior-dotnet
description: World-class C# and .NET development skill for enterprise applications,
  ASP.NET Core web APIs, and cloud-native systems. Expertise in .NET 8, Entity Framework
  Core, ASP.NET Core Identity, Blazor, and microservices architecture. Includes project
  scaffolding, dependency management, security implementation, and performance optimization.
license: MIT
metadata:
  author: Claude Skills Team
  compatibility:
    python-version: 3.8+
    platforms:
    - macos
    - linux
    - windows
  contributors: []
  created: 2025-12-16
  dependencies:
    scripts:
    - dotnet_project_scaffolder.py
    - dependency_analyzer.py
    - entity_generator.py
    - api_endpoint_generator.py
    - security_config_generator.py
    - performance_profiler.py
    references:
    - dotnet-best-practices.md
    - aspnet-core-patterns.md
    - ef-core-guide.md
    - dotnet-security-reference.md
    - dotnet-performance-tuning.md
    assets: []
  difficulty: advanced
  domain: engineering
  examples:
  - title: ASP.NET Core Web API Scaffolding
    input: python scripts/dotnet_project_scaffolder.py my-service --type webapi --db
      sqlserver
    output: Complete ASP.NET Core 8 project with Clean Architecture, Docker setup,
      and CI/CD pipeline
  - title: Entity Generation
    input: python scripts/entity_generator.py Product --fields 'Id:int,Name:string,Price:decimal,CreatedAt:DateTime'
    output: EF Core entity with repository, service, controller, and DTO classes
  featured: false
  frequency: Daily for enterprise development teams
  orchestrated-by:
  - dotnet-engineer
  related-agents:
  - dotnet-engineer
  related-commands: []
  related-skills:
  - senior-backend
  - senior-architect
  stats:
    downloads: 0
    stars: 0
    rating: 0.0
    reviews: 0
  subdomain: dotnet-development
  tags:
  - csharp
  - dotnet
  - aspnet-core
  - entity-framework
  - ef-core
  - webapi
  - blazor
  - microservices
  - enterprise
  - cloud-native
  - azure
  - nuget
  - api
  - backend
  tech-stack:
  - C# 12
  - .NET 8 LTS
  - ASP.NET Core 8
  - Entity Framework Core 8
  - ASP.NET Core Identity
  - Blazor
  - Minimal APIs
  - MediatR/CQRS
  - FluentValidation
  - AutoMapper
  - xUnit
  - Docker
  - Kubernetes
  time-saved: 60%+ on project scaffolding, 50% on security implementation
  title: Senior .NET Skill Package
  updated: 2025-12-16
  use-cases:
  - Building enterprise ASP.NET Core Web APIs with production-ready configuration
  - Designing microservices with .NET Aspire and service discovery
  - Implementing Entity Framework Core data layers with optimized queries
  - Setting up ASP.NET Core Identity with OAuth2 and JWT authentication
  - Performance tuning .NET applications and async patterns
  verified: true
  version: v1.0.0
---

# Senior .NET

World-class C# and .NET development skill for enterprise applications, ASP.NET Core web APIs, and cloud-native systems. Expertise in .NET 8, Entity Framework Core, ASP.NET Core Identity, Blazor, and microservices architecture.

## Overview

This skill provides production-ready C# and .NET development capabilities through six Python automation tools and comprehensive reference documentation. Whether building enterprise monoliths, microservices architectures, or Blazor applications, this skill ensures best practices, scalable architecture, and enterprise-grade security.

**What This Skill Provides:**
- ASP.NET Core project scaffolding with Clean Architecture
- EF Core entity and repository generation with optimized queries
- RESTful API endpoint scaffolding (Controllers and Minimal APIs)
- ASP.NET Core Identity configuration (OAuth2, JWT, OIDC)
- NuGet dependency analysis and upgrade recommendations
- .NET performance profiling and optimization guidance

**Use this skill when:**
- Starting new ASP.NET Core projects or microservices
- Implementing Entity Framework Core data layers
- Designing RESTful APIs with Controllers or Minimal APIs
- Setting up authentication and authorization
- Optimizing .NET application performance
- Reviewing C# code for quality and patterns

**Core Value:** Save 60%+ time on project scaffolding while ensuring enterprise-grade architecture, security compliance, and performance optimization.

## Core Capabilities

- **ASP.NET Core Scaffolding** - Generate production-ready .NET 8 projects with Clean Architecture, Docker configuration, and CI/CD pipelines
- **Entity Generation** - Create EF Core entities with repositories, services, controllers, DTOs, and AutoMapper profiles following DDD patterns
- **API Development** - Scaffold REST endpoints with validation, error handling, pagination, and OpenAPI documentation
- **Security Implementation** - Configure ASP.NET Core Identity with OAuth2, JWT, role-based access control, and security best practices
- **Dependency Management** - Analyze NuGet packages for vulnerabilities, outdated versions, and upgrade paths
- **Performance Optimization** - Profile .NET applications, identify bottlenecks, and generate optimization recommendations

## Quick Start

### Create ASP.NET Core Project

```bash
# Create Web API with SQL Server
python scripts/dotnet_project_scaffolder.py order-service --type webapi --db sqlserver

# Create MVC application with PostgreSQL and Identity
python scripts/dotnet_project_scaffolder.py ecommerce-app --type mvc --db postgresql --auth identity

# Create Minimal API service
python scripts/dotnet_project_scaffolder.py notification-service --type minimal-api --db sqlite
```

### Generate EF Core Entity

```bash
# Generate complete entity stack (entity, repository, service, controller, DTO)
python scripts/entity_generator.py Product --fields "Id:int,Name:string,Price:decimal,Category:string,CreatedAt:DateTime"

# Generate with relationships
python scripts/entity_generator.py Order --fields "Id:int,CustomerId:int,Total:decimal" --relations "Customer:ManyToOne,Items:OneToMany"
```

### Analyze Dependencies

```bash
# Check for vulnerabilities and updates
python scripts/dependency_analyzer.py MyProject.csproj --check-security

# Generate upgrade report
python scripts/dependency_analyzer.py src/ --output report.md
```

### Access Documentation

- .NET patterns: `references/dotnet-best-practices.md`
- ASP.NET Core design: `references/aspnet-core-patterns.md`
- EF Core guide: `references/ef-core-guide.md`
- Security reference: `references/dotnet-security-reference.md`
- Performance tuning: `references/dotnet-performance-tuning.md`

## Key Workflows

### 1. New ASP.NET Core Web API

**Time:** 30-45 minutes

1. **Scaffold Project** - Generate Web API with .NET 8, Docker, and CI/CD
   ```bash
   python scripts/dotnet_project_scaffolder.py inventory-service --type webapi --db sqlserver --auth jwt
   ```

2. **Configure Environment** - Set up appsettings.json with environment-specific settings
   ```bash
   cd inventory-service
   # Edit src/Api/appsettings.json
   # Configure database connection, JWT settings, and service endpoints
   ```

3. **Generate Entities** - Create domain model with EF Core entities
   ```bash
   python scripts/entity_generator.py Inventory --fields "Id:int,ProductId:int,Quantity:int,Warehouse:string"
   python scripts/entity_generator.py InventoryMovement --fields "Id:int,InventoryId:int,Quantity:int,Type:string,Timestamp:DateTime"
   ```

4. **Implement Business Logic** - Add service layer logic and validation rules

5. **Add Tests** - Generate unit and integration tests
   ```bash
   # Run tests
   dotnet test
   dotnet test --filter Category=Integration
   ```

6. **Build and Deploy**
   ```bash
   dotnet publish -c Release
   docker build -t inventory-service:latest .
   ```

See [dotnet-best-practices.md](references/dotnet-best-practices.md) for complete setup patterns.

### 2. REST API Development

**Time:** 20-30 minutes per endpoint group

1. **Design API Contract** - Define endpoints following REST conventions
   ```bash
   python scripts/api_endpoint_generator.py products --methods GET,POST,PUT,DELETE --paginated
   ```

2. **Implement Validation** - Add FluentValidation validators and custom rules

3. **Configure Error Handling** - Set up global exception handler with ProblemDetails (RFC 7807)

4. **Add OpenAPI Documentation** - Configure Swagger/OpenAPI for automatic API docs

5. **Test Endpoints** - Generate integration tests with WebApplicationFactory

See [aspnet-core-patterns.md](references/aspnet-core-patterns.md) for API design patterns.

### 3. Entity Framework Core Optimization

**Time:** 1-2 hours for complex data models

1. **Analyze Current Queries** - Profile repository methods for N+1 problems
   ```bash
   python scripts/performance_profiler.py --analyze-queries src/
   ```

2. **Optimize Fetch Strategies** - Configure eager loading with Include/ThenInclude

3. **Add Query Projections** - Use Select projections for read-only queries

4. **Configure Caching** - Set up distributed caching with Redis

5. **Implement Pagination** - Use cursor-based or offset pagination for large datasets

See [ef-core-guide.md](references/ef-core-guide.md) for optimization patterns.

### 4. ASP.NET Core Security Implementation

**Time:** 1-2 hours

1. **Generate Security Config** - Create security configuration for chosen auth method
   ```bash
   python scripts/security_config_generator.py --type jwt --roles Admin,User,Manager
   ```

2. **Configure JWT/OAuth2** - Set up token generation, validation, and refresh

3. **Implement Authorization** - Add policy-based authorization to endpoints

4. **Add Attribute Authorization** - Configure [Authorize] attributes with policies

5. **Test Security** - Generate security integration tests

See [dotnet-security-reference.md](references/dotnet-security-reference.md) for security patterns.

## Python Tools

### dotnet_project_scaffolder.py

Generate production-ready ASP.NET Core project structures with complete configuration.

**Key Features:**
- .NET 8 with C# 12 support
- Multiple project types (webapi, mvc, blazor, minimal-api, worker)
- Database configuration (SQL Server, PostgreSQL, MySQL, SQLite, Cosmos DB)
- Docker and Docker Compose setup
- GitHub Actions CI/CD pipeline
- Clean Architecture template
- FluentValidation and AutoMapper integration

**Common Usage:**
```bash
# Web API with SQL Server and JWT security
python scripts/dotnet_project_scaffolder.py user-service --type webapi --db sqlserver --auth jwt

# MVC with PostgreSQL and Identity
python scripts/dotnet_project_scaffolder.py ecommerce --type mvc --db postgresql --auth identity

# Minimal API service with SQLite
python scripts/dotnet_project_scaffolder.py notification-service --type minimal-api --db sqlite

# Help
python scripts/dotnet_project_scaffolder.py --help
```

### entity_generator.py

Generate complete EF Core entity stacks with repository, service, controller, and DTO.

**Key Features:**
- EF Core entity with data annotations or Fluent API
- Generic repository pattern implementation
- Service layer with dependency injection
- REST controller with validation
- DTO and AutoMapper profile
- Relationship support (OneToMany, ManyToOne, ManyToMany)

**Common Usage:**
```bash
# Basic entity
python scripts/entity_generator.py Customer --fields "Id:int,Name:string,Email:string"

# Entity with relationships
python scripts/entity_generator.py Order --fields "Id:int,CustomerId:int,Total:decimal" --relations "Customer:ManyToOne"

# Entity with audit fields
python scripts/entity_generator.py Product --fields "Id:int,Name:string,Price:decimal" --auditable

# Help
python scripts/entity_generator.py --help
```

### api_endpoint_generator.py

Scaffold RESTful API endpoints with validation and documentation.

**Key Features:**
- Controller or Minimal API endpoint generation
- Request/response DTOs
- FluentValidation validators
- OpenAPI/Swagger annotations
- Pagination support
- Error handling

**Common Usage:**
```bash
# Full CRUD endpoints
python scripts/api_endpoint_generator.py orders --methods GET,POST,PUT,DELETE

# Read-only with pagination
python scripts/api_endpoint_generator.py reports --methods GET --paginated

# Minimal API style
python scripts/api_endpoint_generator.py products --style minimal-api

# Help
python scripts/api_endpoint_generator.py --help
```

### security_config_generator.py

Generate ASP.NET Core security configuration for various authentication methods.

**Key Features:**
- JWT Bearer authentication setup
- ASP.NET Core Identity configuration
- OAuth2/OIDC configuration
- Policy-based authorization
- CORS configuration
- Security middleware

**Common Usage:**
```bash
# JWT security with roles
python scripts/security_config_generator.py --type jwt --roles Admin,User

# Identity with external login
python scripts/security_config_generator.py --type identity --external-providers Google,Microsoft

# OAuth2/OIDC
python scripts/security_config_generator.py --type oidc --authority https://auth.example.com

# Help
python scripts/security_config_generator.py --help
```

### dependency_analyzer.py

Analyze NuGet dependencies for vulnerabilities and updates.

**Key Features:**
- Security vulnerability scanning
- Outdated package detection
- Upgrade path recommendations
- Dependency tree analysis
- License compliance checking

**Common Usage:**
```bash
# Analyze project
python scripts/dependency_analyzer.py MyProject.csproj

# Analyze with security focus
python scripts/dependency_analyzer.py src/ --check-security

# Generate markdown report
python scripts/dependency_analyzer.py MyProject.csproj --output report.md

# Help
python scripts/dependency_analyzer.py --help
```

### performance_profiler.py

Profile .NET applications and generate optimization recommendations.

**Key Features:**
- EF Core query analysis for N+1 detection
- Async/await antipattern detection
- Memory allocation patterns
- Connection pool recommendations
- GC behavior analysis

**Common Usage:**
```bash
# Analyze source for performance issues
python scripts/performance_profiler.py --analyze-queries src/

# Generate optimization report
python scripts/performance_profiler.py src/ --output performance-report.md

# Help
python scripts/performance_profiler.py --help
```

## Reference Documentation

### When to Use Each Reference

**[dotnet-best-practices.md](references/dotnet-best-practices.md)** - .NET Patterns
- Solution/project structure and Clean Architecture
- Dependency injection patterns
- Configuration management with Options pattern
- Logging and monitoring with Serilog
- Testing strategies (unit, integration, functional)

**[aspnet-core-patterns.md](references/aspnet-core-patterns.md)** - ASP.NET Core
- Middleware pipeline configuration
- Minimal APIs vs Controllers
- Request/response handling
- Filters and attributes
- Background services with IHostedService

**[ef-core-guide.md](references/ef-core-guide.md)** - Data Layer
- Entity configuration with Fluent API
- Migrations management
- Query optimization and projections
- Change tracking strategies
- Concurrency handling with row versioning

**[dotnet-security-reference.md](references/dotnet-security-reference.md)** - Security
- Authentication schemes (JWT, Identity, OIDC)
- Authorization patterns (roles, policies, requirements)
- Data Protection API
- OWASP compliance checklist
- Security headers and HTTPS

**[dotnet-performance-tuning.md](references/dotnet-performance-tuning.md)** - Performance
- Async/await best practices
- Memory management and allocation
- Caching strategies (in-memory, distributed)
- Connection pooling configuration
- Profiling tools and diagnostics

## Best Practices

### Quality Standards

- **Code Coverage:** Target 80%+ for business logic, 60%+ overall
- **API Documentation:** 100% endpoint coverage with OpenAPI
- **Security Scanning:** Zero critical/high vulnerabilities
- **Performance:** P99 latency < 200ms for CRUD operations

### Common Pitfalls to Avoid

- **N+1 Queries** - Always use Include/ThenInclude or projections for relationships
- **Missing Async** - Use async/await consistently for I/O operations
- **Sync over Async** - Never use .Result or .Wait() on async methods
- **Hardcoded Configuration** - Use IOptions pattern and environment variables
- **Missing Validation** - Always validate input with FluentValidation
- **EF Core Tracking** - Use AsNoTracking() for read-only queries

See [dotnet-best-practices.md](references/dotnet-best-practices.md) for detailed guidelines.

## Performance Metrics

**Development Efficiency:**
- Project scaffolding time (target: < 30 minutes)
- Entity stack generation (target: < 5 minutes per entity)
- Security setup time (target: < 1 hour)

**Code Quality:**
- Test coverage (target: 80%+)
- Static analysis issues (target: 0 critical/high)
- Documentation coverage (target: 100% public APIs)

**Runtime Performance:**
- P99 latency (target: < 200ms)
- Throughput (target: > 1000 RPS per instance)
- Memory efficiency (target: < 256MB for typical service)

## Integration

This skill works best with:
- **senior-backend** - For general API patterns and database design
- **senior-architect** - For system design and microservices architecture decisions
- **senior-devops** - For CI/CD pipeline and Kubernetes deployment
- **senior-security** - For security audits and penetration testing

See [dotnet-best-practices.md](references/dotnet-best-practices.md) for CI/CD and automation integration examples.

## Composability & Integration

### Skill Composition Patterns

**This skill receives input from:**
- **senior-architect** - Architecture decisions inform project scaffolding choices
- **business-analyst-toolkit** - Requirements define entity models and API contracts
- **product-manager-toolkit** - User stories guide feature implementation

**This skill provides output to:**
- **senior-devops** - Generated projects include Dockerfile and CI/CD configuration
- **senior-qa** - Generated code includes test scaffolding for QA automation
- **technical-writer** - OpenAPI specs feed API documentation generation

### Recommended Skill Combinations

**Workflow Pattern 1: Microservices Development**
```
senior-architect → senior-dotnet → senior-devops
```
Use this pattern for designing and deploying microservices with proper architecture review.

**Workflow Pattern 2: Full-Stack Feature**
```
senior-dotnet → senior-frontend → senior-qa
```
Use this pattern for end-to-end feature implementation with backend API, frontend UI, and testing.

## Benefits

**Time Savings:**
- 60% faster project scaffolding vs. manual setup
- 50% reduction in boilerplate code through generation
- 40% faster security implementation with templates

**Quality Improvements:**
- Consistent architecture across projects
- Built-in security best practices
- Comprehensive test coverage templates

**Business Impact:**
- Faster time-to-market for new services
- Reduced technical debt through standardization
- Lower maintenance costs through consistency

## Next Steps

**Getting Started:**
1. Run `python scripts/dotnet_project_scaffolder.py my-service --type webapi --db sqlserver` to create your first project
2. Review generated structure and customize configuration
3. Generate entities with `python scripts/entity_generator.py`

**Advanced Usage:**
- Configure .NET Aspire for service discovery
- Implement CQRS patterns with MediatR
- Set up distributed tracing with OpenTelemetry

## Additional Resources

- **Quick commands** - See tool documentation above
- **Best practices** - See [dotnet-best-practices.md](references/dotnet-best-practices.md)
- **Troubleshooting** - See [dotnet-performance-tuning.md](references/dotnet-performance-tuning.md)
- **External documentation** - [ASP.NET Core Documentation](https://learn.microsoft.com/aspnet/core)

---

**Documentation:** Full skill guide and workflows available in this file

**Support:** For issues or questions, refer to domain guide at `../CLAUDE.md` and agent authoring guide at `../../../agents/agent-author.md`

**Version:** 1.0.0 | **Last Updated:** 2025-12-16 | **Status:** Production Ready
