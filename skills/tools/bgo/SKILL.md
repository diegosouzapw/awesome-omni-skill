---
name: bgo
description: Automated Blender build-go workflow. Automatically builds, removes old version, installs, enables, and launches Blender with your extension/add-on. Use when you want to quickly test changes, execute complete build-to-launch cycle, or run custom packaging scripts with automatic Blender launch.
---

# Blender Build-Go (bgo)

Automate the complete workflow from building to launching Blender with your extension/add-on installed and enabled.

## When to use this skill

Use `/bgo` when:
- You want to quickly test extension/add-on changes in a fresh Blender instance
- You need to execute the full build → remove → install → enable → launch cycle
- You want to run a custom packaging script and automatically launch Blender
- You're iterating rapidly and want one command to see results

## Quick Start

**Auto-detect workflow (recommended):**

```bash
/bgo
```

The skill will:
1. Detect whether you're working with an extension (Blender 4.2+) or add-on (legacy)
2. Build/package your code
3. Remove old version from Blender
4. Install fresh build
5. Enable the extension/add-on
6. Launch Blender

**With custom script:**

```bash
/bgo --script build_custom.py
```

Runs your custom packaging script, then proceeds with install/enable/launch.

## Core Workflow

### Step 1: Build/Package

**For Extensions (Blender 4.2+):**
- Validates `blender_manifest.toml`
- Builds extension as `.zip` using `blender --command extension build`
- Stores output in `./build/` directory

**For Legacy Add-ons:**
- Packages add-on files (`.py` or directory)
- Prepares for installation via Blender preferences

**For Custom Scripts:**
- Executes your specified script
- Expects script to produce installable package

### Step 2: Remove Old Version

**For Extensions:**
```bash
blender --command extension remove <extension_id>
```

**For Add-ons:**
```python
bpy.ops.preferences.addon_disable(module="<module_name>")
bpy.ops.preferences.addon_remove(module="<module_name>")
```

Ensures clean slate before installation.

### Step 3: Install

**For Extensions:**
```bash
blender --command extension install-file ./build/<package>.zip
```

**For Add-ons:**
- Copies to Blender scripts directory
- Installs via `bpy.ops.preferences.addon_install()`

### Step 4: Enable

**For Extensions:**
Extensions are typically auto-enabled on install. Verifies enabled state.

**For Add-ons:**
```python
bpy.ops.preferences.addon_enable(module="<module_name>")
```

### Step 5: Launch Blender

Opens Blender with the extension/add-on active:

```bash
blender
```

Or with specific options:
```bash
blender --log "*" --log-level 0  # With debug logging
blender --python-expr "import bpy; print(bpy.context.preferences.addons.keys())"  # Verify load
```

## Usage Patterns

### Pattern 1: Quick iteration (default)

```bash
/bgo
```

Auto-detects project type and executes full workflow. Best for rapid development cycles.

### Pattern 2: Custom build script

```bash
/bgo --script scripts/build_release.py
```

Useful when you have:
- Custom packaging logic
- Multi-target builds
- Pre/post-build hooks
- Version management automation

### Pattern 3: Specific extension/add-on

```bash
/bgo --source-dir ./my_extension --extension-id my_extension_id
```

Explicitly specify:
- `--source-dir`: Path to extension/add-on source
- `--extension-id` or `--module`: Extension ID or add-on module name
- `--output-dir`: Custom build output directory (default: `./build`)

### Pattern 4: Build with options

```bash
/bgo --validate-only  # Just validate, don't install/launch
/bgo --no-launch      # Build and install, but don't open Blender
/bgo --clean          # Remove build artifacts before building
```

## Workflow Checklist

When `/bgo` executes, it performs:

```
Blender Build-Go Progress:
- [ ] Step 1: Detect project type (extension vs add-on)
- [ ] Step 2: Validate manifest/structure
- [ ] Step 3: Build/package code
- [ ] Step 4: Remove old version from Blender
- [ ] Step 5: Install fresh build
- [ ] Step 6: Enable extension/add-on
- [ ] Step 7: Launch Blender
- [ ] Step 8: Verify successful load
```

