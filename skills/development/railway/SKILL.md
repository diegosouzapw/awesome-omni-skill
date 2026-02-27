---
name: railway
description: Deploy and manage applications on Railway.app. Use for deploying projects, managing services, viewing logs, setting environment variables, and managing databases. Railway is a modern cloud platform for deploying apps with zero configuration.
metadata:
  {
    "openclaw":
      {
        "emoji": "🚂",
        "requires": { "bins": ["railway"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "railway",
              "bins": ["railway"],
              "label": "Install Railway CLI (brew)",
            },
            {
              "id": "npm",
              "kind": "npm",
              "package": "@railway/cli",
              "bins": ["railway"],
              "label": "Install Railway CLI (npm)",
            },
          ],
      },
  }
---

# Railway

Deploy and manage applications on [Railway.app](https://railway.app) - a modern cloud platform with zero-config deployments.

## Authentication

```bash
# Login (opens browser)
railway login

# Login with token (CI/CD)
railway login --token <TOKEN>

# Check login status
railway whoami

# Logout
railway logout
```

## Project Management

### Link & Initialize

```bash
# Link current directory to existing project
railway link

# Link to specific project
railway link --project <PROJECT_ID>

# Create new project
railway init

# Unlink project
railway unlink
```

### View Projects

```bash
# List all projects
railway list

# Open project in browser
railway open

# Show project status
railway status
```

## Deployment

### Deploy

```bash
# Deploy current directory
railway up

# Deploy without watching logs
railway up --detach

# Deploy specific service
railway up --service <SERVICE_NAME>

# Deploy to specific environment
railway up --environment production

# Redeploy latest version
railway redeploy

# Redeploy specific service
railway redeploy --service <SERVICE_NAME>
```

### Deploy from Template

```bash
# Deploy a template
railway deploy --template <TEMPLATE_NAME>

# With variables
railway deploy --template postgres --variable POSTGRES_USER=myuser
```

## Services

```bash
# List services in project
railway service

# Create new service
railway service create

# Delete service
railway service delete <SERVICE_NAME>
```

## Environment Variables

```bash
# List all variables
railway variables

# Set variable
railway variables set KEY=value

# Set multiple variables
railway variables set KEY1=value1 KEY2=value2

# Delete variable
railway variables delete KEY

# View specific variable
railway variables get KEY
```

## Logs

```bash
# View logs (live)
railway logs

# View logs for specific service
railway logs --service <SERVICE_NAME>

# View recent logs (not live)
railway logs --no-follow

# View logs with timestamps
railway logs --timestamps
```

## Run Commands

```bash
# Run command with Railway env vars
railway run <command>

# Examples
railway run npm start
railway run python manage.py migrate
railway run prisma db push

# SSH into running service
railway ssh

# SSH into specific service
railway ssh --service <SERVICE_NAME>
```

## Domains

```bash
# List domains
railway domain

# Add custom domain
railway domain add <DOMAIN>

# Remove domain
railway domain delete <DOMAIN>
```

## Databases

Railway supports one-click database provisioning:

```bash
# Add PostgreSQL
railway add --plugin postgresql

# Add MySQL
railway add --plugin mysql

# Add Redis
railway add --plugin redis

# Add MongoDB
railway add --plugin mongodb
```

Database connection strings are automatically added to environment variables.

## Environments

```bash
# List environments
railway environment

# Switch environment
railway environment <ENV_NAME>

# Create environment
railway environment create <ENV_NAME>

# Delete environment
railway environment delete <ENV_NAME>
```

## Volumes

```bash
# List volumes
railway volume

# Create volume
railway volume create --mount /data

# Delete volume
railway volume delete <VOLUME_ID>
```

## Common Workflows

### Deploy a New Project

```bash
# 1. Initialize in your project directory
cd my-app
railway init

# 2. Add a database if needed
railway add --plugin postgresql

# 3. Set environment variables
railway variables set NODE_ENV=production

# 4. Deploy
railway up
```

### Connect to Production Database

```bash
# Run local command with production env vars
railway run psql $DATABASE_URL

# Or use SSH
railway ssh
# Then inside container:
psql $DATABASE_URL
```

### View Deployment Status

```bash
# Check status
railway status

# View logs
railway logs

# Open dashboard
railway open
```

### Rollback Deployment

```bash
# View deployments in dashboard
railway open

# Redeploy previous version (via dashboard)
# Or redeploy current code
railway redeploy
```

## CI/CD Integration

For GitHub Actions or other CI:

```yaml
# .github/workflows/deploy.yml
name: Deploy to Railway
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Railway CLI
        run: npm i -g @railway/cli
      - name: Deploy
        run: railway up --detach
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
```

## Resources

- [Railway Documentation](https://docs.railway.com)
- [Railway CLI Reference](https://docs.railway.com/reference/cli-api)
- [Railway Templates](https://railway.app/templates)
- [Railway GitHub](https://github.com/railwayapp/cli)
