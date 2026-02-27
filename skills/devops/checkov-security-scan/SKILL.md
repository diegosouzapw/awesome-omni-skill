---
name: checkov-security-scan
description: Scan Infrastructure as Code (IaC) for security misconfigurations and compliance violations using Checkov. (1) Primary use for Terraform, CloudFormation, Kubernetes manifests, Dockerfiles, Helm charts, ARM/Bicep templates, GitHub Actions, GitLab CI, and CI/CD pipelines. (2) Detects cloud misconfigurations, exposed secrets, overly permissive IAM policies, unencrypted storage, public access risks, container security issues. (3) Use for IaC security audits, compliance scanning (CIS, SOC2, HIPAA, PCI-DSS), pre-deployment validation, CI/CD security gates. Do NOT use for application source code vulnerabilities (use bandit, graudit) or dependency/package audits (use guarddog, dependency-check).
---

# Checkov Security Scanning Skill

This skill enables scanning Infrastructure as Code (IaC) for security misconfigurations and compliance violations using **Checkov** - an open-source static analysis tool by Bridgecrew/Palo Alto Networks that covers 20+ frameworks with 1000+ built-in security checks.

> **Key Distinction**: Checkov detects **IaC misconfigurations** (insecure cloud resource configurations). For **application source code** vulnerabilities, use `bandit` (Python) or `graudit` (multi-language). For **dependency audits**, use `guarddog` or `dependency-check`.

## Quick Reference

| Task | Command |
|------|---------|
| Scan a directory | `checkov -d /path/to/iac/` |
| Scan a specific file | `checkov -f main.tf` |
| Scan Terraform only | `checkov -d . --framework terraform` |
| Scan Kubernetes manifests | `checkov -d ./k8s --framework kubernetes` |
| Scan Dockerfile | `checkov -f Dockerfile --framework dockerfile` |
| JSON output for automation | `-o json` |
| SARIF output for CI/CD | `-o sarif` |
| List all available checks | `checkov --list` |
| Skip specific checks | `--skip-check CKV_AWS_1,CKV_DOCKER_7` |
| Fail only on HIGH/CRITICAL | `--hard-fail-on HIGH,CRITICAL` |

## When to Use This Skill

**PRIMARY USE CASES:**
- Audit Terraform configurations for AWS, Azure, GCP, and other cloud providers
- Scan Kubernetes manifests for container security issues
- Check Dockerfiles for security best practices
- Validate CloudFormation templates before deployment
- Scan Helm charts and Kustomize configurations
- Audit GitHub Actions, GitLab CI, and other CI/CD workflows
- Compliance scanning against CIS benchmarks, SOC2, HIPAA, PCI-DSS
- Pre-commit/pre-deployment security gates

**DO NOT USE FOR:**
- Application source code vulnerabilities → use `bandit` (Python) or `graudit`
- Dependency/package audits → use `guarddog` or `dependency-check`
- Shell script security → use `shellcheck`
- Runtime security scanning → use specialized runtime tools

## Decision Tree: Choosing the Right Scan

```
What are you scanning?
│
├── Infrastructure as Code (IaC)?
│   ├── Terraform (.tf, .tf.json) → checkov -d . --framework terraform
│   ├── Kubernetes manifests → checkov -d . --framework kubernetes
│   ├── Dockerfile → checkov -f Dockerfile --framework dockerfile
│   ├── CloudFormation (.yaml/.json) → checkov -d . --framework cloudformation
│   ├── Helm charts → checkov -d . --framework helm
│   ├── ARM/Bicep → checkov -d . --framework arm,bicep
│   └── GitHub Actions → checkov -d . --framework github_actions
│
├── CI/CD configuration?
│   ├── GitHub Actions → checkov --framework github_actions
│   ├── GitLab CI → checkov --framework gitlab_ci
│   ├── Azure Pipelines → checkov --framework azure_pipelines
│   └── CircleCI/BitBucket → checkov --framework circleci_pipelines,bitbucket_pipelines
│
├── Python/JavaScript source code?
│   └── Use bandit or graudit instead
│
└── Third-party dependencies?
    └── Use guarddog or dependency-check instead
```

## Prerequisites

Checkov must be installed. If not available, install it:

```bash
# Install via pip (recommended)
pip install checkov

# Or use pipx for isolated install
pipx install checkov

# Or use Homebrew (macOS/Linux)
brew install checkov

# Or use Docker (isolated, no local install)
docker pull bridgecrew/checkov
alias checkov='docker run --tty --rm --volume "$(pwd):/tf" --workdir /tf bridgecrew/checkov'

# Verify installation
checkov --version
```

**Troubleshooting Installation:**
```bash
# If pip install fails, ensure Python 3.8+
python --version

# If permissions issues occur
pip install --user checkov

# Check if checkov is in PATH
which checkov || echo "Add to PATH or use full path"
```

