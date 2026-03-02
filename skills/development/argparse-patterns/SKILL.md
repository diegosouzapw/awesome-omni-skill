---
name: argparse-patterns
description: Standard library Python argparse examples with subparsers, choices, actions, and nested command patterns. Use when building Python CLIs without external dependencies, implementing argument parsing, creating subcommands, or when user mentions argparse, standard library CLI, subparsers, argument validation, or nested commands.
allowed-tools: Read, Write, Edit, Bash
---

# argparse-patterns

Python's built-in argparse module for CLI argument parsing - no external dependencies required.

## Overview

Provides comprehensive argparse patterns using only Python standard library. Includes subparsers for nested commands, choices for validation, custom actions, argument groups, and mutually exclusive options.

## Instructions

### Basic Parser Setup

1. Import argparse and create parser with description
2. Add version info with `action='version'`
3. Set formatter_class for better help formatting
4. Parse arguments with `parser.parse_args()`

### Subparsers (Nested Commands)

1. Use `parser.add_subparsers(dest='command')` to create command groups
2. Add individual commands with `subparsers.add_parser('command-name')`
3. Each subparser can have its own arguments and options
4. Nest subparsers for multi-level commands (e.g., `mycli config get key`)

### Choices and Validation

1. Use `choices=['opt1', 'opt2']` to restrict values
2. Implement custom validation with type functions
3. Add validators using argparse types
4. Set defaults with `default=value`

### Actions

1. `store_true/store_false` - Boolean flags
2. `store_const` - Store constant value
3. `append` - Collect multiple values
4. `count` - Count flag occurrences
5. `version` - Display version and exit
6. Custom actions with Action subclass

### Argument Types

1. Positional arguments - Required by default
2. Optional arguments - Prefix with `--` or `-`
3. Type coercion - `type=int`, `type=float`, `type=pathlib.Path`
4. Nargs - `'?'` (optional), `'*'` (zero or more), `'+'` (one or more)

## Available Templates

### Python Templates

- **basic-parser.py** - Simple parser with arguments and options
- **subparser-pattern.py** - Single-level subcommands
- **nested-subparser.py** - Multi-level nested commands (e.g., git config get)
- **choices-validation.py** - Argument choices and validation
- **boolean-flags.py** - Boolean flag patterns
- **custom-actions.py** - Custom action classes
- **mutually-exclusive.py** - Mutually exclusive groups
- **argument-groups.py** - Organizing related arguments
- **type-coercion.py** - Custom type converters
- **variadic-args.py** - Variable argument patterns

### TypeScript Templates

- **argparse-to-commander.ts** - argparse patterns translated to commander.js
- **argparse-to-yargs.ts** - argparse patterns translated to yargs
- **parser-comparison.ts** - Side-by-side argparse vs Node.js patterns

## Available Scripts

- **generate-parser.sh** - Generate argparse parser from specifications
- **validate-parser.sh** - Validate parser structure and completeness
- **test-parser.sh** - Test parser with various argument combinations
- **convert-to-click.sh** - Convert argparse code to Click decorators

## Examples

See `examples/` directory for comprehensive patterns:

- **basic-usage.md** - Simple CLI with arguments
- **subcommands.md** - Multi-command CLI (like git, docker)
- **nested-commands.md** - Deep command hierarchies
- **validation-patterns.md** - Argument validation strategies
- **advanced-parsing.md** - Complex parsing scenarios

## Common Patterns

### Pattern 1: Simple CLI with Options

```python
parser = argparse.ArgumentParser(description='Deploy application')
parser.add_argument('--env', choices=['dev', 'staging', 'prod'], default='dev')
parser.add_argument('--force', action='store_true')
args = parser.parse_args()
```

### Pattern 2: Subcommands (git-like)

```python
parser = argparse.ArgumentParser()
subparsers = parser.add_subparsers(dest='command')

deploy_cmd = subparsers.add_parser('deploy')
deploy_cmd.add_argument('environment')

config_cmd = subparsers.add_parser('config')
```

### Pattern 3: Nested Subcommands (git config get/set)

```python
parser = argparse.ArgumentParser()
subparsers = parser.add_subparsers(dest='command')

config = subparsers.add_parser('config')
config_subs = config.add_subparsers(dest='config_command')

config_get = config_subs.add_parser('get')
config_get.add_argument('key')

config_set = config_subs.add_parser('set')
config_set.add_argument('key')
config_set.add_argument('value')
```

### Pattern 4: Mutually Exclusive Options

```python
group = parser.add_mutually_exclusive_group()
group.add_argument('--json', action='store_true')
group.add_argument('--yaml', action='store_true')
```

### Pattern 5: Custom Validation

```python
def validate_port(value):
    ivalue = int(value)
    if ivalue < 1 or ivalue > 65535:
        raise argparse.ArgumentTypeError(f"{value} is not a valid port")
    return ivalue

parser.add_argument('--port', type=validate_port, default=8080)
```

## Best Practices

1. **Always provide help text** - Use `help=` for every argument
2. **Set sensible defaults** - Use `default=` to avoid None values
3. **Use choices for fixed options** - Better than manual validation
4. **Group related arguments** - Use `add_argument_group()` for clarity
5. **Handle missing subcommands** - Check if `args.command` is None
6. **Use type coercion** - Prefer `type=int` over manual conversion
7. **Provide examples** - Use `epilog=` for usage examples

## Advantages Over External Libraries

- **No dependencies** - Built into Python standard library
- **Stable API** - Won't break with updates
- **Universal** - Works everywhere Python works
- **Well documented** - Extensive official documentation
- **Lightweight** - No installation or import overhead

## When to Use argparse

Use argparse when:
- Building simple to medium complexity CLIs
- Avoiding external dependencies is important
- Working in restricted environments
- Learning CLI patterns (clear, explicit API)

Consider alternatives when:
- Need decorator-based syntax (use Click/Typer)
- Want type safety and auto-completion (use Typer)
- Rapid prototyping from existing code (use Fire)

## Integration

This skill integrates with:
- `cli-setup` agent - Initialize Python CLI projects
- `cli-feature-impl` agent - Implement command logic
- `cli-verifier-python` agent - Validate argparse structure
- `click-patterns` skill - Compare with Click patterns
- `typer-patterns` skill - Compare with Typer patterns

## Requirements

- Python 3.7+ (argparse included in standard library)
- No external dependencies required
- Works on all platforms (Windows, macOS, Linux)

---

**Purpose**: Standard library Python CLI argument parsing patterns
**Used by**: Python CLI projects prioritizing zero dependencies
