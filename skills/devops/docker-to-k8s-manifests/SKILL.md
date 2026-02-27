---
name: docker-to-k8s-manifests
description: Automatically generate optimized Kubernetes deployment manifests from Dockerfile and docker-compose configurations with proper resource limits and health checks.
version: 1.0.0
triggers:
  - "Convert my Dockerfile to Kubernetes manifests"
  - "Generate k8s deployment from docker-compose.yml"
  - "Create Kubernetes resources for this Docker application"
  - "Optimize my Docker setup for Kubernetes deployment"
  - "Add health checks and resource limits to my k8s deployment"
  - "Migrate Docker Compose to Kubernetes manifests"
---

# docker-to-k8s-manifests

Converts Dockerfile and docker-compose.yml files into production-ready Kubernetes deployment manifests.

## Usage

### CLI

```bash
# From a Dockerfile
docker-to-k8s --dockerfile ./Dockerfile --output ./k8s/

# From a docker-compose.yml
docker-to-k8s --compose ./docker-compose.yml --output ./k8s/

# With custom app name
docker-to-k8s --dockerfile ./Dockerfile --name my-app --output ./k8s/
```

### As a library

```javascript
const { DockerfileParser } = require('./scripts/dockerfile-parser');
const { ComposeParser } = require('./scripts/compose-parser');
const { K8sGenerator } = require('./scripts/k8s-generator');

// Parse Dockerfile
const parser = new DockerfileParser();
const config = parser.parseFile('./Dockerfile');

// Generate K8s manifests
const generator = new K8sGenerator();
const manifests = generator.generate(config, { name: 'my-app' });
```

## Features

- Parses Dockerfile instructions and extracts metadata
- Parses docker-compose.yml multi-service configurations
- Generates Deployment, Service, and Ingress manifests
- Estimates resource limits based on application type
- Configures liveness, readiness, and startup probes
- Validates output against Kubernetes schemas
- Applies security best practices (non-root, read-only FS, dropped capabilities)

## Output

Generates the following Kubernetes resources:

- **Deployment** — with resource limits, probes, security context
- **Service** — ClusterIP with correct port mappings
- **Ingress** — optional, for HTTP services
- **ConfigMap** — for environment variables
- **PersistentVolumeClaim** — for volume mounts
