---
name: "environment-setup-mac2"
description: "Set up and validate a Mac2 Appium environment for macOS application automation"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:25:00 GMT"

---
# appium-mac2-environment-setup

## Goal
Prepares a reliable Appium Mac2 execution environment by installing Node.js and Appium prerequisites, configuring Xcode and accessibility permissions, running Appium doctor checks, and iterating until all mandatory checks pass.

## Decision Logic
- If the host OS is not macOS: stop and ask the user to use a supported macOS system.
- If macOS version is below 11: stop and ask the user to upgrade to macOS 11 or later.
- If current Node.js does not satisfy `engines.node` for both `appium` and `appium-mac2-driver`: install/upgrade Node.js to a compatible active LTS version.
- If Appium CLI is not installed: install `appium` globally.
- Use global npm/Appium commands by default (`npm install -g appium`, `appium ...`).
- Use local Appium commands (`npx appium ...`) only when the user explicitly requests local execution.
- If Xcode is not installed or version is below 13: install/upgrade Xcode before continuing.
- If xcode-select is not pointing to Xcode.app/Contents/Developer: configure it correctly.
- If Xcode Helper app lacks Accessibility access: guide user to enable it manually.
- If testmanagerd requires UIAutomation authentication (macOS 12+): provide guidance on disabling it for CI environments.
- If the `mac2` driver is not installed: install it via Appium CLI.
- If install returns "already installed", ignore the error and continue (or run driver update).
- If `appium driver doctor mac2` reports missing dependencies: resolve each missing item and re-run doctor.

## Instructions
1. **Verify macOS version compatibility**
   ```bash
   sw_vers
   ```
   Ensure macOS version is 11 (Big Sur) or later. If not, stop and ask the user to upgrade.

2. **Prepare Node.js + npm environment on macOS**
   ```bash
   node -v
   npm -v
   ```
   If `node` is missing, install a compatible active LTS release and re-run the commands.

3. **Install Appium npm command and Mac2 driver**
   ```bash
   npm install -g appium
   appium driver install mac2 || appium driver update mac2
   appium driver list --installed
   ```
   If the install command fails only because `mac2` is already installed, continue and do not stop preparation.

4. **Validate Appium npm commands and Node compatibility (after driver setup)**
   ```bash
   appium -v
   appium driver list --installed
   npm view appium engines --json
   npm view appium-mac2-driver engines --json
   ```
   If current Node.js does not satisfy the reported `engines.node` ranges, install/upgrade Node.js to a compatible active LTS version and re-run the setup checks.

5. **Verify Xcode installation and configuration**
   First validate Xcode availability (version 13 or later required):
   ```bash
   xcodebuild -version
   ```
   If Xcode is not installed or version is below 13, install/upgrade Xcode from the App Store.
   
   Check the selected developer path:
   ```bash
   xcode-select -p
   ```
   The path should be `/Applications/Xcode.app/Contents/Developer` (not `/Library/Developer/CommandLineTools`).
   
   If the path is incorrect, set it correctly:
   ```bash
   sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
   ```
   
   Re-validate:
   ```bash
   xcodebuild -version
   xcode-select -p
   ```
   
   Ensure Xcode license and first-launch tasks are complete:
   ```bash
   sudo xcodebuild -license accept
   xcodebuild -runFirstLaunch
   ```

6. **Enable Xcode Helper Accessibility access (manual step)**
   This is a mandatory one-time setup that requires manual user action:
   
   1. Open the Xcode Helper app location in Finder:
   ```bash
   open /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/Library/Xcode/Agents/
   ```
   
   2. Manually drag and drop "Xcode Helper.app" to:
      - System Preferences → Security & Privacy → Privacy → Accessibility
   
   3. Ensure the checkbox next to "Xcode Helper" is enabled
   
   This action must only be done once per system.

7. **Optional: Disable UIAutomation authentication for CI (macOS 12+)**
   If running on macOS 12 or later in a CI environment, you may need to disable UIAutomation authentication:
   ```bash
   # Check if automationmodetool is available
   which automationmodetool
   
   # Disable authentication (requires administrator privileges)
   sudo automationmodetool enable-automationmode-without-authentication
   ```
   This is particularly useful in CI environments. See [Apple forum thread](https://developer.apple.com/forums/thread/686810) for more details.

8. **Run Appium doctor for Mac2 and fix in a loop**
   ```bash
   appium driver doctor mac2
   ```
   If doctor reports issues, apply targeted fixes and re-run until mandatory checks pass.
   
   The doctor command validates:
   - Xcode installation and version
   - xcode-select configuration
   - Accessibility permissions
   - WebDriverAgentMac build requirements
   - Other optional dependencies

9. **Optional: Open WebDriverAgentMac in Xcode**
   If you need to configure code signing or inspect the WebDriverAgentMac project:
   ```bash
   appium driver run mac2 open-wda
   ```
   This will open the bundled WebDriverAgentMac source in Xcode and print the path to the .xcodeproj file.

10. **Start Appium server smoke test**
    ```bash
    appium server
    ```
    Keep this server process running in Terminal A.
    In Terminal B, run:
    ```bash
    curl -s http://127.0.0.1:4723/status
    ```
    First confirm `/status` responds successfully from `curl`.
    Then confirm startup/readiness from server logs and ensure the `Available drivers:` block contains `mac2` (for example: `- mac2@3.2.16 (automationName 'Mac2')`).
    After smoke validation, clean up the running Appium server:
    - In Terminal A, stop the server with `Ctrl+C`.
    - Verify no leftover Appium server process (Terminal B):
    ```bash
    pgrep -fl "appium.*server" || echo "no appium server process"
    ```

11. **Agent completion criteria**
    Mark the skill complete only when all are true:
    - macOS version is 11 or later
    - Xcode 13 or later is installed
    - xcode-select points to `/Applications/Xcode.app/Contents/Developer`
    - Xcode Helper app has Accessibility access enabled
    - `appium driver list --installed` includes `mac2`
    - `appium -v` succeeds
    - `appium driver doctor mac2` has no failing mandatory checks
    - `curl -s http://127.0.0.1:4723/status` returns a successful status response
    - Appium server logs show startup/readiness successfully after the curl check
    - Appium server logs include `Available drivers:` with a `mac2` entry
    - Appium smoke-test server process is cleanly stopped after validation

## Constraints
- This skill is macOS-only; do not provide Linux/Windows alternatives.
- Requires macOS 11 (Big Sur) or later.
- Requires Xcode 13 or later.
- Use global npm/Appium commands as the default execution mode.
- Use `npx appium` only if the user explicitly asks for local execution.
- Always run `appium driver doctor mac2` after each environment change.
- Treat optional doctor warnings as non-blocking.
- Ask the user before installing optional dependencies, and install them only when explicitly needed.
- Xcode Helper Accessibility access must be enabled manually by the user.
- xcode-select must point to Xcode.app/Contents/Developer, not CommandLineTools.
- For CI environments on macOS 12+, consider disabling UIAutomation authentication.
- Use shell-appropriate commands (bash for macOS).
- Prefer deterministic CLI checks over assumptions.
- If elevated privileges are required, pause and provide exact commands for the user to run.
- Do not claim success until doctor and smoke-test checks are actually green.
- Parallel execution of multiple Mac2 driver instances is not supported (single-threaded accessibility layer).