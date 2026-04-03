# WebRTC iOS — Build from Source Guide

A step-by-step guide to download, compile, and package Google WebRTC for iOS as a Swift Package (XCFramework).

> **Branches:** https://chromiumdash.appspot.com/branches


> **Target:** iOS device (arm64) + Simulator | **Output:** `WebRTC.xcframework`

---

## Prerequisites

- macOS with Xcode installed (Xcode 15 recommended for best compatibility)
- Python 3
- Git
- ~10GB free disk space (WebRTC source is ~6GB)

---

## Step 1 — Install depot_tools

Google uses `depot_tools` to manage WebRTC and Chromium repositories.

```bash
mkdir webrtc-builds && cd webrtc-builds
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH="`pwd`/depot_tools:$PATH"
```

### Make PATH permanent

```bash
echo 'export PATH="/Users/<your-username>/webrtc-builds/depot_tools:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

---

## Step 2 — Fetch WebRTC Source

```bash
fetch --nohooks webrtc_ios
cd src
```

> This fetches the full WebRTC source with iOS-specific parts. Expect ~6GB download and several minutes.

---

## Step 3 — Checkout a Specific Version

Find your target branch from the [Chromium Dash](https://chromiumdash.appspot.com/branches) by matching the Chrome milestone to a branch number.

```bash
# Example: Milestone 146 = branch-heads/7680
git checkout -b my-build branch-heads/7680
gclient sync
```

> `gclient sync` pulls all dependencies for that branch. This can take 10–20 minutes.

---

## Step 4 — Fix Missing Modulemap Files (Xcode 16 Only)

If you're on **Xcode 16.x**, Apple restructured the SDK and removed modulemap files that WebRTC expects. Create them manually:

```bash
# iOS Device SDK
sudo touch /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS18.2.sdk/usr/include/DarwinFoundation1.modulemap
sudo touch /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS18.2.sdk/usr/include/DarwinFoundation2.modulemap
sudo touch /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS18.2.sdk/usr/include/DarwinFoundation3.modulemap

# iOS Simulator SDK
sudo touch /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator18.2.sdk/usr/include/DarwinFoundation1.modulemap
sudo touch /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator18.2.sdk/usr/include/DarwinFoundation2.modulemap
sudo touch /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator18.2.sdk/usr/include/DarwinFoundation3.modulemap

# macOS SDK
sudo touch /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX15.2.sdk/usr/include/DarwinFoundation1.modulemap
sudo touch /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX15.2.sdk/usr/include/DarwinFoundation2.modulemap
sudo touch /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX15.2.sdk/usr/include/DarwinFoundation3.modulemap
```

Then write unique module content into each file (empty files cause redefinition errors):

```bash
# iOS Device SDK
echo 'module _DarwinFoundation1 [system] {}' | sudo tee /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS18.2.sdk/usr/include/DarwinFoundation1.modulemap
echo 'module _DarwinFoundation2 [system] {}' | sudo tee /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS18.2.sdk/usr/include/DarwinFoundation2.modulemap
echo 'module _DarwinFoundation3 [system] {}' | sudo tee /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS18.2.sdk/usr/include/DarwinFoundation3.modulemap

# iOS Simulator SDK
echo 'module _DarwinFoundation1 [system] {}' | sudo tee /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator18.2.sdk/usr/include/DarwinFoundation1.modulemap
echo 'module _DarwinFoundation2 [system] {}' | sudo tee /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator18.2.sdk/usr/include/DarwinFoundation2.modulemap
echo 'module _DarwinFoundation3 [system] {}' | sudo tee /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator18.2.sdk/usr/include/DarwinFoundation3.modulemap

# macOS SDK
echo 'module _DarwinFoundation1 [system] {}' | sudo tee /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX15.2.sdk/usr/include/DarwinFoundation1.modulemap
echo 'module _DarwinFoundation2 [system] {}' | sudo tee /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX15.2.sdk/usr/include/DarwinFoundation2.modulemap
echo 'module _DarwinFoundation3 [system] {}' | sudo tee /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX15.2.sdk/usr/include/DarwinFoundation3.modulemap
```

---

## Step 5 — Build using Official Python Script (Recommended)

```bash
cd src

