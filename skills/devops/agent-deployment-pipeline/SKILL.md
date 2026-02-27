---
name: agent-deployment-pipeline
description: Implement CI/CD pipelines for AI agent deployment with evaluation gates. Use for GitHub Actions workflows, GitOps with ArgoCD, container image building, and automated testing. Triggers on "CI/CD", "pipeline", "GitHub Actions", "GitOps", "ArgoCD", "deployment automation", "continuous deployment", or when implementing safe agent release workflows.
---

# Agent Deployment Pipeline

## Overview

Implement CI/CD pipelines for AgentStack agents with evaluation-based safety gates, GitOps workflows, and progressive rollouts.

## Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   Deployment Pipeline                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐         │
│  │  Push   │──▶│  Build  │──▶│  Test   │──▶│  Eval   │──▶      │
│  │         │   │  Image  │   │  Unit   │   │  MLflow │         │
│  └─────────┘   └─────────┘   └─────────┘   └─────────┘         │
│                                                │                │
│                                                ▼                │
│                                        ┌─────────────┐         │
│                                        │ Safety Gate │         │
│                                        │   Pass?     │         │
│                                        └─────────────┘         │
│                                           │    │               │
│                                    Yes ───┘    └─── No         │
│                                    │                │          │
│                                    ▼                ▼          │
│                            ┌─────────────┐  ┌─────────────┐    │
│                            │   Deploy    │  │   Block     │    │
│                            │   Staging   │  │   + Alert   │    │
│                            └─────────────┘  └─────────────┘    │
│                                    │                           │
│                                    ▼                           │
│                            ┌─────────────┐                     │
│                            │  Canary     │                     │
│                            │  Rollout    │                     │
│                            └─────────────┘                     │
│                                    │                           │
│                                    ▼                           │
│                            ┌─────────────┐                     │
│                            │ Production  │                     │
│                            │   100%      │                     │
│                            └─────────────┘                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## GitHub Actions Workflow

### Main Pipeline

```yaml
# .github/workflows/agent-deploy.yaml
name: Agent Deployment Pipeline

on:
  push:
    branches: [main]
    paths:
      - 'agents/**'
  pull_request:
    branches: [main]
    paths:
      - 'agents/**'

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      agents: ${{ steps.changes.outputs.agents }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: changes
        with:
          filters: |
            agents:
              - 'agents/**'
          list-files: json

  build:
    needs: detect-changes
    if: needs.detect-changes.outputs.agents != '[]'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        agent: ${{ fromJson(needs.detect-changes.outputs.agents) }}
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
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.agent }}
          tags: |
            type=sha,prefix=
            type=ref,event=branch
            type=ref,event=pr
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: agents/${{ matrix.agent }}
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  test:
    needs: build
    runs-on: ubuntu-latest
    strategy:
      matrix:
        agent: ${{ fromJson(needs.detect-changes.outputs.agents) }}
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          pip install -r agents/${{ matrix.agent }}/requirements.txt
          pip install pytest pytest-cov
      
      - name: Run unit tests
        run: |
          pytest agents/${{ matrix.agent }}/tests/ \
            --cov=agents/${{ matrix.agent }}/src \
            --cov-report=xml \
            --junitxml=test-results.xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          file: coverage.xml
          flags: ${{ matrix.agent }}

  evaluate:
    needs: [build, test]
    runs-on: ubuntu-latest
    strategy:
      matrix:
        agent: ${{ fromJson(needs.detect-changes.outputs.agents) }}
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install MLflow
        run: pip install mlflow>=3.0.0 openai
      
      - name: Run evaluation
        id: eval
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          MLFLOW_TRACKING_URI: ${{ secrets.MLFLOW_TRACKING_URI }}
        run: |
          cd agents/${{ matrix.agent }}
          python -m eval.run_evaluation \
            --agent-image ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.agent }}:${{ github.sha }} \
            --dataset eval/datasets/golden.yaml \
            --output-file results.json
          
          # Parse results for gate check
          SAFETY_SCORE=$(jq '.scores.safety' results.json)
          QUALITY_SCORE=$(jq '.scores.quality' results.json)
          
          echo "safety_score=$SAFETY_SCORE" >> $GITHUB_OUTPUT
          echo "quality_score=$QUALITY_SCORE" >> $GITHUB_OUTPUT
      
      - name: Safety gate check
        run: |
          SAFETY=${{ steps.eval.outputs.safety_score }}
          QUALITY=${{ steps.eval.outputs.quality_score }}
          
          if (( $(echo "$SAFETY < 0.95" | bc -l) )); then
            echo "❌ Safety score $SAFETY below threshold 0.95"
            exit 1
          fi
          
          if (( $(echo "$QUALITY < 0.80" | bc -l) )); then
            echo "❌ Quality score $QUALITY below threshold 0.80"
            exit 1
          fi
          
          echo "✅ All quality gates passed"
      
      - name: Upload evaluation artifacts
        uses: actions/upload-artifact@v4
        with:
          name: eval-${{ matrix.agent }}
          path: agents/${{ matrix.agent }}/results.json

  deploy-staging:
    needs: [evaluate]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Configure kubeconfig
        run: |
          echo "${{ secrets.KUBE_CONFIG_STAGING }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig
      
      - name: Deploy to staging
        run: |
          for agent in ${{ join(fromJson(needs.detect-changes.outputs.agents), ' ') }}; do
            kubectl set image deployment/$agent \
              agent=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/$agent:${{ github.sha }} \
              -n agentstack-staging
          done
      
      - name: Wait for rollout
        run: |
          for agent in ${{ join(fromJson(needs.detect-changes.outputs.agents), ' ') }}; do
            kubectl rollout status deployment/$agent -n agentstack-staging --timeout=5m
          done

  integration-tests:
    needs: [deploy-staging]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run integration tests
        run: |
          pip install pytest requests
          pytest tests/integration/ \
            --base-url=${{ vars.STAGING_URL }} \
            --api-key=${{ secrets.STAGING_API_KEY }}

  deploy-production:
    needs: [integration-tests]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy with canary
        uses: ./.github/actions/canary-deploy
        with:
          image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          initial-weight: 10
          increment: 20
          interval: 300  # 5 minutes
          error-threshold: 0.01
```

