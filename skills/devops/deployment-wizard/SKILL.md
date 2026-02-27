---
name: deployment-wizard
description: Deploy local websites to the internet instantly via Cloudflare Tunnel. Zero hosting, zero domain needed.
---

# Deployment Wizard

Expose any local server to the internet using **Cloudflare Tunnel**.
No account, no domain, no hosting needed.

## Quick Start

```bash
# Find free port + get serve command for your stack:
python .agent/skills/deployment-wizard/scripts/tunnel.py --find-port --serve-cmd static --quiet
# → FREE_PORT=3000
# → SERVE_CMD=python -m http.server 3000

# Start tunnel:
python .agent/skills/deployment-wizard/scripts/tunnel.py --port 3000 --quiet
# → TUNNEL_URL=https://xxx.trycloudflare.com
```

## Agent Deploy
📖 Read the compact recipe: `.agent/skills/deployment-wizard/data/deploy_recipe.md`

## Available Stacks
`static` · `react` · `vite` · `nextjs` · `nuxt` · `django` · `flask` · `fastapi` · `express` · `php` · `hugo` · `ruby` · `custom`

## Other Commands
```bash
--check                    # Check cloudflared installed
--install                  # Auto-install cloudflared
--find-port                # Find free port (pre-flight)
--serve-cmd STACK          # Get serve command for stack
--find-port --serve-cmd X  # Combined: free port + serve cmd
```
