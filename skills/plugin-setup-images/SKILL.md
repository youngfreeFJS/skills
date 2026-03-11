---
name: "plugin-setup-images"
description: "Install and validate Appium Images plugin for image matching and comparison"
metadata:
  last_modified: "Tue, 11 Mar 2026 20:00:00 GMT"

---
# plugin-setup-images

## Goal
Install and validate the Images plugin to enable image-based element finding and comparison features, allowing tests to locate elements using visual matching instead of traditional locators.

## Decision Logic
- If Appium is not installed: reference `environment-setup-node` and install Appium first
- If OpenCV dependencies are missing: install platform-specific OpenCV packages before plugin installation
- If plugin is already installed: skip installation or update to latest version
- Validate plugin with image comparison test using sample images

## Instructions

1. **Verify Appium installation**
   ```bash
   appium -v
   ```
   If Appium is not installed, follow `skills/environment-setup-node/SKILL.md` first.

2. **Install system dependencies (OpenCV)**
   
   macOS (Homebrew):
   ```bash
   brew install opencv
   ```
   
   Linux (Debian/Ubuntu):
   ```bash
   sudo apt-get update
   sudo apt-get install -y libopencv-dev python3-opencv
   ```
   
   Linux (RHEL/CentOS/Fedora):
   ```bash
   sudo dnf install -y opencv opencv-devel
   ```
   
   Windows (using Chocolatey):
   ```powershell
   choco install opencv
   ```
   
   Or download pre-built binaries from: https://opencv.org/releases/

3. **Install images plugin**
   ```bash
   appium plugin install images
   ```
   If the plugin is already installed, you can update it:
   ```bash
   appium plugin update images
   ```

4. **Verify plugin installation**
   ```bash
   appium plugin list --installed
   ```
   Verify that `images` appears in the list.

5. **Prepare test images**
   Create a test directory and prepare sample images:
   ```bash
   mkdir -p /tmp/appium-images-test
   # You'll need two similar images for comparison testing
   ```

6. **Start Appium server with plugin enabled**
   ```bash
   appium server --use-plugins=images
   ```
   Keep this server running in Terminal A.

7. **Test image comparison functionality**
   In Terminal B, test the image comparison endpoint:
   ```bash
   # This requires a running session with a driver
   # Example using curl (requires base64-encoded images)
   curl -X POST http://127.0.0.1:4723/session/{sessionId}/appium/compare_images \
     -H "Content-Type: application/json" \
     -d '{
       "mode": "matchTemplate",
       "firstImage": "<base64-encoded-image-1>",
       "secondImage": "<base64-encoded-image-2>"
     }'
   ```
   
   Note: For actual testing, you'll need an active driver session and properly encoded images.

8. **Verify plugin capabilities**
   Check that image-related commands are available:
   ```bash
   curl -s http://127.0.0.1:4723/status | grep -i image
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
    - OpenCV is installed and accessible
    - `appium plugin list --installed` includes `images`
    - Appium server starts successfully with `--use-plugins=images`
    - Image comparison endpoint is accessible
    - Server process is cleanly stopped after validation

## Doctor Checks

The Images plugin provides built-in Doctor checks to help diagnose and fix common setup issues. These checks verify system dependencies, plugin configuration, and installation status.

### Running Doctor Checks

After installing the Images plugin, run the following command to diagnose potential issues:

```bash
appium plugin doctor images
```

Expected output when all checks pass:
```
✔ OpenCV library is installed and accessible
✔ Images plugin is correctly installed
✔ Plugin configuration is valid
✔ Image processing dependencies are available

info AppiumDoctor Everything looks good, bye!
info AppiumDoctor
```

### Available Doctor Checks

The Images plugin includes the following diagnostic checks:

#### 1. System Dependencies Check

**Purpose**: Verifies that OpenCV libraries are installed and accessible

**Check Details**:
- Checks for OpenCV installation on the system
- Verifies OpenCV version compatibility (3.0+ or 4.0+)
- Validates library paths and environment variables

**Diagnosis Command**:
```bash
# Manual verification
opencv_version
# or
python3 -c "import cv2; print(cv2.__version__)"
```

**Common Issues and Fixes**:

| Issue | Diagnosis | Fix |
|-------|-----------|-----|
| OpenCV not installed | `opencv_version: command not found` | Install OpenCV using package manager (see Installation step 2) |
| Wrong OpenCV version | Version < 3.0 | Upgrade OpenCV: `brew upgrade opencv` (macOS) or `sudo apt-get upgrade libopencv-dev` (Linux) |
| Library path not set | `Library not loaded` error | Set `DYLD_LIBRARY_PATH` (macOS) or `LD_LIBRARY_PATH` (Linux) |

**Autofix Available**: No (requires manual installation)

---

#### 2. Plugin Installation Check

**Purpose**: Verifies that the Images plugin is correctly installed

**Check Details**:
- Confirms plugin is listed in installed plugins
- Validates plugin package integrity
- Checks plugin version compatibility with Appium

**Diagnosis Command**:
```bash
appium plugin list --installed
```

**Common Issues and Fixes**:

| Issue | Diagnosis | Fix |
|-------|-----------|-----|
| Plugin not found | Plugin not in list | Run `appium plugin install images` |
| Corrupted installation | Installation errors | Reinstall: `appium plugin uninstall images && appium plugin install images` |
| Version mismatch | Compatibility warnings | Update plugin: `appium plugin update images` |

**Autofix Available**: Yes (can reinstall plugin automatically)

---

#### 3. Configuration Check

**Purpose**: Validates plugin configuration and settings

**Check Details**:
- Checks for valid image format support (PNG, JPEG, BMP)
- Verifies image processing settings
- Validates memory allocation for image operations

**Diagnosis Command**:
```bash
# Check plugin configuration
appium server --show-config | grep -A 10 "images"
```

**Common Issues and Fixes**:

| Issue | Diagnosis | Fix |
|-------|-----------|-----|
| Invalid config file | Parse errors | Create valid config file (see Configuration example) |
| Unsupported format | Format errors | Use supported formats: PNG, JPEG, BMP |
| Memory limits | Out of memory errors | Increase Node.js heap size: `NODE_OPTIONS=--max-old-space-size=4096` |

**Autofix Available**: Yes (can create default configuration)

---

#### 4. Runtime Environment Check

**Purpose**: Verifies the runtime environment for image processing

**Check Details**:
- Checks available system memory
- Validates temporary directory permissions
- Verifies image cache directory

**Diagnosis Command**:
```bash
# Check system resources
free -h  # Linux
vm_stat  # macOS

