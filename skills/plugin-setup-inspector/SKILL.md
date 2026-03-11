---
name: "plugin-setup-inspector"
description: "Install and validate Appium Inspector plugin for integrated debugging"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"

---
# plugin-setup-inspector

## Goal
Install and validate the Inspector plugin to integrate Appium Inspector directly into the Appium server installation, providing a web-based UI for inspecting app elements and testing commands.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- If plugin is already installed: skip installation or update to latest version
- Validate plugin by accessing Inspector web interface through browser
- Check browser compatibility (modern browsers required)

## Instructions

1. **Verify Appium installation**
   ```bash
   appium -v
   ```
   If Appium is not installed, follow `skills/environment-setup-node/SKILL.md` first.

2. **Install inspector plugin**
   ```bash
   appium plugin install inspector
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update inspector
   ```

3. **Verify plugin installation**
   ```bash
   appium plugin list --installed
   ```
   Verify that `inspector` appears in the list.

4. **Start Appium server with inspector enabled**
   ```bash
   appium server --use-plugins=inspector
   ```
   Keep this server running in Terminal A.
   
   The server will log the Inspector URL, typically:
   ```
   Appium Inspector available at http://localhost:4723/inspector
   ```

5. **Access Inspector web interface**
   Open a web browser and navigate to:
   ```
   http://localhost:4723/inspector
   ```
   
   You should see the Appium Inspector UI with:
   - Session configuration panel
   - Capability editor
   - Start Session button

6. **Verify Inspector functionality**
   In the Inspector UI:
   - Check that the interface loads correctly
   - Verify capability editor is functional
   - Confirm connection status shows "Ready"
   
   Note: You don't need to start an actual session for validation, just verify the UI is accessible.

7. **Test Inspector API endpoint**
   In Terminal B, verify the Inspector endpoint:
   ```bash
   curl -s http://127.0.0.1:4723/inspector
   ```
   Should return HTML content of the Inspector UI.

8. **Stop Appium server**
   In Terminal A, stop the server with `Ctrl+C`.
   
   Verify no leftover process (Terminal B):
   macOS/Linux:
   ```bash
   pgrep -fl "appium.*server" || echo "no appium server process"
   ```
   Windows PowerShell:
   ```powershell
   if (Get-CimInstance Win32_Process | Where-Object { $_.CommandLine -match 'appium.*server' }) {
      Get-CimInstance Win32_Process | Where-Object { $_.CommandLine -match 'appium.*server' } | Select-Object ProcessId, Name, CommandLine
   } else {
      "no appium server process"
   }
   ```

9. **Agent completion criteria**
   Mark complete only when all are true:
   - `appium plugin list --installed` includes `inspector`
   - Appium server starts successfully with `--use-plugins=inspector`
   - Inspector web interface is accessible at http://localhost:4723/inspector
   - Inspector UI loads correctly in browser
   - Server process is cleanly stopped after validation

## Constraints
- Requires Appium 2.0+
- Requires modern web browser (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- Plugin must be explicitly enabled when starting server using `--use-plugins=inspector`
- Default port: 4723 (configurable via --port flag)
- Web interface accessible at http://localhost:4723/inspector
- Inspector UI requires JavaScript enabled in browser
- Network access required to load Inspector assets
- Inspector session capabilities are stored in browser local storage
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
