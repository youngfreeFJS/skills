---
name: "plugin-setup-relaxed-caps"
description: "Install and validate Appium Relaxed Caps plugin to relax vendor prefix requirements"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"

---
# plugin-setup-relaxed-caps

## Goal
Install and validate the Relaxed Caps plugin to allow capabilities without vendor prefixes (e.g., `platformName` instead of `appium:platformName`), useful for legacy test suites and backward compatibility.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- If plugin is already installed: skip installation or update to latest version
- Validate plugin by starting a test session with non-prefixed capabilities
- Ensure backward compatibility with existing test suites

## Instructions

1. **Verify Appium installation**
   ```bash
   appium -v
   ```
   If Appium is not installed, follow `skills/environment-setup-node/SKILL.md` first.

2. **Install relaxed-caps plugin**
   ```bash
   appium plugin install relaxed-caps
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update relaxed-caps
   ```

3. **Verify plugin installation**
   ```bash
   appium plugin list --installed
   ```
   Verify that `relaxed-caps` appears in the list.

4. **Start Appium server with plugin enabled**
   ```bash
   appium server --use-plugins=relaxed-caps
   ```
   Keep this server running in Terminal A.

5. **Test session creation with non-prefixed caps**
   In Terminal B, test creating a session with non-prefixed capabilities:
   ```bash
   curl -X POST http://127.0.0.1:4723/session \
     -H "Content-Type: application/json" \
     -d '{
       "capabilities": {
         "alwaysMatch": {
           "platformName": "Android",
           "automationName": "UiAutomator2",
           "deviceName": "Android Emulator"
         }
       }
     }'
   ```
   
   Note: This will attempt to create a session. Without a connected device/emulator, it will fail, but the plugin should accept the non-prefixed capabilities without error.

6. **Verify plugin accepts non-prefixed capabilities**
   Check server logs in Terminal A for:
   - No capability validation errors
   - Plugin processes the capabilities correctly
   - Session creation attempt is made (even if it fails due to no device)

7. **Test with prefixed capabilities (should also work)**
   ```bash
   curl -X POST http://127.0.0.1:4723/session \
     -H "Content-Type: application/json" \
     -d '{
       "capabilities": {
         "alwaysMatch": {
           "appium:platformName": "Android",
           "appium:automationName": "UiAutomator2",
           "appium:deviceName": "Android Emulator"
         }
       }
     }'
   ```
   
   Both prefixed and non-prefixed capabilities should be accepted.

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
   - `appium plugin list --installed` includes `relaxed-caps`
   - Appium server starts successfully with `--use-plugins=relaxed-caps`
   - Non-prefixed capabilities are accepted without validation errors
   - Prefixed capabilities continue to work normally
   - Server process is cleanly stopped after validation

## Constraints
- Requires Appium 2.0+
- Plugin must be explicitly enabled when starting server using `--use-plugins=relaxed-caps`
- Useful for legacy test suites that don't use vendor prefixes
- May conflict with strict capability validation in production environments
- Recommended for development and testing environments only
- Both prefixed and non-prefixed capabilities are supported simultaneously
- Standard W3C capabilities (like `browserName`, `platformName`) work without prefix
- Appium-specific capabilities can be used with or without `appium:` prefix
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