# Check temp directory
ls -la /tmp/appium-images
```

**Common Issues and Fixes**:

| Issue | Diagnosis | Fix |
|-------|-----------|-----|
| Insufficient memory | Memory < 2GB available | Close other applications or increase system memory |
| Permission denied | Cannot write to temp dir | Fix permissions: `chmod 755 /tmp/appium-images` |
| Disk space low | < 1GB free space | Clean up disk space |

**Autofix Available**: Partial (can create directories, cannot fix memory/disk)

---

### Doctor Check Results Interpretation

Doctor checks return one of the following statuses:

- ✔ **OK**: Check passed, no action needed
- ⚠ **WARNING**: Optional issue detected, plugin may work with limitations
- ✖ **ERROR**: Critical issue detected, must be fixed before using plugin

### Example Doctor Output

**Scenario 1: All checks pass**
```bash
$ appium plugin doctor images

info AppiumDoctor ### Diagnostic for images plugin ###
✔ OpenCV library is installed (version 4.5.0)
✔ Images plugin is correctly installed (version 2.1.0)
✔ Plugin configuration is valid
✔ Runtime environment is ready

info AppiumDoctor Everything looks good, bye!
```

**Scenario 2: OpenCV not installed**
```bash
$ appium plugin doctor images

info AppiumDoctor ### Diagnostic for images plugin ###
✖ OpenCV library is NOT installed
  → Fix: Install OpenCV using your package manager
     macOS: brew install opencv
     Linux: sudo apt-get install libopencv-dev
     Windows: choco install opencv

✔ Images plugin is correctly installed (version 2.1.0)
✔ Plugin configuration is valid
⚠ Runtime environment check skipped (OpenCV required)

info AppiumDoctor ✖ 1 required fix needed
```

**Scenario 3: Plugin not installed**
```bash
$ appium plugin doctor images

info AppiumDoctor ### Diagnostic for images plugin ###
✖ Images plugin is NOT installed
  → Fix: Run 'appium plugin install images'
  → Autofix available: Run with --fix flag to install automatically

info AppiumDoctor ✖ 1 required fix needed
```

### Using Autofix

Some Doctor checks support automatic fixes. To attempt automatic fixes, run:

```bash
appium plugin doctor images --fix
```

**Autofix Capabilities**:
- ✅ Can install missing plugin
- ✅ Can create default configuration files
- ✅ Can create required directories
- ❌ Cannot install system dependencies (OpenCV)
- ❌ Cannot fix system resource issues

**Example with Autofix**:
```bash
$ appium plugin doctor images --fix

info AppiumDoctor ### Diagnostic for images plugin ###
✖ Images plugin is NOT installed
  → Attempting autofix...
  → Running: appium plugin install images
  ✔ Plugin installed successfully

✔ OpenCV library is installed (version 4.5.0)
✔ Plugin configuration is valid
✔ Runtime environment is ready

info AppiumDoctor All issues fixed! Everything looks good, bye!
```

### Troubleshooting Doctor Checks

If Doctor checks fail or produce unexpected results:

1. **Update Appium and plugin to latest versions**:
   ```bash
   npm update -g appium
   appium plugin update images
   ```

2. **Run with verbose logging**:
   ```bash
   appium plugin doctor images --verbose
   ```

3. **Check Appium server logs**:
   ```bash
   appium server --log-level debug
   ```

4. **Verify system requirements**:
   - Node.js 14+ installed
   - Appium 2.0+ installed
   - Sufficient system resources (2GB+ RAM, 1GB+ disk space)

### Integration with CI/CD

Doctor checks can be integrated into CI/CD pipelines:

```yaml
# Example GitHub Actions workflow
- name: Install Appium and Images plugin
  run: |
    npm install -g appium
    appium plugin install images

- name: Run Doctor checks
  run: |
    appium plugin doctor images
    if [ $? -ne 0 ]; then
      echo "Doctor checks failed"
      exit 1
    fi
```

### Best Practices

1. **Run Doctor checks after installation**: Always run `appium plugin doctor images` after installing the plugin
2. **Run before test execution**: Include Doctor checks in your test setup to catch configuration issues early
3. **Monitor in CI/CD**: Add Doctor checks to your CI/CD pipeline to ensure consistent environment setup
4. **Keep dependencies updated**: Regularly update OpenCV and the Images plugin to avoid compatibility issues
5. **Review warnings**: Even if tests pass, review and address any warnings from Doctor checks

## Constraints
- Requires Appium 2.0+
- Requires OpenCV libraries (3.0+ or 4.0+)
- Plugin must be explicitly enabled when starting server using `--use-plugins=images`
- Supported image formats: PNG, JPEG, BMP
- Image comparison modes: matchTemplate, features, ssim
- Performance depends on image size and complexity
- Large images may cause memory issues
- Image matching accuracy depends on image quality and similarity threshold
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows)
