# Code Signing Setup Guide

This guide will help you set up code signing for your iOS app in GitHub Actions.

## Prerequisites

1. **Apple Developer Account** (Individual or Organization)
2. **Xcode** installed on macOS
3. Access to GitHub repository settings

## Step 1: Create Signing Certificate

### Option A: Using Xcode (Recommended)

1. Open **Xcode**
2. Go to **Xcode → Preferences → Accounts**
3. Sign in with your Apple ID
4. Select your team → **Manage Certificates**
5. Click **+** → **Apple Distribution** (for App Store) or **Apple Development**
6. The certificate will be created and stored in your Keychain

### Option B: Using Apple Developer Portal

1. Go to [Apple Developer](https://developer.apple.com/account/resources/certificates)
2. Click **+** to create a new certificate
3. Choose **iOS App Development** or **iOS Distribution**
4. Follow the instructions to create a Certificate Signing Request (CSR)
5. Download the certificate and install it in Keychain

## Step 2: Export Certificate as .p12

1. Open **Keychain Access** on macOS
2. Select **login** keychain and **My Certificates** category
3. Find your certificate (e.g., "Apple Development: your@email.com")
4. Right-click → **Export "Apple Development..."**
5. Save as **Certificates.p12**
6. Set a **password** (you'll need this later)

## Step 3: Create App ID

1. Go to [Identifiers](https://developer.apple.com/account/resources/identifiers)
2. Click **+** to register a new identifier
3. Select **App IDs** → **Continue**
4. Choose **App** → **Continue**
5. Set:
   - **Description**: FirebaseDistributionApp
   - **Bundle ID**: `com.example.firebasedistributionapp` (or your custom ID)
   - **Capabilities**: Select as needed
6. Click **Continue** → **Register**

## Step 4: Create Provisioning Profile

1. Go to [Profiles](https://developer.apple.com/account/resources/profiles)
2. Click **+** to create a new profile
3. Choose:
   - **iOS App Development** (for testing)
   - **Ad Hoc** (for Firebase distribution)
   - **App Store** (for App Store submission)
4. Select your **App ID** created in Step 3
5. Select the **certificate** created in Step 1
6. Select **devices** (for Development/Ad Hoc profiles)
7. Enter a **Profile Name**: e.g., "FirebaseDistributionApp AdHoc"
8. Download the **.mobileprovision** file

## Step 5: Encode Files to Base64

### On macOS/Linux:

```bash
# Encode certificate
base64 -i Certificates.p12 -o certificate.txt

# Encode provisioning profile
base64 -i YourProfile.mobileprovision -o profile.txt
```

### On Windows (PowerShell):

```powershell
# Encode certificate
[Convert]::ToBase64String([IO.File]::ReadAllBytes("Certificates.p12")) | Out-File certificate.txt

# Encode provisioning profile
[Convert]::ToBase64String([IO.File]::ReadAllBytes("YourProfile.mobileprovision")) | Out-File profile.txt
```

## Step 6: Create ExportOptions.plist

Create a file called `ExportOptions.plist`:

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
        <string>FirebaseDistributionApp AdHoc</string>
    </dict>
</dict>
</plist>
```

**Replace:**
- `YOUR_TEAM_ID` - Your Apple Team ID (found in Apple Developer Account)
- `com.example.firebasedistributionapp` - Your bundle ID
- `FirebaseDistributionApp AdHoc` - Your provisioning profile name

**Valid `method` values:**
- `ad-hoc` - For Firebase/TestFlight distribution
- `app-store` - For App Store submission
- `development` - For development builds
- `enterprise` - For enterprise distribution

### Encode ExportOptions.plist:

```bash
# macOS/Linux
base64 -i ExportOptions.plist -o exportoptions.txt

# Windows PowerShell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("ExportOptions.plist")) | Out-File exportoptions.txt
```

## Step 7: Add GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings → Secrets and variables → Actions**
3. Click **New repository secret**
4. Add the following secrets:

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `BUILD_CERTIFICATE_BASE64` | Content of `certificate.txt` | Base64 encoded .p12 certificate |
| `P12_PASSWORD` | Your certificate password | Password you set when exporting .p12 |
| `BUILD_PROVISION_PROFILE_BASE64` | Content of `profile.txt` | Base64 encoded provisioning profile |
| `KEYCHAIN_PASSWORD` | Any strong password | Temporary keychain password (e.g., `actions`) |
| `DEVELOPMENT_TEAM` | Your Team ID | Found in Apple Developer account |
| `EXPORT_OPTIONS_PLIST` | Content of `exportoptions.txt` | Base64 encoded ExportOptions.plist |

## Step 8: Update Bundle ID (if needed)

If you're using a custom bundle ID, update `project.yml`:

```yaml
PRODUCT_BUNDLE_IDENTIFIER: com.yourcompany.yourapp
```

## Step 9: Trigger Workflow

1. Commit and push your changes
2. Go to **Actions** tab in GitHub
3. The workflow will run automatically
4. Download the signed IPA from **Artifacts**

## Troubleshooting

### Error: "No signing identity found"

**Solution**: 
- Verify certificate is properly encoded
- Check P12_PASSWORD is correct
- Ensure certificate hasn't expired

### Error: "No provisioning profile matches"

**Solution**:
- Verify bundle ID matches in all places:
  - `project.yml`
  - Provisioning profile
  - `ExportOptions.plist`
- Ensure provisioning profile hasn't expired

### Error: "User interaction is not allowed"

**Solution**: 
- This is handled by `security set-key-partition-list` in the workflow
- If it persists, check keychain unlock step

### How to find Team ID

1. Go to [Apple Developer Membership](https://developer.apple.com/account/#/membership)
2. Your **Team ID** is displayed at the top

### How to verify Bundle ID

In Xcode project or `project.yml`, check:
```yaml
PRODUCT_BUNDLE_IDENTIFIER: com.example.firebasedistributionapp
```

## Security Best Practices

1. ✅ Never commit certificates or profiles to git
2. ✅ Use strong passwords for P12 export
3. ✅ Rotate certificates regularly
4. ✅ Use repository secrets, not organization secrets
5. ✅ Limit access to repository settings
6. ✅ Review workflow runs for sensitive data leaks

## Next Steps

After successful signing:
1. Download signed IPA from GitHub Actions artifacts
2. Upload to Firebase App Distribution
3. Distribute to testers

## Resources

- [Apple Developer Documentation](https://developer.apple.com/documentation/)
- [Code Signing Guide](https://developer.apple.com/support/code-signing/)
- [GitHub Actions Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
