---
name: aws-iam
description: "AWS security patterns: IAM policy design, least-privilege principles, Secrets Manager vs Parameter Store, KMS envelope encryption, IRSA/OIDC for Kubernetes, and compliance checklists. Triggered by: IAM, least privilege, Secrets Manager, Parameter Store, KMS, IRSA, OIDC, permission boundary, SCP, CloudTrail, compliance, secret rotation."
user-invocable: true
argument-hint: "[policy|secrets|kms|irsa|audit]"
---

# AWS IAM

Security patterns for IAM policy design, secrets management, encryption, and compliance. Covers preventive controls (IAM) rather than detective controls (GuardDuty, Security Hub — see `security-review` skill).

---

## When to Use

- Designing IAM policies with least privilege
- Choosing between Secrets Manager and Parameter Store
- Configuring KMS CMKs, key policies, and envelope encryption
- Setting up IRSA (IAM Roles for Service Accounts) for EKS
- Establishing OIDC trust for GitHub Actions or other CI systems
- Auditing IAM posture against CIS AWS Foundations

**Related:** `/aws` (credential chain, SDK setup) | `security-review` skill (runtime posture, GuardDuty)

---

## Quick Start

```
/aws-iam policy    # Least-privilege policy design, conditions, resource ARNs
/aws-iam secrets   # Secrets Manager vs Parameter Store decision + rotation setup
/aws-iam kms       # CMK design, key policies, envelope encryption patterns
/aws-iam irsa      # IAM Roles for Service Accounts (EKS) + OIDC providers
/aws-iam audit     # CIS AWS Foundations compliance checklist, CloudTrail verification
```

---

## Mode: policy

### Least-Privilege Design Process

1. Start with zero permissions. Add only what the workload demonstrably needs.
2. Use `aws:RequestedRegion` condition to restrict to your operating regions.
3. Scope `Resource` to specific ARNs — never `"*"` unless the action requires it (e.g., `s3:ListAllMyBuckets`).
4. Add `Condition` keys for time-based, IP-based, or MFA-based restrictions.
5. Validate with IAM Access Analyzer before attaching.

### Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadSpecificS3Bucket",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "us-east-1"
        }
      }
    }
  ]
}
```

### Policy Evaluation Logic

Evaluation order (deny wins at each step):
1. **Explicit Deny** in any policy → DENY (no exceptions)
2. **SCPs (Service Control Policies)** — org-level ceiling; must Allow
3. **Resource-based policy** — cross-account: both identity + resource policy must Allow
4. **Permission Boundaries** — max permissions ceiling for a role/user
5. **Identity-based policy (IAM)** — what the principal is allowed to do
6. **Session policies** — further restrict assumed-role sessions

### Common Policy Conditions

| Condition Key | Example Use |
|---------------|-------------|
| `aws:RequestedRegion` | Restrict to `["us-east-1", "eu-west-1"]` |
| `aws:SourceIp` | Lock to corporate IP ranges |
| `aws:MultiFactorAuthPresent` | Require MFA for sensitive actions |
| `aws:PrincipalTag/team` | Tag-based ABAC |
| `s3:prefix` | Restrict S3 access to a key prefix |
| `kms:ViaService` | Enforce KMS usage only from specific AWS service |

### Permission Boundaries

Use permission boundaries to delegate IAM administration without privilege escalation:

```json
{
  "Statement": [{
    "Effect": "Allow",
    "Action": "iam:CreateRole",
    "Resource": "*",
    "Condition": {
      "StringEquals": {
        "iam:PermissionsBoundary": "arn:aws:iam::ACCOUNT:policy/DeveloperBoundary"
      }
    }
  }]
}
```

IAM policy templates: `assets/policy-templates/`

Reference: `references/policy-design.md`

---

## Mode: secrets

### Decision: Secrets Manager vs Parameter Store

| Feature | Secrets Manager | Parameter Store |
|---------|----------------|-----------------|
| Automatic rotation | Yes (built-in Lambda rotation) | No (manual) |
| Pricing | $0.40/secret/month + $0.05/10K API calls | Free (standard) / $0.05/10K (advanced) |
| Cross-account access | Yes, via resource policy | Yes (advanced tier) |
| Binary secrets | Yes | No |
| Max size | 65KB | 4KB (standard) / 8KB (advanced) |
| **Use when** | Passwords, API keys needing rotation | Config values, feature flags, non-sensitive params |

**Rule of thumb:** Use Secrets Manager for anything that rotates or is high-sensitivity. Use Parameter Store for config that doesn't change often and doesn't need rotation.

### Injecting Secrets into Lambda

```python
# boto3 — fetch at cold start, cache for the execution environment's lifetime
import boto3
import json
from functools import lru_cache

@lru_cache(maxsize=None)
def get_secret(name: str) -> dict:
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=name)
    return json.loads(response['SecretString'])

def handler(event, context):
    creds = get_secret('prod/myapp/db')
    # Use creds['username'], creds['password']
```

**Avoid:** Reading secrets on every invocation — it adds latency and cost. Cache in the execution environment.

### Rotation Configuration

```bash
aws secretsmanager rotate-secret \
  --secret-id prod/myapp/db \
  --rotation-lambda-arn arn:aws:lambda:...:MyRotationLambda \
  --rotation-rules AutomaticallyAfterDays=30
