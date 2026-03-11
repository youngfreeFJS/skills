---
name: "plugin-setup-storage"
description: "Install and validate Appium Storage plugin for server-side data management"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"

---
# plugin-setup-storage

## Goal
Install and validate the Storage plugin to add server-side storage with client-side management capabilities, allowing tests to store and retrieve data across sessions.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- If plugin is already installed: skip installation or update to latest version
- Validate plugin by storing and retrieving test data
- Check storage persistence behavior (memory vs persistent storage)

## Instructions

1. **Verify Appium installation**
   ```bash
   appium -v
   ```
   If Appium is not installed, follow `skills/environment-setup-node/SKILL.md` first.

2. **Install storage plugin**
   ```bash
   appium plugin install storage
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update storage
   ```

3. **Verify plugin installation**
   ```bash
   appium plugin list --installed
   ```
   Verify that `storage` appears in the list.

4. **Start Appium server with plugin enabled**
   ```bash
   appium server --use-plugins=storage
   ```
   Keep this server running in Terminal A.

5. **Test data storage**
   In Terminal B, store test data:
   ```bash
   curl -X POST http://127.0.0.1:4723/session/storage/set \
     -H "Content-Type: application/json" \
     -d '{
       "key": "test-key",
       "value": "test-value"
     }'
   ```

6. **Test data retrieval**
   Retrieve the stored data:
   ```bash
   curl -X POST http://127.0.0.1:4723/session/storage/get \
     -H "Content-Type: application/json" \
     -d '{
       "key": "test-key"
     }'
   ```
   
   Should return: `{"value": "test-value"}`

7. **Test data deletion**
   Delete the stored data:
   ```bash
   curl -X POST http://127.0.0.1:4723/session/storage/delete \
     -H "Content-Type: application/json" \
     -d '{
       "key": "test-key"
     }'
   ```

8. **Verify deletion**
   Try to retrieve the deleted data:
   ```bash
   curl -X POST http://127.0.0.1:4723/session/storage/get \
     -H "Content-Type: application/json" \
     -d '{
       "key": "test-key"
     }'
   ```
   
   Should return null or empty response.

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
    - `appium plugin list --installed` includes `storage`
    - Appium server starts successfully with `--use-plugins=storage`
    - Data can be stored successfully
    - Data can be retrieved successfully
    - Data can be deleted successfully
    - Server process is cleanly stopped after validation

## Constraints
- Requires Appium 2.0+
- Plugin must be explicitly enabled when starting server using `--use-plugins=storage`
- Data stored in memory by default (not persistent across server restarts)
- Persistent storage requires additional configuration
- Storage cleared on server restart unless configured for persistence
- Storage is session-independent (data accessible across different sessions)
- No built-in data expiration mechanism
- Storage size limited by available memory
- Suitable for temporary test data, not for production data storage
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
