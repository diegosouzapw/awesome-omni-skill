---
name: Server
description: Configure, deploy, and troubleshoot web servers, application servers, and containerized services.
---

## Scope

This skill covers the **software layer** — what runs inside a machine.
For infrastructure (provisioning VMs, SSH hardening, firewalls, backups), use the `vps` skill.

## When to Use

- Configuring nginx, Caddy, or Apache
- Deploying Node.js, Python, Go, or other apps
- Docker and Docker Compose setups
- SSL/TLS certificates with Let's Encrypt
- Reverse proxy and load balancing
- Process management (pm2, systemd services)
- Troubleshooting port conflicts, CORS, connection errors
- Self-hosting (Plex, Nextcloud, game servers)
- Local development servers (Vite, webpack-dev-server)

## Common Pitfalls

- **Port already in use** — Check with `lsof -i :PORT` or `ss -tlnp | grep PORT`, kill stale processes
- **CORS in development** — Proxy through your dev server or configure backend headers, don't disable in production
- **SSL certificate issues** — Certbot needs port 80/443 open, use `--standalone` or `--webroot` mode appropriately
- **nginx config not loading** — Always run `nginx -t` before `systemctl reload nginx`
- **Docker container can't reach host** — Use `host.docker.internal` (Docker Desktop) or `172.17.0.1` (Linux)
- **Process dies after SSH disconnect** — Use systemd, pm2, or run inside tmux/screen
- **Wrong permissions on mounted volumes** — Match container user UID with host file ownership

## Patterns by Use Case

| Use Case | Recommended Stack |
|----------|-------------------|
| Static site | Caddy (auto-SSL, zero config) |
| Node.js app | PM2 + nginx reverse proxy |
| Python (Django/FastAPI) | Gunicorn + nginx |
| Multiple services | Docker Compose + Traefik |
| Game server | Dedicated container + port mapping |

For framework-specific configs, see `configs.md`.
For Docker Compose patterns, see `docker.md`.

## Debugging Checklist

1. **Is the process running?** `systemctl status` or `docker ps`
2. **Is it listening?** `ss -tlnp | grep PORT`
3. **Can you reach it locally?** `curl localhost:PORT`
4. **Firewall blocking?** Check `ufw status` or cloud security groups
5. **Reverse proxy misconfigured?** Check nginx logs: `/var/log/nginx/error.log`
6. **DNS pointing correctly?** `dig domain.com` or `nslookup`
