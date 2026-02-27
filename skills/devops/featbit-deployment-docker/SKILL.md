---
name: featbit-deployment-docker
description: Expert guidance for deploying FeatBit with Docker Compose across three tiers - Standalone (PostgreSQL only), Standard (PostgreSQL/MongoDB + Redis), and Professional (+ ClickHouse + Kafka). Use when user mentions "docker-compose", "deploy with Docker", "standalone vs standard vs pro", works with docker-compose.yml files, or asks about container configuration, environment variables, or production Docker setup.
appliesTo:
  - "**/docker-compose.yml"
  - "**/docker-compose.yaml"
  - "**/Dockerfile"
license: MIT
metadata:
  version: 2.0.0
  category: deployment
---

# FeatBit Docker Compose Deployment

Expert guidance for deploying FeatBit with Docker Compose. This skill provides deployment instructions for all three tiers with links to detailed configuration files.

## Core Concepts

FeatBit offers three deployment architectures optimized for different scales:

| Tier | Database | Cache | Message Queue | Analytics | Best For |
|------|----------|-------|---------------|-----------|----------|
| **Standalone** | PostgreSQL | None | PostgreSQL | PostgreSQL | Low to moderate concurrent connections, moderate API calls, limited event volume |
| **Standard** | PostgreSQL/MongoDB | Redis | Redis | MongoDB | Moderate to high concurrent connections & API calls, moderate event volume |
| **Professional** | PostgreSQL/MongoDB | Redis | Kafka | ClickHouse | Moderate to high concurrent connections & API calls, high event volume |

**Quick Selection Guide**:
- **Standalone**: Minimal setup, single server, handles low to moderate concurrent WebSocket connections
- **Standard**: Production-ready with caching, handles high concurrent connections with moderate event volume
- **Professional**: Maximum scale for high connections AND high event volume, requires DevOps expertise

**Note**: Traffic includes both concurrent WebSocket connections from frontend clients and API calls to clients - these have different scales.

📚 **Architecture Details**: https://docs.featbit.co/installation/deployment-options

## Deployment Guides

**⚠️ Important**: Before starting any deployment, clone the FeatBit repository as it contains required scripts:

```bash
git clone https://github.com/featbit/featbit.git
cd featbit
```

These scripts are accessed during Docker execution.

**📍 Access Information** (applies to all deployment tiers):
- **URL**: http://localhost:8081
- **Default Login**: test@featbit.com / 123456

### Standalone Deployment

**Quick Start**:
```bash
# Clone repository (if not already done)
git clone https://github.com/featbit/featbit.git
cd featbit

# Start services using the included docker-compose.yml
docker compose up -d
```

**Prerequisites**: Docker 20.10+, Docker Compose 2.0+, 2GB RAM

📄 **Complete Guide**: [references/standalone-configuration.md](references/standalone-configuration.md)
- Full docker-compose.yml with health checks
- Step-by-step setup instructions
- Using managed PostgreSQL
- Resource requirements
- Limitations and when to upgrade

### Standard Deployment

**Two Options Available**:

**Option A: PostgreSQL + Redis**
- Best for teams familiar with PostgreSQL
- Simpler than MongoDB option

```bash
# Clone repository (if not already done)
git clone https://github.com/featbit/featbit.git
cd featbit

# Start services with PostgreSQL Standard configuration
docker compose -f docker-compose-standard.yml up -d
```

**Option B: MongoDB + Redis** 
- Better for document-oriented workloads
- Includes initialization script

```bash
# Clone repository (if not already done)
git clone https://github.com/featbit/featbit.git
cd featbit

# Start services with MongoDB configuration
docker compose -f docker-compose-mongodb.yml up -d
```

**Prerequisites**: Docker 20.10+, Docker Compose 2.0+, 4GB RAM (8GB recommended)

📄 **Complete Guide**: [references/standard-configuration.md](references/standard-configuration.md)
- Full configurations for both PostgreSQL and MongoDB options
- Production setup with managed services
- Resource requirements
- When to choose Standard vs Professional

### Professional Deployment

**Enterprise-Scale Architecture**:
- Kafka for high-throughput messaging
- ClickHouse for advanced analytics
- Horizontal scalability
- Handles millions of events per day

```bash
# Clone repository (if not already done)
git clone https://github.com/featbit/featbit.git
cd featbit

# Start all services (Docker Compose handles startup order via depends_on)
docker compose -f docker-compose-pro.yml up -d
```

**Prerequisites**: Docker 20.10+, Docker Compose 2.0+, 8GB+ RAM, 4+ CPU cores

⚠️ **Complexity Warning**: Requires significant DevOps expertise

📄 **Complete Guide**: [references/professional-configuration.md](references/professional-configuration.md)
- Full docker-compose.yml with all services
- Infrastructure startup sequence
- Horizontal scaling configuration
- Using managed services (AWS MSK, ClickHouse Cloud)
- Performance tuning
- Monitoring setup

## Reference Guides

### Environment Variables

Complete reference for all configuration options:

