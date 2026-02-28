---
name: security
description: Generate Tekton Task that uses Trivy to scan generated container for vulnerabilities.
metadata:
  version: "0.1"
  author: "Alex Soto"
  standard: "agentskills.io"
---

# Vulnerabilities Scan

Generate a Tekton task that uses Trivy to scan a container image and abort the pipeline in case of critical or high vulnerabilities.

## Instructions for Bob

### Step 1: Generate a Tekton Task using Trivy 

The task should receive as parameter the container image name.
The task should only fail if the container image contains CRITICAL or HIGH vulnerabilities.

1. Use the `aquasec/trivy:0.50.0` container to run the `trivy` command.
2. The `exit-code` should be 1 when a vulnerability is found.

The following snippet shows an example:

```yaml
apiVersion: tekton.dev/v1
kind: Task
metadata:
  name: trivy-scan
spec:
  params:
    - name: image-name
      type: string
  steps:
    - name: scan
      image: aquasec/trivy:0.50.0
      script: |
        trivy image --severity CRITICAL,HIGH \
          --exit-code 1 \
          $(params.image-name):$(git rev-parse --short HEAD)

```

## Critical
- Use only trivy tool for vulnerabilities