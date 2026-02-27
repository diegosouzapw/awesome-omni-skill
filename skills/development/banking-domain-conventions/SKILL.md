---
name: banking-domain-conventions
description: Use when writing code in a Firefly Banking Platform (firefly-oss) service — applies domain organization, service taxonomy, inter-service communication patterns, POM inheritance, and platform-specific conventions
---

# Firefly Banking Platform Conventions

## 1. Service Taxonomy

The Firefly Banking Platform organizes ~55 repositories into six domain prefixes. Each prefix defines the service's role, architectural tier, and framework starter.

| Prefix | Role | Tier | Framework Starter | Examples |
|--------|------|------|-------------------|----------|
| `core-banking-*` | Banking primitives: accounts, cards, ledger, payments | Core (CRUD + business rules) | `fireflyframework-starter-core` | `core-banking-accounts`, `core-banking-ledger`, `core-banking-payments`, `core-banking-cards`, `core-banking-psdx` |
| `core-lending-*` | Lending primitives: origination, servicing, credit, collateral | Core (CRUD + business rules) | `fireflyframework-starter-core` | `core-lending-loan-origination`, `core-lending-loan-servicing`, `core-lending-credit-scoring`, `core-lending-collateral-management`, `core-lending-mortgage` |
| `core-common-*` | Cross-cutting capabilities: customers, products, documents, security | Core (CRUD + business rules) | `fireflyframework-starter-core` | `core-common-customer-mgmt`, `core-common-product-mgmt`, `core-common-kycb-mgmt`, `core-common-user-mgmt`, `core-common-security-vault` |
| `core-data-*` | External data integration and templates | Core (data access) | `fireflyframework-starter-core` | `core-data-credit-bureaus`, `core-data-template` |
| `domain-*` | Orchestration of multiple core services into business workflows | Domain (orchestration) | `fireflyframework-starter-domain` | `domain-customer-people`, `domain-customer-kyc-kyb`, `domain-lending-loan-origination`, `domain-product-catalog`, `domain-distributor-branding` |
| `exp-*` | Backend-for-Frontend organized by user journey | Experience (BFF) | `fireflyframework-starter-application` | `exp-onboarding`, `exp-lending`, `exp-profile`, `exp-payments` |
| `app-*` | Entry-point applications: gateways, authenticators, configurators | Application (gateway/edge) | `fireflyframework-starter-application` | `app-authenticator`, `app-configurator-masters` |

Additional non-service repos:

| Repo | Purpose |
|------|---------|
| `firefly-parent` | Platform parent POM (extends `fireflyframework-parent`) |
| `firefly-bom` | Bill of Materials for all `com.firefly` artifacts |
| `firefly-config` | Centralized Spring Cloud Config repository (dev/pre/pro profiles) |

## 2. Package Naming

All Firefly platform code uses `groupId` = `com.firefly`. Java packages follow the pattern:

```
com.firefly.{tier}.{domain-short-name}.{module-layer}.{sub-package}
```

Real examples from the codebase:

| Repo | Module | Package Root |
|------|--------|-------------|
| `core-common-customer-mgmt` | `-interfaces` | `com.firefly.core.customer.interfaces.dtos` / `...interfaces.enums` |
| `core-common-customer-mgmt` | `-models` | `com.firefly.core.customer.models.entities` / `...models.repositories` |
| `core-common-customer-mgmt` | `-core` | `com.firefly.core.customer.core.services` / `...core.mappers` |
| `core-common-customer-mgmt` | `-web` | `com.firefly.core.customer.web.controllers` |
| `core-common-customer-mgmt` | `-sdk` | `com.firefly.core.customer.sdk.api` / `...sdk.model` / `...sdk.invoker` |
| `core-lending-loan-origination` | `-interfaces` | `com.firefly.core.lending.origination.interfaces.dtos` / `...interfaces.enums` |
| `core-lending-loan-origination` | `-core` | `com.firefly.core.lending.origination.core.services` / `...core.mappers` |
| `core-lending-loan-origination` | `-web` | `com.firefly.core.lending.origination.web` |
| `domain-customer-people` | `-core` | `com.firefly.domain.people.core.customer.commands` / `...core.customer.handlers` |
| `domain-customer-people` | `-infra` | `com.firefly.domain.people.infra` |
| `domain-customer-people` | `-web` | `com.firefly.domain.people.web.controller` |
| `domain-customer-people` | `-sdk` | `com.firefly.domain.people.sdk.api` / `...sdk.model` |
| `exp-onboarding` | `-web` | `com.firefly.exp.onboarding.web` / `...web.controllers` |
| `app-authenticator` | `-web` | `com.firefly.app.orchestrator.web` / `...web.controllers` |

