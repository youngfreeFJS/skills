---
name: "plugin-setup-interceptor"
description: "Install and validate Appium Interceptor plugin for API mocking"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"
  maintainer: "@AppiumTestDistribution"

---
# plugin-setup-interceptor

## Goal
Install and validate the Interceptor plugin to intercept and mock API requests and responses, enabling network traffic manipulation for testing purposes.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- Requires proxy configuration for HTTP/HTTPS interception
- If plugin is already installed: skip installation or update to latest version
- Validate plugin with mock API test configuration

## Instructions

1. **Verify Appium installation**
   ```bash
   appium -v
   ```
   If Appium is not installed, follow `skills/environment-setup-node/SKILL.md` first.

2. **Install interceptor plugin from npm**
   ```bash
   appium plugin install --source=npm appium-interceptor-plugin
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update appium-interceptor-plugin
   ```

3. **Verify plugin installation**
   ```bash
   appium plugin list --installed
   ```
   Verify that `interceptor` appears in the list.

4. **Configure interception rules**
   Create an interception configuration file `interceptor-config.json`:
   ```json
   {
     "intercept": [
       {
         "urlPattern": "https://api.example.com/.*",
         "method": "GET",
         "response": {
           "statusCode": 200,
           "body": {
             "message": "Mocked response"
           }
         }
       }
     ]
   }
   ```

5. **Start Appium server with plugin enabled**
   ```bash
   appium server --use-plugins=interceptor --plugin-interceptor-config=./interceptor-config.json
   ```
   Keep this server running in Terminal A.

6. **Verify plugin is loaded**
   In Terminal B, check server status:
   ```bash
   curl -s http://127.0.0.1:4723/status
   ```
   
   Check server logs in Terminal A for interceptor initialization messages.

7. **Test API interception (requires active session)**
   Note: Full interception testing requires an active driver session with app making network requests.
   
   Configure interception rules via API:
   ```bash
   curl -X POST http://127.0.0.1:4723/session/{sessionId}/appium/interceptor/rules \
     -H "Content-Type: application/json" \
     -d '{
       "rules": [
         {
           "urlPattern": "https://api.example.com/users",
           "method": "GET",
           "mockResponse": {
             "statusCode": 200,
             "body": {"users": []}
           }
         }
       ]
     }'
   ```

8. **Verify mock responses**
   When the app makes a request matching the pattern, it should receive the mocked response instead of the real API response.

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
    - `appium plugin list --installed` includes `interceptor`
    - Appium server starts successfully with `--use-plugins=interceptor`
    - Plugin initialization is confirmed in server logs
    - Interception configuration is loaded successfully
    - Server process is cleanly stopped after validation
    
    Note: Full functional validation requires an active driver session with app making network requests.

## Constraints
- Requires Appium 2.0+
- Plugin must be explicitly enabled when starting server using `--use-plugins=interceptor`
- Requires SSL certificate configuration for HTTPS interception
- May affect app performance due to proxy overhead
- Interception rules must be defined before session start or via API during session
- Supports URL pattern matching with regex
- Can mock HTTP methods: GET, POST, PUT, DELETE, PATCH
- Mock responses can include custom headers, status codes, and body
- Useful for testing offline scenarios, error handling, and edge cases
- Not recommended for production testing
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
