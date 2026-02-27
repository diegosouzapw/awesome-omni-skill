---
name: add-nixvim-plugin
description: Add Neovim plugins not available in NixVim's official plugin set using extraPlugins and extraConfigLua. Use this skill when the user requests adding a plugin by GitHub URL or mentions a plugin that doesn't exist as a native NixVim module.
---

# Add NixVim Plugin

## Overview

Add custom Neovim plugins to NixVim using `extraPlugins` when the plugin isn't available in NixVim's official plugin set. This skill guides through fetching plugin information, creating configuration files, and maintaining consistency with the existing plugin structure.

## When to Use This Skill

Use this skill when:
- User provides a GitHub URL for a Neovim plugin
- User requests a plugin that doesn't have a native NixVim module (e.g., `plugins.plugin-name`)
- User asks how to add plugins not in NixVim

Do NOT use this skill when:
- The plugin already has native NixVim support (use the standard `plugins.plugin-name` configuration)
- User is asking about configuring an already-added plugin

## Workflow

### Step 1: Gather Plugin Information

Fetch information about the plugin from its GitHub repository:

1. Use WebFetch to visit the plugin's GitHub URL
2. Extract:
   - Plugin purpose and key features
   - Default configuration structure
   - Any setup requirements
   - Latest stable commit hash or release tag

### Step 2: Determine Plugin Category

Identify the appropriate subdirectory in `config/plugins/`:
- `ui/` - UI enhancements, windows, buffers
- `editing/` - Text editing, manipulation
- `lsp/` - Language server related
- `navigation/` - Movement, jumping
- `git/` - Git integration
- `files/` - File management
- `completion/` - Completion engines
- `search/` - Search and replace
- `terminal/` - Terminal integration
- `workflow/` - Productivity, helpers
- `testing/` - Test frameworks
- `statusline/` - Status line plugins
- `treesitter/` - Treesitter related
- `snippets/` - Snippet engines
- `ai/` - AI assistants
- `langs/` - Language-specific

### Step 3: Create Plugin Configuration File

Create `config/plugins/<category>/<plugin-name>.nix` with this structure:

```nix
# ABOUTME: [Brief description of what the plugin does]
# ABOUTME: [Secondary description line if needed]
{pkgs, ...}: {
  extraPlugins = [
    (pkgs.vimUtils.buildVimPlugin {
      name = "plugin-name";
      src = pkgs.fetchFromGitHub {
        owner = "github-username";
        repo = "repo-name";
        rev = "commit-hash-or-tag";
        sha256 = "sha256-AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=";  # Use fake hash initially
      };
    })
  ];

  extraConfigLua = ''
    require("plugin-name").setup({
      -- Start with minimal default configuration
      -- Add essential settings only
    })
  '';

  keymaps = [
    {
      mode = "n";
      key = "<leader>xy";
      action = "<cmd>PluginCommand<CR>";
      options = {
        desc = "Brief description";
        silent = true;
      };
    }
  ];
}
```

**Important notes:**
- Start with fake SHA256 hash - Nix will provide correct hash on first build
- Follow existing keymap groups (`<leader>f` = file, `<leader>u` = UI, `<leader>w` = window, etc.)
- Keep initial configuration minimal - add incrementally as needed
- Include ABOUTME comments at the top of the file

### Step 4: Update default.nix

Add the new plugin file to `config/default.nix` imports in the appropriate section:

```nix
{
  imports = [
    # ... other imports
    ./plugins/<category>/<plugin-name>.nix
  ];
}
```

Maintain alphabetical or logical ordering within each section.

### Step 5: Update Documentation

Update three locations to document the new plugin:

#### 5a. Update recent_plugins in config/options.nix

Add entry at the TOP of the `recent_plugins` table (around line 96):

```lua
{
  name = "plugin-name",
  date = "YYYY-MM-DD",  -- Today's date
  description = "Brief description of functionality",
  keymaps = {
    "<leader>xy = Action Description",
    -- additional keymaps
  },
  usage = "How to use the plugin, key features"
},
```

#### 5b. Add whichkey entries in config/plugins/workflow/whichkey.nix

Add entries to the `spec` array for each keymap:

```nix
{
  "__unkeyed-1" = "<leader>xy";
  desc = "Action Description";
  icon = {
    icon = "󰈙";  # Choose appropriate Nerd Font icon
    color = "blue";  # Choose appropriate color
  };
}
```

Icon colors available: blue, yellow, orange, cyan, purple, green, red, azure

#### 5c. Update WISHLIST.md (if applicable)

If the plugin was on the wishlist, mark it as completed or remove the entry.

### Step 6: Build and Test

1. Stage files: `git add .` (required for Nix flakes)
2. Build: `nix build` or `just build`
3. If SHA256 hash error occurs:
   - Copy correct hash from error message
   - Update the config file with correct hash
   - Build again
4. Test: `nix run .` or `just run`

### Step 7: Lazy Loading (Optional)

If the plugin should be lazy-loaded to improve startup time:

1. Read `references/lazy-loading.md` for detailed patterns
2. Modify the plugin configuration to use lz.n
3. Test that the plugin loads correctly when triggered

Common lazy-loading scenarios:
- Load on command: `cmd = "CommandName"`
- Load on keypress: `keys = "<leader>key"`
- Load on event: `event = "BufEnter"`
- Load on filetype: `ft = "python"`

## Troubleshooting

**Build fails with "file not found"**
- Ensure `git add .` was run before building
- Nix flakes only see tracked files

**Wrong SHA256 hash error**
- Expected on first build
- Copy correct hash from error message
- Update config file and rebuild

**Plugin not loading**
- Check plugin is imported in `config/default.nix`
- Verify `extraConfigLua` has no syntax errors
- Ensure plugin name matches exactly (case-sensitive)

**Formatter changes files**
- Pre-commit hooks with alejandra automatically format Nix files
- This is expected behavior

## Resources

See `references/lazy-loading.md` for detailed lazy loading configuration patterns.
