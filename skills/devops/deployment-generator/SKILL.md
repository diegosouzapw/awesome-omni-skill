---
name: deployment-generator
description: Use when users request Kubernetes deployment configs, CI/CD pipelines, or Docker configurations - ensures systematic discovery, complete artifact generation, and production-ready best practices through structured workflow
---

# Deployment Generator Skill

## When to Use This Skill

Use this skill when user requests:
- Kubernetes deployment configurations
- CI/CD pipelines (GitHub Actions, GitLab CI, etc.)
- Docker/container configurations
- Infrastructure-as-code for deployments
- "Deploy my app to Kubernetes"
- "Create GitHub Actions workflow"
- "Setup CI/CD pipeline"

## Mandatory Workflow

Follow these phases in order. DO NOT skip phases.

### Phase 1: Discovery

Ask systematic questions to understand requirements. Use AskUserQuestion tool.

#### Core Questions (Always Ask)

**Traffic & Scaling:**
- Expected traffic level? (Low/Medium/High)
- Need autoscaling? (Yes/No)
- If yes: Min/max replicas, CPU threshold?
- If no: Fixed replica count per component?

**Environment & Branching:**
- How many environments? (Dev/Staging/Prod)
- Branch strategy? (main, develop, feature branches)
- Environment triggers? (branches, tags, manual)

**Resources & Constraints:**
- Resource limits per environment?
- Storage requirements?
- External dependencies? (databases, queues, caches)

**Deployment Details:**
- Kubernetes cluster access method?
- Image registry? (DockerHub, GCR, ECR, ACR)
- Domain/ingress requirements?
- Secrets management approach?

#### DockerHub Configuration (Critical)

**Always ask about DockerHub repository pattern:**

```
How do you want to organize Docker images?

A) Separate repositories per component
   Example: username/myapp-backend:latest
            username/myapp-frontend:latest

B) Single repository with component tags (cost-effective for private repos)
   Example: username/store:myapp-backend-v1.0.0
            username/store:myapp-frontend-v1.0.0
```

**If user chooses Option B (Single Repository), ask:**
- DockerHub username?
- Repository name? (e.g., "store", "images", "containers")
- Project name prefix? (e.g., "myapp" → "myapp-backend-v1.0.0")
- Image naming pattern preference? (project-component-version OR custom)

**Detection Patterns for Single Repo:**
- User mentions "single repo" or "one repository"
- User shows example like `username/repo:image-tag`
- User mentions "private repo" or "cost savings"
- User already has existing private DockerHub repo
- User has multiple projects they want to store together

### Phase 2: Codebase Analysis

Before generating configs, analyze the codebase:

1. **Identify Components:**
   - Look for Dockerfiles (Dockerfile, Dockerfile.*, *.Dockerfile)
   - Identify services (backend, frontend, workers, etc.)
   - Check for monorepo structure (packages/, apps/)

2. **Check Build Systems:**
   - Package managers (npm, pnpm, yarn, maven, gradle)
   - Build commands in package.json, Makefile, scripts/
   - Multi-stage builds needed?

3. **Database & Dependencies:**
   - Database type (PostgreSQL, MySQL, MongoDB, SQLite)
   - Database migrations (Prisma, Sequelize, Flyway, Liquibase)
   - Queues (Redis, RabbitMQ, Kafka)
   - Caches (Redis, Memcached)

4. **Configuration Patterns:**
   - Environment variables in .env.example
   - Config files (config/, .config/)
   - Health check endpoints
   - Port configurations

5. **Optional Components:**
   - Identify optional services (operators, controllers, admin tools)
   - Check if they're required for basic deployment
   - Ask user if they want to include them

### Phase 3: Generation

Generate complete deployment artifacts using TodoWrite to track:

#### For GitHub Actions Workflow

