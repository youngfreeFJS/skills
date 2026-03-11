---
name: "environment-setup-chromium"
description: "Set up and validate a Chromium Appium environment for Chrome/Chromium browsers"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:25:00 GMT"

---
# appium-chromium-environment-setup

## Goal
Prepares a reliable Appium Chromium execution environment by installing Node.js and Appium prerequisites, configuring Chrome/Chromium browser dependencies, running Appium doctor checks, and iterating until all mandatory checks pass.

## Decision Logic
- If the host OS is not macOS, Linux, or Windows: stop and ask the user to use a supported OS.
- If current Node.js does not satisfy `engines.node` for both `appium` and `appium-chromium-driver`: install/upgrade Node.js to a compatible active LTS version.
- If Appium CLI is not installed: install `appium` globally.
- Use global npm/Appium commands by default (`npm install -g appium`, `appium ...`).
- Use local Appium commands (`npx appium ...`) only when the user explicitly requests local execution.
- If Chrome/Chromium browser is not installed: prompt user to install it first.
- If the user explicitly requests media features that require FFmpeg: run `environment-setup-ffmpeg` before final validation.
- If the `chromium` driver is not installed: install it via Appium CLI.
- If install returns "already installed", ignore the error and continue (or run driver update).
- If `appium driver doctor chromium` reports missing dependencies: resolve each missing item and re-run doctor.

## Instructions
1. **Prepare Node.js + npm environment**
   macOS/Linux:
   ```bash
   node -v
   npm -v
   ```
   Windows PowerShell:
   ```powershell
   node -v
   npm -v
   ```
   If `node` is missing, install a compatible active LTS release and re-run the commands.

2. **Install Appium npm command**
   ```bash
   npm install -g appium
   appium driver install chromium || appium driver update chromium
   appium driver list --installed
   ```
   If the install command fails only because `chromium` is already installed, continue and do not stop preparation.

3. **Validate Appium npm commands and Node compatibility (after driver setup)**
   macOS/Linux:
   ```bash
   appium -v
   appium driver list --installed
   npm view appium engines --json
   npm view appium-chromium-driver engines --json
   ```
   Windows PowerShell:
   ```powershell
   appium -v
   appium driver list --installed
   npm view appium engines --json
   npm view appium-chromium-driver engines --json
   ```
   If current Node.js does not satisfy the reported `engines.node` ranges, install/upgrade Node.js to a compatible active LTS version and re-run the setup checks.

4. **Verify Chrome/Chromium browser installation**
   macOS:
   ```bash
   /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --version || \
   /Applications/Chromium.app/Contents/MacOS/Chromium --version
   ```
   Linux:
   ```bash
   google-chrome --version || chromium-browser --version || chromium --version
   ```
   Windows PowerShell:
   ```powershell
   & "C:\Program Files\Google\Chrome\Application\chrome.exe" --version
   ```
   If no browser is found, prompt the user to install Chrome or Chromium before continuing.

5. **Optional: Install specific Chromedriver version**
   If the user needs a specific Chromedriver version or wants to pre-download it:
   
   macOS/Linux:
   ```bash
   appium driver run chromium install-chromedriver
   ```
   
   Windows PowerShell:
   ```powershell
   appium driver run chromium install-chromedriver
   ```
   
   For a specific version:
   macOS/Linux:
   ```bash
   CHROMEDRIVER_VERSION=131.0.6778.3 appium driver run chromium install-chromedriver
   ```
   
   Windows PowerShell:
   ```powershell
   $env:CHROMEDRIVER_VERSION='131.0.6778.3'; appium driver run chromium install-chromedriver; Remove-Item Env:\CHROMEDRIVER_VERSION
   ```

   Optional shared dependency:
   - If the user explicitly requests FFmpeg-related capability, run `environment-setup-ffmpeg` before continuing.

6. **Run Appium doctor for Chromium and fix in a loop**
   ```bash
   appium driver doctor chromium
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
   Then confirm startup/readiness from server logs and ensure the `Available drivers:` block contains `chromium` (for example: `- chromium@2.1.6 (automationName 'Chromium')`).
   After smoke validation, clean up the running Appium server:
   - In Terminal A, stop the server with `Ctrl+C`.
   - Verify no leftover Appium server process (Terminal B, macOS/Linux):
   ```bash
   pgrep -fl "appium.*server" || echo "no appium server process"
   ```
   - Verify no leftover Appium server process (Terminal B, Windows PowerShell):
   ```powershell
   if (Get-CimInstance Win32_Process | Where-Object { $_.CommandLine -match 'appium.*server' }) {
      Get-CimInstance Win32_Process | Where-Object { $_.CommandLine -match 'appium.*server' } | Select-Object ProcessId, Name, CommandLine
   } else {
      "no appium server process"
   }
   ```

8. **Agent completion criteria**
   Mark the skill complete only when all are true:
   - `appium driver list --installed` includes `chromium`
   - `appium -v` succeeds
   - `appium driver doctor chromium` has no failing mandatory checks
   - Chrome or Chromium browser is installed and version is verified
   - `curl -s http://127.0.0.1:4723/status` returns a successful status response
   - Appium server logs show startup/readiness successfully after the curl check
   - Appium server logs include `Available drivers:` with a `chromium` entry
   - Appium smoke-test server process is cleanly stopped after validation

## Constraints
- Always run `appium driver doctor chromium` after each environment change.
- Use global npm/Appium commands as the default execution mode.
- Use `npx appium` only if the user explicitly asks for local execution.
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows).
- Treat optional doctor warnings as non-blocking.
- Ask the user before installing optional dependencies, and install them only when the user explicitly needs that capability.
- Prefer deterministic CLI checks over assumptions.
- If elevated privileges are required, pause and provide exact commands for the user to run.
- Do not claim success until doctor and smoke-test checks are actually green.
- The driver automatically downloads appropriate Chromedriver versions, but users can pre-install specific versions if needed.