Key pattern: the domain short name drops the prefix category and management suffix. For example, `core-common-customer-mgmt` becomes `core.customer`, and `domain-customer-people` becomes `domain.people`.

## 3. POM Inheritance Chain

Every service follows a three-level POM hierarchy:

```
org.fireflyframework:fireflyframework-parent (1.0.0-SNAPSHOT)
    |
    v
com.firefly:firefly-parent (1.0.0-SNAPSHOT)
    |
    v
com.firefly:{service-artifactId} (1.0.0-SNAPSHOT)
```

### What `fireflyframework-parent` provides

- Spring Boot parent (version management for Spring Boot, Spring Cloud, etc.)
- Plugin management (maven-compiler-plugin, spring-boot-maven-plugin, etc.)
- Dependency versions for framework artifacts (`org.fireflyframework` groupId)

### What `firefly-parent` adds on top

- **Imports two BOMs**: `fireflyframework-bom` (version `26.02.06`) and `firefly-bom` (version `1.0.0-SNAPSHOT`)
- **OpenAPI conventions**: Properties `base.package`, `openapi.base.package`, `openapi.model.package`, `openapi.api.package`
- **OpenAPI generation profile** (`generate-openapi`): always active unless `-DskipOpenApiGen`. Uses `spring-boot-maven-plugin` to start/stop a lightweight `OpenApiGenApplication`, then `springdoc-openapi-maven-plugin` to fetch the spec from `http://localhost:18080/v3/api-docs.yaml` and write it to `target/openapi/openapi.yml`
- **Skip-by-default control**: `openapi.gen.skip=true` by default; only `-web` modules set it to `false`

### Service-level POM pattern

```xml
<parent>
    <groupId>com.firefly</groupId>
    <artifactId>firefly-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <relativePath/>
</parent>

<artifactId>core-common-customer-mgmt</artifactId>
<packaging>pom</packaging>
<modules>
    <module>core-common-customer-mgmt-core</module>
    <module>core-common-customer-mgmt-interfaces</module>
    <module>core-common-customer-mgmt-models</module>
    <module>core-common-customer-mgmt-sdk</module>
    <module>core-common-customer-mgmt-web</module>
</modules>

<properties>
    <java.version>25</java.version>
</properties>
```

All services use Java 25 with `maven.compiler.release` set accordingly.

## 4. BOM Usage

`firefly-bom` (`com.firefly:firefly-bom:1.0.0-SNAPSHOT`) is a flat POM that manages versions for every artifact in the platform. It is imported automatically by `firefly-parent`, so individual services never need to import it directly.

**What it manages** (all at `1.0.0-SNAPSHOT`):

- **7 common libraries**: `library-common-core`, `library-common-web`, `library-common-r2dbc`, `library-common-auth`, `library-common-eda`, `library-common-cache`, `library-common-batch`
- **2 domain libraries**: `library-psps`, `library-rails`
- **3 utility libraries**: `library-utils`, `library-commons-validators`, `library-workflow-engine`
- **2 plugin manager libs**: `library-plugin-manager-core`, `library-plugin-manager-api`
- **5 core-banking** service artifacts + **5 matching SDKs**
- **11 core-lending** service artifacts + **11 matching SDKs**
- **15 core-common** service artifacts + **15 matching SDKs**
- **2 core-data** service artifacts + **2 matching SDKs**
- **10 domain** service artifacts + **10 matching SDKs**
- **3 application** artifacts + **1 app SDK** (`bfc-initconfig-sdk`)
- **1 flow SDK**: `firefly-flow-sdk`

**Version strategy**: All artifacts use `1.0.0-SNAPSHOT` during development. The BOM ensures no version conflicts across the platform.

When you add a dependency on another Firefly service's SDK, you omit the `<version>` tag because the BOM manages it:

```xml
<dependency>
    <groupId>com.firefly</groupId>
    <artifactId>core-common-customer-mgmt-sdk</artifactId>
    <!-- version managed by firefly-bom via firefly-parent -->
</dependency>
```