```

AWS provides managed rotation Lambda functions for RDS, Redshift, and DocumentDB — prefer these over custom implementations.

Reference: `references/secrets-patterns.md`

---

## Mode: kms

### CMK vs AWS Managed Keys

| Key Type | Management | Cost | Use When |
|----------|-----------|------|----------|
| AWS Managed (`aws/s3`) | AWS manages rotation | Free | Default; no control needed |
| Customer Managed (CMK) | You manage policy + rotation | $1/key/month + $0.03/10K requests | Compliance, cross-account, audit trail, access revocation |
| Imported Key Material | You provide key bytes | $1/key/month | HSM or regulatory key source requirements |

### Envelope Encryption Pattern

```
Plaintext Data
  ↓ encrypted with
Data Key (DEK) ← generated by KMS per-encrypt call
  ↓ encrypted with
CMK (stored in KMS, never leaves KMS)

Store: { ciphertext_blob, encrypted_data_key }
Decrypt: KMS.Decrypt(encrypted_data_key) → plaintext DEK → decrypt ciphertext_blob
```

Use `GenerateDataKey` + local encryption for large data. Never call `Encrypt` on data >4KB — that's for encrypting small values like data keys.

### Key Policy Essentials

```json
{
  "Statement": [
    {
      "Sid": "EnableRootAccess",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::ACCOUNT:root"},
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "AllowServiceUse",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::ACCOUNT:role/MyAppRole"},
      "Action": ["kms:Decrypt", "kms:GenerateDataKey"],
      "Resource": "*",
      "Condition": {
        "StringEquals": {"kms:ViaService": "s3.us-east-1.amazonaws.com"}
      }
    }
  ]
}
```

**Warning:** Removing the root principal from a key policy locks you out permanently. Always keep a break-glass root statement.

Reference: `references/kms-patterns.md`

---

## Mode: irsa

### IRSA (IAM Roles for Service Accounts — EKS)

IRSA allows Kubernetes pods to assume IAM roles without node-level credentials.

**Setup steps:**

1. Create OIDC provider for the cluster:
```bash
eksctl utils associate-iam-oidc-provider \
  --cluster my-cluster \
  --approve
```

2. Create IAM role with OIDC trust policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Federated": "arn:aws:iam::ACCOUNT:oidc-provider/oidc.eks.REGION.amazonaws.com/id/CLUSTER_ID"
    },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": {
        "oidc.eks.REGION.amazonaws.com/id/CLUSTER_ID:sub": "system:serviceaccount:NAMESPACE:SERVICE_ACCOUNT_NAME",
        "oidc.eks.REGION.amazonaws.com/id/CLUSTER_ID:aud": "sts.amazonaws.com"
      }
    }
  }]
}
```

3. Annotate the Kubernetes ServiceAccount:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: my-namespace
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT:role/MyPodRole
```

4. Pods using this ServiceAccount automatically receive short-lived credentials via the projected token volume.

### GitHub Actions OIDC (no long-lived keys)

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::ACCOUNT:role/GitHubActionsRole
      aws-region: us-east-1
```

Trust policy for GitHub Actions:
```json
{
  "Condition": {
    "StringEquals": {
      "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
    },
    "StringLike": {
      "token.actions.githubusercontent.com:sub": "repo:ORG/REPO:ref:refs/heads/main"
    }
  }
}
```

OIDC policy templates: `assets/policy-templates/irsa-role-trust-policy.json`

Reference: `references/irsa-oidc.md`

---

## Mode: audit

### CIS AWS Foundations — Key Controls

| Control | Check | Remediation |
|---------|-------|-------------|
| 1.1 | Root account has no active access keys | Delete root access keys |
| 1.4 | Root account has MFA enabled | Enable MFA on root |
| 1.5 | IAM password policy: 14+ chars, mixed case, symbols | Set account password policy |
| 1.12 | No credentials unused >90 days | Remove inactive keys/passwords |
| 1.14 | Access keys rotated every 90 days | Rotate or migrate to roles |
| 2.1 | CloudTrail enabled in all regions | Enable CloudTrail with multi-region |
| 2.6 | S3 bucket access logging enabled | Enable server access logging |
| 3.x | CloudTrail log metric filters + alarms | Create metric filters for auth failures, policy changes |
| 4.1 | No security groups allow 0.0.0.0/0 on SSH/RDP | Restrict ingress rules |

### CloudTrail Compliance Setup

```bash
# Multi-region trail with S3 + CloudWatch integration
aws cloudtrail create-trail \
  --name org-trail \
  --s3-bucket-name my-cloudtrail-bucket \
  --is-multi-region-trail \
  --enable-log-file-validation \
  --cloud-watch-logs-log-group-arn arn:aws:logs:...

aws cloudtrail start-logging --name org-trail
```

### IAM Access Analyzer

```bash
# Find externally shared resources
aws accessanalyzer create-analyzer \
  --analyzer-name account-analyzer \
  --type ACCOUNT

aws accessanalyzer list-findings --analyzer-name account-analyzer
```

Run Access Analyzer before any IAM policy change that touches `Principal: *` or cross-account trust.

Reference: `references/compliance-checklist.md`

---

## Cross-References

| Topic | Skill |
|-------|-------|
| Credential chain, SDK credential loading | `/aws` |
| Lambda execution roles, ECS task roles | `/aws-serverless` |
| Runtime threat detection (GuardDuty, Security Hub) | `security-review` skill |
| CloudWatch log metric filters | `observability` skill |
| GitHub Actions OIDC pipeline setup | `cicd-pipeline` skill |
