---
name: packaging-rules
description: BrainDrive plugin packaging and ZIP rules - use when creating the final distributable package or validating ZIP structure
---


## What is the Final Plugin Package?

A BrainDrive plugin is distributed as a **ZIP file** that users can install via the Plugin Installer UI.

### ZIP Contents

```
my-plugin-1.0.0.zip
└── my-plugin/                          # Plugin folder name
    ├── package.json                    # npm metadata
    ├── lifecycle_manager.py           # Backend lifecycle hooks
    ├── webpack.config.js              # Build configuration
    ├── tsconfig.json                  # TypeScript configuration
    ├── src/                           # Source code
    │   ├── index.tsx
    │   ├── MyPlugin.tsx
    │   ├── MyPlugin.css
    │   ├── types.ts
    │   ├── components/
    │   ├── services/
    │   └── utils/
    ├── public/                        # Static assets
    │   └── icon.png
    ├── dist/                          # Build output (MUST exist)
    │   ├── remoteEntry.js            # Module Federation entry (REQUIRED)
    │   ├── main.js
    │   └── ... other chunks
    ├── README.md                      # Installation/usage instructions
    ├── LICENSE                        # License file
    ├── .npmignore                     # Files to exclude from npm (if publishing)
    └── (optional) CHANGELOG.md
```

## ZIP File Naming

### Format
```
{plugin-slug}-{version}.zip
```

### Examples
```
✅ VALID
- my-plugin-1.0.0.zip
- task-manager-0.1.0.zip
- data-dashboard-2.3.1.zip

❌ INVALID
- my-plugin.zip (missing version)
- my-plugin-v1.0.0.zip (has "v" prefix)
- my-plugin-latest.zip (not semver)
- my_plugin-1.0.0.zip (uses underscore)
```

## Build Output Requirements

### MUST Include: dist/ Folder

Before creating the ZIP, run the build:

```bash
npm install
npm run build
```

This creates the `dist/` folder with:
- **dist/remoteEntry.js** (REQUIRED) - Module Federation entry point
- **dist/main.js** - Your plugin code bundle
- **dist/{other-chunks}.js** - Split code chunks if any

### MUST Exist Before ZIP

```bash
# Verify dist folder and remoteEntry.js exist
ls -la dist/
# Output:
# -rw-r--r--  main.js
# -rw-r--r--  remoteEntry.js
```

### Build Configuration (webpack.config.js)

```javascript
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  mode: 'production',
  entry: './src/index.tsx',
  output: {
    path: __dirname + '/dist',
    filename: 'main.js',
    publicPath: 'auto'  // Critical for Module Federation
  },
  
  plugins: [
    new ModuleFederationPlugin({
      name: 'MyPlugin',              // Must match plugin_slug
      filename: 'remoteEntry.js',   // MUST be exactly this
      exposes: {
        './index': './src/index.tsx'
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.2.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.2.0' }
      }
    })
  ]
};
```

## Step-by-Step Packaging Process

### Step 1: Verify Source Code
```bash
# Check that all source files exist
ls -la
# Files MUST exist:
# - package.json
# - lifecycle_manager.py
# - webpack.config.js
# - tsconfig.json
# - src/index.tsx
```

### Step 2: Install Dependencies
```bash
npm install
```

### Step 3: Build for Production
```bash
npm run build
```

Verify build output:
```bash
ls -la dist/
# Must show:
# remoteEntry.js
# main.js
```

### Step 4: Verify dist/ Folder Size

```bash
du -sh dist/
# Typical plugin: 100KB - 500KB
# If > 2MB: check for large dependencies or assets
```

### Step 5: Create ZIP File

Using command line:
```bash
# Option 1: Include dist/
zip -r my-plugin-1.0.0.zip my-plugin/

# Option 2: Manual folder structure
mkdir package_dir
cp -r my-plugin/* package_dir/
cd package_dir
zip -r ../my-plugin-1.0.0.zip .
cd ..
rm -rf package_dir
```

Using Node.js:
```javascript
const archiver = require('archiver');
const fs = require('fs');

const output = fs.createWriteStream('my-plugin-1.0.0.zip');
const archive = archiver('zip', { zlib: { level: 9 } });

output.on('close', () => {
  console.log('ZIP created:', archive.pointer() + ' total bytes');
});

archive.pipe(output);
archive.directory('my-plugin/', 'my-plugin');
archive.finalize();
```

### Step 6: Verify ZIP Contents

```bash
# List ZIP contents
unzip -l my-plugin-1.0.0.zip

# Should show structure like:
# my-plugin-1.0.0/my-plugin/package.json
# my-plugin-1.0.0/my-plugin/lifecycle_manager.py
# my-plugin-1.0.0/my-plugin/dist/remoteEntry.js
# ... etc
```

