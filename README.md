# Firebase Distribution iOS App

A simple iOS application configured for **signed IPA generation** and Firebase App Distribution.

## Project Structure

```
.
├── FirebaseDistributionApp/
│   ├── AppDelegate.swift       # App entry point
│   ├── ViewController.swift    # Main view controller
│   └── Info.plist             # App metadata
├── project.yml                 # XcodeGen configuration
├── SIGNING_SETUP.md           # Code signing setup guide
└── .github/
    └── workflows/
        └── build-ipa.yml       # GitHub Actions workflow
```

## Features

- ✅ Clean iOS app with UIKit
- ✅ **Signed IPA generation** with code signing
- ✅ GitHub Actions CI/CD
- ✅ Ready for Firebase App Distribution
- ✅ Secure certificate management via GitHub Secrets

## Local Development

### Prerequisites
- macOS with Xcode installed
- XcodeGen (`brew install xcodegen`)

### Setup and Build

```bash
# Generate Xcode project
xcodegen generate

# Open in Xcode
open FirebaseDistributionApp.xcodeproj

# Or build from command line
xcodebuild -project FirebaseDistributionApp.xcodeproj \
  -scheme FirebaseDistributionApp \
  -destination 'platform=iOS Simulator,name=iPhone 14' \
  build
```

### Create IPA Locally

```bash
# Generate project
xcodegen generate

# Archive
xcodebuild clean archive \
  -project FirebaseDistributionApp.xcodeproj \
  -scheme FirebaseDistributionApp \
  -archivePath ./build/FirebaseDistributionApp.xcarchive \
  -destination 'generic/platform=iOS' \
  CODE_SIGNING_ALLOWED=NO

# Create IPA
mkdir -p Payload
cp -R ./build/FirebaseDistributionApp.xcarchive/Products/Applications/FirebaseDistributionApp.app Payload/
zip -r FirebaseDistributionApp.ipa Payload
rm -rf Payload
```

## CI/CD Workflow

The GitHub Actions workflow automatically:

1. **Generates** Xcode project using XcodeGen
2. **Sets up** code signing with certificates from GitHub Secrets
3. **Builds** the iOS app with signing
4. **Archives** with proper code signing
5. **Exports** signed IPA file
6. **Uploads** IPA as artifact (30-day retention)

### Required GitHub Secrets

Before the workflow can create signed IPAs, you need to configure these secrets:

| Secret | Description |
|--------|-------------|
| `BUILD_CERTIFICATE_BASE64` | Base64 encoded .p12 signing certificate |
| `P12_PASSWORD` | Password for the .p12 certificate |
| `BUILD_PROVISION_PROFILE_BASE64` | Base64 encoded provisioning profile |
| `KEYCHAIN_PASSWORD` | Temporary keychain password (e.g., "actions") |
| `DEVELOPMENT_TEAM` | Your Apple Developer Team ID |
| `EXPORT_OPTIONS_PLIST` | Base64 encoded ExportOptions.plist |

**📖 See [SIGNING_SETUP.md](SIGNING_SETUP.md) for detailed setup instructions.**

### Workflow Triggers
- Push to `main` branch
- Pull requests to `main`
- Manual workflow dispatch

### Download IPA

1. Go to **Actions** tab in GitHub
2. Select latest workflow run
3. Download **FirebaseDistributionApp-IPA-Signed** artifact

## Code Signing Setup

To enable signed IPA generation, follow these steps:

### Quick Start

1. **Read the guide**: [SIGNING_SETUP.md](SIGNING_SETUP.md)
2. **Generate certificates** from Apple Developer Portal
3. **Encode to Base64** (certificate, provisioning profile, ExportOptions.plist)
4. **Add GitHub Secrets** (6 secrets required)
5. **Push to trigger** signed build

### ExportOptions.plist Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>ad-hoc</string>
    <key>teamID</key>
    <string>YOUR_TEAM_ID</string>
    <key>signingStyle</key>
    <string>manual</string>
    <key>provisioningProfiles</key>
    <dict>
        <key>com.example.firebasedistributionapp</key>
        <string>YOUR_PROFILE_NAME</string>
    </dict>
</dict>
</plist>
```

For complete instructions, see **[SIGNING_SETUP.md](SIGNING_SETUP.md)**.

## Firebase App Distribution Setup

### Step 1: Install Firebase CLI

```bash
npm install -g firebase-tools
firebase login
```

### Step 2: Initialize Firebase

```bash
firebase init
# Select "App Distribution"
```

### Step 3: Upload IPA to Firebase

```bash
# Download IPA from GitHub Actions artifact
# Then upload to Firebase
firebase appdistribution:distribute FirebaseDistributionApp.ipa \
  --app YOUR_FIREBASE_APP_ID \
  --groups "testers" \
  --release-notes "Build from GitHub Actions"
```

### Step 4: Automate with GitHub Actions (Optional)

Add Firebase App Distribution to your workflow:

```yaml
- name: Upload to Firebase App Distribution
  uses: wzieba/Firebase-Distribution-Github-Action@v1
  with:
    appId: ${{ secrets.FIREBASE_APP_ID }}
    token: ${{ secrets.FIREBASE_TOKEN }}
    groups: testers
    file: output/FirebaseDistributionApp.ipa
    releaseNotes: "Automated build from commit ${{ github.sha }}"
```

Required secrets:
- `FIREBASE_APP_ID` - Your Firebase iOS app ID
- `FIREBASE_TOKEN` - Firebase CI token (`firebase login:ci`)

## Configuration

### Bundle ID
Change bundle ID in `project.yml`:
```yaml
PRODUCT_BUNDLE_IDENTIFIER: com.yourcompany.yourapp
```

### App Version
Update in `FirebaseDistributionApp/Info.plist`:
```xml
<key>CFBundleShortVersionString</key>
<string>1.0</string>
<key>CFBundleVersion</key>
<string>1</string>
```

### Deployment Target
Modify in `project.yml`:
```yaml
deploymentTarget:
  iOS: "13.0"
```

## Troubleshooting

### Issue: "No signing identity found"
**Solution**: Verify GitHub Secrets are configured correctly. See [SIGNING_SETUP.md](SIGNING_SETUP.md)

### Issue: "No provisioning profile matches"
**Solution**: Ensure bundle ID matches in `project.yml`, provisioning profile, and `ExportOptions.plist`

### Issue: "No .app found in archive"
**Solution**: Ensure the archive step completed successfully. Check GitHub Actions logs.

### Issue: Certificate expired
**Solution**: Regenerate certificate in Apple Developer Portal and update `BUILD_CERTIFICATE_BASE64` secret

### Issue: Workflow fails at signing step
**Solution**: Check that all 6 GitHub Secrets are set with correct Base64 values

## Next Steps

1. ✅ Download IPA from GitHub Actions
2. ⬜ Set up Firebase project
3. ⬜ Upload IPA to Firebase App Distribution
4. ⬜ Invite testers
5. ⬜ Automate distribution in workflow

## Repository

- **GitHub**: [lalitpipeline/IOS-CICD-POC](https://github.com/lalitpipeline/IOS-CICD-POC)
- **Workflow**: [build-ipa.yml](.github/workflows/build-ipa.yml)

## License

MIT
