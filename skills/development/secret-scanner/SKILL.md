---
name: secret-scanner
description: Scans files, repos, and directories for leaked secrets — API keys, tokens, passwords, connection strings, private keys, and credentials. Detects 40+ secret patterns across all major cloud providers and services.
version: 0.1.0
---

# Secret Scanner

Security skill that scans code, config files, and repos for accidentally leaked secrets and credentials.

## When to Use This Skill

Use this skill when the user:

- Asks to "check for leaked secrets" or "scan for API keys"
- Wants to audit a repo or folder before committing or publishing
- Says "are there any hardcoded passwords in this code?"
- Asks to "find credentials" or "check for exposed tokens"
- Wants pre-commit or pre-publish security checks
- Mentions concern about accidentally checking in secrets

## Capabilities

- Detect **40+ secret patterns** including:
  - AWS Access Keys, Secret Keys, Session Tokens
  - Azure Storage Keys, Connection Strings, SAS Tokens
  - GCP Service Account Keys, API Keys
  - GitHub / GitLab / Bitbucket Personal Access Tokens
  - OpenAI, Anthropic, Hugging Face API Keys
  - Slack Bot Tokens, Webhooks
  - Stripe, Twilio, SendGrid Keys
  - Database connection strings (MongoDB, PostgreSQL, MySQL, Redis)
  - SSH Private Keys, PEM/PFX Certificates
  - JWT Tokens, Bearer Tokens
  - Generic passwords in config files (password=, secret=, token=)
- Scan individual files, directories, or entire repos recursively
- Ignore binary files, node_modules, .git, and other non-relevant paths
- Output results as Markdown report or JSON
- Provide severity ratings (Critical, High, Medium, Low)
- Suggest remediation for each finding

## How to Scan

### Scan a directory
```bash
python secret_scanner.py /path/to/project
```

### Scan with JSON output
```bash
python secret_scanner.py /path/to/project --json
```

### Scan and save report
```bash
python secret_scanner.py /path/to/project --output report.md
```

### Within an Agent
```
"Scan this project for leaked secrets"
"Check if there are any API keys in the codebase"
"Run secret-scanner on the current directory"
"Find hardcoded passwords in my config files"
"Audit this repo before I push to GitHub"
```

## Secret Patterns Detected

### Cloud Provider Keys
| Provider | Secrets Detected |
|----------|-----------------|
| **AWS** | Access Key ID (`AKIA...`), Secret Access Key, Session Token |
| **Azure** | Storage Account Key, Connection String, SAS Token, Client Secret |
| **GCP** | API Key (`AIza...`), Service Account JSON, OAuth Client Secret |

### AI / LLM Keys
| Service | Pattern |
|---------|---------|
| **OpenAI** | `sk-` prefixed API keys |
| **Anthropic** | `sk-ant-` prefixed keys |
| **Hugging Face** | `hf_` prefixed tokens |
| **Cohere** | API keys in config |

### Developer Platforms
| Platform | Secrets Detected |
|----------|-----------------|
| **GitHub** | `ghp_`, `gho_`, `ghu_`, `ghs_`, `ghr_` tokens |
| **GitLab** | `glpat-` tokens |
| **Slack** | `xoxb-`, `xoxp-`, `xoxs-` tokens, webhook URLs |
| **Stripe** | `sk_live_`, `sk_test_`, `rk_live_` keys |
| **Twilio** | Account SID, Auth Token |
| **SendGrid** | `SG.` prefixed API keys |

### Databases & Infrastructure
| Type | Pattern |
|------|---------|
| **MongoDB** | `mongodb://` or `mongodb+srv://` with credentials |
| **PostgreSQL** | `postgresql://` with embedded password |
| **MySQL** | `mysql://` with embedded password |
| **Redis** | `redis://` with password |
| **SSH** | `-----BEGIN (RSA\|EC\|OPENSSH) PRIVATE KEY-----` |
| **Certificates** | PEM, PFX, P12 with embedded keys |

### Generic Patterns
| Pattern | Description |
|---------|-------------|
| **password=** | Hardcoded passwords in config/env files |
| **secret=** | Hardcoded secrets |
| **token=** | Hardcoded tokens |
| **Bearer** | Bearer tokens in code |
| **Basic Auth** | Base64-encoded basic auth headers |
| **JWT** | `eyJ` prefixed JWT tokens |
| **High Entropy** | Long random strings that look like secrets |

## Severity Levels

| Severity | Description | Examples |
|----------|-------------|----------|
| 🔴 **Critical** | Active production credentials | AWS Secret Key, Private Keys, DB passwords |
| 🟠 **High** | Service tokens with broad access | GitHub PAT, Slack Bot Token, Stripe Live Key |
| 🟡 **Medium** | Keys that may be test/dev | Test API keys, example tokens |
| 🟢 **Low** | Potential false positives | Generic password= in comments, placeholder values |

## Files Scanned

Scans these file types by default:
- Source code: `.py`, `.js`, `.ts`, `.java`, `.go`, `.rb`, `.php`, `.cs`, `.rs`
- Config: `.json`, `.yaml`, `.yml`, `.toml`, `.ini`, `.cfg`, `.conf`
- Environment: `.env`, `.env.local`, `.env.production`
- Shell: `.sh`, `.bash`, `.zsh`, `.ps1`
- Docs: `.md`, `.txt`
- Other: `Dockerfile`, `docker-compose.yml`, `Makefile`

## Ignored Paths

Automatically skips:
- `node_modules/`, `vendor/`, `venv/`, `.venv/`
- `.git/`, `.svn/`
- `__pycache__/`, `.pytest_cache/`
- Binary files, images, compiled outputs
- `package-lock.json`, `yarn.lock`

## Remediation Guidance

When secrets are found, the skill recommends:
1. **Rotate the secret immediately** — assume it's compromised
2. **Remove from code** — use environment variables or a secrets manager instead
3. **Add to .gitignore** — prevent `.env` and credential files from being committed
4. **Use git-filter-repo** — to remove secrets from git history
5. **Enable pre-commit hooks** — to catch secrets before they're committed

## Requirements
- Python 3.7+
- No additional dependencies (uses Python standard library)

## Entry Point
- **CLI:** `secret_scanner.py`

## Tags
#security #secrets #credentials #api-keys #tokens #passwords #scanner #audit #pre-commit #leak-detection #cloud #aws #azure #gcp #devops
