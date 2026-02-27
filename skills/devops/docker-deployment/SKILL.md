---
name: docker-deployment
description: Docker containerization and deployment for Java/Spring Boot applications. Multi-stage builds, docker-compose, health checks, and CI/CD with GitHub Actions.
version: "1.0.0"
author: GazApps
tags: [docker, deployment, container, docker-compose, ci-cd, github-actions, devops]
dependencies: [java-fundamentals]
compatibility: [antigravity, claude-code, gemini-cli]
---

# Docker Deployment for Java/Spring Boot Applications

## When to Use This Skill

**Use when:**
- Containerizing a Java or Spring Boot application for consistent deployment
- Setting up local development environments with Docker Compose
- Building CI/CD pipelines that produce Docker images
- Deploying to container orchestration platforms (Kubernetes, ECS, Cloud Run)
- Ensuring reproducible builds across development, staging, and production
- Managing multi-service architectures locally (app + database + cache)

**Do NOT use when:**
- Building serverless functions (use cloud-native packaging instead)
- The application is a CLI tool or batch job that does not need containerization
- Deploying to a traditional VM-based environment with no container runtime
- The project is a library/SDK (publish to Maven Central, not a container registry)

---

## Multi-Stage Dockerfile for Java

Always use multi-stage builds to separate the build environment from the runtime. This produces smaller, more secure images.

### Standard Pattern

```dockerfile
# Stage 1: Build
FROM eclipse-temurin:21-jdk AS builder
WORKDIR /app
COPY pom.xml .
COPY .mvn/ .mvn/
COPY mvnw .
RUN chmod +x mvnw && ./mvnw dependency:go-offline -B
COPY src/ src/
RUN ./mvnw package -DskipTests -B

# Stage 2: Runtime
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY --from=builder /app/target/*.jar app.jar
RUN chown -R appuser:appgroup /app
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget -qO- http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Key Principles

1. **Use JRE for runtime, not JDK** -- the JDK includes compilers and tools not needed at runtime
2. **Use Alpine-based images** when possible for smaller footprint (~200MB vs ~400MB)
3. **Run as non-root user** -- never run Java applications as root in containers
4. **Copy dependency resolution step first** to leverage Docker layer caching
5. **Use `.dockerignore`** to exclude unnecessary files from the build context

---

## JVM Options for Containers

Modern JVMs are container-aware (Java 10+), but explicit tuning is still recommended.

### Essential Flags

```bash
# Memory: use percentage-based limits so the JVM respects container memory limits
-XX:MaxRAMPercentage=75.0
-XX:InitialRAMPercentage=50.0
-XX:MinRAMPercentage=25.0

# GC selection
-XX:+UseG1GC                    # General-purpose, good for most workloads
-XX:+UseZGC                     # Low-latency, good for large heaps (Java 17+)
-XX:+UseShenandoahGC            # Alternative low-latency GC

# Container awareness (enabled by default in Java 10+, but explicit is safer)
-XX:+UseContainerSupport

# Diagnostics
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/heapdump.hprof
-XX:+ExitOnOutOfMemoryError
```

### Memory Sizing Guidelines

| Container Memory Limit | MaxRAMPercentage | Reasoning                                     |
|------------------------|------------------|------------------------------------------------|
| 256 MB                 | 75%              | Leaves ~64 MB for OS, native memory, threads   |
| 512 MB                 | 75%              | Leaves ~128 MB for overhead                    |
| 1 GB                   | 75%              | Standard for most microservices                 |
| 2 GB+                  | 80%              | Larger containers can use a higher percentage   |

### Common Pitfall: OOMKilled

If your container is killed by the OOM killer (exit code 137), the JVM heap is exceeding the container memory limit. Solutions:
- Lower `MaxRAMPercentage` (e.g., from 75% to 60%)
- Increase the container memory limit
- Check for native memory leaks (Netty buffers, JNI, thread stacks)
- Use `-XX:NativeMemoryTracking=summary` for diagnostics

---

## Docker Compose for Local Development

Use Docker Compose to run your application alongside its dependencies.

### Structure

```
project/
  docker-compose.yml          # Base configuration (production-like)
  docker-compose.dev.yml      # Dev overrides (debug, live reload)
  docker-compose.test.yml     # Test overrides (test DB, mocks)
  .env                        # Environment variables (not committed)
  .env.example                # Template for environment variables (committed)
```

### Override Pattern

```bash
# Development
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Testing
docker compose -f docker-compose.yml -f docker-compose.test.yml up

# Production (base only)
docker compose up -d
```

### Service Dependencies and Health Checks

Always use `depends_on` with `condition: service_healthy` so your application starts only after its dependencies are ready:

```yaml
services:
  app:
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
```

---

## Health Checks

### Spring Boot Actuator Health Endpoint

Add the Actuator dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Configure in `application.yml`:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    health:
      show-details: when-authorized
      probes:
        enabled: true    # Enables /actuator/health/liveness and /readiness
  health:
    livenessstate:
      enabled: true
    readinessstate:
      enabled: true
```