📄 **[references/environment-variables.md](references/environment-variables.md)**
- Provider configuration (DbProvider, MqProvider, CacheProvider)
- Database connection strings (PostgreSQL, MongoDB, Redis, Kafka, ClickHouse)
- UI configuration (API_URL, EVALUATION_URL)
- Service-specific variables
- Using .env files for secrets
- Environment-specific configurations
- OpenTelemetry configuration

### Troubleshooting

Common issues and solutions:

📄 **[references/troubleshooting.md](references/troubleshooting.md)**
- Port conflicts
- UI connection issues  
- Database connection failures
- Service startup problems
- WebSocket connection failures
- Redis and MongoDB issues
- Kafka and ClickHouse troubleshooting
- Performance problems
- Emergency recovery procedures
- Debugging tips

## Quick Commands

### Service Management

```bash
# Start all services (Standalone)
docker compose up -d

# Start all services (Standard PostgreSQL)
docker compose -f docker-compose-standard.yml up -d

# Start all services (Standard MongoDB)
docker compose -f docker-compose-mongodb.yml up -d

# Start all services (Professional)
docker compose -f docker-compose-pro.yml up -d

# Stop all services (use same -f flag as used for starting)
docker compose down
# or for non-default configs:
docker compose -f docker-compose-standard.yml down

# View status
docker compose ps

# View logs
docker compose logs -f

# View specific service logs
docker compose logs -f api-server

# Restart service
docker compose restart api-server

# Scale service (Professional tier)
docker compose up -d --scale evaluation-server=3
```

### Maintenance

```bash
# Update to latest images
docker compose pull
docker compose up -d

# Clean up
docker system prune -a --volumes

# Backup PostgreSQL
docker compose exec postgresql pg_dump -U postgres featbit > backup.sql

# Backup MongoDB
docker compose exec mongodb mongodump --out /backup --db featbit
```

## When to Choose Each Tier

### Choose Standalone If:
✅ **Production or non-production** with low to moderate traffic:  
   - Low to moderate concurrent WebSocket connections from frontend clients
   - Moderate API call volume to clients
✅ Low event volume (feature flag usage events & custom events)
✅ Simple single-server deployment preferred
✅ Cost-effective solution for small-scale production use
✅ Quick evaluation of FeatBit

❌ Not Recommended For:
- Very high concurrent WebSocket connections
- High event volume requiring event streaming
- Need for caching layer to improve performance

### Choose Standard If:
✅ **Very high concurrent WebSocket connections & API calls**  
✅ Need caching layer (Redis) for improved performance
✅ Moderate event volume (feature flag usage events & custom events)
✅ Production environment requiring reliability
✅ Good balance of complexity vs features

❌ Consider Professional If:
- Very high event volume requiring Kafka event streaming
- Require real-time analytics at scale with ClickHouse
- Need horizontal scalability for data analytics

### Choose Professional If:
✅ **Very high concurrent WebSocket connections & API calls**  
✅ **Very high event volume** (feature flag usage events & custom events)
✅ Need Kafka for high-throughput event streaming
✅ Need ClickHouse for real-time analytics at scale
✅ Have dedicated DevOps resources
✅ Budget for infrastructure

❌ Consider Standard If:
- Limited DevOps expertise
- Moderate event volume (Redis sufficient for message queue)
- Cost-sensitive

## Official Resources

### Documentation
- **Installation Guide**: https://docs.featbit.co/installation/docker-compose
- **Deployment Options**: https://docs.featbit.co/installation/deployment-options
- **Infrastructure Components**: https://docs.featbit.co/tech-stack/infrastructure-components
- **FAQ**: https://docs.featbit.co/installation/faq

### Source Code & Configurations
- **Main Repository**: https://github.com/featbit/featbit
- **Standalone**: https://github.com/featbit/featbit/blob/main/docker-compose.yml
- **Standard (MongoDB)**: https://github.com/featbit/featbit/blob/main/docker-compose-mongodb.yml
- **Professional**: https://github.com/featbit/featbit/blob/main/docker-compose-pro.yml
- **PostgreSQL Init**: https://github.com/featbit/featbit/tree/main/infra/postgresql/docker-entrypoint-initdb.d
- **MongoDB Init**: https://github.com/featbit/featbit/blob/main/infra/mongodb/docker-entrypoint-initdb.d

### Docker Images
- **Docker Hub**: https://hub.docker.com/u/featbit
- Images: featbit-ui, featbit-api-server, featbit-evaluation-server, featbit-data-analytics-server

### Alternative Deployments
- **Kubernetes**: https://github.com/featbit/featbit-charts
- **Azure Container Apps**: https://github.com/featbit/featbit-aspire
- **Terraform (AWS)**: https://github.com/featbit/featbit-terraform-aws

## Related Skills

- **featbit-deployment-kubernetes**: Kubernetes/Helm deployments with auto-scaling
- **featbit-deployment**: Overview of all deployment options and architecture
- **featbit-getting-started**: Initial setup, creating feature flags, SDK integration
- **featbit-opentelemetry**: Observability and monitoring setup
