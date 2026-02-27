---
name: codebase-mapper
description: "Generate a deterministic architecture map of Python, C#, and TypeScript codebases using AST parsing. Outputs a token-efficient structure section for CLAUDE.md. Use when mapping a codebase, generating architecture documentation, updating claude.md structure, or onboarding to a new project."
allowed-tools: Read Write Edit Bash Glob Grep
---

# Codebase Structure Mapper

Deterministically parse Python, C#, and TypeScript codebases to extract structural maps (namespaces, classes, methods, inheritance, dependencies) and output a concise, token-efficient architecture section for CLAUDE.md.

---

## Why This Skill?

- **Zero AI tokens for parsing** — Pure script execution using AST parsers
- **Deterministic & repeatable** — Same code = same output, version-controllable
- **Token-efficient output** — Compact format optimized for LLM consumption
- **Marker-based updates** — Preserves hand-written CLAUDE.md sections

---

## Workflow

| Phase | Name | Description |
|-------|------|-------------|
| 1 | DETECT | Auto-detect project languages and entry points |
| 2 | PARSE | Run appropriate AST-based mapper scripts |
| 3 | MERGE | Insert/update map in CLAUDE.md with markers |
| 4 | REPORT | Display summary and token count |

---

## Phase 1: DETECT

### Identify Project Languages

Scan for language indicators:

| Language | Detection Files |
|----------|-----------------|
| Python | `*.py`, `pyproject.toml`, `requirements.txt`, `setup.py` |
| C# | `*.cs`, `*.csproj`, `*.sln` |
| TypeScript | `*.ts`, `*.tsx`, `tsconfig.json` |

```bash
# Count files by type
find . -name "*.py" -type f | wc -l
find . -name "*.cs" -type f | wc -l
find . -name "*.ts" -o -name "*.tsx" -type f | wc -l
```

### Identify Source Roots

Look for common source directory patterns:
- Python: `src/`, `app/`, `<project_name>/`
- C#: `src/`, solution folder structure from `.sln`
- TypeScript: `src/`, `lib/`, `app/`

---

## Phase 2: PARSE

Run the appropriate mapper scripts based on detected languages.

### Python Projects

```bash
python scripts/map_python.py --root ./src --output /tmp/map_python.md
```

**What it extracts:**
- Modules and packages
- Classes with base classes
- Public methods and their signatures
- Key decorators (`@app.route`, `@property`, `@staticmethod`)
- Module-level functions
- Import dependencies (FastAPI, Django, Flask, SQLAlchemy detection)

### C# Projects

```bash
python scripts/map_csharp.py --root ./src --output /tmp/map_csharp.md
```

**What it extracts:**
- Namespaces
- Classes, interfaces, enums, records
- Inheritance and interface implementations
- Public methods and properties
- Key attributes (`[ApiController]`, `[HttpGet]`, `[Authorize]`)
- Using statements for dependency detection

### TypeScript Projects

```bash
python scripts/map_typescript.py --root ./src --output /tmp/map_typescript.md
```

**What it extracts:**
- Modules and exports
- Classes with inheritance
- Interfaces and types
- Functions (exported and module-level)
- React components (function and class-based)
- Key decorators (Angular `@Component`, `@Injectable`, etc.)

---

## Phase 3: MERGE

Use the update script to merge maps into CLAUDE.md:

```bash
python scripts/update_claude_md.py \
  --claude-md ./CLAUDE.md \
  --maps /tmp/map_python.md /tmp/map_csharp.md /tmp/map_typescript.md
```

### Marker System

The script uses markers to preserve hand-written sections:

```markdown
<!-- CODEBASE-MAP:AUTO-START -->
## Architecture Map

[Auto-generated content here]
<!-- CODEBASE-MAP:AUTO-END -->
```

- If markers exist: Replace content between them
- If no markers: Append after `## Project Context` section
- Hand-written sections outside markers are preserved

---

## Phase 4: REPORT

Display summary:

```
Codebase Map Generated
======================
Languages: Python, C#
Files scanned: 142
Classes mapped: 47
Methods mapped: 312

Output added to CLAUDE.md
Token estimate: ~450 tokens

Markers: <!-- CODEBASE-MAP:AUTO-START/END -->
Re-run this skill anytime to update.
```

---

## Output Format

Compact, scannable format optimized for LLM consumption:

```markdown
<!-- CODEBASE-MAP:AUTO-START -->
## Architecture Map

### Python: src/
- `api/` — FastAPI REST endpoints
  - `routes/users.py`: `UserRouter` → get_user(), create_user(), update_user()
  - `routes/orders.py`: `OrderRouter` → list_orders(), create_order()
- `models/` — SQLAlchemy ORM models
  - `user.py`: `User(Base)` — id, email, name, created_at
  - `order.py`: `Order(Base)` — id, user_id, total, status

### C#: src/MyApp/
- `MyApp.Api` — ASP.NET Core Web API
  - `Controllers/UserController.cs`: `UserController : ControllerBase` → Get(), Post(), Put()
- `MyApp.Core` — Domain models and interfaces
  - `Models/User.cs`: `User` — Id, Email, Name, CreatedAt
  - `Interfaces/IUserRepository.cs`: `IUserRepository` → GetByIdAsync(), CreateAsync()

### TypeScript: src/
- `components/` — React components
  - `UserCard.tsx`: `UserCard` (FC) — props: user, onSelect
  - `OrderList.tsx`: `OrderList` (FC) — props: orders, loading
- `services/` — API clients
  - `userService.ts`: fetchUser(), createUser(), updateUser()
<!-- CODEBASE-MAP:AUTO-END -->
```

---

## Configuration Options

### Depth Control

Control output verbosity via `--depth` flag:

| Depth | Output |
|-------|--------|
| `classes` | Namespaces and classes only |
| `methods` | Classes + public methods (default) |
| `full` | Classes + methods + properties + private members |

```bash
python scripts/map_python.py --root ./src --depth classes
```

### Exclusions

Exclude directories via `--exclude`:

```bash
python scripts/map_python.py --root ./src --exclude tests,migrations,__pycache__
```

Default exclusions: `node_modules`, `bin`, `obj`, `.git`, `__pycache__`, `venv`, `.venv`

---

## Integration with /ai-plugins-and-skills-init

This skill can be invoked automatically during `/ai-plugins-and-skills-init`:

1. After CLAUDE.md is generated, user is asked: "Generate codebase architecture map?"
2. If yes, this skill runs and inserts the map section
3. The map updates whenever this skill is re-run

To manually invoke after init:
```
Using the codebase-mapper skill, generate an architecture map for this project
```

---

## Script Locations

Scripts are located in `scripts/` relative to this skill:

| Script | Purpose |
|--------|---------|
| `map_python.py` | Python AST-based structure extractor |
| `map_csharp.py` | C# structure extractor (regex-based) |
| `map_typescript.py` | TypeScript structure extractor |
| `update_claude_md.py` | Merges map output into CLAUDE.md |

All scripts use Python stdlib only (no external dependencies).

---

## Troubleshooting

### "No source files found"
- Verify source root directory is correct
- Check that file extensions match expected patterns

### "CLAUDE.md not found"
- Run `/ai-plugins-and-skills-init` first to create CLAUDE.md
- Or specify path: `--claude-md ./path/to/CLAUDE.md`

### "Parse errors in file X"
- Script continues on parse errors, logs warning
- Check for syntax errors in source file
- Non-UTF8 files are skipped with warning
