---
name: ios-ci-cd
description: Set up CI/CD pipelines for iOS apps using GitHub Actions, Fastlane, and Xcode Cloud. Use when configuring automated builds, testing, code signing, TestFlight deployment, or App Store release automation. Part of the idea-to-App-Store workflow before publishing.
---

# iOS CI/CD

Automate building, testing, and deploying iOS applications.

## CI/CD Overview

```
Push → Build → Test → Sign → Deploy → Notify
         ↓       ↓      ↓       ↓
      SwiftLint  Unit  Match  TestFlight
                 UI          App Store
```

## GitHub Actions

### Basic Workflow
```yaml
# .github/workflows/ios.yml
name: iOS CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: macos-14  # Latest macOS with Xcode
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Select Xcode
        run: sudo xcode-select -s /Applications/Xcode_15.2.app
        
      - name: Show Xcode version
        run: xcodebuild -version
        
      - name: Install dependencies
        run: |
          brew install swiftlint
          
      - name: Run SwiftLint
        run: swiftlint --strict
        
      - name: Build
        run: |
          xcodebuild build \
            -workspace MyApp.xcworkspace \
            -scheme MyApp \
            -destination 'platform=iOS Simulator,name=iPhone 15' \
            -configuration Debug \
            CODE_SIGN_IDENTITY="" \
            CODE_SIGNING_REQUIRED=NO
            
      - name: Run Tests
        run: |
          xcodebuild test \
            -workspace MyApp.xcworkspace \
            -scheme MyApp \
            -destination 'platform=iOS Simulator,name=iPhone 15' \
            -enableCodeCoverage YES \
            -resultBundlePath TestResults
            
      - name: Upload Test Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: TestResults
```

### TestFlight Deployment
```yaml
# .github/workflows/deploy.yml
name: Deploy to TestFlight

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: macos-14
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Select Xcode
        run: sudo xcode-select -s /Applications/Xcode_15.2.app
        
      - name: Install Fastlane
        run: gem install fastlane
        
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true
          
      - name: Install Certificates
        env:
          MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
          MATCH_GIT_URL: ${{ secrets.MATCH_GIT_URL }}
          MATCH_GIT_BASIC_AUTHORIZATION: ${{ secrets.MATCH_GIT_BASIC_AUTHORIZATION }}
        run: fastlane match appstore --readonly
        
      - name: Build and Upload
        env:
          APP_STORE_CONNECT_API_KEY_KEY_ID: ${{ secrets.APP_STORE_CONNECT_KEY_ID }}
          APP_STORE_CONNECT_API_KEY_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_ISSUER_ID }}
          APP_STORE_CONNECT_API_KEY_KEY: ${{ secrets.APP_STORE_CONNECT_KEY }}
        run: fastlane beta
```

## Fastlane Configuration

### Setup
```bash
# Install Fastlane
gem install fastlane

# Initialize in project
cd /path/to/project
fastlane init

# This creates:
# fastlane/Fastfile
# fastlane/Appfile
```

### Appfile
```ruby
# fastlane/Appfile
app_identifier("com.example.myapp")
apple_id("developer@example.com")
team_id("XXXXXXXXXX")

# App Store Connect API Key (preferred over password)
# app_store_connect_api_key(
#   key_id: "D383SF739",
#   issuer_id: "6053b7fe-68a8-4acb-89be-165aa6465141",
#   key_filepath: "./AuthKey_D383SF739.p8"
# )
```

### Fastfile
```ruby
# fastlane/Fastfile
default_platform(:ios)

platform :ios do
  
  # Run before all lanes
  before_all do
    ensure_git_status_clean
  end
  
  # MARK: - Testing
  
  desc "Run all tests"
  lane :test do
    scan(
      workspace: "MyApp.xcworkspace",
      scheme: "MyApp",
      device: "iPhone 15",
      code_coverage: true,
      output_directory: "./fastlane/test_output"
    )
  end
  
  # MARK: - Beta
  
  desc "Push a new beta build to TestFlight"
  lane :beta do
    # Increment build number
    increment_build_number(
      build_number: Time.now.strftime("%Y%m%d%H%M")
    )
    
    # Sync certificates
    match(type: "appstore", readonly: true)
    
    # Build
    build_app(
      workspace: "MyApp.xcworkspace",
      scheme: "MyApp",
      configuration: "Release",
      export_method: "app-store"
    )
    
    # Upload to TestFlight
    upload_to_testflight(
      skip_waiting_for_build_processing: true
    )
    
    # Notify
    slack(
      message: "New beta build uploaded to TestFlight! 🚀",
      success: true
    )
  end
  
  # MARK: - Release
  
  desc "Push a new release to App Store"
  lane :release do
    # Run tests first
    test
    
    # Increment version
    increment_version_number(
      bump_type: "patch"  # major, minor, patch
    )
    
    # Build and upload
    match(type: "appstore", readonly: true)
    build_app(
      workspace: "MyApp.xcworkspace",
      scheme: "MyApp",
      configuration: "Release",
      export_method: "app-store"
    )
    
    # Upload to App Store
    upload_to_app_store(
      submit_for_review: false,
      automatic_release: false,
      skip_screenshots: true,
      skip_metadata: false
    )
    
    # Create git tag
    add_git_tag(tag: "v#{get_version_number}")
    push_to_git_remote
  end
  
  # MARK: - Code Signing
  
  desc "Sync certificates and profiles"
  lane :sync_signing do
    match(type: "development")
    match(type: "appstore")
  end
  
  # MARK: - Screenshots
  
  desc "Capture screenshots"
  lane :screenshots do
    capture_screenshots
    frame_screenshots(silver: true)
  end
  
  # Error handling
  error do |lane, exception|
    slack(
      message: "Build failed: #{exception.message}",
      success: false
    )
  end
end
```

