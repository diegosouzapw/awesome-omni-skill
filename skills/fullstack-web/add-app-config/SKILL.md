---
name: add-app-config
description: Use when adding configuration for a new application to the dotfiles, setting up a new tool's config, or when user says "add config for X"
---

# Add App Configuration

Add a new application's configuration to Juliet dotfiles with proper stow symlinks.

## Process

1. **Create config directory**
   ```bash
   mkdir -p ~/.config/Juliet/symlinked/config/<app-name>/
   ```

2. **Add configuration files**
   - Create config files inside `symlinked/config/<app-name>/`
   - Use the app's expected filenames (e.g., `config`, `config.toml`, `settings.json`)

3. **Run stow to create symlinks**
   ```bash
   stow --target=$HOME/.config --dir=$HOME/.config/Juliet/symlinked config
   ```

4. **Apply Bamboo theme** (if applicable)
   - Check if app supports Bamboo theme
   - Apply consistent colors: dark green-tinted background, soft white foreground
   - Document theme setting in `.claude/rules/theming.md`

## Pre-flight Check

Before creating, verify the target doesn't already exist:
```bash
ls -la ~/.config/<app-name>
```

If it exists and is NOT a symlink, back it up first:
```bash
mv ~/.config/<app-name> ~/.config/<app-name>.backup
```

## Example: Adding Fuzzel Config

```bash
# Create directory
mkdir -p ~/.config/Juliet/symlinked/config/fuzzel/

# Create config file
# (write fuzzel.ini content)

# Stow it
stow --target=$HOME/.config --dir=$HOME/.config/Juliet/symlinked config

# Verify
ls -la ~/.config/fuzzel  # Should show symlink to Juliet
```

## Common Mistakes

- Creating files directly in `~/.config/` instead of `symlinked/config/`
- Forgetting to run `stow` after adding files
- Not checking if target folder already exists (could overwrite user data)
