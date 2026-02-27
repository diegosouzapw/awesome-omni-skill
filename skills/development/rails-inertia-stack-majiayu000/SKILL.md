---
name: rails-inertia-stack
description: Generate production-ready Rails applications with Inertia.js, React, TypeScript, Server-Side Rendering (SSR), and ShadcnUI components, configured for deployment with Kamal. Use when creating new Rails projects that need modern SPA-like frontend with SEO-friendly SSR, or when helping users set up Inertia.js with Rails. Supports PostgreSQL, MySQL, and SQLite databases.
---

# Rails Inertia Stack Generator

## Overview

Generate complete, production-ready Rails applications with Inertia.js for SPA-like user experience, React + TypeScript frontend, Server-Side Rendering for SEO, ShadcnUI component library, and Kamal deployment configuration. The skill handles the entire setup process including Dockerfile configuration for SSR, database accessories, and deployment configuration.

## When to Use This Skill

Use this skill when:
- Creating new Rails projects with modern frontend stack
- Setting up Inertia.js with React and TypeScript in Rails
- Configuring Server-Side Rendering for Rails + Inertia applications
- Setting up ShadcnUI with Rails projects
- Preparing Rails applications for Kamal deployment
- Migrating between databases (PostgreSQL/MySQL/SQLite)

## Core Workflow

### Step 1: Determine Project Requirements

Ask the user for project configuration:
- **Project name** (required)
- **Database choice**: PostgreSQL (recommended for production), MySQL, or SQLite
- **Server IP addresses** for deployment (can be configured later)
- **Domain name** for SSL (optional, can be configured later)

### Step 2: Create Rails Project

Execute the appropriate Rails new command based on database choice:

**PostgreSQL:**
```bash
rails new PROJECT_NAME -d postgresql --skip-javascript
cd PROJECT_NAME
```

**MySQL:**
```bash
rails new PROJECT_NAME -d mysql --skip-javascript
cd PROJECT_NAME
```

**SQLite:**
```bash
# Default with Solid stack
rails new PROJECT_NAME --skip-javascript
cd PROJECT_NAME

# Or without Solid stack (if user requests)
rails new PROJECT_NAME --skip-javascript --skip-solid
cd PROJECT_NAME
```

### Step 3: Install Inertia Rails Stack

Run these commands in sequence:

```bash
# Add Inertia Rails gem
bundle add inertia_rails

# Install frontend stack (non-interactive)
bin/rails generate inertia:install \
  --framework=react \
  --typescript \
  --vite \
  --tailwind \
  --no-interactive
```

**When prompted about bin/dev conflict:** Choose `Y` to overwrite.

After installation:
```bash
# Setup databases
bin/rails db:setup
bin/rails db:migrate
```

### Step 4: Configure RuboCop (Optional but Recommended)

Rails 8 includes RuboCop Rails Omakase by default. Create or update `.rubocop.yml`:

```yaml
# Omakase Ruby styling for Rails
inherit_gem: { rubocop-rails-omakase: rubocop.yml }

# Overwrite or add rules to create your own house style
#
# # Use `[a, [b, c]]` not `[ a, [ b, c ] ]`
# Layout/SpaceInsideArrayLiteralBrackets:
#   Enabled: false

# Restore strict 2-space indentation enforcement
Layout/IndentationConsistency:
  Enabled: true

Layout/IndentationWidth:
  Enabled: true
  Width: 2
```

**Why this matters:**
- Maintains consistent code style across the project
- Omakase provides sensible Rails defaults
- Strict 2-space indentation ensures readability

### Step 5: Fix Development Configuration

Apply two critical fixes for development:

**Fix 1: Procfile.dev** (ensure Rails runs on port 3000)

Read `Procfile.dev` and swap the order:
```
web: bin/rails s
vite: bin/vite dev
```

**Fix 2: config/vite.json** (enable 127.0.0.1 access)

Add `"host": "127.0.0.1"` to development section:
```json
{
  "development": {
    "autoBuild": true,
    "publicOutputDir": "vite-dev",
    "port": 3036,
    "host": "127.0.0.1"
  }
}
```

### Step 6: Setup ShadcnUI

Configure TypeScript for ShadcnUI (CRITICAL: update BOTH files):

