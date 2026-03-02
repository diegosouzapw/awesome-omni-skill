---
name: app-icon-generator
description: Generates app icons in all required sizes for iOS, Android, and PWA from a single source image. Use when user asks to "generate app icons", "create ios icons", "android app icons", "favicon", or "pwa icons".
allowed-tools: [Read, Write, Bash, Glob]
---

# App Icon Generator

Generates app icons in all required sizes for iOS, Android, PWA, and web from a single source image.

## When to Use

- "Generate app icons for iOS"
- "Create Android app icons"
- "Generate all icon sizes"
- "Create favicon"
- "PWA icons"
- "App icon set"

## Instructions

### 1. Verify Source Image

Check for source icon file:

```bash
# Look for icon source
find . -name "*icon*.png" -o -name "*logo*.png"
```

**Source image requirements:**
- Minimum 1024x1024 pixels (recommended)
- Square (1:1 aspect ratio)
- PNG format with transparency (if needed)
- High quality, not compressed
- No text too close to edges (safe area: center 70%)

Present findings and ask for source if not found.

### 2. Check for Image Processing Tools

Verify available tools:

```bash
# Check for ImageMagick
which convert || echo "ImageMagick not found"

# Check for sharp-cli
which sharp || echo "sharp-cli not found"

# Check for sips (macOS)
which sips || echo "sips not found (macOS only)"
```

**Installation guide if needed:**

```bash
# macOS
brew install imagemagick

# Ubuntu/Debian
sudo apt-get install imagemagick

# Node.js (cross-platform, recommended)
npm install -g sharp-cli
```

### 3. Generate iOS Icons

**iOS requires multiple sizes:**

```bash
# iOS App Icons (all required sizes)
declare -a ios_sizes=(
  "20"    # iPhone Notification (2x, 3x)
  "29"    # iPhone Settings (2x, 3x)
  "40"    # iPhone Spotlight (2x, 3x)
  "60"    # iPhone App (2x, 3x)
  "76"    # iPad App (1x, 2x)
  "83.5"  # iPad Pro App (2x)
  "1024"  # App Store
)

# Generate using ImageMagick
for size in "${ios_sizes[@]}"; do
  convert icon-source.png -resize ${size}x${size} "ios/icon-${size}.png"

  # Generate 2x
  size2x=$((size * 2))
  convert icon-source.png -resize ${size2x}x${size2x} "ios/icon-${size}@2x.png"

  # Generate 3x (for relevant sizes)
  if [[ $size -eq 20 || $size -eq 29 || $size -eq 40 || $size -eq 60 ]]; then
    size3x=$((size * 3))
    convert icon-source.png -resize ${size3x}x${size3x} "ios/icon-${size}@3x.png"
  fi
done
```

**Or using sharp-cli:**

```bash
# Generate all iOS sizes
sharp -i icon-source.png -o ios/icon-{size}.png \
  resize 20 40 60 58 80 87 120 180 76 152 167 1024
```

**Contents.json for Xcode:**

```json
{
  "images": [
    {
      "size": "20x20",
      "idiom": "iphone",
      "filename": "icon-40.png",
      "scale": "2x"
    },
    {
      "size": "20x20",
      "idiom": "iphone",
      "filename": "icon-60.png",
      "scale": "3x"
    },
    {
      "size": "29x29",
      "idiom": "iphone",
      "filename": "icon-58.png",
      "scale": "2x"
    },
    {
      "size": "29x29",
      "idiom": "iphone",
      "filename": "icon-87.png",
      "scale": "3x"
    },
    {
      "size": "40x40",
      "idiom": "iphone",
      "filename": "icon-80.png",
      "scale": "2x"
    },
    {
      "size": "40x40",
      "idiom": "iphone",
      "filename": "icon-120.png",
      "scale": "3x"
    },
    {
      "size": "60x60",
      "idiom": "iphone",
      "filename": "icon-120.png",
      "scale": "2x"
    },
    {
      "size": "60x60",
      "idiom": "iphone",
      "filename": "icon-180.png",
      "scale": "3x"
    },
    {
      "size": "1024x1024",
      "idiom": "ios-marketing",
      "filename": "icon-1024.png",
      "scale": "1x"
    }
  ],
  "info": {
    "version": 1,
    "author": "xcode"
  }
}
```

### 4. Generate Android Icons

**Android adaptive icons:**

Android uses adaptive icons with separate foreground and background layers.

