---
name: kamal-deploy
description: Expert-level Kamal deployment guidance for deploying containerized applications to any server. Use this skill when users ask about Kamal, container deployment, zero-downtime deployments, deploying Rails/web apps to VPS/cloud servers, kamal setup, kamal deploy, Docker deployment without Kubernetes, or deploying to Hetzner/DigitalOcean/AWS with Kamal. Also use when users mention DHH's deployment tool, 37signals deployment, or want an alternative to Heroku/Render/Vercel with self-hosted infrastructure.
---

# Kamal Deploy Expert

Expert guidance for deploying applications with Kamal - DHH's zero-downtime deployment tool from 37signals.

## Step 1: Fetch Latest Documentation (MANDATORY)

**BEFORE answering ANY Kamal question, you MUST use the WebFetch tool to get current documentation.** The docs below may be outdated - always fetch fresh docs first.

Execute these WebFetch calls in parallel:

1. `WebFetch(url: "https://kamal-deploy.org/docs/installation/", prompt: "Extract complete installation and setup guide")`

2. `WebFetch(url: "https://kamal-deploy.org/docs/configuration/overview/", prompt: "Extract all configuration options and deploy.yml structure")`

3. `WebFetch(url: "https://kamal-deploy.org/docs/commands/view-all-commands/", prompt: "Extract all Kamal commands and usage")`

4. `WebFetch(url: "https://kamal-deploy.org/docs/configuration/proxy/", prompt: "Extract proxy, SSL, and health check configuration")`

Fetch these additional docs based on user's question:
- Servers/roles: `https://kamal-deploy.org/docs/configuration/servers/`
- Accessories (DB, Redis): `https://kamal-deploy.org/docs/configuration/accessories/`
- Environment variables: `https://kamal-deploy.org/docs/configuration/environment-variables/`
- Docker build options: `https://kamal-deploy.org/docs/configuration/builders/`
- Deployment hooks: `https://kamal-deploy.org/docs/hooks/overview/`
- Upgrading v1→v2: `https://kamal-deploy.org/docs/upgrading/overview/`

**Only after fetching fresh docs, use the reference material below as supplementary context.**

## What is Kamal?