**tsconfig.app.json** - Add to compilerOptions:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./app/frontend/*"]
    }
  }
}
```

**tsconfig.json** - Add to root:
```json
{
  "compilerOptions": {
    "baseUrl": "./app/frontend",
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

**Initialize ShadcnUI (non-interactive):**
```bash
npx shadcn@latest init --defaults --yes
npx shadcn@latest add button --yes --overwrite
```

### Step 7: Configure Server-Side Rendering (SSR)

**Create SSR entry point** at `app/frontend/ssr/ssr.tsx`:
```typescript
import { createInertiaApp } from '@inertiajs/react'
import createServer from '@inertiajs/react/server'
import ReactDOMServer from 'react-dom/server'

createServer((page) =>
  createInertiaApp({
    page,
    render: ReactDOMServer.renderToString,
    resolve: (name) => {
      const pages = import.meta.glob('../pages/**/*.tsx', { eager: true })
      return pages[`../pages/${name}.tsx`]
    },
    setup: ({ App, props }) => <App {...props} />,
  }),
)
```

**Enable client-side hydration** in `app/frontend/entrypoints/inertia.ts`:

Add `hydrateRoot` import:
```typescript
import { createRoot, hydrateRoot } from 'react-dom/client'
```

Update setup function:
```typescript
setup({ el, App, props }) {
  if (el) {
    if (import.meta.env.MODE === "production") {
      hydrateRoot(el, createElement(App, props))
    } else {
      createRoot(el).render(createElement(App, props))
    }
  } else {
    console.error('Missing root element...')
  }
}
```

**Enable SSR in Vite** - Add to `config/vite.json`:
```json
{
  "production": {
    "ssrBuildEnabled": true
  }
}
```

**Enable SSR in Inertia Rails** - Update `config/initializers/inertia_rails.rb`:
```ruby
InertiaRails.configure do |config|
  config.version = ViteRuby.digest
  config.encrypt_history = true
  config.always_include_errors_hash = true
  config.ssr_enabled = ViteRuby.config.ssr_build_enabled
end
```

### Step 8: Configure Dockerfile for SSR

TWO modifications needed to the generated Dockerfile!

Reference `references/dockerfile-ssr-patterns.md` for complete examples.

**Modification 1: Install Node.js in BASE stage**

Add AFTER "Install base packages", BEFORE bundler installation:

```dockerfile
# Install JavaScript runtime (prebuilt Node per-arch)
ARG NODE_VERSION=25.0.0
ARG TARGETARCH
ENV PATH=/usr/local/node/bin:$PATH
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y xz-utils && \
    case "${TARGETARCH}" in \
      amd64) NODEARCH=x64 ;; \
      arm64) NODEARCH=arm64 ;; \
      *) echo "Unsupported TARGETARCH: ${TARGETARCH}" >&2; exit 1 ;; \
    esac && \
    mkdir -p /usr/local/node && \
    curl -fsSL "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-${NODEARCH}.tar.xz" | \
      tar -xJ -C /usr/local/node --strip-components=1 && \
    /usr/local/node/bin/node -v && \
    /usr/local/node/bin/npm -v && \
    apt-get purge -y --auto-remove xz-utils && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives
```

**Modification 2: Install Bundler with Locked Version**

Add AFTER Node.js installation, BEFORE "Set production environment":

```dockerfile
# Ensure the Bundler version matches Gemfile.lock to avoid per-build upgrades.
RUN gem install bundler -v 2.7.2 -N
```

**Important:** Check `Gemfile.lock` bottom section for `BUNDLED WITH` version and update to match:
```bash
tail Gemfile.lock
# BUNDLED WITH
#    2.7.2
```

**That's all!** No changes to BUILD stage (Vite Ruby handles npm install), no changes to final stage, no EXPOSE, no CMD modifications.

**Database client variations** - adjust base packages only:
- PostgreSQL: `postgresql-client` (base) and `libpq-dev` (build)
- MySQL: `default-mysql-client` (base) and `libmysqlclient-dev` (build)
- SQLite: No database client needed

### Step 9: Configure Kamal Deployment

Update `config/deploy.yml` with SSR and database configuration.

Reference `references/kamal-ssr-deployment.md` and `references/complete-guide.md` for detailed examples.

**Key additions:**
1. Add `vite` server with `init: true` and `network-alias: vite_ssr` options
2. Add `INERTIA_SSR_URL: http://vite_ssr:13714` to env
3. Configure database accessory (PostgreSQL/MySQL) or volumes (SQLite)
4. Update database environment variables
5. (Optional) Add Redis/Valkey accessory if needed for caching, queues, or Action Cable

