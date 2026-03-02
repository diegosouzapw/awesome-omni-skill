---
name: aspnet-core-devops
version: "2.0.0"
description: Master ASP.NET Core deployment, Docker, Azure cloud, CI/CD pipelines, and production infrastructure for enterprise applications.
sasmp_version: "1.3.0"
bonded_agent: aspnet-core-devops
bond_type: PRIMARY_BOND

# Skill Configuration
skill_type: atomic
responsibility: single

# Parameter Validation Schema
parameters:
  cloud_provider:
    type: string
    required: false
    default: "azure"
    validation:
      allowed_values: ["azure", "aws", "gcp", "on-premise"]
  container_runtime:
    type: string
    required: false
    default: "docker"
    validation:
      allowed_values: ["docker", "containerd", "podman"]
  orchestrator:
    type: string
    required: false
    default: "kubernetes"
    validation:
      allowed_values: ["kubernetes", "docker-swarm", "ecs", "none"]
  ci_platform:
    type: string
    required: false
    default: "github-actions"
    validation:
      allowed_values: ["github-actions", "azure-pipelines", "gitlab-ci", "jenkins"]

# Retry Logic Configuration
retry_config:
  enabled: true
  max_attempts: 5
  backoff_type: exponential
  initial_delay_ms: 2000
  max_delay_ms: 60000
  jitter: true
  retryable_errors:
    - NETWORK_ERROR
    - DEPLOYMENT_TIMEOUT
    - CONTAINER_PULL_ERROR

# Observability Hooks
observability:
  logging:
    enabled: true
    level: Information
    include_deployment_details: true
    sensitive_fields: ["password", "secret", "key", "token", "connectionString"]
  metrics:
    enabled: true
    namespace: skill.aspnetcore.devops
    dimensions:
      - deployment_stage
      - environment
      - status
  tracing:
    enabled: true
    span_name: aspnetcore-devops
    include_infrastructure_spans: true

# Unit Test Templates
test_templates:
  framework: infrastructure_tests
  tools:
    - terratest
    - pester
  coverage_target: 70
---

# ASP.NET Core DevOps & Production

## Skill Overview

Production-grade DevOps skill for ASP.NET Core applications. Covers containerization, cloud deployment, CI/CD pipelines, and infrastructure as code with comprehensive observability.

## DevOps Skills

### Docker & Containerization
```yaml
dockerfile_best_practices:
  multi_stage_builds:
    - Separate build and runtime stages
    - Minimize final image size
    - Use specific base image tags

  security:
    - Run as non-root user
    - Use distroless/alpine images
    - Scan for vulnerabilities
    - Read-only filesystem

  optimization:
    - Order layers by change frequency
    - Use .dockerignore
    - Leverage build cache
    - Multi-platform builds

docker_compose:
  use_cases:
    - Local development
    - Integration testing
    - Multi-container apps
  features:
    - Service dependencies
    - Volume mounts
    - Network isolation
    - Environment variables

container_security:
  practices:
    - Non-root user
    - Drop capabilities
    - Read-only rootfs
    - Security scanning
  tools:
    - Trivy
    - Snyk
    - Docker Scout
```

### Azure Cloud Services
```yaml
compute:
  app_service:
    tiers: [Basic, Standard, Premium, Isolated]
    features:
      - Auto-scaling
      - Deployment slots
      - Custom domains
      - SSL certificates
    best_for: Web apps, APIs

  container_apps:
    features:
      - Serverless containers
      - Auto-scaling
      - Dapr integration
      - Revision management
    best_for: Microservices

  aks:
    features:
      - Managed Kubernetes
      - Node pools
      - Azure CNI
      - AAD integration
    best_for: Complex workloads

data:
  sql_database:
    tiers: [Basic, Standard, Premium]
    features:
      - Geo-replication
      - Automatic backups
      - Elastic pools

  cosmos_db:
    apis: [SQL, MongoDB, Cassandra, Gremlin]
    features:
      - Global distribution
      - Multi-model
      - Automatic scaling

  redis_cache:
    tiers: [Basic, Standard, Premium]
    features:
      - Clustering
      - Geo-replication
      - Data persistence

security:
  key_vault:
    purpose: Secrets management
    features:
      - Access policies
      - Managed identity
      - Key rotation
      - Audit logging

  managed_identity:
    types: [System-assigned, User-assigned]
    benefits:
      - No credential management
      - Automatic rotation
      - Azure AD integration
```

