---
name: claude-skill-bash
description: Apply comprehensive bash scripting standards including main function pattern, usage documentation, argument parsing, dependency checking, and error handling. Triggers when creating/editing .sh files, bash scripts, or discussing shell scripting, deployment scripts, automation tasks, or bash conventions.
---

# Bash Best Practices Skill

This skill ensures all bash scripts follow enterprise-grade best practices for maintainability, reliability, and user-friendliness.

## When This Skill Applies

This skill should be automatically invoked when:
- Creating new bash scripts or shell scripts
- Editing existing .sh or .bash files
- User requests scripts for automation, deployment, backup, or system tasks
- Reviewing or refactoring bash code
- User mentions: bash, shell script, main function, argument parsing, script structure

## Script Complexity Guidelines

This skill recognizes two categories of scripts with different structural requirements:

### Simple Scripts
Scripts that do one simple thing well without unnecessary ceremony.

**Characteristics:**
- Less than ~30 lines of actual logic (excluding comments and whitespace)
- No command-line arguments required
- Single, focused purpose that's self-evident
- Linear flow or minimal branching
- Output is often consumed by other programs or scripts

**Examples:** Version detection, environment checks, data format conversion, simple calculations

**Structural Requirements:**
- Main function and guard clause: **Optional**
- Usage function: **Not needed** (no arguments to document)
- Argument parsing: **Not applicable**
- Color output: **Optional** (often unnecessary)
- Can use direct execution without function wrapping

### Ordinary Scripts
Scripts with broader scope requiring full structure for maintainability.

**Characteristics:**
- More than ~30 lines of logic OR
- Takes command-line arguments OR
- Multiple functions or complex branching OR
- Interactive use by humans OR
- Requires detailed documentation

**Examples:** Deployment tools, backup utilities, system maintenance scripts, development tools

**Structural Requirements:**
- Main function and guard clause: **Required**
- Usage function: **Required**
- Argument parsing: **Required** (if taking arguments)
- Full error handling: **Required**
- User-friendly output: **Required** (for interactive scripts)

## Core Principles to Apply

### For Ordinary Scripts

When writing ordinary scripts, ensure:

1. **Main function pattern with guard clause** - Every script must have a main() function
2. **Usage function** - Clear help documentation
3. **Argument parsing** - Structured option handling in main
4. **Dependency checking** - Explicit validation of required tools
5. **Explicit error handling** - Never use `set -e`, handle errors explicitly
6. **Function organization** - Correct ordering and single responsibility
7. **User-friendly output** - Colored, structured output with status indicators

### For Simple Scripts

When writing simple scripts, ensure:

1. **Clear purpose comment** - Explain what the script does at the top
2. **Explicit error handling** - Never use `set -e`, handle errors explicitly
3. **Proper exit codes** - 0 for success, non-zero for failure
4. **Error output to stderr** - Use `>&2` for error messages
5. **Variable safety** - Quote variables, handle undefined cases

## Script Structure Templates

### Template for Ordinary Scripts

Every new bash script should follow this structure:

```bash
#!/usr/bin/env bash

# Global declarations
DEPENDENCIES=(jq curl git)  # List external tools needed
SCRIPT_NAME=$(basename "$0")
VERSION="1.0.0"

# Color definitions (if outputting to terminal)
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Allow users to disable colors
if [[ -n "${NO_COLOR:-}" ]] || [[ "${TERM:-}" == "dumb" ]]; then
    RED=""
    GREEN=""
    YELLOW=""
    BLUE=""
    NC=""
fi

function usage() {
    cat <<EOM

Brief description of what this script does.

usage: ${SCRIPT_NAME} [options]

options:
    -i|--input        <file>     Input file to process (required)
    -o|--output       <file>     Output file (optional, defaults to stdout)
    -v|--verbose                 Enable verbose output
    -h|--help                    Show this help message
    --version                    Show version information

dependencies: ${DEPENDENCIES[@]}

examples:
    ${SCRIPT_NAME} -i data.txt -o report.json
    ${SCRIPT_NAME} --input data.txt --verbose

EOM
    exit 1
}

function main() {
    # Default values
    local input_file=""
    local output_file=""
    local verbose=false

    # Parse arguments
    while [ "$1" != "" ]; do
        case $1 in
        -i | --input)
            shift
            input_file="$1"
            ;;
        -o | --output)
            shift
            output_file="$1"
            ;;
        -v | --verbose)
            verbose=true
            ;;
        --version)
            echo "${SCRIPT_NAME} version ${VERSION}"
            exit 0
            ;;
        -h | --help)
            usage
            ;;
        *)
            echo "Error: Unknown option '$1'"
            usage
            ;;
        esac
        shift
    done

    # Validate required arguments
    if [ -z "$input_file" ]; then
        echo "Error: Input file is required" >&2
        usage
    fi

    # Check dependencies
    exit_on_missing_tools "${DEPENDENCIES[@]}"

    # Main script logic
    process_file "$input_file" "$output_file" "$verbose"
}

# Business logic functions
function process_file() {
    local input="$1"
    local output="$2"
    local verbose="$3"

    # Implementation here
    echo "Processing ${input}..."
}

# Utility functions
function exit_on_missing_tools() {
    for cmd in "$@"; do
        if command -v "$cmd" &>/dev/null; then
            continue
        fi
        printf "Error: Required tool '%s' is not installed or not in PATH\n" "$cmd"
        exit 1
    done
}

# Guard clause - only execute main if script is run directly
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
    exit 0
fi
```

### Template for Simple Scripts

For scripts that do one simple thing without arguments:

```bash
#!/usr/bin/env bash
# Purpose: Determine version for CI builds based on git state
# Usage: determine-version.sh
# Output: Version string to stdout

# For simple scripts, direct execution is fine - no main() required

# Check if we're on a tag
TAG=$(git describe --exact-match --tags HEAD 2>/dev/null)

if [ -n "$TAG" ]; then
    # On a tag - use the tag as version
    echo "${TAG#v}"
else
    # Not on tag - determine from branch
    if [ -n "$GITHUB_HEAD_REF" ]; then
        # Pull request
        BRANCH="$GITHUB_HEAD_REF"
    elif [ -n "$GITHUB_REF_NAME" ]; then
        # Push to branch
        BRANCH="$GITHUB_REF_NAME"
    else
        # Fallback to current branch
        BRANCH=$(git symbolic-ref -q --short HEAD 2>/dev/null)
        if [ -z "$BRANCH" ]; then
            echo "Error: Unable to determine branch" >&2
            exit 1
        fi
    fi

    # Sanitize branch name for version string
    SANITIZED=$(echo "$BRANCH" | sed 's/[^a-zA-Z0-9._-]/_/g')
    echo "${SANITIZED}-SNAPSHOT"
fi
```

**Key points for simple scripts:**
- Clear purpose comment at the top
- Direct execution without main() wrapper
- Errors still go to stderr with >&2
- Meaningful exit codes (0 for success, non-zero for failure)
- Variables are still properly quoted
- No need for usage() function when there are no arguments
- No need for color output when output is data

## Detailed Best Practices

### 1. Script Bootstrapping

#### Main Function Pattern
- **ALWAYS** use a `main` function as the entry point
- This enables safe sourcing by other scripts
- Provides clear structure and testability

#### The Guard Clause
```bash
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
    exit 0
fi
```
- Must be at the very end of the script
- Enables the script to be sourced without execution
- Always include explicit `exit 0` after main

### 2. Usage Function and Argument Parsing

#### Usage Function Requirements
- Must be defined before main() function
- Use heredoc for clean formatting
- Include: description, options, dependencies, examples
- Exit with status 1