**Redis/Valkey Configuration (if requested):**
Use Valkey instead of Redis due to licensing concerns:
```yaml
accessories:
  redis:
    image: valkey/valkey:9  # Use Valkey, not redis image
    host: SERVER_IP
    port: "127.0.0.1:6379:6379"
    directories:
      - redis_data:/data

env:
  clear:
    REDIS_URL: redis://PROJECT_NAME-redis:6379/1
```

**Solid Queue Configuration:**
- If user created project WITH Solid (default), include:
  ```yaml
  env:
    clear:
      SOLID_QUEUE_IN_PUMA: true
  ```
- If user created project with `--skip-solid`, OMIT this variable

**Optional: Async Job Server (Advanced):**
For dedicated job processing (alternative to `SOLID_QUEUE_IN_PUMA`), add a job server:
```yaml
servers:
  job:
    hosts:
      - SERVER_IP
    cmd: bundle exec async-job-adapter-active_job-server
```
This runs jobs in a separate container for better resource isolation.

**Create database initialization file** at `db/production.sql`:
```sql
CREATE DATABASE PROJECT_NAME_production_cache;
CREATE DATABASE PROJECT_NAME_production_queue;
CREATE DATABASE PROJECT_NAME_production_cable;
```

**Update database.yml** production section:
```yaml
production:
  primary: &primary_production
    <<: *default
    host: <%= ENV["DB_HOST"] %>
    database: PROJECT_NAME_production
    username: PROJECT_NAME
    password: <%= ENV["POSTGRES_PASSWORD"] %>  # or MYSQL_ROOT_PASSWORD
```

### Step 10: Test SSR Build

Verify the setup works:
```bash
export RAILS_ENV=production
./bin/rails assets:precompile
```

Check for successful output:
- Client bundle build completes
- SSR bundle build completes
- `public/vite-ssr/ssr.js` exists

### Step 11: Provide Deployment Instructions

Inform the user about deployment configuration:

1. **Configure secrets** in `.env` (git-ignored):
   ```bash
   POSTGRES_PASSWORD=secure-password
   KAMAL_REGISTRY_PASSWORD=docker-hub-token
   ```

2. **Update `.kamal/secrets`**:
   ```bash
   KAMAL_REGISTRY_PASSWORD=$KAMAL_REGISTRY_PASSWORD
   RAILS_MASTER_KEY=$(cat config/master.key)
   POSTGRES_PASSWORD=$POSTGRES_PASSWORD
   ```

3. **Update `config/deploy.yml`** with actual server IPs and domain

4. **Deploy**:
   ```bash
   export $(grep -v '^#' .env | xargs)
   git add . && git commit -m "Setup Inertia Rails with SSR"
   kamal setup
   kamal deploy
   ```

## Critical Patterns

### Non-Interactive Installation
Always use these flags to avoid prompts:
```bash
# ShadcnUI initialization
npx shadcn@latest init --defaults --yes
npx shadcn@latest add COMPONENT --yes --overwrite

# Inertia installation
bin/rails generate inertia:install --no-interactive
```

### SSR Build Process
- ✅ Use `rails assets:precompile` (handles both client and SSR, plus npm install)
- ❌ Do NOT run `bin/vite build --ssr` separately (redundant)
- ❌ Do NOT run `npm ci` separately (Vite Ruby handles it)

### Dockerfile Node.js
- ✅ Install Node.js in base stage using prebuilt binaries (supports multi-arch)
- ✅ Node.js automatically included in final stage
- ❌ Do NOT remove Node.js after build (breaks SSR)
- ❌ Do NOT run npm ci separately (Vite Ruby handles it)

### Kamal SSR Architecture
```
Web Container (Rails) ←→ vite_ssr Container (Node.js)
              via http://vite_ssr:13714
```

Fixed hostname via `network-alias: vite_ssr` enables reliable connection.

## Database-Specific Variations