```bash
# Android icon sizes (in dp)
# mdpi = 1x, hdpi = 1.5x, xhdpi = 2x, xxhdpi = 3x, xxxhdpi = 4x

# mipmap-mdpi (48x48)
convert icon-source.png -resize 48x48 android/mipmap-mdpi/ic_launcher.png

# mipmap-hdpi (72x72)
convert icon-source.png -resize 72x72 android/mipmap-hdpi/ic_launcher.png

# mipmap-xhdpi (96x96)
convert icon-source.png -resize 96x96 android/mipmap-xhdpi/ic_launcher.png

# mipmap-xxhdpi (144x144)
convert icon-source.png -resize 144x144 android/mipmap-xxhdpi/ic_launcher.png

# mipmap-xxxhdpi (192x192)
convert icon-source.png -resize 192x192 android/mipmap-xxxhdpi/ic_launcher.png

# Play Store (512x512)
convert icon-source.png -resize 512x512 android/playstore-icon.png
```

**Adaptive icon XML:**

```xml
<!-- res/mipmap-anydpi-v26/ic_launcher.xml -->
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@color/ic_launcher_background"/>
    <foreground android:drawable="@mipmap/ic_launcher_foreground"/>
</adaptive-icon>
```

**Round icon variant:**

```bash
# Generate round icons (same sizes)
for size in 48 72 96 144 192; do
  density=$(get_density $size)  # mdpi, hdpi, etc.
  convert icon-source.png -resize ${size}x${size} \
    \( +clone -threshold -1 -negate -fill white -draw "circle $((size/2)),$((size/2)) $((size/2)),0" \) \
    -alpha off -compose copy_opacity -composite \
    "android/mipmap-${density}/ic_launcher_round.png"
done
```

### 5. Generate PWA Icons

**Progressive Web App icons:**

```bash
# PWA icon sizes
sharp -i icon-source.png -o pwa/icon-{size}.png \
  resize 72 96 128 144 152 192 384 512

# Also generate maskable icons (with safe area)
# Maskable icons need 40% safe area
sharp -i icon-source.png -o pwa/icon-{size}-maskable.png \
  resize 72 96 128 144 152 192 384 512 \
  --extend top=10 bottom=10 left=10 right=10
```

**manifest.json:**

```json
{
  "name": "My App",
  "short_name": "App",
  "icons": [
    {
      "src": "/icons/icon-72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192-maskable.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable"
    },
    {
      "src": "/icons/icon-512-maskable.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ]
}
```

### 6. Generate Favicons

**Web favicons (multiple sizes):**

```bash
# Standard sizes
convert icon-source.png -resize 16x16 favicon-16.png
convert icon-source.png -resize 32x32 favicon-32.png
convert icon-source.png -resize 48x48 favicon-48.png

# Create multi-size .ico file
convert favicon-16.png favicon-32.png favicon-48.png favicon.ico

# Apple touch icon
convert icon-source.png -resize 180x180 apple-touch-icon.png

# Microsoft tile
convert icon-source.png -resize 144x144 mstile-144.png
```

**HTML head tags:**

```html
<!-- Favicons -->
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16.png">
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32.png">
<link rel="icon" type="image/png" sizes="48x48" href="/favicon-48.png">
<link rel="shortcut icon" href="/favicon.ico">

<!-- Apple touch icon -->
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">

<!-- Android -->
<link rel="icon" type="image/png" sizes="192x192" href="/android-chrome-192.png">
<link rel="icon" type="image/png" sizes="512x512" href="/android-chrome-512.png">

<!-- Microsoft -->
<meta name="msapplication-TileColor" content="#ffffff">
<meta name="msapplication-TileImage" content="/mstile-144.png">
```

### 7. Generate React Native Icons

**For React Native apps:**

```bash
# iOS (place in ios/AppName/Images.xcassets/AppIcon.appiconset/)
# Same as iOS native above

# Android (place in android/app/src/main/res/)
# Same as Android native above
```

**Or use react-native-make:**

```bash
npx react-native set-icon --path icon-source.png
```

### 8. Generate Flutter Icons

**Using flutter_launcher_icons:**

```yaml
# pubspec.yaml
dev_dependencies:
  flutter_launcher_icons: ^0.13.1

flutter_launcher_icons:
  android: "launcher_icon"
  ios: true
  image_path: "assets/icon.png"
  adaptive_icon_background: "#ffffff"
  adaptive_icon_foreground: "assets/icon-foreground.png"
```

```bash
# Generate icons
flutter pub get
flutter pub run flutter_launcher_icons
```

### 9. Create Automated Script

**Complete icon generation script:**

