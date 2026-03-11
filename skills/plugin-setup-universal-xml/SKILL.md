---
name: "plugin-setup-universal-xml"
description: "Install and validate Appium Universal XML plugin for unified page source format"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"

---
# plugin-setup-universal-xml

## Goal
Install and validate the Universal XML plugin to translate iOS and Android XML formats into a common unified format, simplifying cross-platform test development.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- Requires at least one driver installed (UiAutomator2, Espresso, or XCUITest)
- If plugin is already installed: skip installation or update to latest version
- Validate plugin by comparing XML output with and without plugin enabled

## Instructions

1. **Verify Appium installation**
   ```bash
   appium -v
   ```
   If Appium is not installed, follow `skills/environment-setup-node/SKILL.md` first.

2. **Verify driver installation**
   ```bash
   appium driver list --installed
   ```
   Ensure at least one of these drivers is installed:
   - `uiautomator2` (Android)
   - `espresso` (Android)
   - `xcuitest` (iOS)
   
   If no driver is installed, follow the appropriate driver setup skill first.

3. **Install universal-xml plugin**
   ```bash
   appium plugin install universal-xml
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update universal-xml
   ```

4. **Verify plugin installation**
   ```bash
   appium plugin list --installed
   ```
   Verify that `universal-xml` appears in the list.

5. **Start Appium server with plugin enabled**
   ```bash
   appium server --use-plugins=universal-xml
   ```
   Keep this server running in Terminal A.

6. **Verify plugin is loaded**
   In Terminal B, check server status:
   ```bash
   curl -s http://127.0.0.1:4723/status
   ```
   
   Check server logs in Terminal A for plugin initialization messages.

7. **Test XML transformation (requires active session)**
   Note: To fully test this plugin, you need an active driver session with a connected device/emulator.
   
   With an active session, get page source:
   ```bash
   curl -X GET http://127.0.0.1:4723/session/{sessionId}/source
   ```
   
   The XML output should be in unified format with standardized element names and attributes across platforms.

8. **Compare with native format**
   To see the difference, restart server without the plugin:
   ```bash
   # Stop current server (Ctrl+C in Terminal A)
   # Start without plugin
   appium server
   ```
   
   Get page source again and compare the XML structure.
   - With plugin: Unified element names (e.g., `<element>`)
   - Without plugin: Platform-specific names (e.g., `<android.widget.TextView>`, `<XCUIElementTypeButton>`)

9. **Stop Appium server**
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

10. **Agent completion criteria**
    Mark complete only when all are true:
    - At least one Appium driver is installed
    - `appium plugin list --installed` includes `universal-xml`
    - Appium server starts successfully with `--use-plugins=universal-xml`
    - Plugin initialization is confirmed in server logs
    - Server process is cleanly stopped after validation
    
    Note: Full functional validation requires an active driver session with device/emulator.

## Constraints
- Requires Appium 2.0+
- Requires at least one driver installed (UiAutomator2, Espresso, or XCUITest)
- Plugin must be explicitly enabled when starting server using `--use-plugins=universal-xml`
- Requires active driver session for full functionality testing
- XML transformation may affect performance for large page hierarchies
- Some platform-specific attributes may be lost in translation
- Unified format uses generic element names (e.g., `<element>` instead of platform-specific types)
- Attributes are standardized across platforms (e.g., `text`, `enabled`, `visible`)
- Simplifies cross-platform test development but may hide platform-specific details
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