**Single Repository Pattern:**
```yaml
name: Build and Deploy

on:
  push:
    branches: [deploy, main]
    tags: ['v*']
  workflow_dispatch:

env:
  DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
  DOCKERHUB_REPO: ${{ secrets.DOCKERHUB_REPO || 'images' }}
  PROJECT_NAME: myapp  # Replace with actual project name
  REGISTRY: docker.io

jobs:
  build-and-push:
    name: Build and Push Images
    runs-on: ubuntu-latest
    strategy:
      matrix:
        component:
          - name: backend
            dockerfile: Dockerfile.backend
            context: .
          - name: frontend
            dockerfile: Dockerfile.frontend
            context: .

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to DockerHub
        uses: docker/login-action@v3
        with:
          registry: docker.io
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Set image names
        id: images
        run: |
          # Determine version from branch or tag
          if [[ $GITHUB_REF == refs/tags/* ]]; then
            VERSION="${GITHUB_REF#refs/tags/}"
          elif [[ $GITHUB_REF == refs/heads/main ]] || [[ $GITHUB_REF == refs/heads/master ]]; then
            VERSION="latest"
          else
            BRANCH="${GITHUB_REF#refs/heads/}"
            VERSION="${BRANCH}-${GITHUB_SHA::7}"
          fi

          # Single-repo pattern: project-component-version
          IMAGE_NAME="${PROJECT_NAME}-${{ matrix.component.name }}-${VERSION}"
          FULL_IMAGE="${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${IMAGE_NAME}"

          echo "version=${VERSION}" >> $GITHUB_OUTPUT
          echo "image_name=${IMAGE_NAME}" >> $GITHUB_OUTPUT
          echo "full_image=${FULL_IMAGE}" >> $GITHUB_OUTPUT

          # Also create :latest tag for main/master branch
          if [[ $VERSION == "latest" ]]; then
            echo "latest_tag=${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${PROJECT_NAME}-${{ matrix.component.name }}-latest" >> $GITHUB_OUTPUT
          fi

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ${{ matrix.component.context }}
          file: ${{ matrix.component.dockerfile }}
          push: true
          tags: |
            ${{ steps.images.outputs.full_image }}
            ${{ steps.images.outputs.latest_tag }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64

  deploy:
    name: Deploy to Kubernetes
    runs-on: ubuntu-latest
    needs: build-and-push

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set version
        id: version
        run: |
          if [[ $GITHUB_REF == refs/tags/* ]]; then
            VERSION="${GITHUB_REF#refs/tags/}"
          else
            BRANCH="${GITHUB_REF#refs/heads/}"
            VERSION="${BRANCH}-${GITHUB_SHA::7}"
          fi
          echo "version=${VERSION}" >> $GITHUB_OUTPUT

      - name: Set up kubectl
        uses: azure/setup-kubectl@v3

      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > kubeconfig.yaml
          echo "KUBECONFIG=$(pwd)/kubeconfig.yaml" >> $GITHUB_ENV

      - name: Update manifests with image names
        run: |
          cd config/k8s

          # Single-repo pattern replacements
          sed -i "s|IMAGE_BACKEND|${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${PROJECT_NAME}-backend-${{ steps.version.outputs.version }}|g" backend.yaml
          sed -i "s|IMAGE_FRONTEND|${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${PROJECT_NAME}-frontend-${{ steps.version.outputs.version }}|g" frontend.yaml
        env:
          DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
          DOCKERHUB_REPO: ${{ secrets.DOCKERHUB_REPO || 'images' }}
          PROJECT_NAME: myapp

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f config/k8s/namespace.yaml
          kubectl apply -f config/k8s/
          kubectl rollout status deployment/myapp-backend -n myapp --timeout=5m
          kubectl rollout status deployment/myapp-frontend -n myapp --timeout=5m

      - name: Verify deployment
        if: always()
        run: |
          echo "=== Pods ==="
          kubectl get pods -n myapp

          echo "=== Services ==="
          kubectl get svc -n myapp

          echo "=== Backend Logs ==="
          kubectl logs -l app=myapp-backend -n myapp --tail=50 || true

          echo "=== Frontend Logs ==="
          kubectl logs -l app=myapp-frontend -n myapp --tail=50 || true
```

**Separate Repository Pattern:**
```yaml
# Use component name as repository
IMAGE_NAME="${{ matrix.component.name }}"
FULL_IMAGE="${DOCKERHUB_USERNAME}/${PROJECT_NAME}-${IMAGE_NAME}:${VERSION}"
```

**Key Differences:**
- Single-repo: `DOCKERHUB_REPO` secret + project prefix in tag
- Separate-repo: Component name becomes repository name
- Single-repo needs project prefix to avoid collisions

#### Kubernetes Manifests

Generate complete manifests in `config/k8s/`:

1. **namespace.yaml** - Namespace definition
2. **backend.yaml** - Backend deployment + service + PVC
3. **frontend.yaml** - Frontend deployment + service + configmap
4. **database.yaml** - Database deployment (if applicable)
5. **redis.yaml** - Redis/cache deployment (if applicable)
6. **ingress.yaml** - Ingress rules
7. **kustomization.yaml** - Kustomize config

**Use IMAGE placeholders:**
```yaml
spec:
  containers:
    - name: backend
      image: IMAGE_BACKEND  # Replaced by CI/CD
```

**Include:**
- Resource requests/limits
- Health checks (liveness, readiness, startup)
- Environment variables from secrets
- PVC for stateful components
- Proper labels and selectors

#### Required Secrets Documentation

**For Single Repository Pattern:**
```markdown
## Required GitHub Secrets

```bash
DOCKERHUB_USERNAME - DockerHub account username
DOCKERHUB_REPO - Repository name for all images (e.g., "store")
DOCKERHUB_TOKEN - Access token from DockerHub
KUBE_CONFIG - Base64-encoded kubeconfig
```

**Image naming:** `{username}/{repo}:{project}-{component}-{version}`

**Examples:**
- `kazishiplu/store:myapp-backend-deploy-abc1234`
- `kazishiplu/store:myapp-frontend-v1.0.0`
```