Kamal deploys containerized apps to any server via SSH + Docker. Created by 37signals (DHH's company) to deploy Basecamp, HEY, and other apps.

**Core architecture:**
- SSHs into servers, installs Docker automatically
- Builds app into Docker container
- Pushes to registry (Docker Hub, GHCR, etc.)
- Pulls and runs on target servers
- kamal-proxy handles routing, SSL (Let's Encrypt), zero-downtime

**Mental model:** Hetzner/DigitalOcean = the computer, Kamal = deploys your app to it

## Before You Start

**Check these first to avoid common friction:**

1. **Kamal version** - Run `kamal version`. If on 1.x, upgrade with `gem install kamal`. Config syntax changed significantly (1.x uses `traefik`, 2.x uses `proxy`).

2. **Local Docker situation** - Ask the user if they have Docker working locally. If not (or if Docker Desktop is problematic on macOS), configure a remote builder:
   ```yaml
   builder:
     arch: amd64
     remote: ssh://root@SERVER_IP
   ```
   This builds on the target server and avoids local Docker entirely.

3. **37signals open-source repos** - If deploying Campfire, HEY, or other 37signals apps, immediately delete `.env.erb` - it uses their internal 1Password setup and will fail with `op: command not found`.

4. **Registry access** - Confirm the user has a container registry (Docker Hub, GHCR) and knows their credentials before writing config.

## Quick Start

```bash
# Install (or upgrade)
gem install kamal

# Initialize in project
kamal init

# First deploy (installs Docker, proxy, deploys app)
kamal setup

# Subsequent deploys
kamal deploy
```

## Essential Commands

| Command | Purpose |
|---------|---------|
| `kamal setup` | First deploy - installs Docker, proxy, deploys |
| `kamal deploy` | Deploy new version |
| `kamal rollback` | Revert to previous version |
| `kamal app logs` | View application logs |
| `kamal app exec -i bash` | SSH into running container |
| `kamal accessory boot <name>` | Start accessory (db, redis) |
| `kamal proxy reboot` | Restart kamal-proxy |
| `kamal remove` | Remove everything from servers |

## Minimal config/deploy.yml

```yaml
service: my-app
image: username/my-app

servers:
  - 123.45.67.89

registry:
  username: username
  password:
    - KAMAL_REGISTRY_PASSWORD

proxy:
  ssl: true
  host: myapp.com

env:
  secret:
    - RAILS_MASTER_KEY
    - DATABASE_URL
```

## Secrets Management

Secrets live in `.kamal/secrets`:

```bash
# .kamal/secrets
KAMAL_REGISTRY_PASSWORD=ghp_xxxxxxxxxxxx
RAILS_MASTER_KEY=abc123def456
DATABASE_URL=postgres://user:pass@db:5432/app
```

Reference in deploy.yml:
```yaml
env:
  clear:
    RAILS_ENV: production
  secret:
    - RAILS_MASTER_KEY
    - DATABASE_URL
```

## Multi-Server with Roles

```yaml
servers:
  web:
    hosts:
      - 123.45.67.89
      - 123.45.67.90
  workers:
    hosts:
      - 123.45.67.91
    cmd: bin/jobs
    proxy: false  # Workers don't need proxy
```

## Accessories (Databases, Redis)

```yaml
accessories:
  db:
    image: postgres:16
    host: 123.45.67.89
    port: 5432
    env:
      clear:
        POSTGRES_DB: app_production
      secret:
        - POSTGRES_PASSWORD
    directories:
      - data:/var/lib/postgresql/data

  redis:
    image: redis:7
    host: 123.45.67.89
    port: 6379
    directories:
      - data:/data
```

## SSL Configuration

**Automatic (Let's Encrypt):**
```yaml
proxy:
  ssl: true
  host: myapp.com  # Must point to server IP
```

**Custom certificate:**
```yaml
proxy:
  ssl:
    certificate_pem:
      - SSL_CERTIFICATE
    private_key_pem:
      - SSL_PRIVATE_KEY
```

## Health Checks

```yaml
proxy:
  healthcheck:
    interval: 3
    path: /up
    timeout: 3
```

App must return 200 on `/up` (Rails default) or configured path.

## Destinations (Staging/Production)

Create `config/deploy.staging.yml`:
```yaml
servers:
  - staging.myapp.com

proxy:
  host: staging.myapp.com
```

Deploy: `kamal deploy -d staging`

Secrets: `.kamal/secrets.staging`

## Hooks

Place in `.kamal/hooks/` (no file extension):

Available hooks:
- `pre-connect`, `pre-build`, `pre-deploy`, `post-deploy`
- `pre-app-boot`, `post-app-boot`
- `pre-proxy-reboot`, `post-proxy-reboot`

Example `.kamal/hooks/post-deploy`:
```bash
#!/bin/bash
curl -X POST "https://api.honeybadger.io/v1/deploys" \
  -d "deploy[revision]=$KAMAL_VERSION"
```

## Dockerfile Requirements

Kamal needs a Dockerfile. For Rails:

```dockerfile
FROM ruby:3.3-slim

WORKDIR /app

# Install dependencies
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev

COPY Gemfile* ./
RUN bundle install

COPY . .

RUN bundle exec rails assets:precompile

EXPOSE 80
CMD ["bin/rails", "server", "-b", "0.0.0.0", "-p", "80"]
```

Note: Kamal 2.x defaults to port 80 (not 3000).

## Common Issues

**"Container not healthy"**
- Check `/up` endpoint returns 200
- Increase `deploy_timeout` if app boots slowly
- Check logs: `kamal app logs`

**"Permission denied"**
- Ensure SSH key is added: `ssh-add ~/.ssh/id_rsa`
- Check SSH user has Docker access

**Registry auth failed**
- Verify `KAMAL_REGISTRY_PASSWORD` in `.kamal/secrets`
- For GHCR: use personal access token with `write:packages`

**"Address already in use"**
- Another service on port 80/443
- Run `kamal proxy reboot` or check `docker ps` on server

## Kamal vs Alternatives

| | Kamal | Kubernetes | Heroku |
|---|---|---|---|
| Complexity | Low | High | None |
| Cost | VPS only | VPS + overhead | $$$ |
| Control | Full | Full | Limited |
| Zero-downtime | Yes | Yes | Yes |
| SSL | Auto | Manual | Auto |
| Learning curve | Hours | Weeks | Minutes |

## Best Practices

1. **Always test locally first**: `docker build . && docker run -p 3000:80 <image>`
2. **Use staging destination** before production
3. **Keep secrets out of git**: `.kamal/secrets` in `.gitignore`
4. **Set up monitoring**: Use hooks to notify on deploy
5. **Regular backups**: Especially accessory volumes
6. **Use asset bridging** for Rails: `asset_path: /app/public/assets`

## Reference Files

For detailed configuration options, see:
- [references/configuration.md](references/configuration.md) - Complete deploy.yml reference
- [references/troubleshooting.md](references/troubleshooting.md) - Common issues and solutions