## Core Scanning Commands

### Scan a Directory

```bash
# Scan all IaC files in a directory (auto-detects frameworks)
checkov -d /path/to/infrastructure/

# Scan current directory
checkov -d .

# Scan with compact output (no code blocks)
checkov -d . --compact
```

### Scan Specific Files

```bash
# Scan a single Terraform file
checkov -f main.tf

# Scan multiple files
checkov -f main.tf -f variables.tf -f outputs.tf

# Scan a Dockerfile
checkov -f Dockerfile --framework dockerfile
```

### Framework-Specific Scans

```bash
# Terraform only
checkov -d . --framework terraform

# Kubernetes manifests
checkov -d ./k8s --framework kubernetes

# CloudFormation templates
checkov -d ./cfn --framework cloudformation

# Dockerfile
checkov -f Dockerfile --framework dockerfile

# Helm charts
checkov -d ./helm-chart --framework helm

# GitHub Actions workflows
checkov -d .github/workflows --framework github_actions

# Multiple frameworks
checkov -d . --framework terraform,kubernetes,dockerfile
```

### Output Formats

```bash
# Default CLI output
checkov -d .

# JSON output for parsing
checkov -d . -o json

# SARIF for GitHub/GitLab Security
checkov -d . -o sarif

# JUnit XML for CI/CD test reporting
checkov -d . -o junitxml

# Multiple outputs simultaneously
checkov -d . -o cli -o json -o sarif --output-file-path console,results.json,results.sarif

# CycloneDX SBOM format
checkov -d . -o cyclonedx_json
```

## Available Check Categories

Checkov includes 1000+ built-in checks across multiple categories.

### Check ID Prefixes by Provider

| Prefix | Provider/Category | Example Checks |
|--------|-------------------|----------------|
| `CKV_AWS_*` | Amazon Web Services | IAM, S3, EC2, RDS, Lambda |
| `CKV_AZURE_*` | Microsoft Azure | VMs, Storage, AKS, SQL |
| `CKV_GCP_*` | Google Cloud Platform | GKE, Cloud Storage, IAM |
| `CKV_DOCKER_*` | Dockerfiles | Image security, USER directive |
| `CKV_K8S_*` | Kubernetes | Pod security, RBAC, network policies |
| `CKV_GHA_*` | GitHub Actions | Workflow security, secrets |
| `CKV_GITLABCI_*` | GitLab CI | Pipeline security |
| `CKV_SECRET_*` | Secrets Detection | Hardcoded credentials |
| `CKV_ALI_*` | Alibaba Cloud | Alibaba-specific resources |
| `CKV_OCI_*` | Oracle Cloud | OCI resources |
| `CKV_TF_*` | Terraform General | Provider-agnostic checks |
| `CKV_OPENAPI_*` | OpenAPI/Swagger | API specification security |
| `CKV_ANSIBLE_*` | Ansible | Playbook security |
| `CKV_ARGO_*` | Argo Workflows | Workflow security |

### Security Check Categories

| Category | Description | MITRE ATT&CK |
|----------|-------------|--------------|
| **Encryption** | Data at rest/transit encryption | T1530 |
| **Access Control** | IAM policies, RBAC, authentication | T1078 |
| **Network Security** | Security groups, firewalls, ingress/egress | T1046, T1190 |
| **Public Access** | Publicly accessible resources | T1190 |
| **Logging & Monitoring** | Audit logs, CloudTrail, CloudWatch | T1562 |
| **Secrets Management** | Hardcoded credentials, API keys | T1552 |
| **Container Security** | Privileged containers, root users | T1610 |
| **Compliance** | CIS benchmarks, SOC2, HIPAA, PCI-DSS | - |

### Example Checks by Framework

**Terraform (AWS):**
| Check ID | Description |
|----------|-------------|
| `CKV_AWS_1` | Ensure IAM policies don't allow full "*-*" admin privileges |
| `CKV_AWS_2` | Ensure ALB protocol is HTTPS |
| `CKV_AWS_3` | Ensure EBS is encrypted by default |
| `CKV_AWS_18` | Ensure S3 access logging is enabled |
| `CKV_AWS_19` | Ensure S3 bucket has server-side encryption |
| `CKV_AWS_20` | Ensure S3 bucket is not publicly readable |
| `CKV_AWS_21` | Ensure S3 bucket versioning is enabled |
| `CKV_AWS_23` | Ensure every security group has a description |
| `CKV_AWS_24` | Ensure no security group allows ingress from 0.0.0.0/0 to port 22 |
| `CKV_AWS_25` | Ensure no security group allows ingress from 0.0.0.0/0 to port 3389 |

