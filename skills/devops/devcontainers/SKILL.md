---
name: devcontainers
description: Expert in creating devcontainer configurations using DuploCloud features. Use when setting up development environments, configuring devcontainer.json, or adding DuploCloud tooling to a workspace. Knows feature detection, cloud CLI setup, Kubernetes access, AI tools, and 1Password SSH integration.
---

# DuploCloud Devcontainer Expert

Create devcontainer configurations using [duplocloud/devcontainers](https://github.com/duplocloud/devcontainers) features.

## Naming Conventions

The devcontainer `name` should be descriptive and creative. Use imagination to create a good name:

**Sources for naming:**
1. Repository name (e.g., `terraform-duplocloud-components` → "DuploCloud Components Devspace")
2. Project nickname from README (e.g., "Brook AI" → "Brook AI Devspace")
3. Company + purpose (e.g., "Acme Infrastructure Devspace")
4. Short descriptive name based on what the repo does

**Format:** `"<Project Name> Devspace"`

**Examples:**
- `"DuploCloud IaC Devspace"` - for infrastructure repos
- `"Brook AI Development"` - from README nickname
- `"Playlab Platform Devspace"` - from org/project name
- `"Customer Portal Dev"` - descriptive of purpose

**If no clear name:** Use repo directory name, title-cased, with "Devspace" suffix.

## First Step: Read Project README

**Always start by reading the project's README.md** to understand:
- What the project does
- Technology stack
- Cloud provider (AWS/GCP)
- Any existing infrastructure references

```
Read: README.md (or README.rst, readme.md)
```

## Feature Detection

Search the workspace for indicators to determine which features to include.

### Detection Patterns

| Feature | Search For | Indicators |
|---------|------------|------------|
| **terraform** | `*.tf`, `modules/`, `terraform/` | Terraform files present |
| **aws-cli** | `aws`, `s3://`, `AWS_`, `us-east`, `us-west` | AWS references in code/config |
| **gcloud-cli** | `gcloud`, `gs://`, `GOOGLE_`, `gcp`, `GCP` | GCP references in code/config |
| **kubernetes** | `kubectl`, `k8s`, `kubernetes`, `helm`, `Chart.yaml` | K8s manifests or Helm charts |
| **github** | `.github/`, `github.com` | GitHub workflows or references |
| **git** | Commit signing needed, version tags | Team uses signed commits |
| **ai-claude** | Team uses Claude | Anthropic AI preference |
| **ai-codex** | Team uses ChatGPT/OpenAI | OpenAI preference |
| **ai-gemini** | Team uses Gemini | Google AI preference |
| **onepassword-cli** | Team uses 1Password | SSH key management needed |
| **direnv** | `.envrc` files, per-dir env vars | direnv usage |
| **openvpn** | VPN access needed | Private network access |

### Search Commands

```bash
# Cloud detection
grep -r "aws\|s3://" --include="*.tf" --include="*.py" --include="*.yaml" .
grep -r "gcloud\|gs://" --include="*.tf" --include="*.py" --include="*.yaml" .

# Kubernetes detection
find . -name "*.yaml" -exec grep -l "kind:\|apiVersion:" {} \;
ls -la terraform/ modules/ 2>/dev/null

# Git provider
grep -r "github.com\|gitlab.com\|bitbucket.org" .
```

## Required Feature

**Always include duploctl** - it's required for all other DuploCloud features:

```json
"ghcr.io/duplocloud/devcontainers/duploctl:latest": {}
```

## Feature Reference

| Feature | Purpose | When to Use |
|---------|---------|-------------|
| `duploctl` | DuploCloud CLI | **Always required** |
| `aws-cli` | AWS CLI + JIT creds | AWS cloud projects |
| `gcloud-cli` | Google Cloud CLI | GCP cloud projects |
| `terraform` | Terraform + `tf` wrapper | Infrastructure as code |
| `kubernetes` | kubectl + JIT kubeconfig | K8s workloads |
| `github` | GitHub CLI + Copilot | GitHub repos |
| `git` | Git config + signing + plugins | Commit signing, version bumping |
| `ai-claude` | Anthropic Claude Code CLI | Claude AI assistance |
| `ai-codex` | OpenAI Codex CLI | OpenAI/ChatGPT assistance |
| `ai-gemini` | Google Gemini CLI | Gemini AI assistance |
| `onepassword-cli` | 1Password + SSH keys | Team SSH management |
| `direnv` | direnv + DuploCloud direnvrc | Per-directory env vars |
| `openvpn` | OpenVPN client | VPN access to private networks |

**Detailed options**: See `references/features.md`

## Config Folder Isolation Pattern

**Important:** All CLI configurations should be isolated to a workspace directory. This keeps state across container restarts and rebuilds.

**Default folder:** `config/` (can also be `.config`, `.secrets`, `.tmp`, `.dev` - ask user if unclear)

### Why This Matters

- Container rebuilds don't lose CLI state
- AWS credentials, kubeconfig, gcloud auth persist
- Team members can gitignore personal configs
- Clean separation of workspace vs container state

### Environment Variables Point to Config Folder

```json
"containerEnv": {
  "AWS_CONFIG_FILE": "${containerWorkspaceFolder}/config/aws",
  "KUBECONFIG": "${containerWorkspaceFolder}/config/kubeconfig.yaml",
  "CLOUDSDK_CONFIG": "${containerWorkspaceFolder}/config/gcloud",
  "GOOGLE_APPLICATION_CREDENTIALS": "${containerWorkspaceFolder}/config/gcloud/application_default_credentials.json",
  "DUPLO_CONFIG": "${containerWorkspaceFolder}/config/duplo.yaml",
  "CODEX_HOME": "${containerWorkspaceFolder}/config/.codex",
  "XDG_CONFIG_HOME": "${containerWorkspaceFolder}/config"
}
```

### Config Folder Structure

```
config/
├── aws              # AWS CLI config
├── kubeconfig.yaml  # kubectl config
├── gcloud/          # gcloud SDK config
├── duplo.yaml       # duploctl config
├── op/              # 1Password CLI (via XDG_CONFIG_HOME)
└── .codex/          # Codex skills and config
```

### Add to .gitignore

```
config/
!config/.gitkeep
```

### init.sh Creates Config Folder

```bash
mkdir -p "${VSCODE_CWD}/config"
```

## Environment Variable Strategy

Use `${localEnv:VAR_NAME}` to pull values from the host machine into feature configuration. This allows team members to customize their devcontainer without modifying the shared config.

### Required Local Environment Variables

These must be set on the host machine (in shell profile or `.env`):

| Variable | Purpose | Required |
|----------|---------|----------|
| `DUPLO_HOST` | DuploCloud portal URL | **Yes** |
| `DUPLO_TOKEN` | DuploCloud API token | **Yes** |
| `GIT_USERNAME` | Git commit author name | Recommended |
| `GIT_EMAIL` | Git commit author email | Recommended |
| `OP_SSH_SECRET` | 1Password SSH key name | If using 1Password |
| `OP_ACCOUNT` | 1Password account domain | If using 1Password |
| `OP_VAULT_NAME` | 1Password vault name | If using 1Password |

### The .env File

Secrets are passed via `.env` file (never committed to git). The `runArgs` mounts this file:

```json
"runArgs": [
  "--env-file=${localWorkspaceFolder}/.env"
]
```

**Minimum .env contents:**
```bash
DUPLO_HOST=https://yourportal.duplocloud.net
DUPLO_TOKEN=your-duplo-token
```

**Add to .gitignore:**
```
.env
```

## Lifecycle Scripts

### init.sh - Ensure .env Exists

Create `.devcontainer/init.sh` to bootstrap required files:

```bash
#!/usr/bin/env bash
set -e

echo "Init Devcontainer"

VSCODE_CWD=${VSCODE_CWD:-$PWD}

mkdir -p "${VSCODE_CWD}/config"

# Create .env file if it doesn't exist
if [ ! -f "${VSCODE_CWD}/.env" ]; then
  echo "Creating .env file with DUPLO credentials..."
  cat > "${VSCODE_CWD}/.env" << EOF
DUPLO_HOST=${DUPLO_HOST}
DUPLO_TOKEN=${DUPLO_TOKEN}
EOF
  echo ".env file created"
else
  echo ".env file already exists"
fi
```

### post-start.sh - Post-Start Commands

Create `.devcontainer/post-start.sh` for commands after container starts:

```bash
#!/usr/bin/env bash
set -e

echo "Post start running"

# Add custom post-start commands here
```

### Lifecycle Configuration

```json
"initializeCommand": ".devcontainer/init.sh",
"postStartCommand": ".devcontainer/post-start.sh"
```

## Basic Template

**Default image:** Python base (unless user specifies otherwise)

```json
{
  "name": "<Project> Devspace",
  "image": "mcr.microsoft.com/vscode/devcontainers/python:3.12",
  "features": {
    "ghcr.io/devcontainers/features/docker-outside-of-docker:1": {
      "moby": "false"
    },
    "ghcr.io/duplocloud/devcontainers/duploctl:latest": {}
  },
  "runArgs": [
    "--env-file=${localWorkspaceFolder}/.env"
  ],
  "initializeCommand": ".devcontainer/init.sh",
  "postStartCommand": ".devcontainer/post-start.sh",
  "containerEnv": {
    "WORKSPACE_FOLDER": "${containerWorkspaceFolder}",
    "DUPLO_CONFIG": "${containerWorkspaceFolder}/config/duplo.yaml",
    "EDITOR": "code --wait"
  },
  "customizations": {
    "vscode": {
      "extensions": [],
      "settings": {}
    }
  }
}
```

## Cloud-Specific Patterns

### AWS Project

```json
"features": {
  "ghcr.io/duplocloud/devcontainers/duploctl:latest": {},
  "ghcr.io/duplocloud/devcontainers/aws-cli:latest": {
    "jit": true,
    "jitAdmin": true
  }
},
"containerEnv": {
  "AWS_CONFIG_FILE": "${containerWorkspaceFolder}/config/aws",
  "AWS_DEFAULT_REGION": "us-east-1",
  "AWS_REGION": "us-east-1"
}
```

### GCP Project

```json
"features": {
  "ghcr.io/duplocloud/devcontainers/duploctl:latest": {},
  "ghcr.io/duplocloud/devcontainers/gcloud-cli:1": {}
},
"containerEnv": {
  "CLOUDSDK_CONFIG": "${containerWorkspaceFolder}/config/gcloud",
  "CLOUDSDK_PYTHON": "/usr/local/bin/python",
  "GOOGLE_APPLICATION_CREDENTIALS": "${containerWorkspaceFolder}/config/gcloud/application_default_credentials.json"
}
```

## Kubernetes Access

```json
"ghcr.io/duplocloud/devcontainers/kubernetes:1": {
  "jit": true,
  "jitAdmin": true,
  "plan": "nonprod01"
},
"containerEnv": {
  "KUBECONFIG": "${containerWorkspaceFolder}/config/kubeconfig.yaml"
}
```

**Options:**
- `jit`: Enable automatic JIT authentication
- `jitAdmin`: Use admin privileges (requires admin token)
- `plan`: DuploCloud infrastructure name

## AI Tools

Choose based on team's AI provider preference:

### Claude (Anthropic)

```json
"ghcr.io/duplocloud/devcontainers/ai-claude:1": {
  "skills": "tf-module"
}
```
Skills installed to: `~/.claude/skills/`

### Codex (OpenAI)

```json
"ghcr.io/duplocloud/devcontainers/ai-codex:1": {
  "skills": "tf-module"
},
"containerEnv": {
  "CODEX_HOME": "${containerWorkspaceFolder}/.codex"
}
```
Skills installed to: `$CODEX_HOME/skills/`

### Gemini (Google)

```json
"ghcr.io/duplocloud/devcontainers/ai-gemini:1": {
  "skills": "tf-module"
}
```
Skills installed to: `~/.gemini/skills/`

### GitHub Copilot

```json
"ghcr.io/duplocloud/devcontainers/github:latest": {
  "installCopilot": true,
  "skills": "tf-module"
}
```

## 1Password SSH Integration

### Required Local Environment Variables

These must be set on the **host machine** for 1Password to work:

| Variable | Purpose | Example |
|----------|---------|--------|
| `OP_ACCOUNT` | 1Password account domain | `my.1password.com` |
| `OP_VAULT_NAME` | Vault containing SSH keys | `Employee` |
| `OP_SSH_SECRET` | SSH key secret name in 1Password | `GitHub SSH Key` |
| `OP_AUTO_SSH` | Enable auto SSH config | `true` |
| `USER_EMAIL` | Email for 1Password account | `you@company.com` |

### Feature Configuration

```json
"ghcr.io/duplocloud/devcontainers/onepassword-cli:1": {
  "autoSsh": "${localEnv:OP_AUTO_SSH}",
  "sshSecretNames": "${localEnv:OP_SSH_SECRET}",
  "vault": "${localEnv:OP_VAULT_NAME}",
  "account": "${localEnv:OP_ACCOUNT}",
  "userEmail": "${localEnv:USER_EMAIL}"
}
```

### SSH Secret Requirements in 1Password

**Critical:** For auto SSH config to work, your 1Password SSH secret needs custom fields:

| Field | Required | Description | Example |
|-------|----------|-------------|--------|
| `host` | **Yes** | Hostname for SSH | `github.com` |
| `username` | Recommended | SSH username | `git` |
| `port` | Optional | SSH port | `22` |

**Without the `host` field**, keys download but SSH config isn't created.

### macOS SSH Agent Setup

**Important for macOS users:** You need to configure a Launch Agent to expose the 1Password SSH agent socket to devcontainers.

Follow: [1Password SSH Agent Compatibility](https://developer.1password.com/docs/ssh/agent/compatibility/#ssh-auth-sock)

**Verify SSH agent is working:**
```bash
ssh-add -l
```

### Debugging 1Password Issues

**Common problems:**

1. **"No vault specified" error**
   - Ensure `OP_VAULT_NAME` or `vault` option is set
   - Service accounts/Connect Server require vault specification

2. **SSH keys not configured**
   - Check SSH secret has `host` custom field in 1Password
   - Verify `autoSsh` is `true` (not string `"true"`)

3. **Repeated account setup on rebuild**
   - Set `XDG_CONFIG_HOME` to workspace config folder
   - 1Password data persists in `$XDG_CONFIG_HOME/op/`

4. **macOS SSH agent not forwarding**
   - Configure Launch Agent per 1Password docs
   - Check `SSH_AUTH_SOCK` is set in container

5. **Interactive login required every time**
   - Set `userEmail` to skip email prompt
   - Use `XDG_CONFIG_HOME` for persistence
   - Consider Service Account token for automation

**For detailed troubleshooting:** Read the full feature README:
`https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/onepassword-cli/README.md`

### Authentication Methods (checked in order)

1. Connect Server (`OP_CONNECT_HOST` + `OP_CONNECT_TOKEN`)
2. Service Account (`OP_SERVICE_ACCOUNT_TOKEN`)
3. Desktop App Agent (Linux only, `~/.1password/agent.sock`)
4. Interactive Login

## Git Configuration

```json
"ghcr.io/duplocloud/devcontainers/git:1": {
  "userName": "${localEnv:GIT_USERNAME}",
  "userEmail": "${localEnv:GIT_EMAIL}",
  "signingKey": "${localEnv:GIT_SIGNING_KEY}"
}
```

**Options:**
- `userName`: Git user.name (fallback: `GIT_USER` env)
- `userEmail`: Git user.email (fallback: `GIT_EMAIL` env)
- `signingKey`: Path to signing key relative to `~/.ssh`
- `installGitKraken`: Install GitKraken CLI

**Included plugins:**
- `git-bump` - Semantic version bumping (`git bump -v patch`)
- `git-setenv` - Export CI env vars (`eval $(git setenv bitbucket)`)

**Integration with 1Password for signed commits:**
```json
{
  "ghcr.io/duplocloud/devcontainers/onepassword-cli:1": {
    "autoSsh": true,
    "sshSecretNames": "GitHub SSH Key"
  },
  "ghcr.io/duplocloud/devcontainers/git:1": {
    "signingKey": "GitHub_SSH_Key.pub"
  }
}
```

## OpenVPN Access

**Note:** Primarily useful for GitHub Codespaces. Can be disabled for local devcontainers.

```json
"ghcr.io/duplocloud/devcontainers/openvpn:1": {
  "enabled": "${localEnv:DEVCONTAINER_VPN_ENABLED:-false}",
  "autoConnect": true
},
"runArgs": ["--device=/dev/net/tun"]
```

**Options:**
- `enabled`: Enable/disable feature (use localEnv to toggle)
- `autoConnect`: Auto-connect on container start (default: true)
- `configDir`: Custom config directory

**Required:** Must add `--device=/dev/net/tun` to runArgs.

## direnv

```json
"ghcr.io/duplocloud/devcontainers/direnv:1": {}
```

Installs direnv with DuploCloud-specific direnvrc at `$HOME/direnv/direnvrc`.

## Terraform Projects

```json
"ghcr.io/duplocloud/devcontainers/terraform:1": {},
"customizations": {
  "vscode": {
    "extensions": ["hashicorp.terraform"],
    "settings": {
      "[terraform]": {
        "editor.defaultFormatter": "hashicorp.terraform",
        "editor.formatOnSave": true
      }
    }
  }
}
```

## Decision Flow

```
1. Read README.md for project context
2. Search workspace for cloud indicators (AWS/GCP)
3. Check for Terraform files
4. Check for Kubernetes references
5. Determine git provider
6. Add AI tools based on project complexity
7. Add 1Password if team uses it
```

## Common Extensions by Stack

| Stack | Extensions |
|-------|------------|
| Python | `ms-python.python` |
| Terraform | `hashicorp.terraform` |
| Kubernetes | `ms-kubernetes-tools.vscode-kubernetes-tools` |
| YAML | `redhat.vscode-yaml` |
| GitHub | `github.vscode-github-actions`, `GitHub.copilot-chat` |
| Database | `cweijan.vscode-database-client2` |
| Git | `eamodio.gitlens` |
| AWS | `amazonwebservices.aws-toolkit-vscode` |

## Extensions Auto-Installed by Features

| Feature | Extension |
|---------|----------|
| `github` | `github.vscode-pull-request-github` |
| `git` | `eamodio.gitlens` |
| `ai-claude` | `anthropic.claude-code` |
| `ai-codex` | `openai.chatgpt` |
| `ai-gemini` | `Google.geminicodeassist` |
| `aws-cli` | `amazonwebservices.aws-toolkit-vscode` |
| `kubernetes` | `ms-kubernetes-tools.vscode-kubernetes-tools` |

## Learning More About Features

To learn about a specific feature's options, use web search to read its README:

```
https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/<feature-name>/README.md
```

**All feature README URLs:**
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/duploctl/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/aws-cli/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/gcloud-cli/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/terraform/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/kubernetes/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/github/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/git/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/ai-claude/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/ai-codex/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/ai-gemini/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/onepassword-cli/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/direnv/README.md`
- `https://raw.githubusercontent.com/duplocloud/devcontainers/refs/heads/main/src/openvpn/README.md`

## Skill Resources

| Resource | Contents |
|----------|----------|
| `references/features.md` | Detailed feature options and configuration |
| `references/detection.md` | Workspace search patterns for feature selection |
| `assets/templates/` | Ready-to-use template files |

## Asset Templates

Use these templates as starting points - copy and customize:

| Template | Use Case |
|----------|----------|
| `devcontainer.json.template` | Minimal base template |
| `devcontainer-aws.json.template` | AWS projects with JIT |
| `devcontainer-terraform.json.template` | Terraform/IaC projects |
| `devcontainer-full.json.template` | Full-featured with all tools |
| `init.sh` | Initialize script (creates .env) |
| `post-start.sh` | Post-start script placeholder |

**Template placeholder:** `{{PROJECT_NAME}}` - replace with devcontainer name
