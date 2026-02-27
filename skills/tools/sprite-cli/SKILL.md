---
name: sprite-cli
description: Manage Sprite environments using the sprite CLI. Use for creating/destroying sprites (only when user explicitly requests), listing sprites, executing commands in remote sprites, managing checkpoints, port forwarding, file transfer between sprites, and API operations. Triggers on sprite management tasks or cross-sprite operations.
---

# Sprite CLI

External CLI for managing Sprite environments from outside a sprite.

## Installation

If `sprite` command is not found, install it:
```bash
curl https://sprites.dev/install.sh | bash
```

After installation, the user must authenticate manually:
> Run `sprite login` to authenticate with your Fly.io account.

Do not run `sprite login` automatically - prompt the user to run it themselves.

## Quick Reference

```bash
sprite list                              # List all sprites
sprite create <name>                     # Create sprite (user must request)
sprite destroy -s <name> -force          # Destroy sprite (see safety note)
sprite use <name>                        # Set sprite context for directory
sprite exec -s <name> -- <cmd>           # Run command in sprite
sprite console -s <name>                 # Interactive shell
sprite checkpoint create -s <name>       # Create checkpoint
sprite checkpoint list -s <name>         # List checkpoints
sprite restore -s <name> <id>            # Restore checkpoint
sprite proxy <port>                      # Forward ports
sprite url -s <name>                     # Show sprite URL
sprite api -s <name> <path>              # API call (curl wrapper)
```

## Sprites

### List
```bash
sprite list
sprite ls
```

### Create (requires explicit user request)
```bash
sprite create my-sprite
sprite create -o myorg my-sprite
```

### Destroy
```bash
sprite destroy -s my-sprite -force
```

**Safety rule:** Only destroy sprites you created in the current session. If asked to destroy a sprite you didn't create, tell the user the command instead:
> To destroy that sprite, run: `sprite destroy -s <name> -force`

### Use (set context)
```bash
sprite use my-sprite          # Creates .sprite file
sprite use --unset            # Remove context
```

## Executing Commands

### Single command
```bash
sprite exec -s my-sprite -- ls -la
sprite x -s my-sprite -- npm start
```

### With proper quoting for shell commands
```bash
sprite exec -s my-sprite -- bash -c 'echo "hello" > /tmp/file.txt'
```

### Interactive console
```bash
sprite console -s my-sprite
sprite c -s my-sprite
```

## Checkpoints

Checkpoints are fast (~300ms). Use frequently.

```bash
sprite checkpoint create -s my-sprite
sprite checkpoint list -s my-sprite
sprite checkpoint ls -s my-sprite
sprite restore -s my-sprite <checkpoint-id>
```

Create with comment via API:
```bash
sprite api -s my-sprite /checkpoint -X POST -d '{"comment":"before refactor"}'
```

## Services (Remote)

Manage long-running services on a sprite via `sprite exec` + `sprite-env`:

### List services
```bash
sprite exec -s my-sprite -- sprite-env services list
```

### Create service
```bash
# Basic service
sprite exec -s my-sprite -- sprite-env services create myapp --cmd python3 --args app.py

# With HTTP port (for web services)
sprite exec -s my-sprite -- sprite-env services create web --cmd python3 --args server.py --http-port 8080

# With dependencies
sprite exec -s my-sprite -- sprite-env services create api --cmd node --args server.js --needs postgres,redis
```

Options:
- `--cmd <command>` - Command to run (required)
- `--args <a,b,c>` - Comma-separated arguments
- `--http-port <port>` - HTTP port for proxy routing (only one service can have this)
- `--needs <svc1,svc2>` - Service dependencies
- `--env <K=V,...>` - Environment variables
- `--dir <path>` - Working directory
- `--duration <time>` - Log streaming duration (default: 5s)
- `--no-stream` - Don't stream logs after creation

### Start/Stop/Restart
```bash
sprite exec -s my-sprite -- sprite-env services start myapp
sprite exec -s my-sprite -- sprite-env services stop myapp
sprite exec -s my-sprite -- sprite-env services restart myapp
```

### Delete service
```bash
sprite exec -s my-sprite -- sprite-env services delete myapp
```

### Send signal
```bash
sprite exec -s my-sprite -- sprite-env services signal myapp TERM
sprite exec -s my-sprite -- sprite-env services signal myapp HUP
```

### View logs
Logs are at `/.sprite/logs/services/<name>.log`:
```bash
sprite exec -s my-sprite -- cat /.sprite/logs/services/myapp.log
sprite exec -s my-sprite -- tail -f /.sprite/logs/services/myapp.log
```

## Port Forwarding

Forward local ports through sprite proxy:
```bash
sprite proxy 8080
sprite proxy 8080 3000 5432
```

## URL Management

Each sprite has a unique URL. By default, URLs require authentication.

```bash
sprite url -s my-sprite                           # Show URL and auth status
sprite url update -s my-sprite --auth public      # Make public (user must request)
sprite url update -s my-sprite --auth sprite      # Require auth (default)
```

**Note:** Only make URLs public when the user explicitly requests it. Public URLs expose the HTTP service to the internet.

## File Transfer

Use `scripts/sprite-transfer.py` to copy files between sprites or local:

```bash
# Local to sprite
python3 scripts/sprite-transfer.py ./local-file.txt my-sprite:/path/to/file

# Sprite to local
python3 scripts/sprite-transfer.py my-sprite:/path/to/file ./local-file.txt

# Sprite to sprite
python3 scripts/sprite-transfer.py sprite-a:/file sprite-b:/file
```

### Direct API file operations
```bash
# Read file
sprite api -s my-sprite '/fs/read?path=/home/sprite/file.txt'

# Write file
echo "content" | sprite api -s my-sprite '/fs/write?path=/tmp/file.txt&mkdir=true' -X PUT --data-binary @-

# List directory
sprite api -s my-sprite '/fs/list?path=/home/sprite'
```

## API Access

`sprite api` wraps curl with authentication:

```bash
sprite api -s my-sprite /checkpoints              # GET
sprite api -s my-sprite /exec -X POST             # POST
sprite api /sprites                               # Management API
```

See `references/api.md` for full API reference.

## Debug Mode

```bash
sprite --debug exec -s my-sprite npm start
sprite --debug=/tmp/debug.log exec -s my-sprite npm start
```