**Step 1: Detection**

Checks for `blender_manifest.toml` (extension) or `bl_info` (add-on) to determine workflow.

**Step 2: Validation**

Runs appropriate validation:
- Extensions: Manifest validation via `validate_manifest.py`
- Add-ons: Structure check for required `bl_info` fields

**Step 3: Build**

Executes build process:
- Extensions: `blender --command extension build`
- Add-ons: Package files for installation
- Custom: Runs specified script

**Step 4: Remove old version**

Cleans previous installation to prevent conflicts.

**Step 5: Install**

Installs fresh build into Blender.

**Step 6: Enable**

Activates extension/add-on in Blender preferences.

**Step 7: Launch**

Opens Blender with your code ready to use.

**Step 8: Verify**

Checks console output for:
- Registration success
- Import errors
- Runtime warnings

## Configuration

Create `.bgorc` in project root for persistent settings:

```json
{
  "type": "extension",
  "extension_id": "my_extension",
  "source_dir": ".",
  "output_dir": "./build",
  "build_script": null,
  "blender_path": "blender",
  "launch_args": ["--log", "*", "--log-level", "1"],
  "auto_validate": true,
  "clean_build": false,
  "auto_screenshot": false
}
```

**Configuration fields:**

- `type`: "extension" or "addon"
- `extension_id` / `module`: Extension ID or add-on module name
- `source_dir`: Path to source code
- `output_dir`: Build output directory
- `build_script`: Custom build script path (optional)
- `blender_path`: Path to Blender executable (default: system PATH)
- `launch_args`: Additional arguments for Blender launch
- `auto_validate`: Run validation before build (default: true)
- `clean_build`: Remove build directory before building (default: false)
- `auto_screenshot`: Capture viewport screenshot after launch (default: false)

## Integration with Other Skills

**Works seamlessly with:**

- `blender-extension-dev`: Uses build/install scripts
- `blender-addon-testing`: Runs tests after build (with `--test` flag)
- `blender-use`: Can capture screenshots after launch for verification

**Example combined workflow:**

```bash
# Build, install, test, and launch
/bgo --test --auto-screenshot

# This executes:
# 1. Build extension
# 2. Install extension
# 3. Run test suite (from blender-addon-testing)
# 4. Launch Blender
# 5. Capture screenshot (from blender-use)
```

## Custom Build Scripts

If using `--script`, your script should:

1. Accept command-line arguments:
   ```python
   import argparse
   parser = argparse.ArgumentParser()
   parser.add_argument('--source-dir', required=True)
   parser.add_argument('--output-dir', required=True)
   args = parser.parse_args()
   ```

2. Perform build operations:
   - Package files
   - Copy dependencies
   - Generate manifests
   - Run pre-processors

3. Output package to `--output-dir`:
   - Extensions: `<extension_id>.zip`
   - Add-ons: `<module_name>.py` or `<module_name>/` directory

4. Exit with code 0 on success, 1 on failure

**Example custom script:**

```python
#!/usr/bin/env python3
import argparse
import shutil
from pathlib import Path

def build_extension(source_dir, output_dir):
    """Custom build logic"""
    source = Path(source_dir)
    output = Path(output_dir)
    output.mkdir(exist_ok=True)

    # Custom preprocessing
    print("Running custom build steps...")

    # Use Blender's build command
    import subprocess
    subprocess.run([
        "blender", "--command", "extension", "build",
        "--source-dir", str(source),
        "--output-dir", str(output)
    ], check=True)

    print(f"Build complete: {output}")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument('--source-dir', required=True)
    parser.add_argument('--output-dir', required=True)
    args = parser.parse_args()

    build_extension(args.source_dir, args.output_dir)
```

## Troubleshooting