### Dockerfile Health Check

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD wget -qO- http://localhost:8080/actuator/health/liveness || exit 1
```

### Docker Compose Health Check

```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8080/actuator/health/liveness"]
      interval: 30s
      timeout: 3s
      start_period: 60s
      retries: 3
```

**Note:** Use `start_period` to give the JVM time to start. Spring Boot applications typically need 15-60 seconds to initialize.

---

## Environment Variables and Secrets

### Best Practices

1. **Never bake secrets into images** -- use environment variables or mounted secrets
2. **Use `.env` files for local development** -- never commit `.env` to version control
3. **Use Docker secrets or external secret managers in production** (Vault, AWS Secrets Manager, etc.)

### Spring Boot Environment Variable Binding

Spring Boot automatically maps environment variables to properties using relaxed binding:

```bash
# These are equivalent:
SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/myapp
spring.datasource.url=jdbc:postgresql://db:5432/myapp
```

### Docker Compose env_file

```yaml
services:
  app:
    env_file:
      - .env
    environment:
      # Override specific values or add non-secret config
      - SPRING_PROFILES_ACTIVE=docker
      - SERVER_PORT=8080
```

### .env.example Template

```bash
# Database
POSTGRES_DB=myapp
POSTGRES_USER=myapp
POSTGRES_PASSWORD=changeme

# Application
SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/myapp
SPRING_DATASOURCE_USERNAME=myapp
SPRING_DATASOURCE_PASSWORD=changeme

# Redis
SPRING_DATA_REDIS_HOST=redis
SPRING_DATA_REDIS_PORT=6379

