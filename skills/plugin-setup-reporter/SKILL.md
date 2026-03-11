---
name: "plugin-setup-reporter"
description: "Install and validate Appium Reporter plugin for HTML report generation"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"
  maintainer: "@AppiumTestDistribution"

---
# plugin-setup-reporter

## Goal
Install and validate the Reporter plugin to generate standalone consolidated HTML reports with screenshots, providing comprehensive test execution documentation.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- Configure report output directory before starting server
- If plugin is already installed: skip installation or update to latest version
- Validate plugin by generating a test report

## Instructions

1. **Verify Appium installation**
   ```bash
   appium -v
   ```
   If Appium is not installed, follow `skills/environment-setup-node/SKILL.md` first.

2. **Install reporter plugin from npm**
   ```bash
   appium plugin install --source=npm appium-reporter-plugin
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update appium-reporter-plugin
   ```

3. **Verify plugin installation**
   ```bash
   appium plugin list --installed
   ```
   Verify that `reporter` appears in the list.

4. **Configure report settings**
   Create a report configuration file `reporter-config.json`:
   ```json
   {
     "outputDir": "./test-reports",
     "reportName": "appium-test-report",
     "screenshotPath": "./screenshots",
     "includeScreenshots": true,
     "reportTitle": "Appium Test Execution Report"
   }
   ```

5. **Create report output directory**
   ```bash
   mkdir -p ./test-reports ./screenshots
   ```

6. **Start Appium server with plugin enabled**
   ```bash
   appium server --use-plugins=reporter --plugin-reporter-config=./reporter-config.json
   ```
   Keep this server running in Terminal A.

7. **Verify plugin is loaded**
   In Terminal B, check server status:
   ```bash
   curl -s http://127.0.0.1:4723/status
   ```
   
   Check server logs in Terminal A for reporter plugin initialization messages.

8. **Run test session (for report generation)**
   Note: Full report generation requires running actual test sessions.
   
   After test execution completes, the plugin will automatically generate an HTML report in the configured output directory.

9. **Verify report generation**
   Check that report files are created:
   ```bash
   ls -la ./test-reports/
   ```
   
   Should contain HTML report files with timestamps.

10. **View generated report**
    Open the HTML report in a browser:
    ```bash
    open ./test-reports/appium-test-report.html
    ```
    
    Or on Linux:
    ```bash
    xdg-open ./test-reports/appium-test-report.html
    ```
    
    Or on Windows:
    ```powershell
    Start-Process ./test-reports/appium-test-report.html
    ```

11. **Stop Appium server**
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

12. **Agent completion criteria**
    Mark complete only when all are true:
    - `appium plugin list --installed` includes `reporter`
    - Appium server starts successfully with `--use-plugins=reporter`
    - Report configuration is loaded successfully
    - Report output directory is created
    - Plugin initialization is confirmed in server logs
    - Server process is cleanly stopped after validation
    
    Note: Full report validation requires running actual test sessions.

## Constraints
- Requires Appium 2.0+
- Plugin must be explicitly enabled when starting server using `--use-plugins=reporter`
- Reports generated after session completion
- Screenshot capture must be enabled in test code for screenshots to appear in report
- Report size depends on test duration and number of screenshots
- HTML reports are standalone (include embedded CSS and JavaScript)
- Report includes: test summary, session details, commands executed, screenshots, errors
- Multiple test sessions are consolidated into a single report
- Report format: HTML with responsive design
- Supports custom report titles and branding
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