```bash
#!/bin/bash
# generate-icons.sh

SOURCE_ICON="icon-source.png"
OUTPUT_DIR="generated-icons"

# Check if source exists
if [ ! -f "$SOURCE_ICON" ]; then
  echo "Error: Source icon not found: $SOURCE_ICON"
  exit 1
fi

# Create directories
mkdir -p "$OUTPUT_DIR"/{ios,android,pwa,web}

echo "🎨 Generating app icons..."

# iOS icons
echo "📱 iOS icons..."
for size in 40 60 58 87 80 120 180 76 152 167 1024; do
  convert "$SOURCE_ICON" -resize ${size}x${size} \
    "$OUTPUT_DIR/ios/icon-${size}.png"
  echo "  ✓ ${size}x${size}"
done

# Android icons
echo "🤖 Android icons..."
convert "$SOURCE_ICON" -resize 48x48 "$OUTPUT_DIR/android/mdpi.png"
convert "$SOURCE_ICON" -resize 72x72 "$OUTPUT_DIR/android/hdpi.png"
convert "$SOURCE_ICON" -resize 96x96 "$OUTPUT_DIR/android/xhdpi.png"
convert "$SOURCE_ICON" -resize 144x144 "$OUTPUT_DIR/android/xxhdpi.png"
convert "$SOURCE_ICON" -resize 192x192 "$OUTPUT_DIR/android/xxxhdpi.png"
convert "$SOURCE_ICON" -resize 512x512 "$OUTPUT_DIR/android/playstore.png"
echo "  ✓ All densities generated"

# PWA icons
echo "🌐 PWA icons..."
for size in 72 96 128 144 152 192 384 512; do
  convert "$SOURCE_ICON" -resize ${size}x${size} \
    "$OUTPUT_DIR/pwa/icon-${size}.png"
  echo "  ✓ ${size}x${size}"
done

# Favicons
echo "🔖 Favicons..."
convert "$SOURCE_ICON" -resize 16x16 "$OUTPUT_DIR/web/favicon-16.png"
convert "$SOURCE_ICON" -resize 32x32 "$OUTPUT_DIR/web/favicon-32.png"
convert "$SOURCE_ICON" -resize 48x48 "$OUTPUT_DIR/web/favicon-48.png"
convert "$OUTPUT_DIR/web/favicon-"{16,32,48}.png "$OUTPUT_DIR/web/favicon.ico"
convert "$SOURCE_ICON" -resize 180x180 "$OUTPUT_DIR/web/apple-touch-icon.png"
echo "  ✓ All favicons generated"

echo "✅ Icon generation complete!"
echo "📁 Icons saved to: $OUTPUT_DIR"
```

### 10. Provide Integration Instructions

**iOS (Xcode):**
1. Open Xcode project
2. Navigate to Assets.xcassets
3. Right-click → New App Icon
4. Drag generated icons to appropriate slots

**Android (Android Studio):**
1. Right-click res folder
2. New → Image Asset
3. Select generated icons
4. Configure adaptive icon layers

**Web:**
1. Copy icons to public/icons/ folder
2. Update manifest.json with icon paths
3. Add favicon links to index.html

**React Native:**
1. Copy iOS icons to `ios/AppName/Images.xcassets/AppIcon.appiconset/`
2. Copy Android icons to `android/app/src/main/res/mipmap-*/`
3. Update Contents.json for iOS

### Best Practices

**Design Guidelines:**
- Keep important content in center 80%
- Avoid thin lines (minimum 2px)
- Test on dark and light backgrounds
- Use simple, recognizable shapes
- Avoid text (too small on icons)
- Use bold colors for visibility

**Technical Guidelines:**
- Source: 1024x1024 minimum
- Format: PNG with transparency
- Color space: sRGB
- No compression on source
- Square aspect ratio
- Safe area for maskable: 40% padding

**Testing:**
- Test on actual devices
- Check all sizes render correctly
- Verify transparency works
- Test dark mode appearance
- Check adaptive icon animations (Android)

### Size Reference

**iOS:**
- 20pt (40x40, 60x60)
- 29pt (58x58, 87x87)
- 40pt (80x80, 120x120)
- 60pt (120x120, 180x180)
- 76pt (76x76, 152x152)
- 83.5pt (167x167)
- 1024x1024 (App Store)

**Android:**
- mdpi: 48x48
- hdpi: 72x72
- xhdpi: 96x96
- xxhdpi: 144x144
- xxxhdpi: 192x192
- Play Store: 512x512

**PWA:**
- 72x72, 96x96, 128x128
- 144x144, 152x152, 192x192
- 384x384, 512x512

**Web:**
- 16x16, 32x32 (favicon)
- 180x180 (Apple touch)
- 192x192, 512x512 (Android)