**Kubernetes:**
| Check ID | Description |
|----------|-------------|
| `CKV_K8S_1` | Do not admit containers sharing host PID namespace |
| `CKV_K8S_2` | Do not admit privileged containers |
| `CKV_K8S_3` | Do not admit containers running as root |
| `CKV_K8S_8` | Ensure liveness probe is configured |
| `CKV_K8S_9` | Ensure readiness probe is configured |
| `CKV_K8S_21` | Default namespace should not be used |
| `CKV_K8S_22` | Use read-only filesystem for containers |
| `CKV_K8S_28` | Ensure seccomp profile is set |
| `CKV_K8S_35` | Prefer using secrets for sensitive environment variables |

**Dockerfile:**
| Check ID | Description |
|----------|-------------|
| `CKV_DOCKER_1` | Ensure port 22 (SSH) is not exposed |
| `CKV_DOCKER_2` | Ensure HEALTHCHECK instructions are added |
| `CKV_DOCKER_3` | Ensure USER instruction is added (non-root) |
| `CKV_DOCKER_7` | Ensure base image uses non-latest tag |
| `CKV_DOCKER_8` | Ensure package manager caches are cleaned |
| `CKV_DOCKER_9` | Ensure COPY is used instead of ADD |
| `CKV_DOCKER_10` | Ensure secrets are not stored in Dockerfile |
| `CKV_DOCKER_11` | Ensure base image is pinned using digest |

**GitHub Actions:**
| Check ID | Description |
|----------|-------------|
| `CKV_GHA_1` | Ensure run step does not use shell injection |
| `CKV_GHA_2` | Ensure actions use pinned SHA |
| `CKV_GHA_3` | Ensure workflow run is not triggered on workflow_run |
| `CKV_GHA_4` | Ensure self-hosted runners are not used |
| `CKV_GHA_7` | Ensure ACTIONS_ALLOW_UNSECURE_COMMANDS is not set |
| `CKV2_GHA_1` | Ensure top-level permissions are read-only |

## Selective Check Scanning

```bash
# Run only specific checks
checkov -d . -c CKV_AWS_1,CKV_AWS_2,CKV_AWS_3

# Run checks by type
checkov -d . --check-type terraform

# Skip specific checks
checkov -d . --skip-check CKV_AWS_1,CKV_DOCKER_7

# Skip paths
checkov -d . --skip-path tests/ --skip-path .terraform/

# List all available checks
checkov --list

# Filter checks by framework
checkov --list --framework terraform
```

## Severity Filtering

```bash
# Only fail on HIGH and CRITICAL severity
checkov -d . --hard-fail-on HIGH,CRITICAL

# Soft fail on LOW and MEDIUM (exit 0)
checkov -d . --soft-fail-on LOW,MEDIUM

# Show only failed checks
checkov -d . --quiet

# Always exit 0 (soft fail all)
checkov -d . --soft-fail
```

## Configuration File

Create `.checkov.yaml` in your project root:

```yaml
# Framework selection
framework:
  - terraform
  - dockerfile
  - kubernetes

# Skip specific checks
skip-check:
  - CKV_AWS_1
  - CKV_DOCKER_7

# Skip paths
skip-path:
  - tests/
  - .terraform/
  - examples/

# Output settings
output: cli
compact: true
quiet: false

# Severity configuration
soft-fail-on:
  - LOW
  - MEDIUM

hard-fail-on:
  - HIGH
  - CRITICAL

# Terraform settings
download-external-modules: true
external-modules-download-path: .external_modules
evaluate-variables: true
var-file:
  - variables.tfvars

# For CI/CD
soft-fail: false
```

Create config from CLI args:
```bash
checkov --create-config .checkov.yaml
```

## Workflow for Security Audit

### Quick Scan (1-2 minutes)

```bash
# Quick scan of IaC directory
checkov -d . --compact

# Focus on critical issues only
checkov -d . --hard-fail-on HIGH,CRITICAL --quiet
```

### Comprehensive Audit (3-5 minutes)

```bash
# Step 1: Full scan with all frameworks
checkov -d . -o cli -o json --output-file-path console,full-report.json

# Step 2: Review critical findings
checkov -d . --hard-fail-on CRITICAL --quiet

# Step 3: Framework-specific deep dive
checkov -d ./terraform --framework terraform -o cli
checkov -d ./k8s --framework kubernetes -o cli
checkov -f Dockerfile --framework dockerfile -o cli
```

### Pre-Deployment Validation

```bash
# Scan Terraform plan (most accurate for runtime config)
terraform init
terraform plan -out=tfplan
terraform show -json tfplan > tfplan.json
checkov -f tfplan.json --framework terraform_plan

# Scan with external modules
checkov -d . --download-external-modules true
```

## CI/CD Integration

### GitHub Actions

```yaml
name: IaC Security Scan
on: [push, pull_request]

jobs:
  checkov:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Checkov
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: .
          framework: terraform,dockerfile,kubernetes
          output_format: cli,sarif
          output_file_path: console,results.sarif
          soft_fail: false
          
      - name: Upload SARIF
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: results.sarif
```

