---
name: kube-audit-kit
description: Performs read-only Kubernetes security audits by exporting resources, sanitizing metadata, grouping applications by topology, and generating PSS/NSA-compliant audit reports. Use when the user requests auditing Kubernetes clusters, Namespaces, security reviews, or configuration analysis.
user-invocable: true
allowed-tools: Read, Write, Bash(python:*), Bash(uv:*), Bash(kubectl:*), Bash(export:*)
examples:
  - "Run a security audit for the payment namespace in prod-cluster"
  - "Check whether the backend apps in staging meet PSS standards"
  - "Analyze sensitive data leakage risk for all resources in the development namespace"
  - "Generate a full audit report for the default namespace in test-cluster"
  - "review k8s cluster security configuration"
  - "kubernetes security audit for production workload"
author: crazygit
repository: https://github.com/crazygit/kube-audit-kit
---

# Kube Audit Kit - Read-Only Kubernetes Security Audit Toolkit

This Skill uses a standardized, scripted workflow to export Kubernetes cluster resources in **read-only** mode, sanitize them, group applications, and perform a deep security audit. The entire process strictly follows the **read-only** principle and does not modify any cluster state.

## Core Principles

- **Read-only**: only `get/list` operations, never `apply/patch/delete`
- **Full coverage**: dynamically discover all resource types without hardcoding lists
- **Scripted**: core logic runs through Python scripts for stability

## Quick Start

### Prerequisites

1. **Environment setup**:

   ```bash
   uv sync
   ```

   See [SETUP.md](SETUP.md) for details.

2. **Verify kubectl**:
   ```bash
   kubectl config get-contexts
   ```

### Run an audit

When a user requests an audit, follow these steps strictly:

**Set the output directory first** (important!):

```bash
# Set the output directory to output/ under the current working directory
# This ensures output files are generated in the user's working directory, not the SKILL install directory
export KUBE_AUDIT_OUTPUT="$(pwd)/output"
```

**Use the progress checklist**:

```
Audit progress:
- [ ] Step 1: Export - Dynamic discovery and full resource export
- [ ] Step 2: Sanitize - Remove metadata and status fields
- [ ] Step 3: Group - Associate applications by workload topology
- [ ] Step 4: Audit - Dual-layer security audit
```

#### Step 1: Export

```bash
# Keep the environment variable effective for each command
export KUBE_AUDIT_OUTPUT="$(pwd)/output" && \
uv run python scripts/export.py --context <context> --namespace <namespace>
```

Output: `{OUTPUT_BASE}/export/`

#### Step 2: Sanitize

```bash
export KUBE_AUDIT_OUTPUT="$(pwd)/output" && \
uv run python scripts/sanitize.py --context <context> --namespace <namespace>
```

Output: `{OUTPUT_BASE}/sanitize/`, `{OUTPUT_BASE}/sanitize_fields/`

#### Step 3: Group

```bash
export KUBE_AUDIT_OUTPUT="$(pwd)/output" && \
uv run python scripts/group_apps.py --context <context> --namespace <namespace>
```

Output: `{OUTPUT_BASE}/group/`, `{OUTPUT_BASE}/ungrouped_resources.txt`

#### Step 4: Audit

**Phase 1 - Script-based static scan**:

```bash
export KUBE_AUDIT_OUTPUT="$(pwd)/output" && \
uv run python scripts/audit.py --context <context> --namespace <namespace>
```

Output:

- `{OUTPUT_BASE}/audit/audit_results.json` - structured audit results
- `{OUTPUT_BASE}/audit/configmap_to_secret.csv` - ConfigMap sensitive data
- `{OUTPUT_BASE}/audit/secret_to_configmap.csv` - Secret non-sensitive data
- `{OUTPUT_BASE}/audit/rbac_issues.csv` - RBAC audit results
- `{OUTPUT_BASE}/audit/network_security.csv` - network security audit results
- `{OUTPUT_BASE}/audit/hostpath_mounts.csv` - hostPath mount findings
- `{OUTPUT_BASE}/audit/security_policies.csv` - seccomp/AppArmor results
- `{OUTPUT_BASE}/audit/pdb_and_secrets.csv` - PDB/Secret/ServiceAccount results

**Phase 2 - AI expert deep review**:

AI independently reviews results without relying on phase 1 output:

1. **Independent analysis**: traverse `{OUTPUT_BASE}/group/*/` and read all original YAML files
2. **Deep review**: identify risks not covered by script rules
   - business logic risks (e.g., plaintext private keys, hardcoded passwords)
   - architecture risks (e.g., missing NetworkPolicy, overly broad RBAC)
   - configuration drift risks (e.g., `latest` images, missing resource limits)
3. **Supplement findings**: if sensitive data was missed, append to the CSV files
4. **Report summary**: merge phase 1 findings with AI analysis into `{OUTPUT_BASE}/audit/audit_report.md`