**Build fails:**
- Check manifest validation output
- Verify source files exist
- Review custom script errors (if using `--script`)
- Check Blender version compatibility

**Remove fails:**
- Extension/add-on may not be installed (non-fatal, continues)
- Check extension ID or module name is correct
- Verify Blender can access preferences

**Install fails:**
- Build output not found in `--output-dir`
- Incorrect package format
- Blender version mismatch
- File permission issues

**Enable fails:**
- Extension/add-on has registration errors
- Missing dependencies
- Incompatible Blender version
- Check Blender console for Python errors

**Launch fails:**
- Blender not in system PATH (use `--blender-path`)
- Blender executable name different on your system
- Permission issues

**Extension/add-on doesn't appear:**
- Check Blender console for import errors
- Verify enabled in Preferences → Add-ons/Extensions
- Look for registration errors in console
- Try manual enable: Edit → Preferences → Add-ons

## Advanced Usage

### Pre-launch validation

```bash
/bgo --validate-only
```

Runs build and validation without installing or launching. Useful for CI/CD.

### Silent mode (CI/CD)

```bash
/bgo --headless --no-launch --exit-code
```

Returns exit code based on success/failure. No interactive prompts.

### With test execution

```bash
/bgo --test --test-dir ./tests --report-format json
```

Runs test suite after installation, before launching Blender.

### Custom Blender version

```bash
/bgo --blender-path /path/to/blender-4.2/blender
```

Use specific Blender version for testing.

## Best Practices

**Use `.bgorc` for projects**: Set up configuration once, then just run `/bgo`.

**Clean builds for releases**: Use `--clean` before release builds to ensure no stale artifacts.

**Test before launch**: Add `--test` to catch errors before manual testing.

**Auto-screenshot**: Enable `auto_screenshot` in `.bgorc` to capture viewport state after launch for documentation.

**Version control**: Add `build/` to `.gitignore`, commit `.bgorc` for team consistency.

**CI/CD integration**: Use `--headless --no-launch --exit-code` in automated pipelines.

## Examples

**Basic development iteration:**
```bash
# Edit code...
/bgo
# Blender opens with changes loaded
```

**Release build:**
```bash
/bgo --clean --validate-only
# Review validation results
/bgo --clean
# Blender opens with clean build
```

**Multi-version testing:**
```bash
/bgo --blender-path /path/to/blender-4.2/blender
/bgo --blender-path /path/to/blender-4.3/blender
```

**Full validation pipeline:**
```bash
/bgo --clean --test --test-dir ./tests --auto-screenshot --launch-args "--log * --log-level 0"
```

## Output and Logs

`/bgo` outputs progress to console:

```
[bgo] Detecting project type...
[bgo] ✓ Found extension manifest: my_extension
[bgo] Validating manifest...
[bgo] ✓ Manifest valid
[bgo] Building extension...
[bgo] ✓ Built: build/my_extension.zip
[bgo] Removing old version...
[bgo] ✓ Removed: my_extension
[bgo] Installing extension...
[bgo] ✓ Installed: my_extension
[bgo] Enabling extension...
[bgo] ✓ Enabled: my_extension
[bgo] Launching Blender...
[bgo] ✓ Blender started (PID: 12345)
[bgo] Verifying load...
[bgo] ✓ Extension loaded successfully
[bgo] Done! Blender is ready.
```

**Log files:**
- `build/bgo.log`: Full execution log
- `build/blender_console.log`: Blender console output (if `--capture-logs`)

## Comparison with Manual Workflow

**Manual workflow:**
```bash
# 1. Build
blender --command extension build --source-dir . --output-dir build/

# 2. Remove old
blender --command extension remove my_extension

# 3. Install
blender --command extension install-file build/my_extension.zip

# 4. Launch
blender

# 5. Enable manually in UI
# Edit → Preferences → Extensions → Enable my_extension
```

**With `/bgo`:**
```bash
/bgo
```

**Time saved:** ~2-3 minutes per iteration × many iterations = significant productivity boost.