### GitLab CI

```yaml
checkov:
  image: bridgecrew/checkov:latest
  script:
    - checkov -d . -o cli -o gitlab_sast --output-file-path console,gl-sast-report.json
  artifacts:
    reports:
      sast: gl-sast-report.json
```

### Pre-commit Hook

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/bridgecrewio/checkov
    rev: '3.2.500'
    hooks:
      - id: checkov
        args:
          - --framework=terraform,dockerfile
```

## Interpreting Results

### Output Format

```
Passed checks: 42, Failed checks: 3, Skipped checks: 0

Check: CKV_AWS_20: "Ensure S3 bucket is not publicly readable"
        FAILED for resource: aws_s3_bucket.public_bucket
        File: /main.tf:15-25
        Guide: https://docs.prismacloud.io/en/...

                15 | resource "aws_s3_bucket" "public_bucket" {
                16 |   bucket = "my-public-bucket"
                17 |   acl    = "public-read"  # <- Issue here
                18 | }
```

### Severity Assessment

| Severity | Action Required |
|----------|------------------|
| **CRITICAL** | Immediate fix before deployment |
| **HIGH** | Fix before production |
| **MEDIUM** | Fix in next sprint |
| **LOW** | Best practice improvement |

### Verification Checklist

For each finding, verify:
- [ ] Is this a real misconfiguration, not intentional?
- [ ] What is the blast radius if exploited?
- [ ] Is there a compensating control?
- [ ] What compliance frameworks does this violate?

## Combining with Other Security Tools

| Tool | Use For | Command |
|------|---------|---------|
| **Checkov** | IaC misconfigurations | `checkov -d .` |
| **Bandit** | Python code vulnerabilities | `bandit -r ./src` |
| **Graudit** | Multi-language secrets/patterns | `graudit -d secrets ./` |
| **GuardDog** | Malicious dependencies | `guarddog pypi verify requirements.txt` |
| **ShellCheck** | Shell script security | `shellcheck *.sh` |
| **Trivy** | Container image scanning | `trivy image myimage:latest` |

### Recommended Full Audit Workflow

```bash
# 1. Scan IaC configurations (Checkov)
checkov -d . --framework terraform,kubernetes,dockerfile

# 2. Scan Python code for vulnerabilities (Bandit)
bandit -r ./src -f json -o bandit-results.json

# 3. Check for hardcoded secrets (Graudit)
graudit -d secrets ./

# 4. Audit dependencies (GuardDog)
guarddog pypi verify requirements.txt

# 5. Scan container images (Trivy)
trivy image --severity HIGH,CRITICAL myimage:latest
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `BC_API_KEY` | Prisma Cloud API key for enhanced checks |
| `CKV_FRAMEWORK` | Default frameworks to scan |
| `CKV_CHECK` | Default checks to run |
| `CKV_SKIP_CHECK` | Default checks to skip |
| `CKV_EVAL_VARS` | Evaluate Terraform variables |
| `DOWNLOAD_EXTERNAL_MODULES` | Download Terraform modules |
| `CHECKOV_CREATE_GRAPH` | Enable graph-based checks |

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| `command not found: checkov` | Run `pip install checkov` or check PATH |
| Slow scan with external modules | Set `--external-modules-download-path` to cache |
| Too many false positives | Use `--skip-check` or create `.checkov.yaml` |
| Memory issues on large repos | Scan specific directories instead of root |
| Missing Terraform modules | Use `--download-external-modules true` |

### Performance Optimization

```bash
# Scan only changed files in CI
checkov -f $(git diff --name-only HEAD~1 | grep -E '\.(tf|yaml|yml|json)$')

# Use baseline for existing issues
checkov -d . --create-baseline
checkov -d . --baseline .checkov.baseline

# Cache external modules
checkov -d . --download-external-modules true --external-modules-download-path .modules_cache
```

## Limitations

- Checkov performs **static analysis** - cannot detect runtime misconfigurations
- Some checks may produce **false positives** in complex configurations
- Terraform plan scanning requires `terraform init` to be run first
- External module scanning requires network access to download modules
- Does not scan application source code - use bandit/graudit for that
- Secrets detection is pattern-based - use dedicated secret scanners for comprehensive coverage
- Always validate critical findings with manual review

## Additional Resources

- [Misconfiguration Patterns Examples](./examples/misconfiguration-patterns.md) - Example IaC patterns Checkov detects
- [Checkov GitHub Repository](https://github.com/bridgecrewio/checkov) - Official documentation
- [Bridgecrew Documentation](https://docs.prismacloud.io/en/enterprise-edition/content-collections/application-security) - Enterprise features
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks) - Compliance reference

```