python3 tools_webrtc/ios/build_ios_libs.py \
  --output-dir out/ios_libs \
  --arch device:arm64 simulator:arm64 simulator:x64 \
  --build_config release
```

> This takes **20–40 minutes**. The output `WebRTC.xcframework` will be at `out/ios_libs/`.

### Alternative — Build Manually with GN + Ninja

```bash
# Device (arm64)
gn gen out/ios_arm64 --args='
  target_os="ios"
  target_cpu="arm64"
  target_environment="device"
  ios_enable_code_signing=false
  is_debug=false
  rtc_include_tests=false
  enable_dsyms=false
  enable_stripping=true
'
ninja -C out/ios_arm64 framework_objc

# Simulator (arm64)
gn gen out/ios_sim --args='
  target_os="ios"
  target_cpu="arm64"
  target_environment="simulator"
  ios_enable_code_signing=false
  is_debug=false
  rtc_include_tests=false
  enable_dsyms=false
  enable_stripping=true
'
ninja -C out/ios_sim framework_objc
```

---

## Step 6 — Package as XCFramework (Manual build only)

If you used the manual GN + Ninja path, combine the outputs:

```bash
xcodebuild -create-xcframework \
  -framework out/ios_arm64/WebRTC.framework \
  -framework out/ios_sim/WebRTC.framework \
  -output out/WebRTC.xcframework
```

---

## Step 7 — Create the Swift Package

Create a folder structure:

```
WebRTC/
├── Package.swift
└── WebRTC.xcframework/   ← copy from build output
```

Copy the framework:

```bash
mkdir -p ~/Desktop/Projects/WebRTC
cp -r out/ios_libs/WebRTC.xcframework ~/Desktop/Projects/WebRTC/
```

Create `Package.swift` — **the `swift-tools-version` line MUST be the very first line:**

```swift
// swift-tools-version:5.3
import PackageDescription

let package = Package(
    name: "WebRTC",
    platforms: [.iOS(.v13)],
    products: [
        .library(name: "WebRTC", targets: ["WebRTC"])
    ],
    targets: [
        .binaryTarget(
            name: "WebRTC",
            path: "WebRTC.xcframework"
        )
    ]
)
```

---

## Step 8 — Add to Your iOS Project

### Via Xcode UI

1. Open your Xcode project
2. **File → Add Package Dependencies**
3. Click **Add Local...** and select the `WebRTC` folder

### Via `Package.swift` (remote, after pushing to GitHub)

```swift
dependencies: [
    .package(url: "https://github.com/AshYadav1/WebRTC.git", exact: "146.0.0")
]
```

### Import in Swift

```swift
import WebRTC
```

---

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `gclient: command not found` | depot_tools not in PATH | Re-export: `export PATH="/path/to/depot_tools:$PATH"` |
| `target_environment must be in [...]` | Missing env arg in gn | Add `target_environment="device"` or `"simulator"` |
| `DarwinFoundationX.modulemap missing` | Xcode 16 SDK restructure | Follow Step 4 above |
| `redefinition of module` | Duplicate module names in modulemap | Use unique names: `_DarwinFoundation1`, `_DarwinFoundation2`, etc. |
| `tools-version backward-incompatible` | `swift-tools-version` not on line 1 | Move it to the absolute first line of `Package.swift` |

---

## Branch → Milestone Reference

| Chrome Milestone | Branch | Notes |
|-----------------|--------|-------|
| M146 | branch-heads/7680 | Latest (Xcode 16 issues) |
| M126 | branch-heads/6478 | Stable |
| M120 | branch-heads/6099 | Good Xcode 15/16 compat |

Use [chromiumdash.appspot.com/branches](https://chromiumdash.appspot.com/branches) to find any branch.
