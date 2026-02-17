# OpenPaw Packaging and Distribution Specification

**Version:** 1.0  
**Date:** February 17, 2026  
**Author:** Release Engineering Team  

## Overview

This document details the complete process of packaging and distributing OpenPaw from source code to user installation. It covers the build system, bundling strategy, code signing, notarization, distribution channels, and automated CI/CD pipeline.

---

## 1. Xcode Project Structure

### Project Organization
```
OpenPaw.xcodeproj/
├── OpenPaw/                          # Main macOS app target
│   ├── Sources/
│   │   ├── AppDelegate.swift         # App lifecycle management
│   │   ├── MainViewController.swift  # Primary chat interface
│   │   ├── SettingsViewController.swift
│   │   ├── OnboardingController.swift
│   │   └── Models/
│   │       ├── ChatMessage.swift
│   │       ├── AIProvider.swift
│   │       └── UserSettings.swift
│   ├── Resources/
│   │   ├── Assets.xcassets           # App icons, images
│   │   ├── Main.storyboard          # UI layouts
│   │   ├── Info.plist               # App configuration
│   │   └── Localizable.strings      # Localization
│   └── Embedded/
│       ├── node_runtime/            # Node.js runtime bundle
│       ├── openclaw_engine/         # OpenClaw engine binary
│       └── python_deps/             # Python dependencies
├── OpenPawTests/                     # Unit tests
├── OpenPawUITests/                   # UI automation tests
└── Frameworks/
    ├── Sparkle.framework            # Auto-update framework
    └── KeychainAccess.framework     # Secure credential storage
```

### Target Configuration
- **Deployment Target:** macOS 12.0 (Monterey)
- **Architectures:** arm64 (Apple Silicon), x86_64 (Intel)
- **Bundle Identifier:** `com.openpaw.OpenPaw`
- **Code Signing:** Developer ID Application
- **Hardened Runtime:** Enabled with entitlements
- **Sandboxing:** Disabled (requires full system access)

### Entitlements Required
```xml
<!-- OpenPaw.entitlements -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- Network access for AI APIs -->
    <key>com.apple.security.network.client</key>
    <true/>
    
    <!-- Outgoing network connections -->
    <key>com.apple.security.network.server</key>
    <true/>
    
    <!-- Accessibility API access -->
    <key>com.apple.security.automation.apple-events</key>
    <true/>
    
    <!-- File system access -->
    <key>com.apple.security.files.user-selected.read-write</key>
    <true/>
    
    <!-- Camera/microphone (if AI features require) -->
    <key>com.apple.security.device.camera</key>
    <true/>
    <key>com.apple.security.device.audio-input</key>
    <true/>
    
    <!-- Disable library validation for embedded runtimes -->
    <key>com.apple.security.cs.disable-library-validation</key>
    <true/>
    
    <!-- Allow execution of unsigned executables -->
    <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
    <true/>
    
    <!-- JIT compilation (for Node.js V8) -->
    <key>com.apple.security.cs.allow-jit</key>
    <true/>
</dict>
</plist>
```

---

## 2. Node.js + OpenClaw Bundling Strategy

### Runtime Embedding Approach
OpenPaw embeds complete runtime environments rather than depending on system installations:

1. **Node.js Runtime:** Official Node.js binaries embedded in app bundle
2. **OpenClaw Engine:** Compiled OpenClaw binary and dependencies
3. **Python Runtime:** Minimal Python installation for OpenClaw dependencies
4. **Native Libraries:** All required dylibs bundled and signed

### Bundle Structure Inside .app
```
OpenPaw.app/
├── Contents/
│   ├── Info.plist
│   ├── MacOS/
│   │   └── OpenPaw                   # Swift app binary
│   ├── Resources/                    # App resources
│   ├── Frameworks/
│   │   ├── Sparkle.framework
│   │   └── [other frameworks]
│   └── Embedded/
│       ├── node/
│       │   ├── bin/
│       │   │   └── node              # Node.js binary (universal)
│       │   ├── lib/
│       │   └── include/
│       ├── openclaw/
│       │   ├── bin/
│       │   │   └── openclaw          # OpenClaw engine binary
│       │   ├── lib/                  # OpenClaw libraries
│       │   └── skills/               # Bundled skills
│       ├── python/
│       │   ├── bin/
│       │   │   └── python3
│       │   ├── lib/
│       │   └── site-packages/        # Required Python packages
│       └── scripts/
│           ├── start_openclaw.sh     # Engine startup script
│           └── health_check.sh       # Engine health monitoring
```

### Runtime Management
The Swift app manages embedded runtimes:

```swift
class EmbeddedRuntimeManager {
    private let bundlePath = Bundle.main.bundlePath
    private let embeddedPath: String
    
    init() {
        embeddedPath = "\(bundlePath)/Contents/Embedded"
    }
    
    func startOpenClawEngine() throws {
        let enginePath = "\(embeddedPath)/openclaw/bin/openclaw"
        let nodePath = "\(embeddedPath)/node/bin/node"
        let pythonPath = "\(embeddedPath)/python/bin/python3"
        
        // Set environment for embedded runtimes
        setEnvironment([
            "NODE_PATH": "\(embeddedPath)/node/lib/node_modules",
            "PYTHONPATH": "\(embeddedPath)/python/lib/python3.x/site-packages",
            "OPENCLAW_HOME": "\(embeddedPath)/openclaw"
        ])
        
        // Launch OpenClaw with embedded Node.js
        let process = Process()
        process.executableURL = URL(fileURLWithPath: nodePath)
        process.arguments = [enginePath, "--embedded-mode"]
        try process.run()
    }
}
```

### Universal Binary Creation
Both Node.js and OpenClaw must support both Intel and Apple Silicon:

```bash
# Create universal Node.js binary
lipo -create \
  node-v18.x.x-darwin-x64/bin/node \
  node-v18.x.x-darwin-arm64/bin/node \
  -output node-universal

# Create universal OpenClaw binary  
lipo -create \
  openclaw-intel \
  openclaw-arm64 \
  -output openclaw-universal
```

---

## 3. Code Signing and Notarization Process

### Prerequisites
- **Apple Developer Account:** Paid developer program membership
- **Developer ID Certificate:** "Developer ID Application" certificate
- **App-Specific Password:** For notarization authentication
- **Xcode Command Line Tools:** For codesign and notarytool

### Code Signing Process
```bash
#!/bin/bash
# sign_app.sh - Code signing script

APP_PATH="OpenPaw.app"
DEVELOPER_ID="Developer ID Application: Your Name (TEAM_ID)"
ENTITLEMENTS="OpenPaw.entitlements"

echo "Signing embedded binaries..."
# Sign all embedded executables first
find "$APP_PATH/Contents/Embedded" -name "*.dylib" -o -name "node" -o -name "python3" -o -name "openclaw" | while read binary; do
    codesign --force --sign "$DEVELOPER_ID" \
             --options runtime \
             --timestamp \
             "$binary"
done

echo "Signing frameworks..."
# Sign frameworks
find "$APP_PATH/Contents/Frameworks" -name "*.framework" | while read framework; do
    codesign --force --sign "$DEVELOPER_ID" \
             --options runtime \
             --timestamp \
             "$framework"
done

echo "Signing main application..."
# Sign the main app bundle
codesign --force --sign "$DEVELOPER_ID" \
         --options runtime \
         --entitlements "$ENTITLEMENTS" \
         --timestamp \
         "$APP_PATH"

echo "Verifying signature..."
codesign --verify --verbose=2 "$APP_PATH"
spctl --assess --type exec "$APP_PATH"
```

### Notarization Process
```bash
#!/bin/bash
# notarize_app.sh - Notarization script

APP_PATH="OpenPaw.app"
ZIP_PATH="OpenPaw.zip"
APPLE_ID="your-apple-id@example.com"
TEAM_ID="YOUR_TEAM_ID"
APP_PASSWORD="your-app-specific-password"

echo "Creating archive for notarization..."
ditto -c -k --keepParent "$APP_PATH" "$ZIP_PATH"

echo "Submitting for notarization..."
xcrun notarytool submit "$ZIP_PATH" \
    --apple-id "$APPLE_ID" \
    --team-id "$TEAM_ID" \
    --password "$APP_PASSWORD" \
    --wait

echo "Stapling notarization ticket..."
xcrun stapler staple "$APP_PATH"

echo "Verifying notarization..."
xcrun stapler validate "$APP_PATH"
spctl --assess --type exec "$APP_PATH"
```

### Automation Integration
```yaml
# .github/workflows/sign-and-notarize.yml
name: Sign and Notarize
on:
  workflow_call:
    inputs:
      app_path:
        required: true
        type: string

jobs:
  sign_notarize:
    runs-on: macos-latest
    steps:
      - name: Import Code Signing Certificate
        env:
          CERTIFICATE_P12: ${{ secrets.CERTIFICATE_P12 }}
          CERTIFICATE_PASSWORD: ${{ secrets.CERTIFICATE_PASSWORD }}
        run: |
          echo "$CERTIFICATE_P12" | base64 --decode > certificate.p12
          security create-keychain -p "" build.keychain
          security import certificate.p12 -k build.keychain -P "$CERTIFICATE_PASSWORD" -T /usr/bin/codesign
          security set-key-partition-list -S apple-tool:,apple: -s -k "" build.keychain
          
      - name: Sign Application
        run: ./scripts/sign_app.sh "${{ inputs.app_path }}"
        
      - name: Notarize Application
        env:
          APPLE_ID: ${{ secrets.APPLE_ID }}
          TEAM_ID: ${{ secrets.TEAM_ID }}
          APP_PASSWORD: ${{ secrets.APP_SPECIFIC_PASSWORD }}
        run: ./scripts/notarize_app.sh "${{ inputs.app_path }}"
```

---

## 4. DMG Creation