# JWT / Security
JWT_SECRET=your-256-bit-secret-here
```

---

## .dockerignore

A proper `.dockerignore` reduces build context size and prevents sensitive files from being included:

```
# Build artifacts
target/
build/
*.jar
*.war
!target/*.jar

# IDE files
.idea/
*.iml
.vscode/
.project
.classpath
.settings/

# Version control
.git/
.gitignore

# Docker files (not needed in context)
docker-compose*.yml
Dockerfile*
.dockerignore

# Documentation
*.md
LICENSE
docs/

# Environment and secrets
.env
.env.*
!.env.example

# OS files
.DS_Store
Thumbs.db

# CI/CD
.github/
.gitlab-ci.yml
Jenkinsfile

# Test reports
surefire-reports/
failsafe-reports/
```

---

## Image Optimization

### Layer Caching Strategy

Order Dockerfile instructions from least to most frequently changing:

```dockerfile
# 1. Base image (rarely changes)
FROM eclipse-temurin:21-jre-alpine

# 2. System dependencies (rarely changes)
RUN apk add --no-cache wget

# 3. Application user (rarely changes)
RUN addgroup -S app && adduser -S app -G app

# 4. Dependencies (changes when pom.xml changes)
COPY --from=builder /app/dependencies/ ./
COPY --from=builder /app/spring-boot-loader/ ./
COPY --from=builder /app/snapshot-dependencies/ ./

# 5. Application code (changes most frequently)
COPY --from=builder /app/application/ ./
```

### Spring Boot Layered JAR

Spring Boot 2.3+ supports layered JARs that split the fat JAR into layers for better Docker caching:

```bash
# Enable layered JAR in pom.xml (Spring Boot 3.x -- enabled by default)
# Extract layers after building
java -Djarmode=layertools -jar app.jar extract --destination extracted

# Layers (from least to most frequently changing):
# 1. dependencies          - third-party libraries
# 2. spring-boot-loader    - Spring Boot loader classes
# 3. snapshot-dependencies  - SNAPSHOT dependencies
# 4. application           - your compiled code and resources
```

### Image Size Comparison

| Base Image                    | Approximate Size |
|-------------------------------|------------------|
| eclipse-temurin:21-jdk        | ~460 MB          |
| eclipse-temurin:21-jre        | ~280 MB          |
| eclipse-temurin:21-jre-alpine | ~180 MB          |
| Custom JRE (jlink)            | ~100 MB          |

### Custom JRE with jlink

For maximum optimization, create a custom JRE with only the modules your application needs:

```dockerfile
FROM eclipse-temurin:21-jdk-alpine AS jre-builder
RUN jlink \
    --add-modules java.base,java.logging,java.sql,java.naming,java.management,java.instrument,java.desktop,java.security.jgss,jdk.unsupported \
    --strip-debug \
    --no-man-pages \
    --no-header-files \
    --compress=zip-6 \
    --output /custom-jre
```

---

## GitHub Actions CI/CD

### Pipeline Structure

```
Push to main
  --> Run tests
  --> Build Docker image
  --> Push to container registry (GHCR)
  --> (optional) Deploy to staging
```

### Key Features

- **Layer caching** with `docker/build-push-action` and GitHub Actions cache
- **Multi-platform builds** with `docker/setup-buildx-action`
- **Semantic versioning** for image tags
- **Security scanning** with Trivy or Snyk

### Image Tagging Strategy

```yaml
tags: |
  ghcr.io/${{ github.repository }}:latest
  ghcr.io/${{ github.repository }}:${{ github.sha }}
  ghcr.io/${{ github.repository }}:${{ github.ref_name }}
```

For production, prefer immutable tags (`sha` or semver) over `latest`.

---

## Container Registries

### GitHub Container Registry (GHCR)

```bash
# Login
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Tag and push
docker tag myapp:latest ghcr.io/OWNER/myapp:latest
docker push ghcr.io/OWNER/myapp:latest
```

### Docker Hub

```bash
docker login -u USERNAME
docker tag myapp:latest USERNAME/myapp:latest
docker push USERNAME/myapp:latest
```

### Amazon ECR

```bash
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin ACCOUNT.dkr.ecr.us-east-1.amazonaws.com
docker tag myapp:latest ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
docker push ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
```

---

## Spring Boot Docker Best Practices

### 1. Use Layered JARs

See `examples/Dockerfile.layered` for the complete example. Layered JARs dramatically improve rebuild times because dependency layers are cached.

### 2. Graceful Shutdown

```yaml
# application.yml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

In the Dockerfile or entrypoint, ensure signals are forwarded to the JVM:

```dockerfile
# Use exec form (not shell form) so PID 1 is the JVM
ENTRYPOINT ["java", "-jar", "app.jar"]

# Or use tini as init process
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--", "java", "-jar", "app.jar"]
```

### 3. Spring Profiles for Docker

```yaml
# application-docker.yml
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:postgres}:${DB_PORT:5432}/${DB_NAME:myapp}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
  data:
    redis:
      host: ${REDIS_HOST:redis}
      port: ${REDIS_PORT:6379}

server:
  port: ${SERVER_PORT:8080}

logging:
  pattern:
    console: "%d{ISO8601} [%thread] %-5level %logger{36} - %msg%n"
```

### 4. Log to stdout/stderr

Never log to files inside containers. Use console appenders and let the container runtime collect logs:

```yaml
logging:
  file:
    name:   # Explicitly empty -- no file logging
  level:
    root: INFO
    com.myapp: DEBUG
```

### 5. Externalize Configuration

Use environment variables or mounted config files. Never bake environment-specific configuration into the image.

---

## Production Deployment Considerations

### Resource Limits

Always set memory and CPU limits:

```yaml
# docker-compose.yml
services:
  app:
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: "1.0"
        reservations:
          memory: 512M
          cpus: "0.5"
```

### Logging

- Use JSON-formatted logs for structured log aggregation
- Ship logs to a centralized system (ELK, Loki, CloudWatch)
- Set appropriate log levels (INFO for production, DEBUG for troubleshooting)

### Security

- Scan images for vulnerabilities (`docker scout`, `trivy`, `snyk`)
- Use read-only file systems where possible (`read_only: true`)
- Drop all capabilities and add only what is needed
- Pin base image versions with digest hashes for reproducibility
- Never store secrets in image layers

### Networking

- Use Docker networks to isolate services
- Only expose ports that need to be publicly accessible
- Use internal DNS names for service-to-service communication

### Monitoring

- Expose Prometheus metrics via Actuator (`/actuator/prometheus`)
- Use liveness and readiness probes
- Monitor container resource usage (memory, CPU, restarts)
- Set up alerts for container OOM kills and restart loops

---

## Code Quality Checklist

Before deploying a containerized Java application, verify:

- [ ] **Multi-stage build**: Build and runtime stages are separate
- [ ] **Non-root user**: Application runs as a non-root user
- [ ] **Health check**: Dockerfile or Compose includes a health check
- [ ] **JVM tuning**: `MaxRAMPercentage` is set appropriately
- [ ] **Container memory limit**: `deploy.resources.limits.memory` is configured
- [ ] **No secrets in image**: All secrets come from environment variables or secret managers
- [ ] **.dockerignore**: Build context excludes unnecessary files
- [ ] **Graceful shutdown**: Spring Boot `server.shutdown=graceful` is enabled
- [ ] **Log to stdout**: No file-based logging inside the container
- [ ] **Image tagged**: Using immutable tags (SHA or semver), not just `latest`
- [ ] **Vulnerability scan**: Image has been scanned for known CVEs
- [ ] **Layered JAR**: Spring Boot layered JAR is used for optimal caching
- [ ] **Base image pinned**: Using a specific version tag, not `latest`
- [ ] **Entrypoint uses exec form**: Ensures proper signal handling
- [ ] **CI/CD pipeline**: Automated build, test, scan, and push workflow exists
