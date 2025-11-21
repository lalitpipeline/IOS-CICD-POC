# Requestify Demo iOS App

This is a minimal iOS app that demonstrates the Requestify Swift Package.

## Structure
- The app is in the `RequestifyApp/` directory
- It links to the Requestify package in the parent directory
- The Xcode project is `RequestifyApp.xcodeproj`

## Building
Open `RequestifyApp/RequestifyApp.xcodeproj` in Xcode and build.

Or use the command line:
```bash
cd RequestifyApp
xcodebuild -project RequestifyApp.xcodeproj -scheme RequestifyApp -destination 'generic/platform=iOS' archive
```
