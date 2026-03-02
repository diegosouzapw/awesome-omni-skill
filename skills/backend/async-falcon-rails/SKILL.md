---
name: async-falcon-rails
description: Transform a Rails application to use Falcon web server with async job processing (async-job), async Action Cable, and Redis-compatible database (Valkey for production). Use when the user wants to add async/Falcon stack to a Rails project, migrate from Puma to Falcon, or set up async job processing with Redis for both development and production environments including Kamal deployment.
---

# Async Falcon Rails

## Overview

Transform a Rails application to use the Falcon web server with async job processing, async Action Cable, and Redis-compatible database (Valkey for production). This skill handles the complete migration from Puma to Falcon, configures async job adapters, sets up Redis/Valkey for Action Cable and job queues, and configures Kamal deployment for production.

## When to Use

Use this skill when the user:
- Wants to add async/Falcon stack to an existing Rails project
- Needs to migrate from Puma to Falcon web server
- Requests async job processing setup with Redis
- Wants to configure async Action Cable
- Needs Kamal deployment configuration for the async stack

## Prerequisites

Before applying this skill, verify:
1. The project is a Rails application (check for `Gemfile`, `config/application.rb`)
2. The project structure includes `config/environments/` directory
3. Bundle is available (`bundle` command works)
4. If Kamal deployment is needed, check for `config/deploy.yml`

## Workflow

Follow these steps in order to transform a Rails application to use the async/Falcon stack:

### Step 1: Update Gemfile Dependencies

Replace Puma with Falcon and add async dependencies:

```bash
bundle remove puma
bundle add falcon
bundle add async-job-processor-redis
bundle add async-job-adapter-active_job
bundle add async-cable
bundle add redis
```

After running these commands, verify the Gemfile includes:
- `gem "falcon"`
- `gem "async-job-processor-redis"`
- `gem "async-job-adapter-active_job"`
- `gem "async-cable"`
- `gem "redis"` (provides `Async::Redis::Endpoint` for endpoint parsing)

### Step 2: Update Dockerfile for SSL Dependencies

**CRITICAL:** The async/Falcon stack requires OpenSSL development libraries to build properly. Without this, Docker builds will fail.

Edit the `Dockerfile` and add `libssl-dev` to the system dependencies.

Find the line that installs build packages (usually around line 40):

```dockerfile
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y build-essential git libyaml-dev pkg-config && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives
```

Update it to include `libssl-dev`:

```dockerfile
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y build-essential git libyaml-dev libssl-dev pkg-config && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives
```

**Why this is needed:** The Falcon web server and async gems depend on native extensions that require OpenSSL headers to compile. Without `libssl-dev`, the Docker build will fail with compilation errors.

### Step 3: Create Async Job Configuration

Create `config/initializers/async_job.rb` with the following content:

```ruby
require "async/job"
require "async/job/processor/aggregate"
require "async/job/processor/redis"
require "async/job/processor/inline"
require "async/redis/endpoint"

Rails.application.configure do
  # Resolve Redis endpoint from REDIS_URL; fallback to localhost for dev/test.
  redis_endpoint = Async::Redis::Endpoint.parse(
    ENV.fetch("REDIS_URL") { "redis://localhost:6379/1" }
  )

  config.async_job.define_queue "default" do
    enqueue Async::Job::Processor::Aggregate
    # Ensure the job runner connects to the accessory container (or localhost in dev).
    dequeue Async::Job::Processor::Redis, endpoint: redis_endpoint
  end

  config.async_job.define_queue "local" do
    dequeue Async::Job::Processor::Inline
  end
end
```

This configuration:
- Sets up a "default" queue using Redis for job processing
- Parses the REDIS_URL environment variable to create a proper Redis endpoint
- Passes the endpoint to the Redis processor for both development and production
- Sets up a "local" queue for inline processing
- Uses Aggregate processor for enqueuing and Redis for dequeuing

### Step 4: Update Procfile for Development

Update `Procfile.dev` to include the async job processor:

Add this line to the existing Procfile.dev:
```
jobs: bundle exec async-job-adapter-active_job-server
```

The complete Procfile.dev should include at minimum:
```
web: bin/rails s
jobs: bundle exec async-job-adapter-active_job-server
```

