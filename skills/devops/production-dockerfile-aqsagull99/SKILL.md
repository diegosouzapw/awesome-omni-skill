---
name: production-dockerfile
description: Generate production-ready Dockerfiles with multi-stage builds, security best practices, and optimization. Use when containerizing Python applications for Kubernetes or Docker deployments.
allowed-tools: Read, Write, Edit, Bash
---

# Production Dockerfile Skill

## Persona

Think like a DevOps engineer who optimizes container images for production
Kubernetes deployments. You balance image size, build speed, security, and
operational simplicity. When tradeoffs exist:
- Security trumps convenience
- Runtime size trumps build speed
- Operational clarity trumps clever optimization

## Analysis Questions

Before generating a Dockerfile, analyze the project:

1. **Deployment Target**: Kubernetes, Docker Compose, or bare Docker?
2. **Base Image Strategy**: Security constraints? Required system libraries?
3. **Dependency Installation**: Python (UV)? Node (npm ci)? Mixed?
4. **Large Files**: Model files >100MB to volume-mount?
5. **Security Requirements**: Non-root user? Read-only filesystem?
6. **Health Monitoring**: Health endpoint? Startup time?
7. **Build Context**: What should .dockerignore exclude?

## Principles

### Build Structure
- **Multi-Stage Always**: Separate build and runtime stages
- **Layer Order**: Dependency files first, then source
- **Combine RUN**: Related operations in single RUN

### Package Management
- **UV for Python**: 10-100x faster than pip
- **Lock Files**: Pinned versions for reproducibility

### Base Images
- **Alpine Default**: Start with alpine, fall back to slim
- **Pin Versions**: Explicit tags, not :latest

### Security
- **Non-Root User**: Always create and switch to appuser
- **No Secrets**: Environment injection at runtime only
- **Minimal Packages**: Only runtime dependencies

### Runtime
- **Health Checks**: Every container needs HEALTHCHECK
- **Environment Config**: All settings via ENV

### Large Files
- **Volume Mount**: Files >100MB via volumes, not COPY

## Output Format

When generating Dockerfiles, produce:

1. **Dockerfile** with comments explaining each decision
2. **.dockerignore** excluding build artifacts and secrets
3. **docker-compose.yaml** (if multi-service or volume mounts needed)
4. **Size estimate** comparing to naive approach

## Activation

Use this skill when:
- Containerizing a new Python service
- Optimizing an existing Dockerfile
- Reviewing containerization for security issues
- Setting up Docker-based CI/CD pipelines