**For Separate Repository Pattern:**
```markdown
## Required GitHub Secrets

```bash
DOCKERHUB_USERNAME - DockerHub account username
DOCKERHUB_TOKEN - Access token from DockerHub
KUBE_CONFIG - Base64-encoded kubeconfig
```

**Image naming:** `{username}/{project}-{component}:{version}`

**Examples:**
- `kazishiplu/myapp-backend:deploy-abc1234`
- `kazishiplu/myapp-frontend:v1.0.0`
```

#### Documentation Files

Generate comprehensive docs:

1. **DEPLOYMENT.md** - Complete deployment guide
   - Architecture overview
   - Prerequisites
   - Step-by-step setup
   - GitHub secrets configuration
   - Kubernetes secrets
   - Verification commands
   - Troubleshooting

2. **QUICK_DEPLOY.md** - Fast-track guide
   - Minimal steps to deploy
   - Assumes familiarity

3. **DOCKERHUB_SINGLE_REPO.md** (if single-repo pattern)
   - Image naming pattern
   - Manual build commands
   - Migration from separate repos
   - Cleanup strategies

### Phase 4: Verification

Create verification checklist using TodoWrite:

**Mandatory Checks:**
- [ ] All Dockerfiles exist and are valid
- [ ] Health check endpoints exist in code
- [ ] Resource limits set for all containers
- [ ] Secrets documented (GitHub + Kubernetes)
- [ ] Image naming pattern documented
- [ ] Manifests use placeholders (IMAGE_*)
- [ ] Rollout verification included in workflow
- [ ] Pod logs displayed on failure
- [ ] Documentation complete

**Single-Repo Specific Checks:**
- [ ] DOCKERHUB_REPO secret documented
- [ ] Image names include project prefix
- [ ] Tag format: `project-component-version`
- [ ] DOCKERHUB_SINGLE_REPO.md created
- [ ] Image pull secret added (if private repo)

**Optional Component Checks:**
- [ ] Optional components clearly marked
- [ ] User confirmed which components to include
- [ ] Documentation explains what's optional

## Detection Logic

### Single Repository Pattern Detection

**Trigger single-repo workflow if user:**
- Mentions "single repo" / "one repository" / "shared repo"
- Shows example: `username/repo:image-tag`
- Says "private DockerHub repo"
- Mentions "cost savings" / "one private repo"
- Shows existing pattern with repo:tag format

**Then ask:**
1. DockerHub username?
2. Repository name?
3. Project name prefix?

### Separate Repository Pattern (Default)

**Use if:**
- No single-repo indicators
- User doesn't specify
- First-time deployment

### Optional Components Detection

**Check codebase for:**
- Kubernetes operators (CRD files, controller code)
- Admin dashboards not in main app
- Development tools not needed in production
- Specialized workers/jobs

**Ask user:**
```
I found these optional components:
- [Component name]: [Purpose]

Include in deployment? These are not required for basic functionality.
```

## Anti-Patterns to Avoid

**DON'T:**
- Generate workflow without discovery questions
- Assume single-repo pattern without asking
- Hardcode image names in manifests
- Skip health checks
- Forget resource limits
- Mix single-repo and separate-repo patterns
- Include optional components without confirmation

**DO:**
- Ask about DockerHub organization preference
- Use IMAGE placeholders in manifests
- Document all secrets clearly
- Distinguish required vs optional components
- Generate verification steps
- Create migration guide if changing patterns

## Example Conversations

### Example 1: Single Repository

```
User: "Create deployment for my app using kazishiplu/store:myapp-backend-v1"
Assistant: I detected single-repo pattern from your example. Questions:
- DockerHub username? (I see: kazishiplu)
- Repository name? (I see: store)
- Project name prefix?

User: "Project is 'chronostash'"

[Proceeds with discovery, codebase analysis, generation]

Result: Workflow uses kazishiplu/store:chronostash-backend-{version}
```

### Example 2: Separate Repository (Default)

```
User: "Setup deployment for my Node.js app to Kubernetes"- DockerHub username? (I see: kazishiplu)
- Repository name? (I see: store)
- Project name prefix?

User: "Project is chronostash"

[Proceeds with full workflow]

Result: Workflow uses kazishiplu/store:chronostash-backend-{version}
```

### Example 2: Separate Repository

```
User: "Setup Kubernetes deployment"

Assistant asks about Docker organization preference

User chooses separate repositories

Result: Separate repos created
```

## Summary

Always detect single-repo pattern from user examples
Ask about DockerHub configuration in discovery phase
Generate appropriate workflow for chosen pattern
Document DOCKERHUB_REPO secret for single-repo
Include verification checklist for pattern used
