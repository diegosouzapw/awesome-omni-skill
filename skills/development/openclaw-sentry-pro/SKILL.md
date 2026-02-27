---
name: openclaw-sentry-pro
description: "Full secret scanning suite: detect leaked API keys, tokens, and credentials, then automatically redact, quarantine exposed files, and enforce .gitignore policies. Everything in openclaw-sentry (free) plus automated countermeasures."
user-invocable: true
metadata: {"openclaw":{"emoji":"🔑","requires":{"bins":["python3"]},"os":["darwin","linux","win32"]}}
---

# OpenClaw Sentry Pro

Everything in [openclaw-sentry](https://github.com/AtlasPA/openclaw-sentry) (free) plus automated countermeasures.

**Free version detects secrets. Pro version eliminates them.**

## Detection Commands (also in free)

### Full Scan

Scan all workspace files for secrets and high-risk files.

```bash
python3 {baseDir}/scripts/sentry.py scan --workspace /path/to/workspace
```

### Check Single File

Check a specific file for secrets.

```bash
python3 {baseDir}/scripts/sentry.py check MEMORY.md --workspace /path/to/workspace
```

### Quick Status

One-line summary of secret exposure risk, quarantine status, and policy state.

```bash
python3 {baseDir}/scripts/sentry.py status --workspace /path/to/workspace
```

## Pro Countermeasures

### Redact Secrets

Find secrets in files and replace them with masked versions (e.g., `sk-ant-abc...xyz` becomes `sk-ant-***REDACTED***`). Creates `.bak` backup before modifying. If no file specified, redact all files in workspace.

```bash
# Redact a single file
python3 {baseDir}/scripts/sentry.py redact config.json --workspace /path/to/workspace

# Redact all files in workspace
python3 {baseDir}/scripts/sentry.py redact --workspace /path/to/workspace
```

### Quarantine a File

Move a file containing secrets to `.quarantine/sentry/` with metadata JSON recording what was found, when, and original location.

```bash
python3 {baseDir}/scripts/sentry.py quarantine .env --workspace /path/to/workspace
```

### Unquarantine a File

Restore a quarantined file to its original location.

```bash
python3 {baseDir}/scripts/sentry.py unquarantine .env --workspace /path/to/workspace
```

### Defend

Auto-generate/update `.gitignore` with common secret patterns (.env, *.pem, *.key, credentials.json, etc.) and create a `.sentry-policy.json` policy file listing which patterns to enforce.

```bash
python3 {baseDir}/scripts/sentry.py defend --workspace /path/to/workspace
```

### Protect (Automated Sweep)

Full automated sweep: scan all files, auto-redact secrets in non-critical files, quarantine files with high-density secrets, update .gitignore. **This is the recommended command for session startup.**

```bash
python3 {baseDir}/scripts/sentry.py protect --workspace /path/to/workspace
```

## Recommended Integration

### Session Startup Hook (Claude Code)

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 scripts/sentry.py protect",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Heartbeat (OpenClaw)

Add to HEARTBEAT.md for periodic protection:
```
- Run secret scanning protection (python3 {skill:openclaw-sentry-pro}/scripts/sentry.py protect)
```

### After Adding New Configs or Secrets

Run `protect` to auto-redact and quarantine any newly exposed credentials.

## What It Detects

| Provider | Patterns |
|----------|----------|
| **AWS** | Access keys (AKIA...), secret keys |
| **GitHub** | PATs (ghp_, gho_, ghs_, ghr_, github_pat_) |
| **Slack** | Bot/user tokens (xox...), webhooks |
| **Stripe** | Secret keys (sk_live_), publishable keys |
| **OpenAI** | API keys (sk-...) |
| **Anthropic** | API keys (sk-ant-...) |
| **Google** | API keys (AIza...), OAuth secrets |
| **Azure** | Storage account keys |
| **Generic** | API keys, secrets, passwords, bearer tokens, connection strings |
| **Crypto** | PEM private keys, .key/.pem/.p12 files |
| **Database** | PostgreSQL/MySQL/MongoDB/Redis URLs with credentials |
| **JWT** | JSON Web Tokens |
| **Environment** | .env files with variables |

## Countermeasure Summary

| Command | Action |
|---------|--------|
| `protect` | Full scan + auto-redact + auto-quarantine + update .gitignore |
| `redact [file]` | Replace secrets with masked versions, backup originals |
| `quarantine <file>` | Move file to quarantine with metadata |
| `unquarantine <file>` | Restore a quarantined file |
| `defend` | Update .gitignore + create enforcement policy |

## No External Dependencies

Python standard library only. No pip install. No network calls. Everything runs locally.

## Cross-Platform

Works with OpenClaw, Claude Code, Cursor, and any tool using the Agent Skills specification.
