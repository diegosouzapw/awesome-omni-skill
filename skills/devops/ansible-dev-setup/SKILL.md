---
name: ansible-dev-setup
description: Generate and manage cross-platform Ansible playbooks for development environment setup across macOS, Linux, and Termux. Use when working with development environment automation, package installation configuration, or Ansible playbook generation.
---

# Ansible Development Setup Generator

This skill helps you work with the Ansible-based development environment setup system in this repository, which generates playbooks for installing development tools across multiple platforms.

## Overview

The repository contains a Go-based generator that creates Ansible playbooks for setting up development environments consistently across:
- **macOS** (using Homebrew)
- **Debian/Ubuntu** (using apt with PPA support)
- **Termux** (Android terminal with pkg)

The generator is located in `devtools/setup-dev/ansible/` and consists of:
- `generate_packages.go` - Main generator and entry point
- `types.go` - Core data structures and interfaces
- `install_methods.go` - Platform-specific installation method implementations
- `templates.go` - Ansible playbook templates
- `packages_data.go` - Package and tool definitions

## Key Concepts

### Package Types

1. **PackageData**: Traditional system packages installed via platform package managers (apt, brew, pkg)
   - Supports different package names per platform
   - Supports Ubuntu PPAs and Homebrew taps
   - Example: emacs, git, curl

2. **PlatformSpecificTool**: Development tools with platform-specific installation methods
   - Uses unified `InstallMethod` interface
   - Can use different installation approaches per platform
   - Example: starship (brew on macOS, cargo on Debian, pkg on Termux)

### Installation Methods

The system supports multiple installation methods via the `InstallMethod` interface:
- `PackageInstallMethod` - System package managers (apt, yum, dnf)
- `BrewInstallMethod` - Homebrew with tap and options support
- `TermuxPkgInstallMethod` - Termux pkg command
- `PipInstallMethod` - Python pip packages
- `GoInstallMethod` - Go install with version checking
- `CargoInstallMethod` - Rust cargo with cargo-update integration
- `NpmInstallMethod` - Node.js npm global packages
- `UvInstallMethod` - Python uv tool
- `ShellInstallMethod` - Custom shell commands with version checking

### Platform-Specific Installation Strategy

The generator follows best practices for each platform:
- **Termux**: Strongly prefer `pkg` due to Termux's unconventional setup requiring patches
- **Debian/Ubuntu**: Prefer alternative methods when packages are outdated; use PPAs for actively developed software
- **Python packages**: Prefer `uv` over `pip` (faster, better dependency resolution)
- **Go packages**: Use `go install` with automatic version checking and upgrade logic
- **macOS**: Use Homebrew with tap and option support

## Common Tasks

### Adding a New Package

To add a system package that uses platform package managers:

1. Add to the `packages` array in `packages_data.go`:
```go
{command: "your-tool", debianPkgName: "debian-name", termuxPkgName: "termux-name", brewPkgName: "brew-name"}
```

2. For packages with different names per platform, specify each:
```go
{command: "ag", debianPkgName: "silversearcher-ag", termuxPkgName: "silversearcher-ag", brewPkgName: "the_silver_searcher"}
```

3. For Ubuntu PPAs:
```go
{command: "emacs", UbuntuPPA: "ppa:ubuntuhandhand1/emacs"}
```

4. For Homebrew taps with options:
```go
{command: "emacs", brewPkgName: "emacs-plus", brewTap: "d12frosted/emacs-plus", brewOptions: []string{"with-native-comp", "with-dbus"}}
```

### Adding a Platform-Specific Tool

To add a tool with different installation methods per platform:

1. For Go tools (simplest case):
```go
GoTool("tool-name", "github.com/user/repo/cmd/tool@latest")
```

2. For tools with different methods per platform:
```go
{
    command: "tool-name",
    platforms: map[string]InstallMethod{
        "darwin":      BrewInstallMethod{Name: "tool-name"},
        "termux":      TermuxPkgInstallMethod{Name: "tool-name"},
        "debian-like": UvInstallMethod{Name: "tool-name"},
    },
    Imports: nil,
}
```