**Report template**: see `audit_report_template.md` in the same directory.

**Key requirements**:

- Must read original YAML files, not just audit_results.json
- Every application must have specific analysis; avoid vague statements like "not reviewed"
- If script misses sensitive data, update the CSV files to keep data complete

## Output Structure

```
output/{context}/{namespace}/
├── export/              # raw export data
├── sanitize/            # sanitized data
├── sanitize_fields/     # sanitization records
├── group/               # application grouping
│   └── {app_name}/
│       ├── *.yaml            # grouped resource files
│       └── config_usage.json # CM/Secret usage record
├── ungrouped_resources.txt   # orphan resources
└── audit/               # audit results
    ├── audit_results.json        # static analysis results
    ├── configmap_to_secret.csv   # sensitive data in ConfigMaps
    ├── secret_to_configmap.csv   # non-sensitive data in Secrets
    ├── rbac_issues.csv            # RBAC audit results
    ├── network_security.csv       # network security audit results
    ├── hostpath_mounts.csv        # hostPath mount findings
    ├── security_policies.csv      # seccomp/AppArmor results
    ├── pdb_and_secrets.csv        # PDB/Secret/ServiceAccount results
    └── audit_report.md           # final AI-generated report
```

## Reference Docs

- **[QUICKSTART.md](QUICKSTART.md)**: 30-second quick start
- **[WORKFLOW.md](WORKFLOW.md)**: full workflow and implementation details
- **[SETUP.md](SETUP.md)**: environment setup and dependency installation
- **[EXAMPLES.md](EXAMPLES.md)**: output examples and typical scenarios

## User Interaction Conventions

### Planning Phase

```
Received. Target: Context `{ctx}`, Namespace `{ns}`.

Execution plan:
1. Set the output directory environment variable: export KUBE_AUDIT_OUTPUT="$(pwd)/output"
2. [Export] Dynamic discovery and full resource export → scripts/export.py
3. [Sanitize] Remove metadata and status fields → scripts/sanitize.py
4. [Group] Associate applications by workload topology → scripts/group_apps.py
5. [Audit] Dual-layer security audit (static scan + AI expert review) → scripts/audit.py

Output directory: $(pwd)/output/{ctx}/{ns}/

Start?
```

### Execution Phase

Output a summary after each step (each command must include the environment variable):

```
✅ [Export completed] Scanned 32 resource types, exported 150 YAMLs
   Output: output/{ctx}/{ns}/export/
```

### Results Phase

```
✅ [Audit completed] Static report and AI expert analysis merged

📊 Audit stats:
- Applications: 12
- Critical risks: X (see audit_results.json)
- Warning risks: Y
- Info recommendations: Z

📁 Output directory: output/{ctx}/{ns}/
📄 Full audit report: output/{ctx}/{ns}/audit/audit_report.md

⚠️ Security reminder: the output/ directory contains decrypted Secret data. Please delete it securely after the audit!
```

## Path Conventions

**{OUTPUT_BASE}** = `output/{context}/{namespace}/`

### Output path mechanism

All paths are computed by `get_output_paths()` in `scripts/utils.py`, with the following precedence:

1. **Environment variable `KUBE_AUDIT_OUTPUT`** (recommended)
   - Set in SKILL.md before running: `export KUBE_AUDIT_OUTPUT="$(pwd)/output"`
   - Ensures output files are created in the **user's working directory**
   - Avoids writing to the SKILL installation directory

2. **Current working directory** (fallback)
   - If the environment variable is not set, use `Path.cwd() / "output"`
   - Note: when a SKILL runs, cwd may be the SKILL directory

### Why the environment variable?

When the SKILL is invoked, the Agent switches to the SKILL installation directory to run scripts. Using `Path.cwd()` directly would write to the wrong location.

By setting `KUBE_AUDIT_OUTPUT="$(pwd)/output"` before each command, you ensure:
- `$(pwd)` resolves to the user's working directory
- Python scripts read the environment variable and write to the intended path
- Output always lands in the user's working directory, regardless of where the SKILL is called

## Key Design Decisions

### Volume vs EnvVar distinction

ConfigMaps/Secrets usage determines whether sensitive data is scanned:

- **Volume mount**: skip sensitive scanning (treated as application config files)
- **EnvVar reference**: scan for sensitive data (may include passwords/keys)

`config_usage.json` records the usage type for each ConfigMap/Secret.

### Permission error handling

Scripts use a fault-tolerant approach:

- If a single resource type is denied, skip it and show a warning
- Other resource types continue normally
- The final report notes which checks are missing due to insufficient permissions

Use a dedicated audit service account (see [SETUP.md](SETUP.md)).

## Security Reminder

**Warning**: the `output/` directory contains decrypted Secret data.

**After the audit**:

- Keep `audit_report.md` (it does not contain sensitive data)
- Securely delete other directories or store them encrypted
- Do not commit `output/` to version control
