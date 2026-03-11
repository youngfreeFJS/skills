---
name: "plugin-setup-wait"
description: "Install and validate Appium Wait plugin for global wait timeout management"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"
  maintainer: "@AppiumTestDistribution"

---
# plugin-setup-wait

## Goal
Install and validate the Wait plugin to manage global element wait timeouts across all commands, providing consistent wait behavior without explicit waits in test code.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- Configure default wait timeout values before starting server
- If plugin is already installed: skip installation or update to latest version
- Validate plugin with timeout behavior test

## Instructions

1. **Verify Appium installation**
   ```bash
   appium -v
   ```
   If Appium is not installed, follow `skills/environment-setup-node/SKILL.md` first.

2. **Install wait plugin from npm**
   ```bash
   appium plugin install --source=npm appium-wait-plugin
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update appium-wait-plugin
   ```

3. **Verify plugin installation**
   ```bash
   appium plugin list --installed
   ```
   Verify that `wait` appears in the list.

4. **Configure wait timeout settings**
   Create a wait configuration file `wait-config.json`:
   ```json
   {
     "implicitWaitMs": 10000,
     "elementWaitMs": 5000,
     "pageLoadWaitMs": 30000
   }
   ```

5. **Start Appium server with plugin enabled**
   ```bash
   appium server --use-plugins=wait --plugin-wait-config=./wait-config.json
   ```
   Keep this server running in Terminal A.

6. **Verify plugin is loaded**
   In Terminal B, check server status:
   ```bash
   curl -s http://127.0.0.1:4723/status
   ```
   
   Check server logs in Terminal A for wait plugin initialization messages.

7. **Test implicit wait behavior (requires active session)**
   Note: Full wait testing requires an active driver session with device/emulator.
   
   Set implicit wait timeout:
   ```bash
   curl -X POST http://127.0.0.1:4723/session/{sessionId}/timeouts \
     -H "Content-Type: application/json" \
     -d '{
       "implicit": 10000
     }'
   ```

8. **Verify timeout configuration**
   Get current timeout settings:
   ```bash
   curl -X GET http://127.0.0.1:4723/session/{sessionId}/timeouts
   ```
   
   Should return configured timeout values.

9. **Test element finding with wait**
   When finding elements, the plugin will automatically wait up to the configured timeout:
   ```bash
   curl -X POST http://127.0.0.1:4723/session/{sessionId}/element \
     -H "Content-Type: application/json" \
     -d '{
       "using": "id",
       "value": "element-id"
     }'
   ```
   
   The command will wait up to the implicit wait timeout before failing.

10. **Stop Appium server**
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

11. **Agent completion criteria**
    Mark complete only when all are true:
    - `appium plugin list --installed` includes `wait`
    - Appium server starts successfully with `--use-plugins=wait`
    - Wait configuration is loaded successfully
    - Plugin initialization is confirmed in server logs
    - Server process is cleanly stopped after validation
    
    Note: Full functional validation requires an active driver session with device/emulator.

## Constraints
- Requires Appium 2.0+
- Plugin must be explicitly enabled when starting server using `--use-plugins=wait`
- Affects all element finding commands globally
- May increase test execution time if timeouts are too long
- Timeout values should be balanced for performance vs reliability
- Implicit wait applies to element finding operations
- Page load timeout applies to page navigation operations
- Script timeout applies to asynchronous script execution
- Recommended timeout values: 5-10 seconds for implicit wait
- Too short timeouts may cause flaky tests
- Too long timeouts may slow down test execution
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