If the project uses Vite or other frontend build tools, keep those lines as well.

### Step 5: Configure Development Environment

Edit `config/environments/development.rb` to use async_job queue adapter.

Find the section about ActiveJob (usually near `config.active_job.verbose_enqueue_logs`) and add:

```ruby
config.active_job.queue_adapter = :async_job
```

### Step 6: Configure Production Environment

Edit `config/environments/production.rb` to configure both async_job and Redis cache:

Find and update or add these configurations:

```ruby
# For cache store (usually around line 50)
config.cache_store = :redis_cache_store, { url: ENV["REDIS_URL"] }

# For queue adapter (usually around line 53)
config.active_job.queue_adapter = :async_job
```

### Step 7: Configure Application for Async/Cable

Edit `config/application.rb` to add async/cable support:

1. At the top of the file, after `require "rails/all"`, add:
```ruby
require "async/cable"
```

2. Inside the `class Application < Rails::Application` block, after `config.load_defaults`, add:
```ruby
# Configure async/fiber support
config.active_record.permanent_connection_checkout = :disallowed
config.active_support.isolation_level = :fiber
```

These settings enable fiber-based isolation for async operations.

### Step 8: Generate Action Cable Base Channel

Run the Rails generator to create the base Action Cable structure:

```bash
bin/rails generate channel BaseChannel
```

This creates:
- `app/channels/application_cable/channel.rb`
- `app/channels/application_cable/connection.rb`
- `app/channels/base_channel.rb`
- `test/channels/base_channel_test.rb`

### Step 9: Mount Action Cable in Routes

Edit `config/routes.rb` to mount the Action Cable server.

Add this line at the top of the `Rails.application.routes.draw` block:

```ruby
mount ActionCable.server => '/cable'
```

### Step 10: Configure Cable to Use Redis

Edit `config/cable.yml` to use Redis for all environments:

Update the configuration to:

```yaml
development:
  adapter: redis
  url: <%= ENV.fetch("REDIS_URL") { "redis://localhost:6379/1" } %>
  channel_prefix: PROJECT_NAME_production

test:
  adapter: test

production:
  adapter: redis
  url: <%= ENV.fetch("REDIS_URL") { "redis://localhost:6379/1" } %>
  channel_prefix: PROJECT_NAME_production
```

Replace `PROJECT_NAME` with the actual Rails application name (found in `config/application.rb` as the module name).

### Step 11: Configure Kamal Deployment (If Applicable)

If the project uses Kamal for deployment (check for `config/deploy.yml`), update the deployment configuration:

#### 11.1: Add Job Server

In the `servers:` section, add or uncomment the job server configuration:

```yaml
servers:
  web:
    - 192.168.0.1
  job:
    hosts:
      - 192.168.0.1
    cmd: bundle exec async-job-adapter-active_job-server
    options:
      init: true
```

**Key configuration:**
- `init: true`: Skips health checks for the job server (avoids 30-second deployment wait)
- Job servers don't expose HTTP endpoints, so health checks would timeout unnecessarily

#### 11.2: Configure Redis/Valkey Accessory

