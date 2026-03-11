---
name: "environment-setup-safari"
description: "Set up and validate a Safari Appium environment on macOS and iOS"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:25:00 GMT"

---
# appium-safari-environment-setup

## Goal
Prepares a reliable Appium Safari execution environment by installing Node.js and Appium prerequisites, enabling safaridriver, configuring iOS device/simulator settings, running Appium doctor checks, and iterating until all mandatory checks pass.

## Decision Logic
- If the host OS is not macOS: stop and ask the user to use a supported macOS system.
- If current Node.js does not satisfy `engines.node` for both `appium` and `appium-safari-driver`: install/upgrade Node.js to a compatible active LTS version.
- If Appium CLI is not installed: install `appium` globally.
- Use global npm/Appium commands by default (`npm install -g appium`, `appium ...`).
- Use local Appium commands (`npx appium ...`) only when the user explicitly requests local execution.
- If safaridriver is not enabled: run `safaridriver --enable` with administrator privileges.
- If testing on iOS Simulator: verify Xcode installation and iOS 13+ SDK availability.
- If testing on iOS Real Device: verify Remote Automation is enabled in device settings.
- If the `safari` driver is not installed: install it via Appium CLI.
- If install returns "already installed", ignore the error and continue (or run driver update).
- If `appium driver doctor safari` reports missing dependencies: resolve each missing item and re-run doctor.

## Instructions
1. **Prepare Node.js + npm environment on macOS**
   ```bash
   sw_vers
   node -v
   npm -v
   ```
   If `node` is missing, install a compatible active LTS release and re-run the commands.

2. **Install Appium npm command and Safari driver**
   ```bash
   npm install -g appium
   appium driver install safari || appium driver update safari
   appium driver list --installed
   ```
   If the install command fails only because `safari` is already installed, continue and do not stop preparation.

3. **Validate Appium npm commands and Node compatibility (after driver setup)**
   ```bash
   appium -v
   appium driver list --installed
   npm view appium engines --json
   npm view appium-safari-driver engines --json
   ```
   If current Node.js does not satisfy the reported `engines.node` ranges, install/upgrade Node.js to a compatible active LTS version and re-run the setup checks.

4. **Enable safaridriver (requires administrator password)**
   This is a mandatory one-time setup step:
   ```bash
   safaridriver --enable
   ```
   You will be prompted for your administrator password. This command must succeed before continuing.
   
   Verify safaridriver is available:
   ```bash
   safaridriver --version
   ```

5. **Platform-specific configuration**

   **For macOS Safari (desktop browser):**
   - No additional configuration required
   - Safari version must be compatible with safaridriver
   - Verify Safari is installed:
   ```bash
   /Applications/Safari.app/Contents/MacOS/Safari --version || \
   system_profiler SPApplicationsDataType | grep -A 3 "Safari:"
   ```

   **For iOS Simulator (requires Xcode):**
   - Verify Xcode installation:
   ```bash
   xcodebuild -version
   xcode-select -p
   ```
   - List available iOS Simulators (iOS 13+):
   ```bash
   xcrun simctl list devices | grep -E "iOS 1[3-9]|iOS [2-9][0-9]"
   ```
   - Verify iOS 13+ runtime availability:
   ```bash
   xcrun simctl list runtimes | grep iOS
   ```

   **For iOS Real Device:**
   - Connect the iOS device via USB
   - Verify device is recognized:
   ```bash
   system_profiler SPUSBDataType | grep -A 10 "iPhone\|iPad"
   ```
   - **Important**: On the iOS device, manually enable Remote Automation:
     1. Open Settings app
     2. Navigate to Safari → Advanced
     3. Enable "Remote Automation" switch
   - Verify device UDID (if needed):
   ```bash
   system_profiler SPUSBDataType | grep "Serial Number" | awk '{print $3}'
   ```

6. **Run Appium doctor for Safari and fix in a loop**
   ```bash
   appium driver doctor safari
   ```
   If doctor reports issues, apply targeted fixes and re-run until mandatory checks pass.

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
   Then confirm startup/readiness from server logs and ensure the `Available drivers:` block contains `safari` (for example: `- safari@4.1.9 (automationName 'Safari')`).
   After smoke validation, clean up the running Appium server:
   - In Terminal A, stop the server with `Ctrl+C`.
   - Verify no leftover Appium server process (Terminal B):
   ```bash
   pgrep -fl "appium.*server" || echo "no appium server process"
   ```

8. **Agent completion criteria**
   Mark the skill complete only when all are true:
   - `appium driver list --installed` includes `safari`
   - `appium -v` succeeds
   - `safaridriver --enable` has been executed successfully
   - `appium driver doctor safari` has no failing mandatory checks
   - For iOS Simulator testing: Xcode is installed and iOS 13+ simulators are available
   - For iOS Real Device testing: Remote Automation is enabled on the device
   - `curl -s http://127.0.0.1:4723/status` returns a successful status response
   - Appium server logs show startup/readiness successfully after the curl check
   - Appium server logs include `Available drivers:` with a `safari` entry
   - Appium smoke-test server process is cleanly stopped after validation

## Constraints
- This skill is macOS-only; do not provide Linux/Windows alternatives.
- Use global npm/Appium commands as the default execution mode.
- Use `npx appium` only if the user explicitly asks for local execution.
- Always run `appium driver doctor safari` after each environment change.
- Treat optional doctor warnings as non-blocking.
- Ask the user before installing optional dependencies, and install them only when explicitly needed.
- `safaridriver --enable` requires administrator privileges and must be run manually by the user.
- For iOS Real Device testing, Remote Automation must be manually enabled on each device.
- Safari driver supports macOS High Sierra or newer, iOS 13 or newer.
- Use shell-appropriate commands (bash for macOS).
- Prefer deterministic CLI checks over assumptions.
- If elevated privileges are required, pause and provide exact commands for the user to run.
- Do not claim success until doctor and smoke-test checks are actually green.