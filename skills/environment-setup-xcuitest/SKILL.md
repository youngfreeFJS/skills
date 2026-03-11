---
name: "environment-setup-xcuitest"
description: "Set up and validate an XCUITest Appium environment on macOS"
metadata:
  last_modified: "Tue, 11 Mar 2026 19:35:00 GMT"

---
# appium-xcuitest-environment-setup

## Goal
Prepares a stable Appium XCUITest execution environment on macOS by validating Node.js and Appium installation, installing and validating Xcode toolchains, and iterating Appium doctor checks until required items pass.

## Decision Logic
- If host OS is not macOS: stop and tell the user this skill only supports macOS.
- If Xcode is missing or command line tools are unconfigured: install/configure them before continuing.
- If current Node.js does not satisfy `engines.node` for both `appium` and `appium-xcuitest-driver`: install/upgrade Node.js to a compatible active LTS version.
- If Appium CLI or `xcuitest` driver is missing: install them via Appium CLI.
- Use global npm/Appium commands by default (`npm install -g appium`, `appium ...`).
- Use local Appium commands (`npx appium ...`) only when the user explicitly requests local execution.
- If the user explicitly requests media features that require FFmpeg: run `environment-setup-ffmpeg` before final validation.
- If install returns "already installed", ignore the error and continue (or run driver update).
- If `appium driver doctor xcuitest` reports missing dependencies: fix each reported dependency and re-run doctor.
- If Xcode 15+ is detected: verify iOS 17+ SDK availability and apply Xcode 15+ specific configurations.
- If host is Apple Silicon (ARM64): enable Rosetta 2 compatibility and use native ARM64 toolchain for optimal performance.
- If testing iOS 17+ apps: verify Privacy Manifest requirements and new permission configurations.

## Instructions
1. **Prepare Node.js + npm environment on macOS**
   ```bash
   sw_vers
   node -v
   npm -v
   ```
   If `node` is missing, install a compatible active LTS release and re-run the commands.

2. **Install Appium npm command and XCUITest driver**
   ```bash
   npm install -g appium
   appium driver install xcuitest || appium driver update xcuitest
   appium driver list --installed
   ```
   If the install command fails only because `xcuitest` is already installed, continue and do not stop preparation.

3. **Validate Appium npm commands and Node compatibility (after driver setup)**
   ```bash
   appium -v
   appium driver list --installed
   npm view appium engines --json
   npm view appium-xcuitest-driver engines --json
   ```
   If current Node.js does not satisfy the reported `engines.node` ranges, install/upgrade Node.js to a compatible active LTS version and re-run the setup checks.

4. **Verify Xcode command line setup and license**
   First validate Xcode availability:
   ```bash
   xcodebuild -version
   ```
   If version info is returned, continue without changing `xcode-select`.
   If version info is not returned, check the selected developer path:
   ```bash
   xcode-select -p
   ```
   If the selected path does not contain `Contents/Developer`, try setting it to the default Xcode app path:
   ```bash
   xcode-select --switch /Applications/Xcode.app/Contents/Developer
   ```
   Re-validate:
   ```bash
   xcodebuild -version
   xcode-select -p
   ```
   Then ensure license and first-launch tasks are complete (use privilege escalation only if prompted/required):
   ```bash
   sudo xcodebuild -license accept
   xcodebuild -runFirstLaunch
   ```
   
   **Xcode 15+ specific configuration**:
   ```bash
   # Detect Xcode version
   XCODE_VERSION=$(xcodebuild -version | head -n 1 | awk '{print $2}')
   XCODE_MAJOR=$(echo $XCODE_VERSION | cut -d. -f1)
   
   if [ "$XCODE_MAJOR" -ge 15 ]; then
     echo "Detected Xcode 15+, applying iOS 17+ configurations..."
     
     # Verify iOS 17 SDK availability
     xcodebuild -showsdks | grep "iOS 17"
     
     # List available iOS simulators
     xcrun simctl list runtimes | grep iOS
     
     # Verify minimum macOS version (Xcode 15 requires macOS 13.5+)
     sw_vers
   fi
   ```