In the `accessories:` section (create if it doesn't exist), add Redis/Valkey configuration:

```yaml
accessories:
  redis:
    image: valkey/valkey:9
    host: 192.168.0.1
    port: "127.0.0.1:6379:6379"
    directories:
      - redis_data:/data
```

Key considerations:
- **Important:** Use Valkey instead of Redis due to Redis licensing changes. Valkey is a Redis-compatible fork maintained by the Linux Foundation
- Use Valkey 9 or latest stable version
- Port binding `"127.0.0.1:6379:6379"` prevents public exposure (localhost only)
- Volume name `redis_data` prevents conflicts with other services (e.g., PostgreSQL)

#### 11.3: Add REDIS_URL Environment Variable

In the `env:` section under `clear:`, add:

```yaml
env:
  clear:
    REDIS_URL: redis://PROJECT_NAME-redis:6379/1
```

Replace `PROJECT_NAME` with the service name from the top of deploy.yml. The format follows Kamal's Docker naming convention: `{service_name}-{accessory_name}`.

#### 11.4: Configure Multi-Architecture Builds

Update the `builder:` section to support multiple architectures:

```yaml
builder:
  arch:
    - amd64
    - arm64
```

This enables building Docker images for:
- **amd64**: Intel/AMD processors (Windows, Linux, older Macs)
- **arm64**: Apple Silicon (M1/M2/M3 Macs), ARM-based Linux servers

## Verification Steps

After completing the workflow, verify the setup:

1. **Gemfile**: Check that Puma is removed and all async gems are added
2. **Dockerfile**: Verify `libssl-dev` is included in the apt-get install line
3. **Initializers**: Verify `config/initializers/async_job.rb` exists and includes endpoint configuration (`endpoint: redis_endpoint`)
4. **Environments**: Confirm `async_job` queue adapter in development.rb and production.rb
5. **Application**: Verify `async/cable` require and fiber isolation config in application.rb
6. **Cable**: Check that Action Cable is mounted in routes.rb
7. **Cable Config**: Confirm cable.yml uses Redis for development and production
8. **Kamal** (if applicable): Verify job server with `init: true`, Redis accessory, REDIS_URL, and multi-arch build config in deploy.yml

## Important Notes

### Redis/Valkey Dependency

This stack requires Redis-compatible database to be running:
- **Development**: Start Redis locally with `redis-server` or use Docker
- **Production**: Valkey (Redis-compatible) is deployed as a Kamal accessory (configured in deploy.yml)
- **Note**: We use Valkey instead of Redis in production due to Redis licensing changes. Valkey is a fully Redis-compatible fork maintained by the Linux Foundation

### Environment Variables

The `REDIS_URL` environment variable must be set:
- **Development**: Defaults to `redis://localhost:6379/1` (configured in cable.yml)
- **Production**: Set via Kamal deploy.yml or environment configuration

### Kamal Naming Conventions

When using Kamal:
- Redis accessory will be named: `{service_name}-redis`
- Use this name in REDIS_URL: `redis://{service_name}-redis:6379/1`
- Volume names should be descriptive: `redis_data`, `postgres_data`, etc.

### Port Binding Security

The Redis port binding `"127.0.0.1:6379:6379"` ensures:
- Redis is accessible to containers on the same Docker network
- Redis is NOT exposed to the public internet
- Only localhost connections are allowed on the host

## Troubleshooting

### Docker Build Errors

If Docker build fails with compilation errors about OpenSSL or missing headers:
- **Symptom**: Build fails during gem installation with errors like "openssl/ssl.h: No such file or directory"
- **Cause**: Missing `libssl-dev` system dependency in Dockerfile
- **Solution**: Add `libssl-dev` to the apt-get install line in Dockerfile (see Step 2)
- **Verification**: Check that the Dockerfile includes `libssl-dev` in the package list alongside `build-essential`, `git`, `libyaml-dev`, and `pkg-config`

This is a critical dependency for Falcon and async gems - without it, the Docker image cannot be built.

### Bundle Errors

If `bundle add` commands fail:
- Check that Gemfile is not locked with version conflicts
- Try `bundle update` to resolve dependency issues
- Verify network connectivity to rubygems.org

### Redis Connection Errors

If the application cannot connect to Redis:
- **Development**: Ensure Redis is running locally (`redis-cli ping` should return `PONG`)
- **Production**: Check that REDIS_URL environment variable is set correctly
- Verify cable.yml configuration matches the REDIS_URL format

### Kamal Deployment Issues

If Kamal deployment fails:
- Verify all placeholders (PROJECT_NAME, IP addresses) are replaced with actual values
- Check that Redis accessory is running: `kamal accessory details redis`
- Ensure REDIS_URL matches the Kamal service naming convention

### Deployment Timeouts

If Kamal deployment hangs for 30 seconds when deploying the job server:
- **Symptom**: Deployment waits and then shows "Container not ready" for job server
- **Cause**: Job server doesn't expose HTTP endpoints, so health checks timeout
- **Solution**: Add `options:` with `init: true` to the job server configuration in deploy.yml (see Step 11.1)
- **Why it works**: `init: true` tells Kamal to skip health checks for this service

## References

For detailed configuration templates and examples, see:
- `references/configuration-templates.md` - Complete file templates and patterns