## GitOps with ArgoCD

### Application Manifest

```yaml
# argocd/applications/agents.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: agentstack-agents
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: agentstack
  source:
    repoURL: https://github.com/org/agentstack-config.git
    targetRevision: HEAD
    path: environments/production/agents
    
  destination:
    server: https://kubernetes.default.svc
    namespace: agentstack
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
      - ApplyOutOfSyncOnly=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        maxDuration: 3m
        factor: 2
```

### Progressive Rollout with Argo Rollouts

```yaml
# agents/customer-support/rollout.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: customer-support-agent
  namespace: agentstack
spec:
  replicas: 5
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: customer-support-agent
  template:
    metadata:
      labels:
        app: customer-support-agent
    spec:
      containers:
        - name: agent
          image: ghcr.io/org/agents/customer-support:latest
          ports:
            - containerPort: 8080
  strategy:
    canary:
      steps:
        # Start with 10% traffic
        - setWeight: 10
        - pause: {duration: 5m}
        
        # Analysis at 10%
        - analysis:
            templates:
              - templateName: agent-success-rate
            args:
              - name: agent-name
                value: customer-support-agent
        
        # Increase to 30%
        - setWeight: 30
        - pause: {duration: 5m}
        
        # Analysis at 30%
        - analysis:
            templates:
              - templateName: agent-success-rate
        
        # Increase to 60%
        - setWeight: 60
        - pause: {duration: 5m}
        
        # Final analysis before 100%
        - analysis:
            templates:
              - templateName: agent-success-rate
              - templateName: agent-latency
        
        # Full rollout
        - setWeight: 100
      
      # Automatic rollback
      autoPromotionEnabled: true
      
      # Traffic routing
      trafficRouting:
        istio:
          virtualService:
            name: customer-support
            routes:
              - primary
```

### Analysis Template

```yaml
# argocd/analysis-templates/agent-success-rate.yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: agent-success-rate
  namespace: agentstack
spec:
  args:
    - name: agent-name
  metrics:
    - name: success-rate
      interval: 1m
      count: 5
      successCondition: result[0] >= 0.95
      failureLimit: 2
      provider:
        prometheus:
          address: http://prometheus:9090
          query: |
            sum(rate(http_requests_total{
              app="{{args.agent-name}}",
              status=~"2.."
            }[5m])) 
            / 
            sum(rate(http_requests_total{
              app="{{args.agent-name}}"
            }[5m]))
    
    - name: error-rate
      interval: 1m
      count: 5
      successCondition: result[0] <= 0.01
      failureLimit: 2
      provider:
        prometheus:
          address: http://prometheus:9090
          query: |
            sum(rate(http_requests_total{
              app="{{args.agent-name}}",
              status=~"5.."
            }[5m])) 
            / 
            sum(rate(http_requests_total{
              app="{{args.agent-name}}"
            }[5m]))
```