### DMG Structure and Design
```bash
#!/bin/bash
# create_dmg.sh - DMG creation script

APP_NAME="OpenPaw"
APP_PATH="OpenPaw.app"
DMG_NAME="OpenPaw-v1.0.0"
DMG_SIZE="500m"  # 500MB DMG size

# Create temporary DMG directory
mkdir -p dmg_temp
cp -R "$APP_PATH" dmg_temp/

# Create Applications symlink
ln -s /Applications dmg_temp/Applications

# Add README and license files
cp README.txt dmg_temp/
cp LICENSE.txt dmg_temp/

# Create background image directory
mkdir -p dmg_temp/.background
cp dmg_background.png dmg_temp/.background/

# Create temporary DMG
hdiutil create -srcfolder dmg_temp \
               -volname "$APP_NAME" \
               -fs HFS+ \
               -fsargs "-c c=64,a=16,e=16" \
               -format UDRW \
               -size "$DMG_SIZE" \
               temp.dmg

# Mount and customize DMG
device=$(hdiutil attach -readwrite -noverify temp.dmg | egrep '^/dev/' | sed 1q | awk '{print $1}')
mount_point="/Volumes/$APP_NAME"

# Set DMG window properties
osascript <<EOD
tell application "Finder"
    tell disk "$APP_NAME"
        open
        set current view of container window to icon view
        set toolbar visible of container window to false
        set statusbar visible of container window to false
        set the bounds of container window to {400, 100, 900, 450}
        set viewOptions to the icon view options of container window
        set arrangement of viewOptions to not arranged
        set icon size of viewOptions to 72
        set background picture of viewOptions to file ".background:dmg_background.png"
        
        # Position icons
        set position of item "$APP_NAME.app" of container window to {150, 200}
        set position of item "Applications" of container window to {350, 200}
        
        close
        open
        update without registering applications
        delay 2
    end tell
end tell
EOD

# Finalize DMG
hdiutil detach "$device"
hdiutil convert temp.dmg -format UDZO -imagekey zlib-level=9 -o "$DMG_NAME.dmg"
rm -rf dmg_temp temp.dmg

echo "DMG created: $DMG_NAME.dmg"
```

### DMG Background and Branding
- **Background Image:** Custom 500x350px background with OpenPaw branding
- **Icon Positioning:** App icon and Applications folder clearly positioned
- **Instructions:** Visual cues for drag-and-drop installation
- **Accessibility:** High contrast, readable text overlay

---

## 5. Homebrew Cask Formula

### Cask Formula Structure
```ruby
# Formula: homebrew-cask/Casks/openpaw.rb
cask "openpaw" do
  version "1.0.0"
  sha256 "abc123def456...sha256_of_dmg_file"

  url "https://github.com/openpaw/openpaw/releases/download/v#{version}/OpenPaw-v#{version}.dmg",
      verified: "github.com/openpaw/openpaw/"
  name "OpenPaw"
  desc "AI-powered Mac assistant with natural language interface"
  homepage "https://openpaw.ai/"

  livecheck do
    url :url
    strategy :github_latest
  end

  auto_updates true
  depends_on macos: ">= :monterey"

  app "OpenPaw.app"

  postflight do
    # Create necessary directories
    system_command "/bin/mkdir",
                   args: ["-p", "#{ENV["HOME"]}/.openpaw/workspace"]
    
    # Set proper permissions
    system_command "/bin/chmod",
                   args: ["755", "#{ENV["HOME"]}/.openpaw"]
  end

  uninstall quit:   "com.openpaw.OpenPaw",
            delete: [
              "#{ENV["HOME"]}/.openpaw",
              "#{ENV["HOME"]}/Library/Application Support/OpenPaw",
              "#{ENV["HOME"]}/Library/Caches/com.openpaw.OpenPaw",
              "#{ENV["HOME"]}/Library/Preferences/com.openpaw.OpenPaw.plist",
              "#{ENV["HOME"]}/Library/Saved Application State/com.openpaw.OpenPaw.savedState"
            ]

  zap trash: [
    "#{ENV["HOME"]}/Library/Logs/OpenPaw",
    "#{ENV["HOME"]}/Library/WebKit/com.openpaw.OpenPaw"
  ]
end
```

### Cask Submission Process
1. **Fork homebrew-cask repository**
2. **Create formula file** in `Casks/openpaw.rb`
3. **Test formula locally:**
   ```bash
   brew cask install --verbose --debug ./openpaw.rb
   ```
4. **Submit pull request** with formula
5. **Address review feedback** from maintainers
6. **Automated testing** via Homebrew CI

### Cask Maintenance
- **Version Updates:** Automated via GitHub Actions when new releases are created
- **SHA256 Updates:** Automatically calculated and updated
- **Dependency Management:** Keep macOS version requirements current
- **Testing:** Regular testing on multiple macOS versions

---

## 6. GitHub Releases Automation