3. For cargo packages with auto-update:
```go
{
    command: "tool-name",
    platforms: map[string]InstallMethod{
        "all": CargoInstallMethod{Name: "crate-name"},
    },
    Imports: nil,
}
```

### Regenerating Playbooks

After modifying the package definitions, regenerate the playbooks from the **repository root**:

```bash
make generate-ansible
```

This will:
1. Generate all `.yml` playbook files
2. Update the `BUILD.bazel` file with test targets for each playbook
3. Update `README.org`'s manual playbooks list

### Testing Playbooks

After regenerating playbooks, validate them with Bazel syntax tests:

```bash
# Test all playbooks
bazel test //devtools/setup-dev/ansible:ansible_syntax_tests

# Test a specific playbook
bazel test //devtools/setup-dev/ansible:emacs_syntax_test
```

**Recommended workflow** when adding a new package:
1. Edit `packages_data.go` to add the package definition
2. Run `make generate-ansible` from the repo root to regenerate playbooks and BUILD.bazel
3. Run `bazel test //devtools/setup-dev/ansible:ansible_syntax_tests` to validate syntax
4. Optionally run `make` from the repo root to run all tests and linting

## Important Files

- `devtools/setup-dev/ansible/generate_packages.go` - Main generator
- `devtools/setup-dev/ansible/packages_data.go` - Package definitions (edit this to add tools)
- `devtools/setup-dev/ansible/types.go` - Type definitions
- `devtools/setup-dev/ansible/install_methods.go` - Installation method implementations
- `devtools/setup-dev/ansible/templates.go` - Ansible YAML templates
- `devtools/setup-dev/ansible/README.org` - Detailed documentation
- `devtools/setup-dev/ansible/ensure.sh` - Script to run playbooks
- `devtools/setup-dev/ansible/BUILD.bazel` - Generated test targets

## Design Patterns

### Include Guards

All generated playbooks include guards to prevent multiple inclusion:
```yaml
- name: Include guard for tool playbook
  block:
    - name: Stop early if the tool playbook is already included
      meta: end_play
      when: tool_playbook_imported is defined
    - name: Ensure the tool playbook is not included
      set_fact:
        tool_playbook_imported: true
```

### Dependency Management

Tools can declare dependencies via the `Imports` field:
```go
{command: "notmuch", Imports: []Import{{Playbook: "python3-notmuch2"}}}
```

Dependencies are automatically imported at the beginning of playbooks.

### Manual Playbooks

Some playbooks are not generated by the Go program and are maintained manually. These playbooks are typically used for:
- Setup tasks that don't fit the package installation model (e.g., `setup-ssh-key.yml`, `setup-homebrew-env.yml`).
- Complex configuration logic (e.g., `setup-shell-profile.yml`).
- Importing other playbooks (e.g., `all.yml`).

When creating a manual playbook:
1. Ensure it follows the naming convention `setup-<feature>.yml` or just `<feature>.yml`.
2. Manual playbooks are automatically detected by the generator and added to `README.org`.

### Verification

ALWAYS run `make` in the `devtools/setup-dev/ansible` directory after making changes, including adding manual playbooks.

```bash
cd devtools/setup-dev/ansible
make
```

The `make` command:
1.  **Generates Artifacts**: Regenerates `.yml` files and updates `README.org` (including the manual playbooks list).
2.  **Verifies Syntax**: Runs syntax checks on all playbooks (generated and manual).
3.  **Tests**: Runs Go tests and Bazel build targets.

### Version Checking

Go and Cargo installation methods include automatic version checking:
- Go: Uses `go version -m` and `go list -m` to check for updates
- Cargo: Uses `cargo-install-update` for update detection
- Shell: Supports custom version checking with regex and GitHub API

## When to Use This Skill

Use this skill when you need to:
- Add a new development tool to the setup system
- Modify package installation configuration
- Understand how the Ansible playbook generator works
- Add support for a new installation method
- Debug playbook generation issues
- Update package definitions or platform-specific installation strategies
- Work with the BUILD.bazel test generation
- Understand the platform-specific installation strategy

## References

For more detailed information, consult:
- `devtools/setup-dev/ansible/README.org` - Complete documentation with design rationale
- Individual `.go` files for implementation details
- Generated `.yml` files for examples of output playbooks
