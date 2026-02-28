---
name: "cloud-native-todo-deployer"
description: "A Claude Code skill to containerize a full-stack Todo app, create Docker images, generate Helm charts, and deploy the app on a local Kubernetes cluster (Minikube) using AI-assisted DevOps tools (Gordon, kubectl-ai, Kagent). Fully spec-driven, no manual coding required."
---

# Cloud-Native Todo Deployer Skill

A comprehensive skill for containerizing full-stack Todo applications and deploying them to Kubernetes using AI-assisted DevOps tools. This skill automates the entire process from containerization to deployment with no manual coding required.

## When to Use This Skill

Use this skill when you need to:
- Containerize a full-stack Todo application (frontend + backend)
- Create production-ready Docker images
- Generate Helm charts for Kubernetes deployment
- Deploy to local Kubernetes (Minikube) or cloud clusters
- Use AI-assisted DevOps tools (Gordon, kubectl-ai, Kagent)
- Implement spec-driven deployment processes

## Prerequisites

Before using this skill, ensure you have:
- Docker installed with Kubernetes enabled OR Minikube
- Helm 3.x installed
- kubectl installed
- Access to the frontend and backend source code
- (Optional) Access to AI tools: Gordon, kubectl-ai, Kagent

## Inputs

- `frontend_path`: Local path to the frontend Todo app source code
- `backend_path`: Local path to the backend Todo app source code
- `docker_registry`: Docker registry to push images (optional, can be local) [default: "local"]
- `helm_output_path`: Path to save generated Helm charts [default: "./helm-charts"]
- `namespace`: Kubernetes namespace for deployment [default: "todo-app"]
- `replicas_frontend`: Number of replicas for the frontend deployment [default: 2]
- `replicas_backend`: Number of replicas for the backend deployment [default: 2]
- `minikube_profile`: Minikube profile name for local deployment [default: "todo-minikube"]

## Execution Workflow

### 1. Containerization Phase
The skill will:
- Generate optimized Dockerfiles for both frontend and backend applications
- Build production-ready container images
- Apply multi-stage builds for security and optimization
- Include health checks and proper resource allocation

For the frontend (Next.js/React), it will create a Dockerfile with:
- Node.js base image (node:20-alpine)
- Multi-stage build with build artifacts separation
- Production build optimization
- Health check endpoint

For the backend (Python/FastAPI), it will create a Dockerfile with:
- Python base image (python:3.11-slim)
- Dependency installation in separate layer
- Security best practices (non-root user)
- Health check endpoint

### 2. Helm Chart Generation Phase
The skill will generate complete Helm charts for:
- Frontend service with deployment, service, and ingress
- Backend service with deployment, service, and proper networking
- ConfigMaps for configuration management
- Secrets for sensitive data
- Horizontal Pod Autoscalers for scaling

### 3. Deployment Phase
The skill will:
- Set up Kubernetes cluster (Minikube if needed)
- Create the specified namespace
- Deploy backend service first (dependency ordering)
- Deploy frontend service with proper service discovery
- Configure auto-scaling and health checks
- Validate deployment completion

## AI Tool Integration

The skill leverages AI-assisted DevOps tools when available:

### Gordon (Docker AI)
- Generate optimized Dockerfiles for both services
- Build and optimize container images
- Apply security scanning and best practices

### kubectl-ai
- Deploy applications to Kubernetes
- Scale deployments based on load
- Troubleshoot deployment issues
- Manage configuration updates

### Kagent
- Monitor cluster health
- Analyze resource utilization
- Optimize deployment performance

## Scripts Available

The skill includes pre-built scripts for common operations:

### Containerization Scripts
- `scripts/build-frontend-image.sh` - Build frontend container image
- `scripts/build-backend-image.sh` - Build backend container image
- `scripts/optimize-images.sh` - Optimize images for production

### Deployment Scripts
- `scripts/deploy-full-stack.sh` - Deploy both frontend and backend
- `scripts/validate-deployment.sh` - Validate deployment status
- `scripts/rollback-deployment.sh` - Rollback to previous version

### Helm Management Scripts
- `scripts/generate-helm-charts.sh` - Generate Helm charts from templates
- `scripts/upgrade-deployment.sh` - Upgrade deployment with new charts
- `scripts/uninstall-deployment.sh` - Remove deployment cleanly

## Configuration Management

The skill implements proper configuration management:
- Environment variables via ConfigMaps
- Sensitive data via Kubernetes Secrets
- Externalized configuration for different environments
- Secure handling of API keys and database connections

## Auto-Scaling Configuration

Both frontend and backend deployments include:
- Horizontal Pod Autoscaler (HPA) configurations
- CPU and memory-based scaling triggers
- Minimum and maximum replica bounds
- Proper resource requests and limits

## Health Checks and Monitoring

Built-in health checks for both services:
- Liveness probes to restart unhealthy pods
- Readiness probes to remove unhealthy pods from service
- Application-level health endpoints
- Kubernetes-native monitoring integration

## Output

Upon successful execution, the skill provides:
- `frontend_image`: Tagged frontend Docker image reference
- `backend_image`: Tagged backend Docker image reference
- `helm_frontend_chart`: Path to generated frontend Helm chart
- `helm_backend_chart`: Path to generated backend Helm chart
- `deployment_status`: Current status of the deployment

## Error Handling

The skill includes comprehensive error handling:
- Validation of prerequisites before starting
- Rollback capabilities if deployment fails
- Detailed error messages for troubleshooting
- Automatic retry mechanisms for transient failures

## Best Practices Implemented

- **Security**: Non-root containers, minimal base images, secrets management
- **Scalability**: Horizontal pod autoscaling, proper resource allocation
- **Reliability**: Health checks, readiness probes, graceful shutdown
- **Maintainability**: Clean separation of concerns, documented configurations
- **Observability**: Built-in monitoring, logging, and metrics

## Troubleshooting

If deployment issues occur, check:
- Docker daemon is running and accessible
- Kubernetes cluster is available and connected
- Required ports are not in use
- Sufficient system resources (memory, disk space)
- Network connectivity for pulling images

## Success Criteria

Deployment is successful when:
- All pods are running and healthy
- Services are accessible via Kubernetes services
- Health checks are passing
- Auto-scaling is configured and functional
- Both frontend and backend can communicate
- All application features are working correctly