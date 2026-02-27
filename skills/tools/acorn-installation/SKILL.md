---
name: "Acorn Installation"
description: "Install and set up the Acorn theorem prover CLI. Use when the environment doesn't have Acorn installed or when the user needs to set up Acorn for the first time."
---

# Acorn Installation Skill

This skill helps install the Acorn theorem prover CLI in various environments.

## Installation

Simply run the installation script:

```bash
bash .claude/skills/acorn-installation/install-acorn.sh
```

The script will:
- Check if Acorn is already installed (if so, it exits successfully)
- Download the latest Acorn binary from GitHub releases
- Install it to `~/.local/bin/acorn`
- Verify the installation works

## Running the Verifier

Once installed, you can verify Acorn proofs by running:

```bash
acorn
```

This should be run after every change to ensure proofs are verifiable.

## Instructions for the Agent

When this skill is invoked:

1. Run the installation script: `bash .claude/skills/acorn-installation/install-acorn.sh`
2. The script handles checking for existing installation automatically
3. After the script completes successfully, run `acorn` to verify proofs in the current project
