---
name: apify-actor
description: Build and deploy Apify actors for web scraping and automation. Use for serverless scraping, data extraction, browser automation, and API integrations with Python.
---

# Apify Actor Development

Build serverless Apify actors for web scraping, browser automation, and data extraction using Python.

## Prerequisites & Setup (MANDATORY)

Before creating or modifying actors, verify that `apify` CLI is installed:
Run `apify --help`.

If it is not installed, you can run:
```bash
curl -fsSL https://apify.com/install-cli.sh | bash

# Or (Mac): brew install apify-cli
# Or (Windows): irm https://apify.com/install-cli.ps1 | iex
# Or: npm install -g apify-cli
```

When the apify CLI is installed, check that it is logged in with:
```bash
apify info  # Should return your username
```

If it is not logged in, check if the APIFY_TOKEN environment variable is defined (if not, ask the user to generate one on https://console.apify.com/settings/integrations and then define APIFY_TOKEN with it).

Then run:
```bash
apify login -t $APIFY_TOKEN
```

## Quick Start Workflow

### Creating a New Actor

1. **Copy template** - Copy all files including hidden ones from the skill's `assets/python-template/` directory to your new actor directory. The template is located at `{base_dir}/assets/python-template/` where `{base_dir}` is the skill's base directory.
2. **Setup pre-commit** - Run `uv run pre-commit install` for automatic quality checks
3. **Add dependencies** - Use `uv add package-name` for each required dependency
4. **Implement logic** - Write the actor code in `src/main.py` (the `src/__main__.py` entry point is already set up)
5. **Configure schemas** - Update input/output schemas in `.actor/input_schema.json` and `.actor/output_schema.json`
6. **Configure platform settings** - Update `.actor/actor.json` with actor metadata
7. **Write documentation** - Create comprehensive `.actor/ACTOR.md` for the marketplace
8. **Test locally** - Run `apify run` to verify functionality
9. **Deploy** - Run `apify push` to deploy the actor on the Apify platform

**CRITICAL REMINDERS:**
- NEVER create `requirements.txt`
- NEVER use `pip install` or `uv pip install`
- ALWAYS use `uv add` to add dependencies
- ALWAYS use `uv sync` to install dependencies
- ALWAYS format with `uv run ruff format .` after file changes
- ALWAYS lint with `uv run ruff check --fix .` after file changes
- ALWAYS check the `apify push` output for build errors before considering deployment complete
- Input/output schemas should be updated when changing actor functionality

## Core Concepts

### Input/Output Pattern

Every actor follows this pattern:

1. **Input**: JSON from key-value store (defined by input schema)
2. **Process**: Actor logic extracts/transforms data
3. **Output**: Results pushed to dataset or key-value store

### Storage Types

- **Dataset**: Structured data (arrays of objects) - use for scraping results and tabular data
- **Key-Value Store**: Arbitrary data (files, objects) - use for screenshots, PDFs, state, and binary files
- **Request Queue**: URLs to crawl - use for deep web crawling and multi-page scraping workflows

### Project Structure

```
my-actor/
├── .actor/
│   ├── actor.json                    # Actor metadata
│   ├── input_schema.json             # Input schema
│   ├── output_schema.json            # Output schema
│   ├── ACTOR.md                      # PUBLIC marketplace documentation (CRITICAL)
│   └── datasets/
│       └── dataset_schema.json       # Dataset schema with views
├── src/ or package_name/             # Source code
│   ├── __init__.py
│   ├── __main__.py                   # Entry point for CLI (REQUIRED)
│   └── main.py                       # Main actor logic
├── tests/                            # Test files
│   └── test_*.py
├── .dockerignore                     # Docker build exclusions
├── .pre-commit-config.yaml           # Pre-commit hooks
├── Dockerfile                        # Container config
├── pyproject.toml                    # Python project config
├── uv.lock                          # Dependency lock file
└── README.md                         # Development docs
```

## Common Patterns

See `references/python-sdk.md` for complete examples of:
- Simple HTTP scraping with BeautifulSoup
- Browser automation with Playwright and Selenium
- Deep crawling with Request Queue
- Proxy management and error handling
- Storage APIs (Dataset, Key-Value Store, Request Queue)

## Input Schema Design

Input schemas use JSON Schema format to define and validate actor inputs. See `references/input-schema.md` for:

- Field types (string, number, boolean, array, object)
- Special editors (requestListSources, globs, pseudoUrls, proxy, json, textarea)
- Validation patterns (regex, length, range, required fields)
- Complete examples with best practices

**Key principles:**
- Always include descriptions and examples
- Provide examples for all fields
- Set sensible defaults for ease of use
- Use appropriate editors for better UX
- Add units for numeric fields (pages, seconds, MB)

## Output Schema Design

Output schemas define where actors store outputs and provide templates for accessing that data. See `references/output-schema.md` for:

- Schema structure and template variables (links.apiDefaultDatasetUrl, links.apiDefaultKeyValueStoreUrl, etc.)
- Dataset and key-value store output configurations
- Multiple output types in a single actor
- Integration with Python code
- Complete examples with emojis and descriptions

**Key principles:**
- Define all outputs explicitly (even if empty)
- Use descriptive titles with emojis for visual clarity
- Include helpful descriptions for users and LLM integrations
- Match templates to actual storage locations in code

## ACTOR.md Documentation (CRITICAL)

The `.actor/ACTOR.md` file is **the public-facing documentation** that users see in the Apify marketplace. This is your actor's main sales page and user guide.

**Required sections:**
1. **Title & Description** - Clear, compelling one-liner
2. **What it does** - Bullet points of key capabilities
3. **Input** - Example JSON with field explanations
4. **Output** - Example JSON showing expected results
5. **Use Cases** - Who benefits and why (with emojis)
6. **Standby Mode** (if applicable) - API usage examples
7. **Tips & Best Practices** - Performance and configuration guidance

See `assets/python-template/.actor/ACTOR.md` for a complete template.

**Key principles:**
- Write for non-technical users - assume no coding knowledge
- Use emojis to make sections scannable (🎯 🔍 ⚡ 🚀)
- Provide copy-paste ready code examples
- Show actual input/output samples, not schemas
- Highlight benefits and use cases clearly

## Modifying Existing Actors

When modifying an existing actor:

1. **Understand current logic** - Read `src/main.py`
2. **Check input schema** - Review `.actor/input_schema.json` for expected inputs
3. **Add dependencies with uv** - Use `uv add package-name` (NEVER pip install)
4. **Make code changes** - Implement the requested features
5. **Format code** - Run `uv run ruff format .` (MANDATORY)
6. **Lint code** - Run `uv run ruff check --fix .` (MANDATORY)
7. **Test changes locally** - Use `apify run` before deploying
8. **Update schema if needed** - Add new fields to input schema
9. **Deploy** - Push changes with `apify push`

## Debugging Actors

1. **Test locally** - Use `apify run` to test actor locally before deployment
2. **Check storage** - Inspect `./storage/` directory for datasets, key-value stores, and request queues
3. **Add logging** - Use `Actor.log.info()`, `Actor.log.debug()`, `Actor.log.error()` (see SDK references)
4. **View logs on platform** - Check actor run logs in Apify Console for production issues

## Best Practices

### Code Quality

- **Validate input** - Always check required fields and formats with clear error messages
- **Handle errors** - Use try/catch with proper error logging and graceful degradation
- **Structured logging** - Use Actor.log with extra fields for better debugging
- **Type hints** - Add type annotations for better code clarity and IDE support
- **Docstrings** - Document functions and modules for maintainability
- **Format with ruff** - ALWAYS run `uv run ruff format .` before committing
- **Lint with ruff** - ALWAYS run `uv run ruff check --fix .` before deploying

### Performance & Scalability

- **Batch processing** - Push data in batches (100-1000 items) for large datasets to reduce API calls
- **Use proxies** - Avoid IP blocking for web scraping with proxy configuration
- **Resource limits** - Set appropriate memory limits and timeouts in `.actor/actor.json`
- **Optimize Docker** - Use multi-stage builds, bytecode compilation, and minimal base images
- **Consider Standby mode** - For low-latency (<100ms), high-frequency use cases

### Security & Configuration

- **Environment variables** - Never hardcode secrets; use `Actor.config` and environment variables
- **Input validation** - Use JSON Schema patterns, required fields, and runtime validation
- **Run as non-root** - Use `myuser` in Dockerfile for container security
- **Minimize image size** - Use `.dockerignore` to exclude unnecessary files and reduce build time

### Development Workflow

- **Testing** - Write tests with pytest; use coverage and snapshot testing for reliability
- **Pre-commit hooks** - Use ruff and pre-commit for consistent code quality (MANDATORY)
- **Use uv exclusively** - NEVER use pip or requirements.txt; only use `uv add` and `uv sync` (MANDATORY)
- **Lock dependencies** - Always commit `uv.lock` for reproducible builds (MANDATORY)
- **Test locally** - Always test with `apify run` before deploying to catch issues early
- **Dataset schemas** - Define `dataset_schema.json` with views for better Apify Console UI
- **CLI support** - Add CLI entry points via `__main__.py` for local testing and development

## Standby Mode (Real-time API)

Standby mode allows actors to run as persistent HTTP servers, providing instant responses without cold start delays.

**Perfect for:**
- Real-time APIs requiring <100ms response times
- Webhook endpoints that need immediate processing
- High-frequency requests (multiple requests per second)
- Integration with real-time services (Slack bots, chat applications, webhooks)
- Low-latency scraping APIs and on-demand data extraction

See `references/standby-mode.md` for complete implementation patterns, authentication, and examples.

## References

Detailed documentation in `references/`:

- `python-sdk.md` - SDK patterns and complete code examples
- `standby-mode.md` - Real-time API implementation
- `input-schema.md` - Input validation and UI configuration
- `output-schema.md` - Output configuration and templates

## Troubleshooting

If you need information not covered in this skill, use the WebFetch tool with https://docs.apify.com/llms.txt to access the complete official documentation.