---
name: "plugin-setup-device-farm"
description: "Install and validate Appium Device Farm plugin for device management"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"
  maintainer: "@AppiumTestDistribution"

---
# plugin-setup-device-farm

## Goal
Install and validate the Device Farm plugin to manage and create driver sessions on connected Android devices and iOS simulators, providing centralized device management capabilities.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- Requires Android SDK for Android devices (reference `environment-setup-android`)
- Requires Xcode for iOS simulators on macOS (reference `environment-setup-xcuitest`)
- If plugin is already installed: skip installation or update to latest version
- Validate plugin by listing available devices

## Instructions

1. **Verify Appium installation**
   ```bash
   appium -v
   ```
   If Appium is not installed, follow `skills/environment-setup-node/SKILL.md` first.

2. **Verify platform-specific dependencies**
   
   For Android devices:
   ```bash
   adb version
   echo $ANDROID_HOME
   ```
   If not set, follow `skills/environment-setup-android/SKILL.md`.
   
   For iOS simulators (macOS only):
   ```bash
   xcrun simctl list devices
   ```
   If not available, follow `skills/environment-setup-xcuitest/SKILL.md`.

3. **Install device-farm plugin from npm**
   ```bash
   appium plugin install --source=npm appium-device-farm
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update appium-device-farm
   ```

4. **Verify plugin installation**
   ```bash
   appium plugin list --installed
   ```
   Verify that `device-farm` appears in the list.

5. **Configure device farm settings (optional)**
   Create a configuration file `device-farm-config.json`:
   ```json
   {
     "platform": "both",
     "androidDeviceType": "both",
     "iosDeviceType": "both",
     "maxSessions": 5
   }
   ```

6. **Start Appium server with plugin enabled**
   ```bash
   appium server --use-plugins=device-farm
   ```
   Keep this server running in Terminal A.

7. **List available devices**
   In Terminal B, query available devices:
   ```bash
   curl -s http://127.0.0.1:4723/device-farm/api/devices
   ```
   
   Should return a list of connected Android devices and iOS simulators.

8. **Test device allocation**
   Request a device allocation:
   ```bash
   curl -X POST http://127.0.0.1:4723/device-farm/api/device \
     -H "Content-Type: application/json" \
     -d '{
       "platform": "android",
       "name": ".*"
     }'
   ```
   
   Should return device details if a matching device is available.

9. **Verify device management UI (optional)**
   Open browser and navigate to:
   ```
   http://localhost:4723/device-farm
   ```
   
   Should display device management dashboard with connected devices.

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
    - `appium plugin list --installed` includes `device-farm`
    - Appium server starts successfully with `--use-plugins=device-farm`
    - Device list endpoint returns valid response
    - At least one device is detected (Android or iOS)
    - Server process is cleanly stopped after validation

## Constraints
- Requires Appium 2.0+
- Plugin must be explicitly enabled when starting server using `--use-plugins=device-farm`
- Requires platform-specific SDKs (Android SDK for Android, Xcode for iOS)
- Android device discovery depends on USB connections and ADB
- iOS support limited to macOS with Xcode installed
- Maximum concurrent sessions configurable (default: 5)
- Device allocation is first-come-first-served
- Supports both real devices and emulators/simulators
- Web dashboard available at http://localhost:4723/device-farm
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
