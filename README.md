# Firebase Distribution iOS App

A simple iOS application configured for unsigned IPA generation and Firebase App Distribution.

## Project Structure

```
.
├── FirebaseDistributionApp/
│   ├── AppDelegate.swift       # App entry point
│   ├── ViewController.swift    # Main view controller
│   └── Info.plist             # App metadata
├── project.yml                 # XcodeGen configuration
└── .github/
    └── workflows/
        └── build-ipa.yml       # GitHub Actions workflow
```

## Features

- ✅ Clean iOS app with UIKit
- ✅ Unsigned IPA generation
- ✅ GitHub Actions CI/CD
- ✅ Ready for Firebase App Distribution
- ✅ No code signing required

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
2. **Builds** the iOS app
3. **Archives** without code signing
4. **Creates** unsigned IPA file
5. **Uploads** IPA as artifact (30-day retention)

### Workflow Triggers
- Push to `main` branch
- Pull requests to `main`
- Manual workflow dispatch

### Download IPA

1. Go to **Actions** tab in GitHub
2. Select latest workflow run
3. Download **FirebaseDistributionApp-IPA** artifact

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

### Issue: "No .app found in archive"
**Solution**: Ensure the archive step completed successfully. Check GitHub Actions logs.

### Issue: "Unsigned IPA won't install"
**Solution**: Unsigned IPAs require:
- Jailbroken device, OR
- Developer provisioning profile, OR
- Firebase App Distribution (recommended)

### Issue: XcodeGen not found
**Solution**: Install XcodeGen
```bash
brew install xcodegen
```

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