4.5. **Apple Silicon optimization configuration**
   On Apple Silicon (M1/M2/M3) Macs, optimize for native performance:
   ```bash
   # Detect Apple Silicon architecture
   if [ "$(uname -m)" = "arm64" ]; then
     echo "Apple Silicon detected, enabling native ARM64 toolchain..."
     
     # Verify Rosetta 2 installation (for x86_64 compatibility if needed)
     if ! /usr/bin/pgrep oahd >/dev/null 2>&1; then
       echo "Installing Rosetta 2 for x86_64 compatibility..."
       softwareupdate --install-rosetta --agree-to-license
     fi
     
     # Verify native ARM64 Xcode toolchain
     xcode-select --print-path
     arch -arm64 xcodebuild -version
     
     # Check available architectures
     lipo -info /usr/bin/xcodebuild
   fi
   ```
   
   Apple Silicon performance tips:
   - iOS Simulator runs natively on ARM64 with excellent performance
   - Use native ARM64 builds for all development tools
   - Rosetta 2 is only needed for legacy x86_64 dependencies
   - Native ARM64 compilation is significantly faster than Rosetta translation

5. **Optional helper tools**
   Install additional iOS helper tools only if the user explicitly requests capabilities that require them.
   - For FFmpeg-related capabilities, run `environment-setup-ffmpeg`.

6. **Run Appium doctor for XCUITest and fix in a loop**
   ```bash
   appium driver doctor xcuitest
   ```
   Resolve each failing mandatory check, then re-run until the doctor output is clean.

6.5. **iOS 17+ specific checks**
   For iOS 17+ testing, verify additional requirements:
   ```bash
   # Check iOS 17+ SDK availability
   xcodebuild -showsdks | grep -E "iOS 17|iOS 18"
   
   # List iOS 17+ simulators
   xcrun simctl list devices | grep -E "iOS 17|iOS 18"
   
   # Verify Privacy Manifest support (required for iOS 17+)
   # Privacy Manifest (PrivacyInfo.xcprivacy) is required for apps using certain APIs
   # Check if your app includes PrivacyInfo.xcprivacy in the bundle
   
   # iOS 17+ introduces new privacy requirements:
   # - Required Reason API usage must be declared
   # - Third-party SDK signatures required
   # - Enhanced privacy manifests for tracking domains
   ```
   
   Note: iOS 17+ Privacy Manifest requirements:
   - Apps using certain APIs (e.g., file timestamps, system boot time, disk space) must declare usage reasons
   - Privacy-impacting SDKs must include signatures
   - Tracking domains must be declared in privacy manifests
   - See: https://developer.apple.com/documentation/bundleresources/privacy_manifest_files

7. **Start Appium server smoke test**
   ```bash
   appium server
   ```
   Keep this server process running in Terminal A.
   In Terminal B, run:
   ```bash
   curl -s http://127.0.0.1:4723/status
   ```
   First confirm `/status` responds successfully from `curl`.
   Then confirm readiness from server logs and ensure the `Available drivers:` block contains `xcuitest` (for example: `- xcuitest@10.23.3 (automationName 'XCUITest')`).
   After smoke validation, clean up the running Appium server:
   - In Terminal A, stop the server with `Ctrl+C`.
   - Verify no leftover Appium server process (Terminal B):
   ```bash
   pgrep -fl "appium.*server" || echo "no appium server process"
   ```

8. **Agent completion criteria**
   Mark the skill complete only when all are true:
   - `appium driver list --installed` includes `xcuitest`
   - `appium -v` succeeds
   - `appium driver doctor xcuitest` has no failing mandatory checks
   - `curl -s http://127.0.0.1:4723/status` returns a successful status response
   - Appium server logs show startup/readiness successfully after the curl check
   - Appium server logs include `Available drivers:` with an `xcuitest` entry
   - Appium smoke-test server process is cleanly stopped after validation

## Constraints
- This skill is macOS-only; do not provide Linux/Windows alternatives.
- Use global npm/Appium commands as the default execution mode.
- Use `npx appium` only if the user explicitly asks for local execution.
- Always re-run `appium driver doctor xcuitest` after every fix.
- Treat optional doctor warnings as non-blocking.
- Ask the user before installing optional dependencies, and install them only when explicitly needed.
- Do not skip Xcode license and first-launch checks.
- If privileged commands are required, pause and provide the exact command for user execution.
- Do not report readiness until doctor and smoke tests pass.
- **Xcode 15+ Requirements**: Xcode 15 requires macOS 13.5 (Ventura) or later. Verify system compatibility before installation.
- **iOS 17+ Privacy Manifest**: Apps targeting iOS 17+ must include Privacy Manifest (PrivacyInfo.xcprivacy) for certain API usage. Declare Required Reason APIs and tracking domains as needed.
- **Apple Silicon Optimization**: On M1/M2/M3 Macs, iOS Simulator runs natively with superior performance. Use native ARM64 toolchain (`arch -arm64`) for all development tasks. Rosetta 2 is only needed for legacy x86_64 dependencies.