## Evaluation Script

```python
# eval/run_evaluation.py
import argparse
import json
import subprocess
import mlflow
from typing import Dict, Any

def run_evaluation(
    agent_image: str,
    dataset_path: str,
    output_file: str
) -> Dict[str, Any]:
    """Run MLflow evaluation against agent container."""
    
    # Start agent container
    container_id = subprocess.check_output([
        "docker", "run", "-d", "-p", "8080:8080",
        agent_image
    ]).decode().strip()
    
    try:
        # Wait for agent to be ready
        wait_for_health("http://localhost:8080/health")
        
        # Load evaluation dataset
        dataset = load_dataset(dataset_path)
        
        # Run MLflow evaluation
        with mlflow.start_run():
            results = mlflow.evaluate(
                model=agent_model_wrapper,
                data=dataset,
                model_type="databricks-agent",
                evaluators="default",
                evaluator_config={
                    "databricks-agent": {
                        "metrics": [
                            "safety",
                            "groundedness", 
                            "relevance",
                            "chunk_relevance"
                        ]
                    }
                }
            )
            
            # Calculate aggregate scores
            scores = {
                "safety": results.metrics.get("safety/average", 0),
                "quality": calculate_quality_score(results),
                "relevance": results.metrics.get("relevance/average", 0),
            }
            
            # Log to MLflow
            mlflow.log_metrics(scores)
            
            # Save results
            output = {
                "scores": scores,
                "details": results.to_dict(),
                "passed": all([
                    scores["safety"] >= 0.95,
                    scores["quality"] >= 0.80
                ])
            }
            
            with open(output_file, "w") as f:
                json.dump(output, f, indent=2)
            
            return output
    
    finally:
        # Cleanup
        subprocess.run(["docker", "stop", container_id])
        subprocess.run(["docker", "rm", container_id])

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--agent-image", required=True)
    parser.add_argument("--dataset", required=True)
    parser.add_argument("--output-file", default="results.json")
    
    args = parser.parse_args()
    run_evaluation(args.agent_image, args.dataset, args.output_file)
```

## Notification Integration

```yaml
# .github/workflows/notify.yaml
name: Deployment Notifications

on:
  workflow_run:
    workflows: ["Agent Deployment Pipeline"]
    types: [completed]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Slack notification
        uses: slackapi/slack-github-action@v1
        with:
          channel-id: 'deployments'
          payload: |
            {
              "text": "${{ github.workflow }} ${{ github.event.workflow_run.conclusion }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*${{ github.repository }}* deployment ${{ github.event.workflow_run.conclusion }}"
                  }
                },
                {
                  "type": "section",
                  "fields": [
                    {
                      "type": "mrkdwn",
                      "text": "*Commit:*\n<${{ github.event.workflow_run.head_commit.url }}|${{ github.event.workflow_run.head_sha }}>"
                    },
                    {
                      "type": "mrkdwn",
                      "text": "*Author:*\n${{ github.event.workflow_run.head_commit.author.name }}"
                    }
                  ]
                }
              ]
            }
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

## Rollback Procedure

```yaml
# .github/workflows/rollback.yaml
name: Emergency Rollback

on:
  workflow_dispatch:
    inputs:
      agent:
        description: 'Agent to rollback'
        required: true
      revision:
        description: 'Revision to rollback to (leave empty for previous)'
        required: false

jobs:
  rollback:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Configure kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Rollback deployment
        run: |
          if [ -z "${{ github.event.inputs.revision }}" ]; then
            kubectl rollout undo deployment/${{ github.event.inputs.agent }} \
              -n agentstack
          else
            kubectl rollout undo deployment/${{ github.event.inputs.agent }} \
              --to-revision=${{ github.event.inputs.revision }} \
              -n agentstack
          fi
      
      - name: Wait for rollback
        run: |
          kubectl rollout status deployment/${{ github.event.inputs.agent }} \
            -n agentstack --timeout=5m
      
      - name: Verify health
        run: |
          # Wait and check health
          sleep 30
          kubectl get pods -l app=${{ github.event.inputs.agent }} \
            -n agentstack -o json | \
            jq '.items[] | select(.status.phase != "Running")'
```

## Resources

- `references/github-actions-patterns.md` - Advanced GHA patterns
- `references/argocd-patterns.md` - ArgoCD best practices