## 5. Module Structure

### Core services (`core-banking-*`, `core-lending-*`, `core-common-*`, `core-data-*`)

| Module | Content | Key Dependencies |
|--------|---------|-----------------|
| `{service}-interfaces` | DTOs, enums, API request/response contracts | `fireflyframework-utils`, `fireflyframework-validators`, `swagger-annotations`, `jackson-databind` |
| `{service}-models` | JPA/R2DBC entities, Spring Data repositories, Flyway migrations | `{service}-interfaces`, `fireflyframework-r2dbc`, `spring-boot-starter-data-r2dbc`, `r2dbc-postgresql`, `flyway-core` |
| `{service}-core` | Service interfaces, service impls, MapStruct mappers | `{service}-interfaces`, `{service}-models`, `fireflyframework-starter-core`, `mapstruct` |
| `{service}-web` | Spring Boot application class, REST controllers, `application.yaml` | `{service}-core`, `{service}-interfaces`, `fireflyframework-web`, `springdoc-openapi-starter-webflux-ui`, `micrometer-registry-prometheus` |
| `{service}-sdk` | Auto-generated API client from OpenAPI spec | `{service}-web` (provided scope), `openapi-generator-maven-plugin`, `okhttp3`, `swagger-annotations` |

### Domain services (`domain-*`)

| Module | Content | Key Dependencies |
|--------|---------|-----------------|
| `{service}-infra` | SDK client factories, `@ConfigurationProperties` for downstream base paths | Core service SDKs (e.g., `core-common-customer-mgmt-sdk`), `fireflyframework-utils` |
| `{service}-interfaces` | Domain-level DTOs and contracts | `{service}-core`, `fireflyframework-utils`, `fireflyframework-validators` |
| `{service}-core` | Commands, handlers, sagas/workflows, domain services (CQRS pattern) | `{service}-infra`, core SDKs, `fireflyframework-starter-domain` |
| `{service}-web` | REST controllers, Spring Boot app, `application.yaml` | `{service}-interfaces`, `fireflyframework-web`, `springdoc-openapi-starter-webflux-ui` |
| `{service}-sdk` | Auto-generated API client from OpenAPI spec | Same pattern as core SDKs |

### Experience services (`exp-*`)

| Module | Content | Key Dependencies |
|--------|---------|-----------------|
| `{service}-interfaces` | Journey-specific DTOs and API contracts | `fireflyframework-utils`, `fireflyframework-validators` |
| `{service}-infra` | SDK client factories for domain services | Domain service SDKs (e.g., `domain-customer-people-sdk`), `fireflyframework-utils` |
| `{service}-core` | Journey orchestration, response shaping, channel logic | `{service}-infra`, domain SDKs, `fireflyframework-starter-application` |
| `{service}-web` | REST controllers, Spring Boot app, `application.yaml` | `{service}-interfaces`, `fireflyframework-web`, `springdoc-openapi-starter-webflux-ui` |
| `{service}-sdk` | Auto-generated API client from OpenAPI spec | Same pattern as domain SDKs |

Note: Experience services consume **only** domain SDKs, never core SDKs directly. They have `-infra` for domain SDK client factories.

### Application services (`app-*`)

| Module | Content | Key Dependencies |
|--------|---------|-----------------|
| `{service}-interfaces` | Lightweight DTOs for gateway-level operations | `fireflyframework-utils` |
| `{service}-core` | Orchestration logic, security configuration | `fireflyframework-utils`, `mapstruct` |
| `{service}-web` | Gateway application, security config, controllers | `{service}-interfaces`, `fireflyframework-starter-application`, `spring-cloud-starter-gateway` |

Note: App services typically do **not** have `-models` or `-sdk` modules. They also do not have `-infra`.

## 6. Inter-Service Communication

### Synchronous: SDK Import

Services communicate synchronously by importing another service's `-sdk` module. The SDK is an auto-generated OpenAPI client using `openapi-generator-maven-plugin` with the `webclient` library (reactive).

**Concrete example** -- `domain-customer-people` calling `core-common-customer-mgmt`:

1. The `-infra` module declares the SDK dependency:
   ```xml
   <dependency>
       <groupId>com.firefly</groupId>
       <artifactId>core-common-customer-mgmt-sdk</artifactId>
   </dependency>
   ```

