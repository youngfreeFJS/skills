---
name: "plugin-setup-ocr"
description: "Install and validate Appium OCR plugin for text-based element finding"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"
  maintainer: "@jlipps"

---
# plugin-setup-ocr

## Goal
Install and validate the OCR plugin to find elements via OCR (Optical Character Recognition) text recognition, enabling text-based element location when traditional locators are unavailable.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- Requires Tesseract OCR engine (4.0+) installed on the system
- If plugin is already installed: skip installation or update to latest version
- Validate plugin with text recognition test

## Instructions

1. **Verify Appium installation**
   ```bash
   appium -v
   ```
   If Appium is not installed, follow `skills/environment-setup-node/SKILL.md` first.

2. **Install Tesseract OCR engine**
   
   macOS (Homebrew):
   ```bash
   brew install tesseract
   ```
   
   Linux (Debian/Ubuntu):
   ```bash
   sudo apt-get update
   sudo apt-get install -y tesseract-ocr
   ```
   
   Linux (RHEL/CentOS/Fedora):
   ```bash
   sudo dnf install -y tesseract
   ```
   
   Windows (using Chocolatey):
   ```powershell
   choco install tesseract
   ```
   
   Or download from: https://github.com/tesseract-ocr/tesseract

3. **Verify Tesseract installation**
   ```bash
   tesseract --version
   ```
   Should display Tesseract version 4.0 or higher.

4. **Install OCR language data (optional but recommended)**
   
   macOS:
   ```bash
   brew install tesseract-lang
   ```
   
   Linux:
   ```bash
   sudo apt-get install -y tesseract-ocr-eng tesseract-ocr-chi-sim
   ```
   
   Or download language data from: https://github.com/tesseract-ocr/tessdata

5. **Install ocr plugin from npm**
   ```bash
   appium plugin install --source=npm appium-ocr-plugin
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update appium-ocr-plugin
   ```

6. **Verify plugin installation**
   ```bash
   appium plugin list --installed
   ```
   Verify that `ocr` appears in the list.

7. **Configure OCR settings (optional)**
   Create OCR configuration file `ocr-config.json`:
   ```json
   {
     "language": "eng",
     "oem": 3,
     "psm": 3
   }
   ```

8. **Start Appium server with plugin enabled**
   ```bash
   appium server --use-plugins=ocr
   ```
   Keep this server running in Terminal A.

9. **Verify plugin is loaded**
   In Terminal B, check server status:
   ```bash
   curl -s http://127.0.0.1:4723/status
   ```
   
   Check server logs in Terminal A for OCR plugin initialization messages.

10. **Test text recognition (requires active session)**
    Note: Full OCR testing requires an active driver session with device/emulator.
    
    Find element by OCR text:
    ```bash
    curl -X POST http://127.0.0.1:4723/session/{sessionId}/element \
      -H "Content-Type: application/json" \
      -d '{
        "using": "ocr",
        "value": "Login"
      }'
    ```
    
    This will use OCR to find elements containing the text "Login".

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
    - Tesseract OCR is installed (version 4.0+)
    - `appium plugin list --installed` includes `ocr`
    - Appium server starts successfully with `--use-plugins=ocr`
    - Plugin initialization is confirmed in server logs
    - Server process is cleanly stopped after validation
    
    Note: Full functional validation requires an active driver session with device/emulator.

## Constraints
- Requires Appium 2.0+
- Requires Tesseract OCR engine (4.0 or higher)
- Plugin must be explicitly enabled when starting server using `--use-plugins=ocr`
- OCR accuracy depends on image quality, font size, and text clarity
- Performance impact on element finding (slower than traditional locators)
- Language data must be installed separately for non-English text
- Supported languages depend on installed Tesseract language packs
- OCR works best with clear, high-contrast text
- May produce false positives with similar-looking text
- Not recommended as primary locator strategy
- Best used as fallback when traditional locators are unavailable
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
