---
name: "plugin-setup-execute-driver"
description: "Install and validate Appium Execute Driver plugin for batch command execution"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"

---
# plugin-setup-execute-driver

## Goal
Install and validate the Execute Driver plugin to enable batch command execution in a single Appium server call, improving test performance by reducing network overhead.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- If plugin is already installed: skip installation or update to latest version
- If plugin installation fails: provide troubleshooting steps and verify npm connectivity
- Validate plugin by executing a batch command test with multiple commands

## Instructions

1. **Verify Appium installation**
   macOS/Linux:
   ```bash
   appium -v
   ```
   Windows PowerShell:
   ```powershell
   appium -v
   ```
   If Appium is not installed, follow `skills/environment-setup-node/SKILL.md` first.

2. **Install execute-driver plugin**
   ```bash
   appium plugin install execute-driver
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update execute-driver
   ```

3. **List installed plugins**
   ```bash
   appium plugin list --installed
   ```
   Verify that `execute-driver` appears in the list.

4. **Start Appium server with plugin enabled**
   ```bash
   appium server --use-plugins=execute-driver
   ```
   Keep this server running in Terminal A.

5. **Verify plugin is loaded**
   In Terminal B, check server status:
   ```bash
   curl -s http://127.0.0.1:4723/status
   ```
   The response should indicate the server is running.

6. **Execute batch command test**
   Test the plugin by executing multiple commands in a single call:
   ```bash
   curl -X POST http://127.0.0.1:4723/session/execute/driver \
     -H "Content-Type: application/json" \
     -d '{
       "script": "execute-driver",
       "args": [{
         "commands": [
           {"command": "getStatus"},
           {"command": "getSessions"}
         ]
       }]
     }'
   ```
   
   The response should contain results from both commands.

7. **Stop Appium server**
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

8. **Agent completion criteria**
   Mark complete only when all are true:
   - `appium plugin list --installed` includes `execute-driver`
   - Appium server starts successfully with `--use-plugins=execute-driver`
   - Batch command execution test returns valid results
   - Server process is cleanly stopped after validation

## Constraints
- Requires Appium 2.0+
- Plugin must be explicitly enabled when starting server using `--use-plugins=execute-driver`
- Batch commands are executed sequentially, not in parallel
- Each command in the batch must be a valid Appium command
- Error in one command does not stop execution of subsequent commands
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
