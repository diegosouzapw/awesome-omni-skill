---
name: provisioning-bedrock
description: Provisions AWS Bedrock API keys for business units. Use when users need AWS Bedrock API keys, tokens, or credentials. Supports short-term tokens (12h, auto-refresh) for humans and long-term IAM keys for systems.
---

You have access to the AWS MCP Server (`aws`) and Bash tools. Use this skill when users need AWS Bedrock API keys.

## Contents

- [Overview](#overview) - What this skill does
- [User Experience](#user-experience) - Dead simple provisioning
- [API Key Types](#api-key-types) - Short-term vs Long-term
- [Natural Language Parsing](#natural-language-parsing) - Extract parameters from user input
- [Workflows](#workflows) - Step-by-step provisioning processes
- [Output Formats](#output-formats) - Minimal, shareable results
- [Critical Rules](#critical-rules) - Security and idempotency
- [Configuration](#configuration) - Required environment variables

## Overview

This skill provisions AWS Bedrock API keys with:
- **Short-term tokens** (recommended): 12-hour Bedrock API keys via `aws-bedrock-token-generator`
- **Long-term IAM keys** (systems only): Traditional AWS access keys for automation
- Automatic model access verification (Claude models in us-east-1)
- Resource tagging for cost tracking
- Ready-to-use environment configuration

**Before this skill:**
- Users manually navigate AWS Console or write complex scripts
- Choose between multiple authentication methods without guidance
- Manually set up IAM users, policies, and credentials
- Understanding of AWS IAM, Bedrock, STS required

**After this skill:**
```
User: I need an API key for AWS Bedrock

Claude: ✓ Bedrock API key generated (expires in 12 hours)

[Shareable setup snippet with auto-refresh example]
```

## User Experience

**Design Principles:**
- **Dead simple** - One command, fully automated
- **Smart defaults** - Short-term tokens for most use cases
- **Minimal output** - Only essential information
- **Shareable snippets** - Copy/paste ready environment setup
- **Secure by default** - API keys handled securely, rotation guidance

## API Key Types

### When to Use Each Type

**Short-term Tokens (RECOMMENDED - 90% of use cases):**
✅ Use for:
- Interactive development and testing
- Individual developer access
- Temporary projects
- Learning and experimentation
- Any human user

❌ Don't use for:
- Long-running production servers
- Automated CI/CD pipelines
- Services that run >12 hours without restart

**Long-term IAM Keys (Systems and Automation):**
✅ Use for:
- Production servers and services
- CI/CD pipelines
- Automated batch jobs
- Long-running applications
- System-to-system integration

❌ Don't use for:
- Individual developer access
- Interactive sessions
- Temporary testing

### Short-term Tokens (Bedrock API Keys)

**Properties:**
- Valid for up to 12 hours (configurable: 15 min - 12 hours)
- Actual expiry = min(requested_ttl, AWS_credentials_expiry)
- Uses existing AWS credentials (no new IAM user needed)
- Auto-renewable via `aws-bedrock-token-generator`
- Format: `bedrock-api-key-<base64-encoded-presigned-url>&Version=1`
- **AWS recommends this over long-term keys**

**Configurable TTL:**
```python
from datetime import timedelta
from aws_bedrock_token_generator import provide_token

# 15 minutes (minimal)
token = provide_token(expiry=timedelta(minutes=15))

# 6 hours (medium)
token = provide_token(expiry=timedelta(hours=6))

# 12 hours (maximum, default)
token = provide_token()  # or expiry=timedelta(hours=12)
```

**Auto-Refresh:**
```python
# Token generator handles expiry automatically
# Just call provide_token() for each request or when needed
from aws_bedrock_token_generator import provide_token

def make_bedrock_call():
    token = provide_token()  # Gets fresh token if expired
    # Use token for API call...
```

### Long-term IAM Access Keys

**Properties:**
- Never expire (unless rotated manually)
- Requires creating IAM user
- Traditional AWS_ACCESS_KEY_ID + AWS_SECRET_ACCESS_KEY
- Works for all AWS services (not Bedrock-specific)
- Requires `iam:CreateAccessKey` permission

**Security Warning:**
⚠️ Long-term keys have higher security risk. Rotate every 30-90 days minimum.

## Natural Language Parsing

Extract these parameters from user input:

**API Key Type (auto-detect):**
- Patterns:
  - "for production" / "for CI/CD" / "for automation" → long-term
  - "for development" / "quick test" / "try out" → short-term
  - "system" / "server" / "service" → long-term
  - Default: short-term (if ambiguous)

**Business Unit (for long-term keys only):**
- Pattern: "for [business-unit]" or "[business-unit] access"
- Normalize: lowercase with hyphens (e.g., "Marketing Team" → "marketing-team")
- Used to name IAM user: `{business_unit}-bedrock-user`

**TTL (for short-term tokens only):**
- Pattern: "[number] hours" or "[number] minutes"
- Examples: "6 hours" → 6h, "30 minutes" → 30m
- Default: 12 hours (maximum)
- Range: 15 minutes - 12 hours

**Owner Email (optional, for long-term keys):**
- Pattern: "owner: [email]" or any email address in the input
- Used for: IAM tags and audit trail

**Region (optional):**
- Pattern: "in [region]" or "region: [region]"
- Default: us-east-1 (most model availability)

## Workflows

### Workflow 1: Generate Short-term Token (Default)

**Trigger Patterns:**
- "I need an API key for Bedrock"
- "Generate Bedrock API key"
- "Quick Bedrock access"
- "Bedrock API key for testing"

**Steps:**

1. **Install Token Generator (if not present)**
```bash
pip install aws-bedrock-token-generator
```

2. **Parse TTL from user input**
```
Default: 12 hours
If user says "6 hours" → timedelta(hours=6)
If user says "30 minutes" → timedelta(minutes=30)
```

3. **Generate Token**
```python
from datetime import timedelta
from aws_bedrock_token_generator import provide_token

# Use parsed TTL or default
token = provide_token(expiry=timedelta(hours=6))
```

4. **Format Output** (see Output Format section)

**Error Handling:**
- If `pip install` fails → Suggest using Bash to install with user's Python environment
- If AWS credentials not configured → Point to setup instructions
- If region not supported → List supported regions

### Workflow 2: Generate Long-term IAM Keys

**Trigger Patterns:**
- "Bedrock access for production"
- "Bedrock key for CI/CD"
- "Long-term Bedrock credentials"
- "Bedrock key for [business-unit]" (implies long-term)

**Steps:**

1. **Parse Business Unit**
```
User input: "bedrock access for production-api"
business_unit = "production-api"
iam_user_name = "production-api-bedrock-user"
```

2. **Check IAM User Existence**
```
Natural language: "Check if IAM user production-api-bedrock-user exists"

If exists → Ask if user wants new key or different name
If not exists → Continue to create
```

3. **Create IAM User (if needed)**
```
Natural language: "Create IAM user production-api-bedrock-user with tags business-unit=production-api, purpose=bedrock, created-by=provisioning-bedrock-skill"
```

4. **Attach Bedrock Policy**
```
Natural language: "Attach AmazonBedrockFullAccess policy to user production-api-bedrock-user"
```

5. **Create IAM Access Keys**
```
Natural language: "Create access key for IAM user production-api-bedrock-user"

Parse output to extract:
- AccessKeyId
- SecretAccessKey
```

6. **Format Output** (see Output Format section)

**Critical:**
- Long-term keys require additional permission: `iam:CreateAccessKey`
- Keys shown only once - user must save immediately

### Workflow 3: Rotate/Refresh Token

**Trigger Patterns:**
- "Refresh my Bedrock token"
- "New Bedrock API key"

**For Short-term Tokens:**
```
Just regenerate:
token = provide_token()

No rotation needed - old token expires automatically
```

**For Long-term IAM Keys:**
```
1. Create new access key
2. Provide to user
3. Ask if they want to delete old key (after testing new one)
4. Optionally delete old key
```

## Output Formats

### Short-term Token Output

```markdown
✓ Bedrock API key generated

**Type:** Short-term token
**Expires:** {expiration_datetime} ({hours} hours from now)
**Region:** {region}

## Quick Setup (copy/paste)
```bash
export AWS_BEARER_TOKEN_BEDROCK="{token}"
export AWS_REGION="{region}"
```

## Test Command (Python)
```bash
pip install aws-bedrock-token-generator anthropic

python -c "
from aws_bedrock_token_generator import provide_token
from anthropic import AnthropicBedrock
import os

# Set token in environment
os.environ['AWS_BEARER_TOKEN_BEDROCK'] = '{token}'

client = AnthropicBedrock(aws_region='{region}')
message = client.messages.create(
    model='anthropic.claude-sonnet-4-5-v2:0',
    max_tokens=100,
    messages=[{'role': 'user', 'content': 'Hello!'}]
)
print(message.content[0].text)
"
```

## Auto-Refresh (for long-running apps)
```python
from aws_bedrock_token_generator import provide_token
from anthropic import AnthropicBedrock
import os

def make_bedrock_call(prompt):
    # Automatically gets fresh token if expired
    token = provide_token()
    os.environ['AWS_BEARER_TOKEN_BEDROCK'] = token

    client = AnthropicBedrock(aws_region='{region}')
    return client.messages.create(
        model='anthropic.claude-sonnet-4-5-v2:0',
        max_tokens=100,
        messages=[{'role': 'user', 'content': prompt}]
    )

# Call this function whenever you need - token refreshes automatically
response = make_bedrock_call("Hello!")
```

## Extend TTL (configure expiry time)
```python
from datetime import timedelta
from aws_bedrock_token_generator import provide_token

# 30 minutes
token = provide_token(expiry=timedelta(minutes=30))

# 6 hours
token = provide_token(expiry=timedelta(hours=6))

# 12 hours (maximum)
token = provide_token(expiry=timedelta(hours=12))
```

## Available Models
- Claude Sonnet 4.5: anthropic.claude-sonnet-4-5-v2:0
- Claude Opus 4.5: anthropic.claude-opus-4-5-v1:0
- Claude Haiku 4.5: anthropic.claude-haiku-4-5-v1:0

## Security Notes
✓ Token expires automatically in {hours} hours
✓ Use `provide_token()` to get fresh token when expired
✓ For apps running >{hours} hours, use auto-refresh pattern above
✓ Never commit tokens to version control

## When to Use Long-term Keys Instead
If you need:
- Production server access (runs >12 hours)
- CI/CD pipeline integration
- Automated batch jobs
- System-to-system integration

→ Run: `/provisioning-bedrock create long-term key for [business-unit]`
```

### Long-term IAM Keys Output

```markdown
✓ Bedrock IAM access keys created for {Business Unit}

**Type:** Long-term IAM access keys
**IAM User:** {iam_user_name}
**Region:** {region}
**Access Key ID:** {access_key_id[:12]}...

⚠️ **CRITICAL: Save these credentials now - they cannot be retrieved later**

## Quick Setup (copy/paste)
```bash
export AWS_ACCESS_KEY_ID="{access_key_id}"
export AWS_SECRET_ACCESS_KEY="{secret_access_key}"
export AWS_REGION="{region}"
```

## Test Command (Python)
```bash
pip install anthropic

python -c "
from anthropic import AnthropicBedrock
import os

client = AnthropicBedrock(aws_region='{region}')
message = client.messages.create(
    model='anthropic.claude-sonnet-4-5-v2:0',
    max_tokens=100,
    messages=[{'role': 'user', 'content': 'Hello!'}]
)
print(message.content[0].text)
"
```

## Available Models
- Claude Sonnet 4.5: anthropic.claude-sonnet-4-5-v2:0
- Claude Opus 4.5: anthropic.claude-opus-4-5-v1:0
- Claude Haiku 4.5: anthropic.claude-haiku-4-5-v1:0

## Security Reminder
⚠️ **Important:** These are long-term credentials with high security risk.
- Store in AWS Secrets Manager or secure vault (1Password, Vault, etc.)
- NEVER commit to version control
- Rotate every 30-90 days minimum
- Set up AWS CloudTrail alerts for unusual activity
- Consider switching to short-term tokens if possible

## Rotation Instructions
After 30-90 days:
```bash
# Create new key
/provisioning-bedrock create long-term key for {business_unit}

# Update all systems with new credentials
# After confirming new key works:
aws iam delete-access-key --user-name {iam_user_name} --access-key-id {old_access_key_id}
```

## Cost Tracking
This IAM user is tagged with:
- business-unit: {business_unit}
- purpose: bedrock
- owner: {owner_email}
- created-by: provisioning-bedrock-skill

Query costs in AWS Cost Explorer by filtering on these tags.

## Why Short-term Tokens Are Better
Short-term tokens (12-hour, auto-refresh):
✓ More secure (automatic expiration)
✓ No IAM user management overhead
✓ Auto-renewable (set and forget)
✓ AWS recommended for most use cases

Unless you specifically need long-running server credentials,
consider using: `/provisioning-bedrock create token`
```

## Critical Rules

**Security:**
- ✓ ALWAYS recommend short-term tokens unless user explicitly needs long-term
- ✓ NEVER log full credentials in any output visible in chat
- ✓ ALWAYS provide full credentials in shareable snippet only
- ✓ ALWAYS warn about security implications of long-term keys
- ✓ For short-term: Emphasize auto-refresh capability
- ✓ For long-term: Emphasize rotation requirements

**API Key Type Selection:**
- ✓ Default to short-term tokens for ambiguous requests
- ✓ Ask "Is this for a production server or CI/CD?" if unclear
- ✓ If user says "for development" → short-term
- ✓ If user says "for production" → ask if it's long-running (>12h)

**TTL Configuration:**
- ✓ For short-term: Default to 12 hours (maximum)
- ✓ Support configurable TTL (15min - 12h) via expiry parameter
- ✓ Explain that actual expiry = min(requested, AWS_credentials_expiry)

**Output:**
- ✓ ALWAYS provide minimal output (not overwhelming)
- ✓ ALWAYS include shareable snippet (copy/paste ready)
- ✓ ALWAYS show auto-refresh example for short-term tokens
- ✓ ALWAYS show rotation instructions for long-term keys
- ✓ ALWAYS link to troubleshooting documentation

**Idempotency (Long-term keys only):**
- ✓ ALWAYS check if IAM user exists before creating
- ✓ NEVER create duplicate IAM users
- ✓ If user exists, offer to create additional access key
- ✓ Safe to re-run command multiple times

## Configuration

**Required AWS Credentials:**

The skill requires AWS credentials to be configured:

**Setup AWS Credentials (one-time):**
```bash
# Option 1: AWS CLI configure
aws configure
# Enter: Access Key ID, Secret Access Key, Region (us-east-1), Output format (json)

# Option 2: Environment variables
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_REGION="us-east-1"

# Option 3: AWS CLI profiles
aws configure --profile bedrock-provisioner
export AWS_PROFILE="bedrock-provisioner"
```

**Required IAM Permissions:**

**For Short-term Tokens:**
- `bedrock:InvokeModel` (inherited from existing IAM user/role)
- `bedrock:ListFoundationModels`
- `sts:GetCallerIdentity`

**For Long-term IAM Keys:**
- `iam:CreateUser`
- `iam:GetUser`
- `iam:TagUser`
- `iam:AttachUserPolicy`
- `iam:CreateAccessKey`
- `iam:ListAccessKeys`
- `iam:DeleteAccessKey`
- `bedrock:ListFoundationModels`
- `bedrock:InvokeModel`

**Validation:**
Before running provisioning commands:
```bash
# Test AWS credentials
aws sts get-caller-identity

# Test Bedrock access
aws bedrock list-foundation-models --region us-east-1

# Should see Claude models in output
```

## Prerequisites

**For Short-term Tokens:**
- ✓ AWS credentials configured (`aws sts get-caller-identity` works)
- ✓ Python 3.7+ (for aws-bedrock-token-generator)
- ✓ pip install access

**For Long-term IAM Keys:**
- ✓ AWS CLI installed (`aws --version`)
- ✓ AWS credentials configured with IAM permissions
- ✓ Permission to create IAM users and access keys

**First-time Bedrock setup:**
If user has never used Bedrock before:
1. Go to AWS Console → Bedrock → Model Access
2. Request access to Anthropic models
3. Wait for approval (usually instant)

**Check before provisioning:**
```
Natural language: "Check my current AWS identity"
Natural language: "List available Bedrock models in us-east-1"
```

## Further Reading

- [COMPARISON.md](COMPARISON.md) - AWS Bedrock vs GCP Vertex AI comparison
- [EXAMPLES.md](EXAMPLES.md) - Provisioning scenarios with step-by-step flows
- [SECURITY.md](SECURITY.md) - API key safety and best practices
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common issues and solutions
- [AWS Bedrock Token Generator](https://github.com/aws/aws-bedrock-token-generator-python) - Official AWS library