2. A `@ConfigurationProperties` class maps the base URL:
   ```java
   @ConfigurationProperties(prefix = "api-configuration.common-platform.customer-mgmt")
   public class CustomerMgmtProperties {
       private String basePath;
   }
   ```

3. A `ClientFactory` creates API beans:
   ```java
   @Component
   public class ClientFactory {
       private final ApiClient apiClient;

       public ClientFactory(CustomerMgmtProperties props) {
           this.apiClient = new ApiClient();
           this.apiClient.setBasePath(props.getBasePath());
       }

       @Bean
       public PartiesApi partiesApi() {
           return new PartiesApi(apiClient);
       }
       // ... more API beans
   }
   ```

4. `application.yaml` provides the base path:
   ```yaml
   api-configuration:
     common-platform.customer-mgmt:
       base-path: http://localhost:8082
   ```

5. Environment-specific overrides come from `firefly-config`:
   ```yaml
   # config/dev/application-dev.yaml
   endpoints:
     common:
       core-common-customer-mgmt: https://app-core-common-customer-mgmt.dev.soon.es
   ```

### Asynchronous: Events via Kafka

Domain services use the Firefly EDA (Event-Driven Architecture) library for async communication:

```yaml
firefly:
  eda:
    enabled: true
    default-publisher-type: KAFKA
    default-connection-id: default
    publishers:
      kafka:
        default:
          default-topic: domain-layer
          bootstrap-servers: localhost:9092
```

Domain services also enable CQRS and saga support:

```yaml
firefly:
  cqrs:
    enabled: true
    command:
      timeout: 30s
      metrics-enabled: true
      tracing-enabled: true
    query:
      timeout: 15s
      caching-enabled: true
      cache-ttl: 15m
  saga.performance.enabled: true
  stepevents.enabled: true
```

## 7. Layer Rules

```
+-------------------+
|  channel (web/    |  Browser, Mobile App, API Consumer
|  mobile/API)      |
+-------------------+
         |
         v
+-------------------+       +-------------------+
|  exp-* (BFF per   |       |  app-* (edge/     |
|  user journey)    |       |  gateway/auth)    |
+-------------------+       +-------------------+
         |                           |
         | calls via domain SDK      | calls via domain SDK or routes
         v                           v
+-------------------+
|   domain-* (orch) |  Workflows, Sagas, CQRS Commands/Queries
+-------------------+
         |
         | calls via SDK (sync) or Kafka events (async)
         v
+-------------------+
| core-* (CRUD/biz) |  Entities, Repositories, Business Rules
+-------------------+
         |
         | R2DBC / PostgreSQL
         v
+-------------------+
|    Database        |
+-------------------+
```

**Strict dependency direction -- violations break the architecture**:

| Rule | Rationale |
|------|-----------|
| Core NEVER imports a domain SDK | Core services are self-contained data owners |
| Core NEVER imports another core service SDK | Core services do not orchestrate each other |
| Domain orchestrates core services via their SDKs | Domain is the orchestration layer |
| Domain may import other domain SDKs only for cross-domain workflows | Keep coupling minimal |
| Experience (`exp-*`) consumes ONLY domain SDKs | Experience is a BFF layer, never accesses core directly |
| App orchestrates domain services (and occasionally core for infra concerns) | App is the edge entry point |
| Data services read from core but never write back | Data is a read projection |

**The `-infra` module exists ONLY in domain services**. It houses SDK client factories and configuration properties that wire up downstream core service connections. Core services do not have an `-infra` module because they do not call other services.

## 8. Configuration

### Spring Cloud Config Repository (`firefly-config`)

Centralized configuration lives in `firefly-config` with the following structure:

```
firefly-config/
  config/
    dev/
      application.yaml            # Local development defaults
      application-dev.yaml        # Dev environment endpoints for all services
      domain/
        customer-domain-people-dev.yaml
        customer-domain-kyc-kyb-dev.yaml
        lending-domain-loan-origination-dev.yaml
        ...
    pre/
      application-pre.yaml        # Pre-production endpoints
      domain/
        customer-domain-people-pre.yaml
        ...
    pro/
      application-pro.yaml        # Production endpoints
      domain/
        customer-domain-people-pro.yaml
        ...
```

### Environment Profiles

| Profile | Purpose | Endpoint pattern |
|---------|---------|-----------------|
| `dev` | Development | `https://app-{service-name}.dev.soon.es` |
| `pre` | Pre-production / staging | `https://app-{service-name}.pre.soon.es` |
| `pro` | Production | `https://app-{service-name}.pro.soon.es` |