### Release Workflow
```yaml
# .github/workflows/release.yml
name: Create Release
on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: macos-latest
    outputs:
      dmg_path: ${{ steps.build.outputs.dmg_path }}
      version: ${{ steps.extract_version.outputs.version }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Xcode
        uses: maxim-lobanov/setup-xcode@v1
        with:
          xcode-version: latest-stable
          
      - name: Extract Version
        id: extract_version
        run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT
        
      - name: Build App
        id: build
        run: |
          xcodebuild -scheme OpenPaw \
                     -configuration Release \
                     -archivePath OpenPaw.xcarchive \
                     archive
          
          xcodebuild -exportArchive \
                     -archivePath OpenPaw.xcarchive \
                     -exportPath export \
                     -exportOptionsPlist ExportOptions.plist
          
          echo "dmg_path=OpenPaw-v${{ steps.extract_version.outputs.version }}.dmg" >> $GITHUB_OUTPUT
          
      - name: Sign and Notarize
        uses: ./.github/workflows/sign-and-notarize.yml
        with:
          app_path: export/OpenPaw.app
          
      - name: Create DMG
        run: ./scripts/create_dmg.sh export/OpenPaw.app
        
      - name: Upload DMG
        uses: actions/upload-artifact@v3
        with:
          name: openpaw-dmg
          path: "*.dmg"

  release:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Download DMG
        uses: actions/download-artifact@v3
        with:
          name: openpaw-dmg
          
      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          tag_name: ${{ github.ref }}
          name: OpenPaw v${{ needs.build.outputs.version }}
          body_path: CHANGELOG.md
          files: |
            *.dmg
          draft: false
          prerelease: false
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  update_homebrew:
    needs: [build, release]
    runs-on: ubuntu-latest
    steps:
      - name: Update Homebrew Cask
        uses: dawidd6/action-homebrew-bump-formula@v3
        with:
          token: ${{ secrets.HOMEBREW_TOKEN }}
          formula: openpaw
          tag: ${{ github.ref }}
```

### Release Notes Generation
```bash
#!/bin/bash
# generate_release_notes.sh
# Automatically generate release notes from commits

PREVIOUS_TAG=$(git describe --tags --abbrev=0 HEAD^)
CURRENT_TAG=$(git describe --tags --abbrev=0 HEAD)

echo "# OpenPaw v${CURRENT_TAG#v}"
echo ""
echo "## What's New"
echo ""

# Get commits since last tag
git log $PREVIOUS_TAG..HEAD --pretty=format:"- %s" --grep="feat:" | sed 's/feat: //'

echo ""
echo "## Bug Fixes"
echo ""

git log $PREVIOUS_TAG..HEAD --pretty=format:"- %s" --grep="fix:" | sed 's/fix: //'

echo ""
echo "## Improvements"
echo ""

git log $PREVIOUS_TAG..HEAD --pretty=format:"- %s" --grep="improve:" | sed 's/improve: //'

echo ""
echo "## Installation"
echo ""
echo "**Direct Download:** [OpenPaw-v${CURRENT_TAG#v}.dmg](https://github.com/openpaw/openpaw/releases/download/${CURRENT_TAG}/OpenPaw-v${CURRENT_TAG#v}.dmg)"
echo ""
echo "**Homebrew:**"
echo "\`\`\`bash"
echo "brew install --cask openpaw"
echo "\`\`\`"
echo ""
echo "## System Requirements"
echo "- macOS 12.0 (Monterey) or later"
echo "- Apple Silicon (M1/M2/M3) or Intel processor"
echo "- 4GB RAM recommended"
echo "- 2GB available disk space"
```

---

## 7. Auto-Update Mechanism (Sparkle Framework)

### Sparkle Integration
```swift
// AppDelegate.swift
import Sparkle

@main
class AppDelegate: NSObject, NSApplicationDelegate {
    var updaterController: SPUStandardUpdaterController!
    
    func applicationDidFinishLaunching(_ aNotification: Notification) {
        // Initialize Sparkle updater
        updaterController = SPUStandardUpdaterController(
            startingUpdater: true,
            updaterDelegate: self,
            userDriverDelegate: nil
        )
        
        // Check for updates on launch (if user preference allows)
        if UserDefaults.standard.bool(forKey: "checkForUpdatesOnLaunch") {
            updaterController.updater.checkForUpdatesInBackground()
        }
    }
    
    @IBAction func checkForUpdates(_ sender: Any?) {
        updaterController.checkForUpdates(sender)
    }
}

extension AppDelegate: SPUUpdaterDelegate {
    func updater(_ updater: SPUUpdater, didFinishLoading appcast: SUAppcast) {
        print("Appcast loaded successfully")
    }
    
    func updater(_ updater: SPUUpdater, failedToDownloadUpdate item: SUAppcastItem, error: Error) {
        print("Failed to download update: \(error.localizedDescription)")
        
        // Show user-friendly error message
        DispatchQueue.main.async {
            let alert = NSAlert()
            alert.messageText = "Update Download Failed"
            alert.informativeText = "Could not download the latest update. Please check your internet connection and try again."
            alert.addButton(withTitle: "OK")
            alert.runModal()
        }
    }
}
```