### CI/CD Pipelines
```yaml
github_actions:
  workflow_structure:
    - Triggers (push, PR, schedule)
    - Jobs (build, test, deploy)
    - Steps (actions, scripts)
  features:
    - Matrix builds
    - Reusable workflows
    - Environment protection
    - OIDC authentication
    - Artifact management

azure_pipelines:
  structure:
    - Stages
    - Jobs
    - Steps
  features:
    - Multi-stage pipelines
    - Deployment groups
    - Variable groups
    - Service connections

deployment_strategies:
  blue_green:
    description: Deploy to inactive slot, swap
    benefits:
      - Zero downtime
      - Instant rollback
    azure: Deployment slots

  canary:
    description: Gradual traffic shift
    benefits:
      - Risk mitigation
      - Real user testing
    implementation: Traffic manager, Azure Front Door

  rolling:
    description: Update instances incrementally
    benefits:
      - No additional resources
      - Gradual rollout
    kubernetes: Rolling update strategy

quality_gates:
  - Unit tests pass
  - Code coverage threshold
  - Static analysis clean
  - Security scan pass
  - Performance benchmarks
```

### Kubernetes Orchestration
```yaml
core_concepts:
  pods:
    - Smallest deployable unit
    - Container grouping
    - Shared network/storage

  deployments:
    - Desired state management
    - Rolling updates
    - Rollback capability

  services:
    types:
      - ClusterIP (internal)
      - NodePort (node access)
      - LoadBalancer (external)

  ingress:
    controllers:
      - NGINX
      - Traefik
      - Azure Application Gateway
    features:
      - Path-based routing
      - TLS termination
      - Host-based routing

advanced_features:
  hpa:
    metrics:
      - CPU utilization
      - Memory utilization
      - Custom metrics
    behavior:
      - Scale up policies
      - Scale down policies
      - Stabilization windows

  pdb:
    purpose: Availability during updates
    settings:
      - minAvailable
      - maxUnavailable

  network_policies:
    purpose: Pod-to-pod traffic control
    types:
      - Ingress rules
      - Egress rules

helm:
  concepts:
    - Charts
    - Values
    - Templates
    - Releases
  best_practices:
    - Parameterize values
    - Use dependencies
    - Version charts
```

### Infrastructure as Code
```yaml
terraform:
  structure:
    - Providers
    - Resources
    - Variables
    - Outputs
    - Modules
  state_management:
    - Remote backend
    - State locking
    - Workspaces
  best_practices:
    - Modular design
    - Version pinning
    - Plan before apply

bicep:
  benefits:
    - Native Azure support
    - Type safety
    - Simpler syntax
  structure:
    - Parameters
    - Variables
    - Resources
    - Outputs
    - Modules

pulumi:
  benefits:
    - Use familiar languages (C#)
    - Type checking
    - IDE support
  structure:
    - Stacks
    - Resources
    - Outputs
```

### Monitoring & Logging
```yaml
observability_pillars:
  metrics:
    tools:
      - Application Insights
      - Prometheus
      - Azure Monitor
    types:
      - Request rate
      - Error rate
      - Latency (p50, p95, p99)
      - Resource utilization

  logs:
    structured_logging:
      - Serilog
      - NLog
    aggregation:
      - Azure Log Analytics
      - ELK Stack
      - Loki
    best_practices:
      - Correlation IDs
      - Log levels
      - Sensitive data masking

  traces:
    distributed_tracing:
      - OpenTelemetry
      - Application Insights
    features:
      - Request correlation
      - Dependency tracking
      - Performance profiling

health_checks:
  types:
    - Liveness (is alive?)
    - Readiness (can serve?)
    - Startup (initialized?)
  implementation:
    - /health/live
    - /health/ready
    - Custom health checks

alerting:
  strategies:
    - Error rate spikes
    - Latency thresholds
    - Resource exhaustion
    - SLO violations
  channels:
    - Email
    - Slack
    - PagerDuty
    - Azure Monitor
```

## Code Examples

### Production-Ready Dockerfile
```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:9.0-alpine AS build
WORKDIR /src

# Copy csproj and restore (cached layer)
COPY ["*.csproj", "./"]
RUN dotnet restore --runtime linux-musl-x64

# Copy source and build
COPY . .
RUN dotnet build -c Release --no-restore -o /app/build

# Test stage
FROM build AS test
RUN dotnet test --no-build -c Release \
    --logger "trx" \
    --results-directory /testresults

# Publish stage
FROM build AS publish
RUN dotnet publish -c Release -o /app/publish \
    --runtime linux-musl-x64 \
    --self-contained true \
    -p:PublishTrimmed=true \
    -p:PublishSingleFile=true

# Runtime stage - distroless equivalent
FROM mcr.microsoft.com/dotnet/runtime-deps:9.0-alpine AS final

# Security: Non-root user
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -D appuser

WORKDIR /app

# Copy published app with correct ownership
COPY --from=publish --chown=appuser:appgroup /app/publish .

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost:8080/health || exit 1

# Expose port (non-privileged)
EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080
ENV ASPNETCORE_ENVIRONMENT=Production

# Entry point
ENTRYPOINT ["./MyApp"]
```