### Step 7: Store ZIP

- **For distribution:** Upload to plugin marketplace
- **For installation:** Keep ZIP file available locally
- **For backup:** Store with version tag

## What NOT to Include in ZIP

❌ **Exclude These:**
- `node_modules/` (too large, users will run npm install)
- `.git/` (version control files)
- `.gitignore`
- `.env` (sensitive data)
- `src/` (source code, not needed for runtime)
- `*.log` (log files)
- `.DS_Store` (macOS metadata)
- `Thumbs.db` (Windows metadata)
- `dist/` if it's pre-built (users may build differently)

❌ **Actually, reconsider `dist/`:**
- **Pro:** Users don't need to build, faster install
- **Con:** Large ZIP, platform-specific build artifacts
- **Recommendation:** Include `dist/` for end users, exclude for developers

### .npmignore or Similar

If you want different files for npm publishing vs plugin distribution, create `.npmignore`:

```
# .npmignore
dist/
node_modules/
.git
.env
*.log
screenshot/
docs/
```

## Installation Instructions (Include in README.md)

```markdown
## Installation

1. Download the plugin ZIP file
2. Open BrainDrive
3. Go to Settings → Plugins → Install Plugin
4. Select the ZIP file
5. BrainDrive will install the plugin and reload
6. Plugin appears in the sidebar / main menu

## Building from Source

If you have the source code:

```bash
npm install
npm run build
zip -r my-plugin-1.0.0.zip my-plugin/
```

Then follow installation steps above.
```

## Validation Before Packaging

Run this checklist:

```bash
# ✅ Verify essential files exist
[ -f package.json ] && echo "✓ package.json"
[ -f lifecycle_manager.py ] && echo "✓ lifecycle_manager.py"
[ -f webpack.config.js ] && echo "✓ webpack.config.js"
[ -f tsconfig.json ] && echo "✓ tsconfig.json"
[ -d src ] && echo "✓ src/ folder"
[ -d dist ] && echo "✓ dist/ folder"
[ -f dist/remoteEntry.js ] && echo "✓ dist/remoteEntry.js"

# ✅ Verify version consistency
echo "package.json version:"
grep '"version"' package.json

echo "lifecycle_manager.py version:"
grep '"version"' lifecycle_manager.py

# ✅ Verify plugin_slug matches scope
echo "plugin_slug:"
grep 'plugin_slug' lifecycle_manager.py

echo "webpack scope:"
grep "name: '" webpack.config.js

# ✅ Verify no sensitive data
! grep -r 'API_KEY\|SECRET\|PASSWORD' src/ && echo "✓ No hardcoded secrets"
! [ -f .env ] && echo "✓ No .env file"
```

## ZIP Size Guidelines

| Size | Status | Recommendation |
|------|--------|-----------------|
| < 100 KB | ✅ Excellent | Small, fast download |
| 100 KB - 500 KB | ✅ Good | Typical for feature-rich plugins |
| 500 KB - 2 MB | ⚠️ Large | Consider optimizing dependencies |
| > 2 MB | ❌ Too Large | Likely includes unnecessary files |

## Optimization Tips

If your ZIP is too large:

1. **Remove node_modules before zipping:**
   ```bash
   rm -rf node_modules/
   zip -r my-plugin-1.0.0.zip my-plugin/
   ```

2. **Exclude src/ if dist/ exists:**
   ```bash
   zip -r my-plugin-1.0.0.zip my-plugin/ -x "my-plugin/src/*"
   ```

3. **Minimize dependencies:**
   ```bash
   # Remove unused packages
   npm prune --production
   ```

4. **Check for large assets:**
   ```bash
   find dist/ -size +100k
   ```

## Common Packaging Mistakes

❌ **Missing dist/remoteEntry.js**
```bash
# Forgot to build!
npm run build  # Fix this first
```

❌ **Version mismatch**
```python
# package.json: 1.0.0
# lifecycle_manager.py: 1.0.1
# ZIP name: my-plugin-1.0.2.zip
# FIX: Make all three match!
```

❌ **Wrong scope in webpack**
```javascript
// plugin_slug: "MyPlugin"
// webpack scope: "DifferentScope"  // WRONG!
```

❌ **Including node_modules**
```bash
# This makes ZIP huge!
# Always exclude node_modules
```

## Re-Packaging for Updates

When releasing a new version:

1. Update version in package.json, lifecycle_manager.py, changelog
2. Rebuild: `npm run build`
3. Create new ZIP: `{slug}-{new-version}.zip`
4. Keep old ZIP for version history
5. Update plugin manifest/changelog

---

Packaging is the final step. Get it right, and users can install your plugin with a single click.
