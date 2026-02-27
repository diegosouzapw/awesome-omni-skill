---
name: setup-automatik
description: Facilitate the installation and management of VPS solutions using the Setup Automatik engine (powered by Orion Design). Use when the user wants to install, configure, or manage tools like Traefik, Portainer, Chatwoot, N8N, and other open-source applications on a Linux VPS.
---

# Setup Automatik

## Credits & Acknowledgements
Special thanks to **Orion Design** for providing the original `SetupOrion.sh` script.

**Developers:**
- **Alltomatos**
- **Rafa Martins**
- **Robot 🤖** (Seu mentor assistente)

This skill is part of the **Mundo Automatik** ecosystem.

- **Website:** [mundoautomatik.com](https://mundoautomatik.com/)
- **Community (WhatsApp):** [links.mundoautomatik.com/automatik-brasil](https://links.mundoautomatik.com/automatik-brasil)
- **Telegram:** [t.me/mundoautomatik](https://t.me/mundoautomatik)
- **YouTube:** [@mundoautomatik](https://www.youtube.com/@mundoautomatik)

## Overview
The `setup-automatik` skill is designed to assist in deploying various open-source solutions on a VPS (Virtual Private Server). It leverages the `assets/SetupOrion.sh` script to automate the installation of containers, databases, and application stacks.

## Available Tools
A wide range of tools are available for installation, categorized by their purpose:
- **Infrastructure**: Traefik, Portainer, PostgreSQL, MongoDB, RabbitMQ, etc.
- **Automation & AI**: N8N, Flowise, Typebot, Dify AI, Ollama, Langflow, etc.
- **Communication**: Chatwoot, Evolution API, Uno API, Mautic, Mattermost, etc.
- **Business & Utilities**: Wordpress, Baserow, Metabase, Odoo, NextCloud, etc.

For a full list of supported tools, refer to [tools.md](references/tools.md).

## Prerequisites

### 🔐 Granting Agent Access
For the agent to perform installations on your VPS, you must provide access. There are two ways to do this:

#### Option 1: OpenClaw Node Pairing (Recommended)
This is the most secure and native way. It allows the agent to execute commands directly on your VPS terminal.
1. Run the installer on your VPS: `curl -fsSL https://get.openclaw.ai | sh`
2. Start the pairing process: `openclaw node pair`
3. Paste the resulting pairing code or command here in the chat.

#### Option 2: SSH Access
Provide the agent with your VPS connection details:
- Public IP Address
- Username (usually `root`)
- SSH Password OR Private Key

## Workflow

### 0. Access Setup
Before starting, ensure the agent has access using one of the methods in the [Prerequisites](#prerequisites) section.

### 1. Preparation
Ensure the VPS is running a compatible Linux distribution (preferably Ubuntu/Debian) and has Docker installed.

### 2. Information Gathering
Before installation, gather necessary information:
- Domain names (for SSL/Traefik).
- Database credentials.
- SMTP details for email notifications.

### 3. Installation
To install a tool, the agent uses the `assets/SetupOrion.sh` script. The skill can extract specific installation blocks or execute the script directly in non-interactive mode when possible.

### 4. Verification
After installation, verify that the services are running:
- Check `docker ps`.
- Access the web interface of the installed tool.
- Check logs if any issues arise.

## Common Tasks

### Install Traefik & Portainer
This is usually the first step to manage other stacks.
1. Run the script and select option `01`.
2. Follow prompts for domain and email.

### Deploy a Stack (e.g., N8N)
1. Ensure Traefik is running.
2. Select the tool from the menu.
3. Provide the required subdomains.

## References
- [tools.md](references/tools.md): Comprehensive list of available tools.
- [SetupOrion.sh](assets/SetupOrion.sh): The main installation script.
