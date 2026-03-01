---
name: dotfiles-guide
description: Use when adding new configurations, packages, or modules to this dotfiles repository. Covers file placement, package lists, and module creation.
---

# Dotfiles Guide

## Overview

Reference guide for where to place files and how to extend this dotfiles repository.

## Quick Reference

| I want to... | Location |
|--------------|----------|
| Add apt package | `packages/linux/apt.txt` |
| Add snap package | `packages/linux/snap.txt` |
| Add flatpak package | `packages/linux/flatpak.txt` |
| Add brew package (macOS) | `packages/macos/brew.txt` |
| Add winget package (Windows) | `packages/windows/winget.txt` |
| Add shell alias | `config/shell/common/aliases.sh` |
| Add shell function | `config/shell/common/functions.sh` |
| Add bash-only config | `config/shell/bash/.bashrc` |
| Add zsh-only config | `config/shell/zsh/.zshrc` |
| Add git config | `config/git/.gitconfig` |
| Add global gitignore | `config/git/.gitignore_global` |
| Add new app config | `config/<app-name>/` |
| Add Linux script | `scripts/linux/` |
| Add macOS script | `scripts/macos/` |
| Add cross-platform script | `scripts/common/` |
| Install Python (uv) | `make uv` |
| Install Node.js (nvm) | `make nvm` |
| Install AI CLI tools | `make ai-agents` |

## Directory Structure

```
dotfiles/
├── install.sh        # Bootstrap for Linux / macOS / WSL2
├── install.ps1       # Bootstrap for Windows (PowerShell 7)
├── config/           # Configuration files (copied to ~)
│   ├── shell/
│   │   ├── common/   # Shared aliases & functions
│   │   ├── bash/     # Bash-specific (.bashrc)
│   │   ├── zsh/      # Zsh-specific (.zshrc)
│   │   └── powershell/  # PowerShell profile + oh-my-posh theme
│   ├── git/          # Git config files
│   └── <app>/        # Other app configs
│
├── scripts/          # Installation scripts
│   ├── common/       # Cross-platform (configs, uv, nvm, ai-agents)
│   ├── linux/        # Linux-only scripts (also run on WSL2)
│   ├── macos/        # macOS scripts
│   └── windows/      # Windows scripts (setup-powershell.ps1)
│
├── packages/         # Package lists (one per line)
│   ├── linux/
│   │   ├── apt.txt
│   │   ├── snap.txt
│   │   └── flatpak.txt
│   ├── macos/
│   │   └── brew.txt
│   └── windows/
│       └── winget.txt
│
└── lib/              # Shared shell functions
    └── utils.sh
```

## Adding Packages

### APT Packages (`packages/linux/apt.txt`)

```bash
# Comments start with #
git
curl
vim

# Use [full] tag for packages only on dev machines
docker.io [full]
nodejs [full]
```

Packages tagged `[full]` are skipped when running `make server` (minimal mode).

### Snap Packages (`packages/linux/snap.txt`)

```bash
# Format: package-name [flags]
code --classic
slack
```

### Flatpak Packages (`packages/linux/flatpak.txt`)

```bash
# Use full application ID
com.spotify.Client
org.gimp.GIMP
```

### Homebrew Packages (`packages/macos/brew.txt`)

```bash
# Comments start with #
git
curl
wget

# Use [full] tag for packages only on dev machines
node [full]
```

## Installing Development Tools

### Python (uv)

```bash
make uv
```

Installs [uv](https://github.com/astral-sh/uv) - fast Python package manager. After installation, use `uv` to manage Python versions and packages.

### Node.js (nvm)

```bash
make nvm
```

Installs [nvm](https://github.com/nvm-sh/nvm) and the latest LTS Node.js. After installation, use `nvm` to manage Node.js versions.

### AI CLI Tools

```bash
make ai-agents
```

Installs AI coding assistants (requires npm, run `make nvm` first):
- GitHub Copilot CLI (`@github/copilot`)
- OpenAI Codex (`@openai/codex`)
- Google Gemini CLI (`@google/gemini-cli`)
- Claude Code (via curl installer)

## Adding Shell Configuration

### Shared (all shells)

Add to `config/shell/common/aliases.sh` or `functions.sh`:

```bash
# aliases.sh
alias ll='ls -la'

# functions.sh
mkcd() {
    mkdir -p "$1" && cd "$1"
}
```

### Shell-Specific

- **Bash only**: Edit `config/shell/bash/.bashrc`
- **Zsh only**: Edit `config/shell/zsh/.zshrc`

## Adding New Application Config

1. Create directory: `config/<app-name>/`
2. Add config files (use dot prefix if needed)
3. Update `scripts/common/configs.sh` to copy config files
4. Optionally add Makefile target

Example for tmux:

```bash
mkdir -p config/tmux
# Add config/tmux/.tmux.conf
```

## Adding New Module (Makefile Target)

1. Add target to `Makefile`:

```makefile
.PHONY: tmux
tmux:
    @bash scripts/common/configs.sh --module tmux
```

1. Update `scripts/common/configs.sh` to handle the module

## Machine-Specific Config

For settings that shouldn't be in git:

- `~/.bashrc.local` - Bash machine-specific
- `~/.zshrc.local` - Zsh machine-specific

These are sourced automatically but not tracked in git.

## Common Workflows

### "I want to install package X on all my Linux machines"

1. Add to `packages/linux/apt.txt` (or snap.txt/flatpak.txt)
2. Commit and push
3. On other machines: `git pull && make packages`

### "I want a new alias available everywhere"

1. Add to `config/shell/common/aliases.sh`
2. Commit and push
3. On other machines: `git pull && source ~/.bashrc` (or `~/.zshrc`)

### "I want to add vim/neovim config"

1. Create `config/vim/`
1. Add `.vimrc` or `init.vim`
1. Update configs script
1. Add `make vim` target to Makefile
1. Update `README.md` (可用指令、目錄結構)

## Important Reminder

**新增功能或模組後，務必更新 `README.md`：**

- 新增 make target → 更新「可用指令」section
- 新增目錄或檔案 → 更新「目錄結構」section
- 新增套件管理器 → 更新「自訂設定」section
- 新增 OS 支援 → 更新「支援的作業系統」section
