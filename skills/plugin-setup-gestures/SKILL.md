---
name: "plugin-setup-gestures"
description: "Install and validate Appium Gestures plugin for W3C Actions-based gestures"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"
  maintainer: "@AppiumTestDistribution"

---
# plugin-setup-gestures

## Goal
Install and validate the Gestures plugin to perform basic gestures (swipe, tap, long-press, drag) using simplified commands built on W3C Actions API.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- Requires at least one driver installed for testing (UiAutomator2, Espresso, or XCUITest)
- If plugin is already installed: skip installation or update to latest version
- Validate plugin with gesture execution test

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
   Ensure at least one driver is installed for gesture testing.

3. **Install gestures plugin from npm**
   ```bash
   appium plugin install --source=npm appium-gestures-plugin
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update appium-gestures-plugin
   ```

4. **Verify plugin installation**
   ```bash
   appium plugin list --installed
   ```
   Verify that `gestures` appears in the list.

5. **Start Appium server with plugin enabled**
   ```bash
   appium server --use-plugins=gestures
   ```
   Keep this server running in Terminal A.

6. **Verify plugin is loaded**
   In Terminal B, check server status:
   ```bash
   curl -s http://127.0.0.1:4723/status
   ```
   
   Check server logs in Terminal A for plugin initialization messages.

7. **Test gesture commands (requires active session)**
   Note: Full gesture testing requires an active driver session with device/emulator.
   
   Example gesture commands available:
   - **Swipe**: `POST /session/{sessionId}/appium/gestures/swipe`
   - **Tap**: `POST /session/{sessionId}/appium/gestures/tap`
   - **Long Press**: `POST /session/{sessionId}/appium/gestures/longPress`
   - **Drag**: `POST /session/{sessionId}/appium/gestures/drag`
   
   Example swipe command:
   ```bash
   curl -X POST http://127.0.0.1:4723/session/{sessionId}/appium/gestures/swipe \
     -H "Content-Type: application/json" \
     -d '{
       "elementId": "element-id",
       "direction": "up",
       "percent": 0.75
     }'
   ```

8. **Verify gesture endpoints are available**
   Check that gesture-related endpoints are registered:
   ```bash
   curl -s http://127.0.0.1:4723/status | grep -i gesture
   ```

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
    - `appium plugin list --installed` includes `gestures`
    - Appium server starts successfully with `--use-plugins=gestures`
    - Plugin initialization is confirmed in server logs
    - Server process is cleanly stopped after validation
    
    Note: Full functional validation requires an active driver session with device/emulator.

## Constraints
- Requires Appium 2.0+
- Requires at least one driver with W3C Actions support
- Plugin must be explicitly enabled when starting server using `--use-plugins=gestures`
- Requires active driver session for full functionality testing
- Gesture accuracy depends on device/simulator capabilities
- Some gestures may not work on all platforms
- Supported gestures: swipe, tap, long-press, drag, pinch, zoom
- Gestures are built on W3C Actions API for cross-platform compatibility
- Gesture parameters (speed, duration, coordinates) are customizable
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