### PostgreSQL (Production Recommended)
- **Gem:** `pg`
- **Dockerfile client:** `postgresql-client`
- **Dockerfile build lib:** `libpq-dev`
- **Accessory image:** `postgres:18` (latest stable)
- **Volume mount:** `data:/var/lib/postgresql` (BREAKING CHANGE in PostgreSQL 18: previous versions used `data:/var/lib/postgresql/data`)
- **Port:** `127.0.0.1:5432:5432`
- **Env vars:** `POSTGRES_USER`, `POSTGRES_DB`, `POSTGRES_PASSWORD`

### MySQL
- **Gem:** `mysql2`
- **Dockerfile client:** `default-mysql-client`
- **Dockerfile build lib:** `libmysqlclient-dev`
- **Accessory image:** `mysql:9.4.0` (latest Innovation) or `mysql:8.4` (LTS recommended for production)
- **Port:** `127.0.0.1:3306:3306`
- **Env vars:** `MYSQL_ROOT_HOST`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_ROOT_PASSWORD`

### SQLite (Development/Small Apps)
- **Gem:** `sqlite3` (default)
- **Dockerfile:** No database client needed
- **Deployment:** Use volumes instead of accessory
- **Important:** SQLite databases are stored in `storage/` directory in Rails 8+
- **Volume configuration:**
  ```yaml
  # CORRECT: Mount storage directory (contains production.sqlite3)
  volumes:
    - "PROJECT_NAME_storage:/rails/storage"

  # INCORRECT: Do not mount db directory
  # volumes:
  #   - "PROJECT_NAME_db:/rails/db"
  ```
- **Database location:** Check `config/database.yml` production section for actual path:
  ```yaml
  production:
    <<: *default
    database: storage/production.sqlite3  # Note: stored in storage/ not db/
  ```

## Common Pitfalls to Avoid

1. **Missing BOTH tsconfig updates** - ShadcnUI requires updating both `tsconfig.json` and `tsconfig.app.json`
2. **Interactive commands** - Always use `--defaults --yes` with shadcn, `--no-interactive` with inertia:install
3. **Removing Node.js from Dockerfile** - Node.js must remain in production for SSR
4. **Missing network-alias** - Rails cannot connect to SSR without `network-alias: vite_ssr`
5. **Wrong Procfile.dev order** - Vite first causes Rails to run on port 3100 instead of 3000
6. **Missing vite.json host** - Without `"host": "127.0.0.1"`, connections to 127.0.0.1 fail
7. **Not updating database.yml** - Must add `host: <%= ENV["DB_HOST"] %>` and password for production
8. **Wrong SQLite volume path** - Must mount `/rails/storage` (not `/rails/db`) since Rails 8 stores SQLite in storage/ directory
9. **Using Redis image instead of Valkey** - Use `valkey/valkey:9` image to avoid Redis licensing concerns
10. **Bundler version mismatch** - Must lock bundler version in Dockerfile to match `Gemfile.lock` to prevent cache invalidation

## Troubleshooting

### SSR Build Fails
- Check Node.js is in Dockerfile base stage
- Verify `config/vite.json` has `"ssrBuildEnabled": true`
- Ensure `app/frontend/ssr/ssr.tsx` exists
- Run `npm ci` to reinstall dependencies

### Rails Can't Connect to SSR
- Verify `vite` server has `network-alias: vite_ssr` in deploy.yml
- Check `INERTIA_SSR_URL: http://vite_ssr:13714` in env
- Ensure both web and vite containers are on same network

### Database Connection Fails
- Verify `DB_HOST` matches accessory service name (e.g., `PROJECT_NAME-db`)
- Check database accessory is running: `kamal accessory details db`
- Confirm password matches between `.kamal/secrets` and `config/database.yml`

### ShadcnUI Components Not Resolving
- Verify BOTH tsconfig files have path aliases
- Ensure imports use `@/` prefix: `import { Button } from '@/components/ui/button'`

## Resources

### references/
- `complete-guide.md` - Complete 778-line implementation guide with all variations
- `dockerfile-ssr-patterns.md` - Dockerfile modifications for PostgreSQL/MySQL/SQLite
- `kamal-ssr-deployment.md` - Kamal SSR architecture and deployment examples

### assets/
- `ssr-entry.tsx` - SSR server entry point template
- `deploy-postgres.yml` - PostgreSQL deployment configuration
- `deploy-mysql.yml` - MySQL deployment configuration
- `deploy-sqlite.yml` - SQLite deployment configuration

Use these references when detailed examples are needed or when helping users with specific database configurations.