### Appcast Generation
```xml
<!-- appcast.xml - Auto-generated by CI/CD -->
<?xml version="1.0" encoding="utf-8"?>
<rss version="2.0" xmlns:sparkle="http://www.andymatuschak.org/xml-namespaces/sparkle" xmlns:dc="http://purl.org/dc/elements/1.1/">
    <channel>
        <title>OpenPaw Updates</title>
        <link>https://openpaw.ai/updates/appcast.xml</link>
        <description>OpenPaw Software Updates</description>
        <language>en</language>
        
        <item>
            <title>Version 1.0.0</title>
            <description><![CDATA[
                <h2>OpenPaw v1.0.0</h2>
                <h3>What's New</h3>
                <ul>
                    <li>Initial release with core AI assistant functionality</li>
                    <li>Gmail and Calendar integration</li>
                    <li>Multi-provider AI support (OpenAI, Anthropic)</li>
                    <li>Accessibility and VoiceOver support</li>
                </ul>
                <h3>System Requirements</h3>
                <ul>
                    <li>macOS 12.0 (Monterey) or later</li>
                    <li>4GB RAM recommended</li>
                    <li>2GB available disk space</li>
                </ul>
            ]]></description>
            <pubDate>Mon, 17 Feb 2026 10:00:00 +0000</pubDate>
            <sparkle:version>1.0.0</sparkle:version>
            <sparkle:shortVersionString>1.0.0</sparkle:shortVersionString>
            <sparkle:minimumSystemVersion>12.0</sparkle:minimumSystemVersion>
            <enclosure url="https://github.com/openpaw/openpaw/releases/download/v1.0.0/OpenPaw-v1.0.0.dmg"
                       sparkle:version="1.0.0"
                       sparkle:shortVersionString="1.0.0"
                       sparkle:dsaSignature="MC0CFQCxxxxxx...signature"
                       length="104857600"
                       type="application/octet-stream"/>
        </item>
    </channel>
</rss>
```

### Update Settings UI
```swift
// UpdatePreferencesViewController.swift
class UpdatePreferencesViewController: NSViewController {
    @IBOutlet weak var automaticUpdatesCheckbox: NSButton!
    @IBOutlet weak var checkOnLaunchCheckbox: NSButton!
    @IBOutlet weak var betaUpdatesCheckbox: NSButton!
    @IBOutlet weak var checkNowButton: NSButton!
    @IBOutlet weak var lastCheckLabel: NSTextField!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        loadUpdatePreferences()
        updateLastCheckTime()
    }
    
    @IBAction func automaticUpdatesChanged(_ sender: NSButton) {
        let enabled = sender.state == .on
        UserDefaults.standard.set(enabled, forKey: "SUAutomaticallyUpdate")
        
        if let updater = (NSApp.delegate as? AppDelegate)?.updaterController.updater {
            updater.automaticallyChecksForUpdates = enabled
        }
    }
    
    @IBAction func checkForUpdatesNow(_ sender: NSButton) {
        if let updater = (NSApp.delegate as? AppDelegate)?.updaterController {
            updater.checkForUpdates(sender)
        }
    }
    
    private func updateLastCheckTime() {
        if let lastCheck = UserDefaults.standard.object(forKey: "SULastCheckTime") as? Date {
            let formatter = DateFormatter()
            formatter.dateStyle = .medium
            formatter.timeStyle = .short
            lastCheckLabel.stringValue = "Last check: \(formatter.string(from: lastCheck))"
        } else {
            lastCheckLabel.stringValue = "Never checked for updates"
        }
    }
}
```

---

## 8. Minimum macOS Version and Hardware Requirements

### System Requirements Matrix

| Component | Minimum | Recommended | Notes |
|-----------|---------|-------------|-------|
| **macOS Version** | 12.0 (Monterey) | 13.0+ (Ventura+) | Uses modern security and accessibility APIs |
| **Processor** | Intel Core i5 (2017+) or Apple M1 | Apple M2/M3 or Intel i7+ | AI processing benefits from newer CPUs |
| **Memory** | 8GB RAM | 16GB+ RAM | OpenClaw engine + AI models are memory-intensive |
| **Storage** | 4GB free space | 10GB+ free space | App bundle ~500MB, logs/cache can grow |
| **Network** | Internet connection | Broadband recommended | Required for AI API calls and updates |

### Feature-Specific Requirements

#### Accessibility Features
- **VoiceOver:** macOS 12.0+ built-in VoiceOver
- **Voice Control:** macOS 13.0+ for enhanced voice commands
- **Switch Control:** macOS 12.0+ for assistive device support

#### AI Provider Requirements
- **OpenAI API:** Any supported macOS version
- **Anthropic Claude:** Any supported macOS version  
- **Local Models:** 16GB+ RAM recommended for on-device inference