#### Argument Parsing Best Practices
```bash
function main() {
    # Set defaults first
    local input_file=""
    local verbose=false

    # Parse with while loop and case
    while [ "$1" != "" ]; do
        case $1 in
        -i | --input)
            shift
            input_file="$1"
            ;;
        -v | --verbose)
            verbose=true
            ;;
        -h | --help)
            usage
            ;;
        *)
            echo "Error: Unknown option '$1'"
            usage
            ;;
        esac
        shift
    done

    # Validate after parsing
    if [ -z "$input_file" ]; then
        echo "Error: Input file is required" >&2
        usage
    fi
}
```

### 3. Dependency Declaration and Checking

#### Always Declare Dependencies
```bash
DEPENDENCIES=(jq curl git docker)
```

#### Check Before Use
```bash
function exit_on_missing_tools() {
    for cmd in "$@"; do
        if command -v "$cmd" &>/dev/null; then
            continue
        fi
        printf "Error: Required tool '%s' is not installed or not in PATH\n" "$cmd"
        exit 1
    done
}
```

#### Include in Usage
Show dependencies in the usage function so users know requirements upfront.

### 4. Function Organization

#### Correct Function Order (MANDATORY)
1. `usage()` - First after global declarations
2. `main()` - Immediately after usage
3. Business logic functions - Core functionality
4. Utility functions - Generic helpers

**NEVER** add section comment headers like "# UTILITY FUNCTIONS". The code structure speaks for itself.

#### Function Design Principles
- **Single Responsibility**: Each function does one thing
- **Clear Inputs/Outputs**: Use local variables, return meaningful status
- **Descriptive Names**: Use verb-noun combinations (create_backup, validate_input)
- **Keep Small**: Break down functions longer than 50 lines
- **Use Local Variables**: Always declare with `local`

### 5. Error Handling

#### AVOID `set -e`
Never use `set -e`. Instead, use explicit error handling:

```bash
# Good - Explicit handling
if ! command; then
    echo "Error: Command failed" >&2
    exit 1
fi

# Good - Chain with ||
cd /some/directory || {
    echo "Error: Cannot change directory" >&2
    exit 1
}

# Good - Capture and check
output=$(command 2>&1)
if [ $? -ne 0 ]; then
    echo "Error: Command failed: $output" >&2
    exit 1
fi
```

#### Error Handling Rules
- Always redirect errors to stderr with `>&2`
- Provide context about what failed
- Use meaningful exit codes
- Clean up temporary files on failure

### 6. User-Friendly Output

#### Color Output (When Appropriate)
```bash
# Define colors at script top
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# Respect NO_COLOR environment
if [[ -n "${NO_COLOR:-}" ]] || [[ "${TERM:-}" == "dumb" ]]; then
    RED="" GREEN="" YELLOW="" BLUE="" NC=""
fi
```

#### Output Functions
```bash
function print_header() {
    echo -e "${BLUE}=======================${NC}"
    echo -e "${BLUE}$1${NC}"
    echo -e "${BLUE}=======================${NC}"
}

function print_success() {
    echo -e "${GREEN}✅ $1${NC}"
}

function print_error() {
    echo -e "${RED}❌ Error: $1${NC}" >&2
}

function print_warning() {
    echo -e "${YELLOW}⚠️  Warning: $1${NC}"
}

function print_step() {
    local current="$1"
    local total="$2"
    local message="$3"
    echo -e "${YELLOW}[${current}/${total}] ${message}${NC}"
}
```

### 7. Keep Main Function Lean

Main should only:
1. Parse arguments
2. Validate inputs
3. Check dependencies
4. Call business functions
5. Handle high-level errors

Business logic belongs in separate functions.

### 8. Code Comments

#### Avoid Excessive Comments
- Well-named functions are self-documenting
- Only comment the "why", not the "what"
- Use comments for:
  - Complex algorithms
  - Non-obvious workarounds
  - Business logic rules
  - External API quirks

```bash
# Bad - Obvious comment
# Increment counter
counter=$((counter + 1))

# Good - Explains why
# Exponential backoff with jitter to prevent thundering herd
delay=$((2 ** attempt * 1000 + RANDOM % 1000))
```

## Script Generation Checklist

