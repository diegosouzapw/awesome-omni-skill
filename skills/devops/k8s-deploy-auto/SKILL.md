---
name: k8s-deploy-auto
description: Kubernetes deployment automation workflows for CI/CD pipelines, GitOps, and scripted deployments. Use when automating k8s deployments, creating deployment scripts, integrating with GitHub Actions/GitLab CI, implementing rollout strategies, or setting up ArgoCD/Flux workflows.
---

# Kubernetes Deployment Automation

## Deployment Script Template

### Basic Deploy Script

```bash
#!/usr/bin/env bash
set -euo pipefail

# Configuration
NAMESPACE="${NAMESPACE:-default}"
MANIFEST_DIR="${MANIFEST_DIR:-.}"
TIMEOUT="${TIMEOUT:-300s}"

log() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"; }
die() { log "ERROR: $*"; exit 1; }

# Validate prerequisites
command -v kubectl >/dev/null || die "kubectl not found"
kubectl auth can-i create deployments -n "$NAMESPACE" >/dev/null || die "No deploy permissions"

# Validate manifests
log "Validating manifests..."
kubectl apply --dry-run=server -f "$MANIFEST_DIR" -n "$NAMESPACE" || die "Validation failed"

# Apply manifests
log "Applying manifests to $NAMESPACE..."
kubectl apply -f "$MANIFEST_DIR" -n "$NAMESPACE"

# Wait for rollout
log "Waiting for rollout..."
for deploy in $(kubectl get deploy -n "$NAMESPACE" -o name 2>/dev/null); do
    log "Checking $deploy..."
    kubectl rollout status "$deploy" -n "$NAMESPACE" --timeout="$TIMEOUT" || die "Rollout failed: $deploy"
done

# Verify pods
log "Verifying pods..."
kubectl get pods -n "$NAMESPACE" -o wide

log "Deployment complete!"
```

### Deploy with Rollback

```bash
#!/usr/bin/env bash
set -euo pipefail

NAMESPACE="${1:?Usage: $0 <namespace> <deployment> <image>}"
DEPLOYMENT="${2:?}"
IMAGE="${3:?}"
TIMEOUT="${TIMEOUT:-300s}"

log() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"; }

# Capture current revision for potential rollback
PREV_REVISION=$(kubectl rollout history "deployment/$DEPLOYMENT" -n "$NAMESPACE" | tail -2 | head -1 | awk '{print $1}')
log "Current revision: $PREV_REVISION"

# Update image
log "Updating $DEPLOYMENT to $IMAGE..."
kubectl set image "deployment/$DEPLOYMENT" "$DEPLOYMENT=$IMAGE" -n "$NAMESPACE"

# Annotate change
kubectl annotate "deployment/$DEPLOYMENT" kubernetes.io/change-cause="Image: $IMAGE" --overwrite -n "$NAMESPACE"

# Monitor rollout with timeout
log "Monitoring rollout..."
if ! kubectl rollout status "deployment/$DEPLOYMENT" -n "$NAMESPACE" --timeout="$TIMEOUT"; then
    log "ERROR: Rollout failed, initiating rollback to revision $PREV_REVISION..."
    kubectl rollout undo "deployment/$DEPLOYMENT" -n "$NAMESPACE" --to-revision="$PREV_REVISION"
    kubectl rollout status "deployment/$DEPLOYMENT" -n "$NAMESPACE" --timeout="$TIMEOUT"
    exit 1
fi

log "Rollout successful!"
kubectl get pods -n "$NAMESPACE" -l "app=$DEPLOYMENT" -o wide
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options: [staging, production]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image_tag: ${{ steps.meta.outputs.tags }}
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Registry
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
            type=sha,prefix=
            type=ref,event=branch

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment || 'staging' }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup kubectl
        uses: azure/setup-kubectl@v3

      - name: Configure kubeconfig
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBECONFIG }}" | base64 -d > ~/.kube/config
          chmod 600 ~/.kube/config

      - name: Deploy
        env:
          NAMESPACE: ${{ vars.K8S_NAMESPACE }}
          IMAGE: ${{ needs.build.outputs.image_tag }}
        run: |
          # Update image in manifests
          kubectl set image deployment/app app=$IMAGE -n $NAMESPACE
          
          # Wait for rollout
          kubectl rollout status deployment/app -n $NAMESPACE --timeout=300s
          
          # Verify
          kubectl get pods -n $NAMESPACE -l app=app

      - name: Rollback on failure
        if: failure()
        run: |
          kubectl rollout undo deployment/app -n ${{ vars.K8S_NAMESPACE }}
```

### GitLab CI

