---
name: asset-catalog-optimizer
description: Analyze and optimize Xcode asset catalogs - find unused assets, missing resolutions, compress images
type: skill
language: python
---

# Asset Catalog Optimizer

Analyze `.xcassets` folders to optimize image assets and find issues.

## Capabilities

- List all assets in catalogs
- Find unused images (not referenced in code)
- Check for missing @2x/@3x versions
- Analyze image file sizes
- Compress images losslessly
- Detect duplicate images
- Validate asset naming
- Generate asset inventory
- Calculate total asset size
- Suggest optimizations

## Tools Included

###  `asset_optimizer.py`
Python script for asset analysis and optimization

**Commands:**
```bash
# Analyze asset catalog
./asset_optimizer.py PaleoRose/Assets.xcassets analyze

# Find unused assets
./asset_optimizer.py PaleoRose/Assets.xcassets find-unused --source-dir PaleoRose

# Check missing resolutions
./asset_optimizer.py PaleoRose/Assets.xcassets check-resolutions

# Compress images
./asset_optimizer.py PaleoRose/Assets.xcassets compress --quality 85

# Generate report
./asset_optimizer.py PaleoRose/Assets.xcassets report --output report.html

# Calculate sizes
./asset_optimizer.py PaleoRose/Assets.xcassets sizes
```

## Features

### Unused Asset Detection
Searches Swift/ObjC code for asset references like:
- `NSImage(named: "icon")`
- `UIImage(named: "logo")`
- `Image("background")`
- `[NSImage imageNamed:@"button"]`

### Resolution Checking
Verifies all imagesets have:
- @1x (universal or Mac)
- @2x (required for Retina)
- @3x (iOS only, warns if missing)

### Compression
- PNG: lossless optimization with pngquant/optipng
- JPEG: quality-based compression
- Preserves transparency
- Maintains color profiles

## Usage

Run when:
- App bundle is too large
- Need to audit assets
- Before App Store submission
- Cleaning up old/unused assets
- Optimizing performance

## Output Examples

```
Asset Catalog Analysis: Assets.xcassets
========================================

Total Assets: 47
Total Size: 12.3 MB

Missing Resolutions (5):
  - icon.imageset: Missing @2x
  - logo.imageset: Missing @3x
  - background.imageset: Missing @2x, @3x

Unused Assets (8):
  - old-button.imageset (45 KB)
  - deprecated-icon.imageset (23 KB)
  - test-image.imageset (156 KB)

Potential Savings:
  - Compression: 3.2 MB (26%)
  - Remove unused: 1.8 MB (15%)
  - Total: 5.0 MB (41%)

Recommendations:
  1. Compress 23 PNG files to save 3.2 MB
  2. Remove 8 unused assets to save 1.8 MB
  3. Add missing @2x/@3x versions for 5 assets
```