### GitHub Actions Complete Workflow
```yaml
name: Build, Test, and Deploy

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  id-token: write
  contents: read
  packages: write
  security-events: write

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  AZURE_WEBAPP_NAME: myapp

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.0.x'

      - name: Restore dependencies
        run: dotnet restore

      - name: Build
        run: dotnet build --no-restore -c Release

      - name: Test with coverage
        run: |
          dotnet test --no-build -c Release \
            --logger trx \
            --collect:"XPlat Code Coverage" \
            --results-directory ./TestResults

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./TestResults/**/coverage.cobertura.xml
          fail_ci_if_error: true

      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: ./TestResults

      - name: Generate version
        id: version
        run: |
          VERSION=$(date +'%Y%m%d').${{ github.run_number }}
          echo "version=$VERSION" >> $GITHUB_OUTPUT

  security-scan:
    runs-on: ubuntu-latest
    needs: build-and-test

    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'

  build-container:
    runs-on: ubuntu-latest
    needs: [build-and-test, security-scan]
    if: github.event_name != 'pull_request'

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=raw,value=${{ needs.build-and-test.outputs.version }}
            type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Scan container image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ needs.build-and-test.outputs.version }}
          exit-code: '1'
          severity: 'CRITICAL,HIGH'

  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build-and-test, build-container]
    if: github.ref == 'refs/heads/main'
    environment: staging

    steps:
      - name: Azure Login (OIDC)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Deploy to Staging
        uses: azure/webapps-deploy@v3
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}-staging
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ needs.build-and-test.outputs.version }}

      - name: Smoke test
        run: |
          sleep 60
          curl --fail --retry 3 --retry-delay 10 \
            https://${{ env.AZURE_WEBAPP_NAME }}-staging.azurewebsites.net/health

  deploy-production:
    runs-on: ubuntu-latest
    needs: [build-and-test, deploy-staging]
    if: github.ref == 'refs/heads/main'
    environment: production

    steps:
      - name: Azure Login (OIDC)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Deploy to Production Slot
        uses: azure/webapps-deploy@v3
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          slot-name: staging
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ needs.build-and-test.outputs.version }}

      - name: Swap slots
        run: |
          az webapp deployment slot swap \
            --name ${{ env.AZURE_WEBAPP_NAME }} \
            --resource-group myResourceGroup \
            --slot staging \
            --target-slot production

      - name: Verify deployment
        run: |
          sleep 30
          curl --fail --retry 5 --retry-delay 10 \
            https://${{ env.AZURE_WEBAPP_NAME }}.azurewebsites.net/health
```

### Kubernetes Production Manifests
```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
  labels:
    name: myapp

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: myapp
data:
  ASPNETCORE_ENVIRONMENT: "Production"
  LOGGING__LOGLEVEL__DEFAULT: "Information"

---
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
  namespace: myapp
type: Opaque
stringData:
  ConnectionStrings__Default: "Server=xxx;Database=xxx;..."

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
    spec:
      serviceAccountName: myapp
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      containers:
        - name: myapp
          image: ghcr.io/myorg/myapp:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          envFrom:
            - configMapRef:
                name: myapp-config
            - secretRef:
                name: myapp-secrets
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir: {}

---
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: myapp
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp
  namespace: myapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70

---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp
  namespace: myapp
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

## Troubleshooting Guide

### Common Issues

| Issue | Symptoms | Resolution |
|-------|----------|------------|
| Image Pull Error | ImagePullBackOff | Check registry auth, image tag |
| Pod CrashLoop | Continuous restarts | Check logs, liveness probe |
| OOMKilled | Pod terminated | Increase memory limits |
| Deployment Timeout | Stuck rollout | Check readiness probe |
| SSL/TLS Errors | Certificate issues | Verify cert-manager, secrets |

### Debug Checklist

```yaml
step_1_container:
  - docker build locally
  - docker run and test
  - Check Dockerfile syntax
  - Verify base image

step_2_kubernetes:
  - kubectl describe pod
  - kubectl logs pod
  - kubectl get events
  - Check resource limits

step_3_networking:
  - kubectl get svc
  - kubectl get endpoints
  - Test service connectivity
  - Check ingress config

step_4_deployment:
  - Check CI/CD logs
  - Verify secrets/configs
  - Test health endpoints
  - Monitor rollout status
```

## Assessment Criteria

- [ ] Create optimized Docker images
- [ ] Deploy to cloud platforms
- [ ] Setup complete CI/CD pipelines
- [ ] Configure monitoring and alerts
- [ ] Manage secrets securely
- [ ] Implement auto-scaling
- [ ] Setup health checks
- [ ] Configure networking
- [ ] Implement disaster recovery
- [ ] Monitor application performance

## References

- [Docker Documentation](https://docs.docker.com)
- [Kubernetes Documentation](https://kubernetes.io/docs)
- [Azure App Service](https://learn.microsoft.com/azure/app-service)
- [GitHub Actions](https://docs.github.com/actions)
- [Terraform Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm)
