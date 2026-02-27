---
name: configuring-devenv
description: Initializes and configures devenv development environments. Searches packages, sets up languages, services, scripts, git hooks, and processes. Use when setting up devenv, adding packages to devenv.nix, configuring languages, services, git hooks, or searching for devenv options.
argument-hint: [action] [query]
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - mcp__mcp_devenv_sh__search_packages
  - mcp__mcp_devenv_sh__search_options
---

# Devenv Configuration

Initializes, configures, and searches devenv development environments.

## Your Task

Determine action from $ARGUMENTS or context:

| Intent | Action |
|--------|--------|
| "init devenv", "set up devenv", "new devenv" | Initialize new project |
| "add package", "configure", "enable" | Modify configuration |
| "search", "find package", "how to add" | Search packages/options |

## Progress Checklist

- [ ] Determine action (init/config/search)
- [ ] Check current state
- [ ] Execute action
- [ ] Validate changes
- [ ] Guide next steps

---

## Initializing Projects

### Step 1: Check Prerequisites

```bash
# Check if devenv exists
test -f devenv.nix && echo "exists" || echo "new"

# Check if devenv is installed
devenv version
```

If not installed: `curl -L https://get.devenv.sh | sh`

### Step 2: Detect Project Type

| File | Project Type |
|------|--------------|
| `package.json` | Node.js |
| `pyproject.toml`, `requirements.txt` | Python |
| `Cargo.toml` | Rust |
| `go.mod` | Go |
| Multiple | Full-stack |

### Step 3: Create Configuration

**Python:**

```nix
{ pkgs, ... }:

{
  languages.python = {
    enable = true;
    version = "3.11";
    venv.enable = true;
  };

  scripts = {
    test.exec = "pytest";
    lint.exec = "ruff check .";
  };

  git-hooks.hooks = {
    black.enable = true;
    ruff.enable = true;
  };
}
```

**Node.js:**

```nix
{ pkgs, ... }:

{
  languages.javascript = {
    enable = true;
    npm.enable = true;  # or bun.enable
  };

  scripts = {
    dev.exec = "npm run dev";
    test.exec = "npm test";
  };

  git-hooks.hooks = {
    prettier.enable = true;
    eslint.enable = true;
  };
}
```

**Rust:**

```nix
{ pkgs, ... }:

{
  languages.rust.enable = true;

  packages = with pkgs; [ cargo-watch ];

  scripts = {
    dev.exec = "cargo watch -x run";
    test.exec = "cargo test";
  };

  git-hooks.hooks = {
    rustfmt.enable = true;
    clippy.enable = true;
  };
}
```

**Go:**

```nix
{ pkgs, ... }:

{
  languages.go.enable = true;

  packages = with pkgs; [ golangci-lint air ];

  scripts = {
    dev.exec = "air";
    test.exec = "go test ./...";
  };

  git-hooks.hooks = {
    gofmt.enable = true;
    govet.enable = true;
  };
}
```

### Step 4: Create Supporting Files

**.envrc:**

```bash
use devenv
```

**.gitignore additions:**

```
.devenv*
devenv.lock
.direnv
```

### Step 5: Validate

```bash
nix-instantiate --parse devenv.nix
devenv info
direnv allow
```

---

## Configuring Projects

### Always Read First

```bash
cat devenv.nix
```

### Languages

```nix
languages = {
  javascript = {
    enable = true;
    npm.enable = true;      # or bun.enable, pnpm.enable
  };
  python = {
    enable = true;
    version = "3.11";
    venv.enable = true;
  };
  rust.enable = true;
  go.enable = true;
};
```

### Packages

```nix
packages = with pkgs; [
  gh            # GitHub CLI
  jq            # JSON processor
  docker
  kubectl
];
```

### Scripts

```nix
scripts = {
  dev.exec = "npm run dev";
  test.exec = "pytest tests/";
  db-reset.exec = ''
    dropdb myapp --if-exists
    createdb myapp
  '';
};
```

### Git Hooks

```nix
git-hooks.hooks = {
  prettier.enable = true;
  eslint.enable = true;
  black.enable = true;
  rustfmt.enable = true;
  shellcheck.enable = true;
  gitleaks.enable = true;
};
```

### Services

```nix
services = {
  postgres = {
    enable = true;
    initialDatabases = [{ name = "myapp_dev"; }];
  };
  redis.enable = true;
  mysql.enable = true;
};
```

### Processes

```nix
processes = {
  backend.exec = "uvicorn main:app --reload";
  frontend.exec = "npm run dev";
};
```

Start all with `devenv up`.

### Environment Variables

```nix
env = {
  DATABASE_URL = "postgresql://localhost/myapp";
  API_KEY = "dev-key";
};
```

### Validation After Changes

```bash
nix-instantiate --parse devenv.nix
devenv info
direnv reload
```

---

## Searching Packages/Options

### Using CLI

```bash
devenv search <query>
```

### Using MCP Tools

Use `mcp__mcp_devenv_sh__search_packages` and `mcp__mcp_devenv_sh__search_options`.

### Common Corrections

| User Types | Search For |
|------------|------------|
| node, npm | nodejs |
| pg, postgresql | postgres |
| k8s | kubectl |
| python3 | python |
| mongo | mongodb |

### Present Results

```markdown
## Search Results for "{query}"

### Configuration Options
| Option | Description |
|--------|-------------|
| `languages.python.enable` | Enable Python support |

### Packages
| Package | Description |
|---------|-------------|
| `python311` | Python 3.11 interpreter |

### Usage Example
(show nix snippet)
```

---

## Full-Stack Configuration

For frontend + backend + database:

```nix
{ pkgs, ... }:

{
  languages = {
    javascript = {
      enable = true;
      npm.enable = true;
    };
    python = {
      enable = true;
      version = "3.11";
      venv.enable = true;
    };
  };

  env = {
    DATABASE_URL = "postgresql://localhost:5432/myapp_dev";
    API_URL = "http://localhost:8000";
  };

  services.postgres = {
    enable = true;
    initialDatabases = [{ name = "myapp_dev"; }];
  };

  processes = {
    backend = {
      exec = "cd backend && uvicorn main:app --reload --port 8000";
      process-compose = {
        depends_on.postgres.condition = "process_healthy";
      };
    };
    frontend = {
      exec = "cd frontend && npm run dev";
      process-compose = {
        depends_on.backend.condition = "process_healthy";
      };
    };
  };

  scripts = {
    dev.exec = "devenv up";
    db-reset.exec = ''
      dropdb myapp_dev --if-exists
      createdb myapp_dev
    '';
  };

  git-hooks.hooks = {
    prettier.enable = true;
    black.enable = true;
  };
}
```

---

## Error Handling

| Issue | Solution |
|-------|----------|
| devenv not found | `curl -L https://get.devenv.sh \| sh` |
| Syntax error | `nix-instantiate --parse devenv.nix` |
| Package not found | Search with MCP tools or `devenv search` |
| Changes not appearing | `direnv reload` or restart shell |
| Service won't start | Check `$DEVENV_STATE` logs |
| Port in use | `lsof -i :<port>` to find process |
| Hook not running | `devenv gc && direnv allow` |

---

## Tips

- Always read devenv.nix before editing
- Use MCP search tools to find correct names
- Prefer `languages.*` over adding interpreter packages
- Use `services.*` for databases (auto-configured)
- Use `processes` for multi-service development
- Test changes with `devenv info` after editing
- Scripts become shell commands automatically