### Service-Level Configuration (`application.yaml`)

Each service's `-web` module contains an `application.yaml` with:

- `spring.application.name` matching the repo name (e.g., `core-common-customer-mgmt`)
- Database connection via environment variables: `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_SSL_MODE`, `DB_USERNAME`, `DB_PASSWORD`
- R2DBC connection pool settings
- Flyway migrations from `classpath:db/migration`
- SpringDoc OpenAPI scanning the controller package
- Actuator endpoints: `health`, `info`, `prometheus`
- Liveness and readiness probes enabled
- Virtual threads enabled (`spring.threads.virtual.enabled: true`)

Domain services additionally configure `firefly.cqrs`, `firefly.eda`, `firefly.saga`, and `api-configuration` base paths for downstream SDKs.

### Profile-Specific Sections

Services embed multi-profile configuration using Spring's `---` document separator:

```yaml
---
spring:
  config:
    activate:
      on-profile: dev
logging:
  level:
    com.firefly: DEBUG

---
spring:
  config:
    activate:
      on-profile: prod
logging:
  level:
    com.firefly: INFO
springdoc:
  api-docs.enabled: false
  swagger-ui.enabled: false
```

## 9. API Contract Conventions

### OpenAPI Spec Ownership

Each service owns its OpenAPI spec. The spec is auto-generated at build time from the `-web` module's annotated controllers:

1. The `-web` module sets `openapi.gen.skip=false` and `openapi.gen.mainClass` to a lightweight Spring Boot application:
   ```xml
   <properties>
       <openapi.gen.skip>false</openapi.gen.skip>
       <openapi.gen.mainClass>com.firefly.core.customer.web.openapi.OpenApiGenApplication</openapi.gen.mainClass>
   </properties>
   ```

2. During `integration-test` phase, `spring-boot-maven-plugin` starts the app on port 18080, `springdoc-openapi-maven-plugin` fetches the spec from `/v3/api-docs.yaml`, and writes it to `target/openapi/openapi.yml`.

### SDK Generation

The `-sdk` module uses `openapi-generator-maven-plugin` (v7.0.1) to generate a Java client from the spec:

```xml
<configuration>
    <inputSpec>${project.parent.basedir}/${project.parent.artifactId}-web/target/openapi/openapi.yml</inputSpec>
    <generatorName>java</generatorName>
    <library>webclient</library>
    <apiPackage>com.firefly.core.customer.sdk.api</apiPackage>
    <modelPackage>com.firefly.core.customer.sdk.model</modelPackage>
    <invokerPackage>com.firefly.core.customer.sdk.invoker</invokerPackage>
    <configOptions>
        <reactive>true</reactive>
        <returnResponse>true</returnResponse>
        <useTags>true</useTags>
        <dateLibrary>java8-localdatetime</dateLibrary>
    </configOptions>
</configuration>
```

### How Consumers Import SDKs

Consumers add the SDK as a Maven dependency (version managed by the BOM) and create API beans through a `ClientFactory` in their `-infra` module. See section 6 for the full pattern.

### REST API Conventions

Controllers observed in the codebase follow these patterns:

- Base path: `/api/v1/{resource}`
- Fully reactive: all endpoints return `Mono<ResponseEntity<T>>`
- `@Tag` annotation on every controller for OpenAPI grouping
- `@Operation` annotation on every endpoint with `summary` and `description`
- `@Valid` on request bodies
- UUID-based resource identifiers
- Sub-resources under parent: `/api/v1/customers/{partyId}/addresses/{addressId}`
- Swagger UI at `/swagger-ui.html`, API docs at `/v3/api-docs`
- Packages scanned via `springdoc.packages-to-scan`
- Paths filtered via `springdoc.paths-to-match: /api/**`

## 10. Quick Reference

## 11. SDK Model DTO Patterns

Generated SDK model DTOs (from `openapi-generator-maven-plugin`) have patterns that differ from hand-written DTOs:

### Read-Only ID Fields

ID fields (`dateCreated`, `dateUpdated`, and the entity ID) are **read-only** -- set only via the constructor:

```java
// CORRECT -- set ID via constructor
KycVerificationDTO dto = new KycVerificationDTO(null, null, UUID.randomUUID());

// WRONG -- setter does not exist for ID fields
KycVerificationDTO dto = new KycVerificationDTO();
dto.setKycVerificationId(UUID.randomUUID());  // Compilation error
```

### Fluent Setters for Mutable Fields

Mutable fields use fluent setters (return `this`):

```java
SendNotificationCommand command = new SendNotificationCommand()
    .partyId(partyId)
    .notificationType(SendNotificationCommand.NotificationTypeEnum.WELCOME)
    .channels(List.of(SendNotificationCommand.ChannelsEnum.EMAIL));
```

### Inline Enums

Enum fields generate **inner enum classes** within the DTO:

```java
// NOT a standalone enum -- it is an inner class of the DTO
SendNotificationCommand.NotificationTypeEnum.WELCOME
SendNotificationCommand.PriorityEnum.NORMAL
```

### Getter Naming

Getters use the **full field name** from the OpenAPI spec:
- `getKycVerificationId()` (NOT `getId()`)
- `getVerificationDocumentId()` (NOT `getDocumentId()`)
- `getComplianceCaseId()` (NOT `getCaseId()`)

## 12. Module Dependency Rules

### -web MUST depend on -core

When controllers reference service interfaces, commands, or any types from `-core`, the `-web` module must declare the dependency:

```xml
<!-- {service}-web/pom.xml -->
<dependency>
    <groupId>com.firefly</groupId>
    <artifactId>{service}-core</artifactId>
</dependency>
```

### -interfaces MUST NOT depend on -core

This creates an inverted/circular dependency. Correct direction:

```
-core → -interfaces (core uses DTOs from interfaces)
-web  → -core       (web uses services from core)
-web  → -interfaces (web uses DTOs from interfaces)
```

### NotImplementedException Requires fireflyframework-web

If a `-core` module needs `NotImplementedException` (from `org.fireflyframework.web.error.exceptions`), it must add `fireflyframework-web` as a dependency. The starters `fireflyframework-starter-core` and `fireflyframework-starter-domain` do NOT include it transitively.

### StepStatus Enum Values

The `StepStatus` enum has values: `PENDING`, `RUNNING`, `DONE`, `FAILED`, `SKIPPED`, `TIMED_OUT`, `RETRYING`. `StepStatus.COMPLETED` does **not** exist -- use `StepStatus.DONE`.

## 13. ClientFactory Conventions

In domain services, the `-infra` module wires SDK clients using `ClientFactory` beans:

```java
// CORRECT -- @Component, not @Configuration
@Component
public class KycbClientFactory {
    private final ApiClient apiClient;

    public KycbClientFactory(KycbMgmtProperties props) {
        this.apiClient = new ApiClient();
        this.apiClient.setBasePath(props.getBasePath());
    }

    @Bean
    public KycVerificationApi kycVerificationApi() {
        return new KycVerificationApi(apiClient);
    }
}
```

**Key conventions:**
- Use `@Component` (not `@Configuration`) -- this is a banking platform convention
- Use `@ConfigurationProperties` without `@Configuration` on the properties class (when `@ConfigurationPropertiesScan` is on the application class)
- One `ClientFactory` per downstream service
- One `@Bean` method per API class from the SDK

## 14. Cross-Layer Reflexive Property

When generating code for an upper-layer service (domain/app) that calls a lower-layer service (core), if the lower-layer method does not exist, you MUST create it:

1. Create the endpoint in the core service's `-web` controller
2. Create the service interface and implementation in `-core`
3. Rebuild the core service's SDK
4. Then implement the upper-layer call

**Never** leave upper-layer methods returning static/mock data or empty results. If the core service is not yet available, define a port interface with a stub adapter that throws `NotImplementedException`.

## 15. Javadoc and README Documentation Standards

### Javadoc

All public classes and methods must include Javadoc:

```java
/**
 * Orchestrates KYC verification lifecycle including identity checks,
 * document collection, and compliance case management.
 */
@Service
public class KycServiceImpl implements KycService {

    /**
     * Initiates identity verification for the given compliance case.
     *
     * @param caseId the compliance case identifier
     * @return a {@link SagaResult} with verification outcome
     */
    public Mono<SagaResult> verify(UUID caseId) { ... }
}
```

### README.md

Every microservice MUST have a `README.md` at the project root with:

1. **Service description** -- purpose and role in the platform
2. **Architecture** -- tier, dependencies, how it fits in the broader platform
3. **Module structure** -- table of modules and their purposes
4. **API reference** -- main endpoints with HTTP method, path, and description
5. **Configuration** -- required environment variables and properties
6. **Getting started** -- build and run instructions
7. **Dependencies** -- upstream (calls) and downstream (called by) services

## 16. PII Logging Rules

Never log personally identifiable information. Use resource identifiers instead:

```java
// WRONG
log.info("Processing customer: name={}, email={}", name, email);

// CORRECT
log.info("Processing customer: partyId={}", partyId);
```

## 17. Quick Reference

| Do | Don't |
|----|-------|
| Extend `firefly-parent` as your service's parent POM | Extend `fireflyframework-parent` directly -- you lose BOM imports and OpenAPI profile |
| Use the five-module structure for core services: `-interfaces`, `-models`, `-core`, `-web`, `-sdk` | Put entities in `-core` or controllers in `-interfaces` |
| Use the domain module structure with `-infra` for SDK client factories | Put `ClientFactory` or `@ConfigurationProperties` in `-core` |
| Omit `<version>` for any `com.firefly` dependency -- the BOM manages it | Hard-code versions for platform artifacts |
| Name packages `com.firefly.{tier}.{domain-short}.{layer}` | Use `com.firefly.{full-repo-name}` -- it creates unnecessarily long packages |
| Place DTOs and enums in `-interfaces`, entities in `-models` | Mix DTOs and entities in the same module |
| Generate SDKs from OpenAPI specs using the `-web` module's controllers | Write SDK clients by hand |
| Configure downstream service base paths via `api-configuration.*` in `application.yaml` | Hard-code URLs in Java classes |
| Use `fireflyframework-starter-core` for core services, `fireflyframework-starter-domain` for domain services, `fireflyframework-starter-application` for experience (`exp-*`) and app services | Mix starters across tiers |
| Use CQRS commands/handlers/sagas in domain `-core` modules | Use plain CRUD services in domain modules -- that belongs in core |
| Set `openapi.gen.skip=false` only in the `-web` module | Enable OpenAPI generation in non-web modules |
| Use R2DBC with PostgreSQL for data access in `-models` | Use blocking JDBC -- the platform is fully reactive (WebFlux) |
| Communicate async between domain services via Kafka (Firefly EDA) | Call domain SDKs synchronously for fire-and-forget operations |
| Use environment variables for database credentials (`DB_HOST`, etc.) | Commit credentials in `application.yaml` |
| Place Flyway migrations in `-models/src/main/resources/db/migration` | Place migrations anywhere else |
| Set `spring.application.name` to the repo name (e.g., `core-common-customer-mgmt`) | Use a different naming convention for the Spring app name |
| Use `spring.threads.virtual.enabled: true` | Use thread pools for request handling |
| Expose actuator endpoints: `health`, `info`, `prometheus` | Expose all actuator endpoints in production |
| Disable Swagger UI in `prod` profile | Leave API docs publicly accessible in production |
| Use `@Component` on `ClientFactory` classes | Use `@Configuration` on `ClientFactory` classes |
| Use `@ConfigurationProperties` alone (with `@ConfigurationPropertiesScan`) | Combine `@Configuration` with `@ConfigurationProperties` |
| Use `@Valid @RequestBody` on controller POST/PUT bodies | Omit `@Valid` on `@RequestBody` |
| Use `scanBasePackages = "org.fireflyframework.web"` | Use `scanBasePackages = "com.firefly.common.web"` |
| Use `springdoc.packages-to-scan: *.web.controllers` (plural) | Use `*.web.controller` (singular) |
| Use `health.show-details: when-authorized` | Use `health.show-details: always` |
| Use profile names `dev`, `pre`, `pro` | Use `testing`, `staging`, `local` |
| Use `StepStatus.DONE` | Use `StepStatus.COMPLETED` (does not exist) |
| Set SDK DTO IDs via constructor: `new DTO(null, null, id)` | Use `dto.setId()` -- SDK IDs are read-only |
| Implement full call chain to lower-layer services | Return static/mock data from service methods |
| Log only resource IDs (`partyId`, `paymentId`) | Log PII (names, emails, IBANs, phone numbers) |
| Include Javadoc on all public classes and methods | Skip documentation entirely |
| Include `README.md` in every microservice root | Leave services undocumented |