## Code Signing with Match

### Setup Match
```bash
# Initialize match
fastlane match init

# Create certificates and profiles
fastlane match development
fastlane match appstore

# On CI, use readonly
fastlane match appstore --readonly
```

### Matchfile
```ruby
# fastlane/Matchfile
git_url("git@github.com:yourcompany/certificates.git")
storage_mode("git")

type("appstore")  # development, adhoc, appstore

app_identifier(["com.example.myapp"])
username("developer@example.com")

# For CI
# git_basic_authorization(ENV["MATCH_GIT_BASIC_AUTHORIZATION"])
```

### CI Secrets Required
```yaml
# GitHub Secrets
MATCH_PASSWORD          # Encryption password for certificates
MATCH_GIT_URL           # Git repo URL for certificates
MATCH_GIT_BASIC_AUTHORIZATION  # base64(username:token)

# App Store Connect API Key
APP_STORE_CONNECT_KEY_ID
APP_STORE_CONNECT_ISSUER_ID
APP_STORE_CONNECT_KEY    # .p8 key content
```

## Xcode Cloud

### Workflow Configuration
```yaml
# ci_scripts/ci_post_clone.sh
#!/bin/sh

# Install dependencies
brew install swiftlint

# Install Swift packages
xcodebuild -resolvePackageDependencies
```

### Custom Scripts
```bash
# ci_scripts/ci_pre_xcodebuild.sh
#!/bin/sh

# Run SwiftLint
swiftlint --strict

# Environment-specific configuration
echo "Building for environment: $CI_WORKFLOW"
```

```bash
# ci_scripts/ci_post_xcodebuild.sh
#!/bin/sh

# Post-build actions
if [ "$CI_XCODEBUILD_EXIT_CODE" -eq 0 ]; then
    echo "Build succeeded!"
    # Send success notification
else
    echo "Build failed!"
    # Send failure notification
fi
```

### Xcode Cloud Workflow Settings
```
Workflow: Beta Releases
├── Start Conditions:
│   └── Tag: v*
├── Environment:
│   ├── Xcode: Latest Release
│   └── macOS: Latest
├── Actions:
│   ├── Build: iOS
│   ├── Test: iOS
│   └── Archive: iOS
└── Post-Actions:
    └── Deploy to TestFlight
```

## Build Matrix

### Multi-Configuration Testing
```yaml
# .github/workflows/matrix.yml
jobs:
  test:
    runs-on: macos-14
    strategy:
      matrix:
        xcode: ['15.0', '15.2']
        ios: ['17.0', '17.2']
        device: ['iPhone 15', 'iPad Pro (12.9-inch) (6th generation)']
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Select Xcode ${{ matrix.xcode }}
        run: sudo xcode-select -s /Applications/Xcode_${{ matrix.xcode }}.app
        
      - name: Test
        run: |
          xcodebuild test \
            -workspace MyApp.xcworkspace \
            -scheme MyApp \
            -destination 'platform=iOS Simulator,name=${{ matrix.device }},OS=${{ matrix.ios }}'
```

## Useful Actions

### Caching
```yaml
- name: Cache Swift packages
  uses: actions/cache@v4
  with:
    path: .build
    key: ${{ runner.os }}-spm-${{ hashFiles('**/Package.resolved') }}
    restore-keys: |
      ${{ runner.os }}-spm-
      
- name: Cache Ruby gems
  uses: actions/cache@v4
  with:
    path: vendor/bundle
    key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
```

### Notifications
```yaml
- name: Slack Notification
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    fields: repo,message,commit,author,action,eventName,ref,workflow
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
  if: always()
```

## Release Checklist

### Pre-Release
- [ ] All tests passing
- [ ] Version number incremented
- [ ] Build number unique
- [ ] Release notes prepared
- [ ] Screenshots updated (if needed)

### Deploy Commands
```bash
# Local deployment
fastlane beta          # TestFlight
fastlane release       # App Store

# Specific version
fastlane beta version:1.2.0

# Skip certain steps
fastlane beta skip_tests:true
```

### Post-Release
- [ ] Tag created and pushed
- [ ] Release notes published
- [ ] Team notified
- [ ] Monitor crash reports
- [ ] Monitor reviews

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| Code signing failed | Run `fastlane match` locally first |
| Build timeout | Increase GitHub Actions timeout |
| Simulator not found | Check Xcode version and device name |
| Upload failed | Verify API key permissions |

### Debugging Tips
```bash
# Verbose output
fastlane beta --verbose

# Print environment
fastlane action env

# Validate signing
fastlane run validate_provisioning_profile
```

## Resources

See [assets/github-actions/](assets/github-actions/) for workflow templates.
See [assets/fastlane/](assets/fastlane/) for Fastfile templates.