```yaml
stages:
  - build
  - deploy

variables:
  DOCKER_HOST: tcp://docker:2376
  DOCKER_TLS_CERTDIR: "/certs"

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main

.deploy_template: &deploy
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config set-cluster k8s --server="$K8S_SERVER" --certificate-authority="$K8S_CA_CERT"
    - kubectl config set-credentials deployer --token="$K8S_TOKEN"
    - kubectl config set-context default --cluster=k8s --user=deployer --namespace="$K8S_NAMESPACE"
    - kubectl config use-context default
    - kubectl set image deployment/$APP_NAME $APP_NAME=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n $K8S_NAMESPACE
    - kubectl rollout status deployment/$APP_NAME -n $K8S_NAMESPACE --timeout=300s

deploy_staging:
  <<: *deploy
  variables:
    K8S_NAMESPACE: staging
    APP_NAME: myapp
  environment:
    name: staging
  only:
    - main

deploy_production:
  <<: *deploy
  variables:
    K8S_NAMESPACE: production
    APP_NAME: myapp
  environment:
    name: production
  when: manual
  only:
    - main
```

## GitOps Patterns

### ArgoCD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/repo.git
    targetRevision: HEAD
    path: k8s/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

### Kustomize Structure

```
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
└── overlays/
    ├── staging/
    │   ├── kustomization.yaml
    │   ├── namespace.yaml
    │   └── patches/
    │       └── replicas.yaml
    └── production/
        ├── kustomization.yaml
        ├── namespace.yaml
        └── patches/
            ├── replicas.yaml
            └── resources.yaml
```

**base/kustomization.yaml:**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
  - configmap.yaml
```

**overlays/production/kustomization.yaml:**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: production
resources:
  - ../../base
  - namespace.yaml
patches:
  - path: patches/replicas.yaml
  - path: patches/resources.yaml
images:
  - name: myapp
    newTag: v1.2.3
```

## Canary Deployment

### Manual Canary

```bash
#!/usr/bin/env bash
# Deploy canary with 10% traffic

NAMESPACE="${NAMESPACE:-default}"
CANARY_REPLICAS=1
STABLE_REPLICAS=9

# Deploy canary version
kubectl apply -f canary-deployment.yaml -n "$NAMESPACE"

# Scale: 10% canary, 90% stable
kubectl scale deployment/app-stable --replicas="$STABLE_REPLICAS" -n "$NAMESPACE"
kubectl scale deployment/app-canary --replicas="$CANARY_REPLICAS" -n "$NAMESPACE"

echo "Canary deployed. Monitor metrics..."
echo "To promote: kubectl scale deployment/app-canary --replicas=10 && kubectl delete deployment/app-stable"
echo "To rollback: kubectl delete deployment/app-canary"
```

### Argo Rollouts

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 10
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:v1
  strategy:
    canary:
      steps:
        - setWeight: 10
        - pause: {duration: 5m}
        - setWeight: 30
        - pause: {duration: 5m}
        - setWeight: 50
        - pause: {duration: 5m}
      canaryService: myapp-canary
      stableService: myapp-stable
```

## Health Check Verification

```bash
#!/usr/bin/env bash
# Post-deployment health verification

NAMESPACE="${1:?namespace required}"
DEPLOYMENT="${2:?deployment required}"
ENDPOINT="${3:-/healthz}"
RETRIES=30
DELAY=10

log() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"; }

# Wait for pods to be ready
log "Waiting for pods..."
kubectl wait --for=condition=ready pod -l "app=$DEPLOYMENT" -n "$NAMESPACE" --timeout=300s

# Get pod IP
POD=$(kubectl get pods -n "$NAMESPACE" -l "app=$DEPLOYMENT" -o jsonpath='{.items[0].metadata.name}')
log "Testing pod: $POD"

# Health check loop
for i in $(seq 1 $RETRIES); do
    if kubectl exec "$POD" -n "$NAMESPACE" -- wget -q -O /dev/null "http://localhost:8080$ENDPOINT" 2>/dev/null; then
        log "Health check passed!"
        exit 0
    fi
    log "Attempt $i/$RETRIES failed, retrying in ${DELAY}s..."
    sleep "$DELAY"
done

log "Health check failed after $RETRIES attempts"
exit 1
```

## Secrets Management

### Sealed Secrets

```bash
# Install kubeseal
brew install kubeseal

# Seal a secret
kubectl create secret generic mysecret --dry-run=client -o yaml \
  --from-literal=password=secret123 | \
  kubeseal --controller-name=sealed-secrets --controller-namespace=kube-system -o yaml > sealed-secret.yaml

# Apply sealed secret (controller decrypts it)
kubectl apply -f sealed-secret.yaml
```

### External Secrets Operator

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: app-secrets
  data:
    - secretKey: database-password
      remoteRef:
        key: prod/app/database
        property: password
```

## Monitoring Deployment

```bash
# Watch deployment progress
watch -n1 'kubectl get pods -n production -o wide; echo "---"; kubectl get events -n production --sort-by=.lastTimestamp | tail -10'

# Stream logs during deploy
kubectl logs -f -l app=myapp -n production --all-containers --since=5m

# Check resource usage
kubectl top pods -n production --sort-by=memory
```