#### System Permissions Required
```swift
// Required permissions check at startup
enum RequiredPermissions: String, CaseIterable {
    case accessibility = "com.apple.security.automation.apple-events"
    case screenRecording = "com.apple.security.screen-recording" 
    case microphone = "com.apple.security.device.audio-input"
    case camera = "com.apple.security.device.camera"
    
    var isRequired: Bool {
        switch self {
        case .accessibility: return true  // Critical for automation
        case .screenRecording: return false // Optional, enhances context
        case .microphone: return false // Optional, for voice input
        case .camera: return false // Optional, for visual AI
        }
    }
    
    var userDescription: String {
        switch self {
        case .accessibility:
            return "Required for OpenPaw to interact with other apps and automate tasks"
        case .screenRecording:
            return "Allows OpenPaw to see your screen for visual assistance (optional)"
        case .microphone:
            return "Enables voice input for hands-free interaction (optional)"
        case .camera:
            return "Allows visual AI features like document scanning (optional)"
        }
    }
}
```

### Hardware Optimization
```swift
// Hardware capability detection
class SystemCapabilities {
    static let shared = SystemCapabilities()
    
    lazy var processorType: ProcessorType = {
        var systemInfo = utsname()
        uname(&systemInfo)
        let machine = withUnsafePointer(to: &systemInfo.machine) {
            $0.withMemoryRebound(to: CChar.self, capacity: 1) {
                String(validatingUTF8: $0) ?? ""
            }
        }
        
        if machine.hasPrefix("arm64") {
            return .appleSilicon
        } else {
            return .intel
        }
    }()
    
    lazy var memoryGB: Int = {
        let physicalMemory = ProcessInfo.processInfo.physicalMemory
        return Int(physicalMemory / (1024 * 1024 * 1024))
    }()
    
    var recommendedAIProvider: AIProvider {
        // Apple Silicon with 16GB+ RAM can handle more complex models
        if processorType == .appleSilicon && memoryGB >= 16 {
            return .anthropicClaude // More capable model
        } else {
            return .openAIGPT35 // Faster, lighter model
        }
    }
    
    var maxConcurrentTasks: Int {
        // Scale concurrent operations based on hardware
        return max(2, min(8, memoryGB / 4))
    }
}

enum ProcessorType {
    case appleSilicon
    case intel
}
```

---

## 9. CI/CD Pipeline (GitHub Actions)

### Complete Pipeline Architecture
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  XCODE_VERSION: "15.2"
  MACOS_VERSION: "macos-14"