When generating a new script, verify:
- [ ] Shebang line: `#!/usr/bin/env bash`
- [ ] DEPENDENCIES array declared
- [ ] usage() function defined first
- [ ] main() function handles argument parsing
- [ ] Dependency checking with exit_on_missing_tools
- [ ] Guard clause at script end
- [ ] Explicit error handling (no `set -e`)
- [ ] Functions properly ordered
- [ ] Local variables in functions
- [ ] Errors redirected to stderr
- [ ] Meaningful exit codes
- [ ] Color output with NO_COLOR support (if applicable)

## Common Patterns

### Interactive Prompts
```bash
# Yes/no confirmation
read -p "Continue? (y/n): " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "Operation cancelled"
    exit 0
fi

# Menu selection
echo "Select an option:"
echo "1) Full backup"
echo "2) Incremental backup"
read -p "Enter choice (1-2): " choice

# Password (hidden)
read -s -p "Enter password: " password
echo  # Add newline
```

### Temporary Files
```bash
# Create temp file safely
temp_file=$(mktemp)
trap "rm -f '$temp_file'" EXIT

# Use temp directory
temp_dir=$(mktemp -d)
trap "rm -rf '$temp_dir'" EXIT
```

### Pipeline Error Handling
```bash
# Without set -e, check PIPESTATUS
command1 | command2 | command3
if [ "${PIPESTATUS[0]}" -ne 0 ]; then
    echo "Error: command1 failed" >&2
    exit 1
fi
```

## Examples to Follow

When users request scripts for common tasks, apply these patterns:

### Backup Script
- Include timestamp in backup names
- Verify source exists before backup
- Check available disk space
- Provide progress feedback
- Clean up old backups based on retention

### Deployment Script
- Validate environment first
- Show clear step progression
- Roll back on failure
- Log all operations
- Provide summary at end

### Data Processing Script
- Validate input format
- Handle large files efficiently
- Provide progress indicators
- Support resume on failure
- Generate detailed reports

## Choosing Between Simple and Ordinary Scripts

### Decision Tree

Ask these questions to determine which pattern to use:

1. **Does the script take command-line arguments?**
   - YES → Use ordinary script pattern (needs usage() and parsing)
   - NO → Continue to question 2

2. **Is the logic more than ~30 lines (excluding comments)?**
   - YES → Use ordinary script pattern (needs structure)
   - NO → Continue to question 3

3. **Does it have multiple functions or complex branching?**
   - YES → Use ordinary script pattern (needs organization)
   - NO → Continue to question 4

4. **Will it be run interactively by humans who need help?**
   - YES → Use ordinary script pattern (needs user-friendly features)
   - NO → Continue to question 5

5. **Is it doing one simple, self-evident task?**
   - YES → Use simple script pattern
   - NO → Use ordinary script pattern

**When in doubt, use the ordinary pattern.** It's better to have structure you don't need than to need structure you don't have.

### Examples of Simple Scripts
- `determine-version.sh` - Outputs version string based on git state
- `check-env.sh` - Verifies environment variables are set
- `format-json.sh` - Reformats JSON from stdin to stdout
- `count-lines.sh` - Counts lines in specified file types
- `is-production.sh` - Returns 0 if on production branch

### Examples of Ordinary Scripts
- `deploy.sh` - Multi-step deployment with rollback
- `backup.sh` - Database backup with retention management
- `setup-dev.sh` - Developer environment initialization
- `test-runner.sh` - Configurable test execution
- `migrate-data.sh` - Data migration with validation

## Testing Scripts

Always test generated scripts for:
1. **Syntax**: `bash -n script.sh`
2. **Undefined variables**: Run with `set -u` temporarily
3. **Dependencies**: Run without required tools installed
4. **Error paths**: Test with invalid inputs
5. **Sourcing**: Verify guard clause works (for ordinary scripts)

## Remember

Every script you generate or modify should be production-ready:
- Maintainable by others
- Self-documenting through good structure
- Gracefully handles errors
- Provides helpful output
- Follows consistent patterns

These practices ensure scripts are professional, reliable, and user-friendly.