jobs:
  test:
    runs-on: macos-14
    strategy:
      matrix:
        xcode: ["15.1", "15.2"]
        
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Full history for version calculations
          
      - name: Setup Xcode
        uses: maxim-lobanov/setup-xcode@v1
        with:
          xcode-version: ${{ matrix.xcode }}
          
      - name: Cache Dependencies
        uses: actions/cache@v3
        with:
          path: |
            ~/Library/Developer/Xcode/DerivedData
            ~/.openclaw/cache
          key: ${{ runner.os }}-xcode-${{ matrix.xcode }}-${{ hashFiles('**/*.swift', '*.xcodeproj') }}
          
      - name: Install OpenClaw Dependencies
        run: |
          # Install Node.js for OpenClaw
          brew install node@18
          npm install -g @openclaw/cli
          
          # Install Python dependencies
          pip3 install -r requirements.txt
          
      - name: Run Unit Tests
        run: |
          xcodebuild test \
            -scheme OpenPawTests \
            -destination 'platform=macOS' \
            -resultBundlePath TestResults.xcresult
            
      - name: Run UI Tests
        run: |
          xcodebuild test \
            -scheme OpenPawUITests \
            -destination 'platform=macOS' \
            -resultBundlePath UITestResults.xcresult
            
      - name: Upload Test Results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: test-results-${{ matrix.xcode }}
          path: "*.xcresult"

  security_scan:
    runs-on: macos-14
    steps:
      - uses: actions/checkout@v4
      
      - name: Security Scan
        run: |
          # Scan for hardcoded secrets
          brew install gitleaks
          gitleaks detect --verbose
          
          # Static analysis
          xcodebuild analyze \
            -scheme OpenPaw \
            -destination 'platform=macOS'

  build:
    needs: [test, security_scan]
    runs-on: macos-14
    if: github.event_name == 'push' && (github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/tags/'))
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Build Environment
        run: |
          # Install build dependencies
          brew install create-dmg
          npm install -g appdmg
          
      - name: Calculate Version
        id: version
        run: |
          if [[ $GITHUB_REF == refs/tags/* ]]; then
            VERSION=${GITHUB_REF#refs/tags/v}
          else
            VERSION=$(date +%Y.%m.%d)-$(git rev-parse --short HEAD)
          fi
          echo "version=$VERSION" >> $GITHUB_OUTPUT
          echo "Building version: $VERSION"
          
      - name: Update Version in Project
        run: |
          # Update Info.plist with build version
          /usr/libexec/PlistBuddy -c "Set :CFBundleVersion ${{ steps.version.outputs.version }}" "OpenPaw/Info.plist"
          /usr/libexec/PlistBuddy -c "Set :CFBundleShortVersionString ${{ steps.version.outputs.version }}" "OpenPaw/Info.plist"
          
      - name: Build Dependencies
        run: |
          # Build embedded Node.js runtime
          ./scripts/build_node_runtime.sh
          
          # Build OpenClaw engine
          ./scripts/build_openclaw_engine.sh
          
          # Prepare embedded Python
          ./scripts/build_python_runtime.sh
          
      - name: Build Application
        run: |
          xcodebuild archive \
            -scheme OpenPaw \
            -configuration Release \
            -archivePath "OpenPaw.xcarchive" \
            -allowProvisioningUpdates
            
          xcodebuild -exportArchive \
            -archivePath "OpenPaw.xcarchive" \
            -exportPath "export" \
            -exportOptionsPlist "ExportOptions.plist"
            
      - name: Sign and Notarize
        env:
          CERTIFICATE_P12: ${{ secrets.DEVELOPER_ID_CERTIFICATE }}
          CERTIFICATE_PASSWORD: ${{ secrets.CERTIFICATE_PASSWORD }}
          APPLE_ID: ${{ secrets.APPLE_ID }}
          APP_PASSWORD: ${{ secrets.APP_SPECIFIC_PASSWORD }}
          TEAM_ID: ${{ secrets.TEAM_ID }}
        run: |
          # Import certificate
          echo "$CERTIFICATE_P12" | base64 --decode > certificate.p12
          security create-keychain -p "" build.keychain
          security default-keychain -s build.keychain
          security unlock-keychain -p "" build.keychain
          security import certificate.p12 -k build.keychain -P "$CERTIFICATE_PASSWORD" -T /usr/bin/codesign
          security set-key-partition-list -S apple-tool:,apple: -s -k "" build.keychain
          
          # Sign and notarize
          ./scripts/sign_app.sh "export/OpenPaw.app"
          ./scripts/notarize_app.sh "export/OpenPaw.app"
          
      - name: Create DMG
        run: |
          ./scripts/create_dmg.sh "export/OpenPaw.app" "${{ steps.version.outputs.version }}"
          
      - name: Upload Build Artifacts
        uses: actions/upload-artifact@v3
        with:
          name: openpaw-build-${{ steps.version.outputs.version }}
          path: |
            *.dmg
            export/OpenPaw.app

  release:
    needs: build
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/')
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download Build Artifacts
        uses: actions/download-artifact@v3
        with:
          pattern: openpaw-build-*
          merge-multiple: true
          
      - name: Generate Release Notes
        run: |
          ./scripts/generate_release_notes.sh > RELEASE_NOTES.md
          
      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          name: OpenPaw ${{ github.ref_name }}
          body_path: RELEASE_NOTES.md
          files: "*.dmg"
          draft: false
          prerelease: ${{ contains(github.ref, '-') }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          
      - name: Update Appcast
        run: |
          ./scripts/update_appcast.sh "${{ github.ref_name }}" "*.dmg"
          
      - name: Deploy Appcast
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./appcast
          destination_dir: updates

  homebrew:
    needs: release
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/') && !contains(github.ref, '-')
    
    steps:
      - name: Update Homebrew Cask
        uses: dawidd6/action-homebrew-bump-formula@v3
        with:
          token: ${{ secrets.HOMEBREW_GITHUB_TOKEN }}
          formula: openpaw
          tag: ${{ github.ref_name }}
          revision: ${{ github.sha }}
          
  metrics:
    needs: [test, build, release]
    runs-on: ubuntu-latest
    if: always()
    
    steps:
      - name: Report Build Metrics
        run: |
          # Report build success/failure to monitoring
          curl -X POST "${{ secrets.WEBHOOK_URL }}" \
            -H "Content-Type: application/json" \
            -d '{
              "pipeline": "openpaw-cicd",
              "version": "${{ github.ref_name }}",
              "status": "${{ needs.build.result }}",
              "tests_passed": "${{ needs.test.result == 'success' }}",
              "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
            }'
```

### Pipeline Monitoring and Alerts
```yaml
# .github/workflows/pipeline-health.yml
name: Pipeline Health Check
on:
  schedule:
    - cron: '0 9 * * MON' # Weekly on Monday 9 AM UTC
  workflow_dispatch:

jobs:
  health_check:
    runs-on: ubuntu-latest
    steps:
      - name: Check Recent Pipeline Success
        run: |
          # Query GitHub API for recent workflow runs
          RECENT_RUNS=$(gh api repos/${{ github.repository }}/actions/runs \
            --jq '.workflow_runs[0:10] | map(select(.name == "CI/CD Pipeline")) | .[].conclusion' \
            | tr '\n' ',' | sed 's/,$//')
            
          FAILURE_COUNT=$(echo "$RECENT_RUNS" | tr ',' '\n' | grep -c "failure" || true)
          
          if [ "$FAILURE_COUNT" -gt 3 ]; then
            echo "⚠️ Pipeline health degraded: $FAILURE_COUNT failures in recent runs"
            # Send alert to team
            curl -X POST "${{ secrets.SLACK_WEBHOOK }}" \
              -d '{"text": "OpenPaw CI/CD pipeline health alert: Multiple recent failures detected"}'
          fi
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 10. Versioning Strategy

### Semantic Versioning
OpenPaw follows [Semantic Versioning 2.0.0](https://semver.org/):

**Format:** `MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]`

- **MAJOR:** Breaking changes, API incompatibilities
- **MINOR:** New features, backward-compatible additions  
- **PATCH:** Bug fixes, backward-compatible changes
- **PRERELEASE:** alpha, beta, rc (release candidate)
- **BUILD:** Build metadata (commit hash, build date)

### Version Examples
- `1.0.0` - Initial stable release
- `1.1.0` - New features added
- `1.1.1` - Bug fixes only
- `1.2.0-beta.1` - Beta release with new features
- `2.0.0` - Major release with breaking changes
- `1.1.0+20260217.abc123` - Build with metadata

### Automated Version Management
```bash
#!/bin/bash
# scripts/calculate_version.sh
# Automatically calculate version based on git history

set -e

# Get the latest tag
LATEST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")
LATEST_VERSION=${LATEST_TAG#v}

# Parse version components
IFS='.' read -ra VERSION_PARTS <<< "$LATEST_VERSION"
MAJOR=${VERSION_PARTS[0]}
MINOR=${VERSION_PARTS[1]}
PATCH=${VERSION_PARTS[2]}

# Check commit messages since last tag for version bump type
COMMITS_SINCE_TAG=$(git log $LATEST_TAG..HEAD --pretty=format:"%s" 2>/dev/null || git log --pretty=format:"%s")

# Determine version bump based on conventional commits
BUMP_TYPE="patch"  # Default to patch

if echo "$COMMITS_SINCE_TAG" | grep -q "^feat!:" || echo "$COMMITS_SINCE_TAG" | grep -q "BREAKING CHANGE:"; then
    BUMP_TYPE="major"
elif echo "$COMMITS_SINCE_TAG" | grep -q "^feat:"; then
    BUMP_TYPE="minor"
fi

# Calculate new version
case $BUMP_TYPE in
    major)
        NEW_VERSION="$((MAJOR + 1)).0.0"
        ;;
    minor)  
        NEW_VERSION="$MAJOR.$((MINOR + 1)).0"
        ;;
    patch)
        NEW_VERSION="$MAJOR.$MINOR.$((PATCH + 1))"
        ;;
esac

# Add prerelease suffix for non-main branches
if [[ "$GITHUB_REF" != "refs/heads/main" && "$GITHUB_REF" != refs/tags/* ]]; then
    BRANCH_NAME=${GITHUB_REF#refs/heads/}
    COMMIT_SHORT=$(git rev-parse --short HEAD)
    NEW_VERSION="$NEW_VERSION-${BRANCH_NAME}.${COMMIT_SHORT}"
fi

echo "$NEW_VERSION"
```

### Version Consistency Across Components
```swift
// Version.swift - Single source of truth for version info
struct AppVersion {
    static let major = 1
    static let minor = 0
    static let patch = 0
    static let prerelease: String? = nil
    static let build = Bundle.main.infoDictionary?["CFBundleVersion"] as? String
    
    static var full: String {
        var version = "\(major).\(minor).\(patch)"
        if let prerelease = prerelease {
            version += "-\(prerelease)"
        }
        if let build = build {
            version += "+\(build)"
        }
        return version
    }
    
    static var marketing: String {
        return "\(major).\(minor).\(patch)"
    }
    
    // Compare versions for update checks
    static func isNewerThan(_ otherVersion: String) -> Bool {
        // Implementation of version comparison logic
        return full.compare(otherVersion, options: .numeric) == .orderedDescending
    }
}
```

---

## Quality Assurance and Testing

### Build Quality Gates
Every release must pass:

1. **Unit Tests:** 100% pass rate, minimum 80% code coverage
2. **UI Tests:** All critical user flows automated and passing
3. **Security Scan:** No high/critical vulnerabilities
4. **Performance Tests:** Launch time <5s, memory usage <200MB idle
5. **Accessibility Tests:** VoiceOver navigation and screen reader compatibility
6. **Code Signing:** Valid signature and successful notarization

### Testing Matrix
| Test Type | Frequency | Automation | Coverage |
|-----------|-----------|------------|----------|
| Unit Tests | Every commit | Full | 80%+ code coverage |
| Integration Tests | Every PR | Full | API integrations |
| UI Tests | Daily | Full | Critical user paths |
| Performance Tests | Weekly | Partial | Memory, CPU, startup |
| Security Tests | Every release | Full | Vulnerability scanning |
| Compatibility Tests | Every release | Manual | Multiple macOS versions |

### Release Criteria Checklist
- [ ] All automated tests passing
- [ ] Security scan clean (no high/critical issues)
- [ ] Performance benchmarks within acceptable ranges
- [ ] Accessibility compliance verified
- [ ] Documentation updated
- [ ] Release notes prepared
- [ ] Code signed and notarized successfully
- [ ] DMG created and tested on clean system
- [ ] Homebrew cask formula prepared
- [ ] Rollback plan documented

This comprehensive packaging and distribution specification ensures OpenPaw can be reliably built, signed, and distributed to users through multiple channels while maintaining high quality and security standards.