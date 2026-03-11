---
name: "environment-setup-android"
description: "Prepare and validate Android SDK, Java, and device tooling for Appium Android drivers"
metadata:
  last_modified: "Tue, 11 Mar 2026 19:35:00 GMT"

---
# environment-setup-android

## Goal
Prepares a working Android automation environment for Appium by validating Java, Android SDK command-line tools, required SDK packages, environment variables, and ADB device visibility, with a verify-and-fix loop until all mandatory checks pass.

## Decision Logic
- If host OS is unsupported for Android SDK setup: stop and ask the user to switch to macOS, Linux, or Windows.
- If host OS is macOS: prioritize Android Studio app setup (`/Applications/Android Studio.app`) for both `ANDROID_HOME` (`$HOME/Library/Android/sdk`) and `JAVA_HOME` (Android Studio JBR).
- If host OS is macOS and Android Studio app is not present: use Homebrew-based setup for SDK tools and Java.
- If host OS is Linux: use package manager + `$HOME/Android/Sdk` conventions.
- If host OS is Windows: use Android SDK tools with persistent user environment variables.
- If `java` or `javac` is missing: install a supported JDK and configure `JAVA_HOME`.
- If the user wants official Android tooling setup flow: install Android Studio from the official site first, then use SDK Manager from Android Studio.
- If `ANDROID_HOME` is unset/empty or the `ANDROID_HOME` path does not exist: run step 2 to install command-line tools and create the SDK path.
- If `JAVA_HOME` is unset/empty or the `JAVA_HOME` path does not exist: run step 3 before Android SDK package/license commands.
- If `adb` is missing: install `platform-tools` via `sdkmanager`.
- If emulator binary is missing under `ANDROID_HOME/emulator/emulator` (or Windows equivalent): install emulator packages.
- Prepare emulator instances using the latest stable system-image version by default.
- Use host-optimized emulator architecture (native architecture first, then fallback architecture).
- Skip step 7 emulator preparation if at least one device is already connected or at least one emulator instance already exists.
- If required SDK packages are missing: install them and re-run checks.
- If testing apps targeting Android 13+ (API 33+): configure runtime permissions including POST_NOTIFICATIONS.
- If Gradle version incompatibility detected: verify Gradle and Android Gradle Plugin (AGP) compatibility matrix.
- If host is Apple Silicon (ARM64): prioritize ARM64 emulator images and enable hardware acceleration for optimal performance.

## Instructions
1. **Detect OS and validate Java/base tooling**
   macOS/Linux:
   ```bash
   uname -s
   java -version
   javac -version
   echo "$JAVA_HOME"
   command -v adb
   ls "$ANDROID_HOME/emulator/emulator"
   test -x "$ANDROID_HOME/emulator/emulator" && echo "emulator binary: OK"
   ```
   Windows PowerShell:
   ```powershell
   [System.Environment]::OSVersion.VersionString
   java -version
   javac -version
   $env:JAVA_HOME
   Get-Command adb.exe -ErrorAction SilentlyContinue
   Test-Path "$env:ANDROID_HOME\emulator\emulator.exe"
   ```

2. **Install Android SDK tooling when `ANDROID_HOME` path is missing**
   Trigger checks:
   - macOS/Linux:
   ```bash
   [ -n "$ANDROID_HOME" ] && [ -d "$ANDROID_HOME" ] || echo "run step 2"
   ```
   - Windows PowerShell:
   ```powershell
   if (-not $env:ANDROID_HOME -or -not (Test-Path $env:ANDROID_HOME)) { "run step 2" }
   ```
   Option A (Android Studio app path; macOS priority when app exists):
   - Download Android Studio from `https://developer.android.com/studio`.
   - Complete first launch and install SDK components from SDK Manager.
   - Use platform default SDK path after setup:
     - macOS: `$HOME/Library/Android/sdk`
     - Linux: `$HOME/Android/Sdk`
     - Windows: `%LOCALAPPDATA%\Android\Sdk`

   Option B (CLI tools only):
   - macOS/Homebrew example:
   ```bash
   [ -d "/Applications/Android Studio.app" ] || brew install --cask android-commandlinetools
   mkdir -p "$HOME/Library/Android/sdk/cmdline-tools/latest"
   cp -R /opt/homebrew/share/android-commandlinetools/* "$HOME/Library/Android/sdk/cmdline-tools/latest/"
   ```
   - Linux example (Debian/Ubuntu-style prerequisites + cmdline tools placement):
   ```bash
   sudo apt-get update
   sudo apt-get install -y unzip wget openjdk-21-jdk
   mkdir -p "$HOME/Android/Sdk/cmdline-tools/latest"
   ```
   - Windows example (PowerShell, after extracting Android command-line tools zip):
   ```powershell
   New-Item -ItemType Directory -Force "$env:LOCALAPPDATA\Android\Sdk\cmdline-tools\latest"
   ```

3. **Configure `JAVA_HOME` when `JAVA_HOME` path is missing**
   Trigger checks:
   - macOS/Linux:
   ```bash
   [ -n "$JAVA_HOME" ] && [ -d "$JAVA_HOME" ] || echo "run step 3"
   ```
   - Windows PowerShell:
   ```powershell
   if (-not $env:JAVA_HOME -or -not (Test-Path $env:JAVA_HOME)) { "run step 3" }
   ```
   macOS (priority: Android Studio JBR, fallback: Homebrew JDK 21):
   ```bash
   if [ -d "/Applications/Android Studio.app/Contents/jbr/Contents/Home" ]; then
     export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
   else
     brew install --cask temurin
     export JAVA_HOME="$(/usr/libexec/java_home -v 21)"
   fi
   export PATH="$JAVA_HOME/bin:$PATH"
   java -version
   javac -version
   ```
   Linux (OpenJDK 21 example):
   ```bash
   export JAVA_HOME="/usr/lib/jvm/java-21-openjdk-amd64"
   export PATH="$JAVA_HOME/bin:$PATH"
   java -version
   javac -version
   ```
   Windows PowerShell (persist for current user):
   ```powershell
   [Environment]::SetEnvironmentVariable('JAVA_HOME', "$env:LOCALAPPDATA\Programs\Android Studio\jbr", 'User')
   $currentPath = [Environment]::GetEnvironmentVariable('Path', 'User')
   if ($currentPath -notlike "*$env:LOCALAPPDATA\Programs\Android Studio\jbr\bin*") {
     [Environment]::SetEnvironmentVariable('Path', "$currentPath;$env:LOCALAPPDATA\Programs\Android Studio\jbr\bin", 'User')
   }
   $env:JAVA_HOME = [Environment]::GetEnvironmentVariable('JAVA_HOME', 'User')
   java -version
   javac -version
   ```

4. **Configure Android environment variables and PATH**
   macOS (priority: Android Studio SDK path):
   ```bash
   export ANDROID_HOME="$HOME/Library/Android/sdk"
   export PATH="$ANDROID_HOME/platform-tools:$ANDROID_HOME/cmdline-tools/latest/bin:$PATH"
   echo "$ANDROID_HOME"
   command -v adb
   ls "$ANDROID_HOME/emulator/emulator"
   ```
   Linux:
   ```bash
   export ANDROID_HOME="$HOME/Android/Sdk"
   export PATH="$ANDROID_HOME/platform-tools:$ANDROID_HOME/cmdline-tools/latest/bin:$PATH"
   echo "$ANDROID_HOME"
   command -v adb
   ls "$ANDROID_HOME/emulator/emulator"
   ```
   Windows PowerShell (persist for current user):
   ```powershell
   [Environment]::SetEnvironmentVariable('ANDROID_HOME', "$env:LOCALAPPDATA\Android\Sdk", 'User')
   $androidPaths = "$env:LOCALAPPDATA\Android\Sdk\platform-tools;$env:LOCALAPPDATA\Android\Sdk\cmdline-tools\latest\bin"
   $currentPath = [Environment]::GetEnvironmentVariable('Path', 'User')
   if ($currentPath -notlike "*$androidPaths*") {
     [Environment]::SetEnvironmentVariable('Path', "$currentPath;$androidPaths", 'User')
   }
   $env:ANDROID_HOME = [Environment]::GetEnvironmentVariable('ANDROID_HOME', 'User')
   ```

5. **Accept SDK licenses and install required packages**
   macOS/Linux:
   ```bash
   if command -v sdkmanager >/dev/null 2>&1; then yes | sdkmanager --licenses; fi
   if command -v sdkmanager >/dev/null 2>&1; then sdkmanager "platform-tools" "build-tools;34.0.0" "platforms;android-34" "emulator"; fi
   ```
   Windows PowerShell:
   ```powershell
   if (Get-Command sdkmanager.bat -ErrorAction SilentlyContinue) { cmd /c "echo y| sdkmanager.bat --licenses" }
   if (Get-Command sdkmanager.bat -ErrorAction SilentlyContinue) { sdkmanager.bat "platform-tools" "build-tools;34.0.0" "platforms;android-34" "emulator" }
   ```

5.5. **Verify Gradle compatibility (for Android app projects)**
   If working with an Android app project, verify Gradle and AGP compatibility:
   macOS/Linux:
   ```bash
   # Check Gradle version in project
   if [ -f "gradle/wrapper/gradle-wrapper.properties" ]; then
     echo "Gradle wrapper configuration:"
     grep "distributionUrl" gradle/wrapper/gradle-wrapper.properties
   fi
   
   # Check Gradle version if gradlew exists
   if [ -f "./gradlew" ]; then
     ./gradlew --version
   fi
   
   # Verify AGP compatibility
   # AGP 8.x requires Gradle 8.0+
   # AGP 7.x requires Gradle 7.0+
   # AGP 4.2+ requires Gradle 6.7.1+
   ```
   Windows PowerShell:
   ```powershell
   # Check Gradle version in project
   if (Test-Path "gradle\wrapper\gradle-wrapper.properties") {
     Write-Host "Gradle wrapper configuration:"
     Select-String -Path "gradle\wrapper\gradle-wrapper.properties" -Pattern "distributionUrl"
   }
   
   # Check Gradle version if gradlew.bat exists
   if (Test-Path ".\gradlew.bat") {
     .\gradlew.bat --version
   }
   ```

6. **Verify Android SDK and ADB state**
   macOS/Linux:
   ```bash
   if command -v sdkmanager >/dev/null 2>&1; then sdkmanager --list | head -n 80; fi
   adb version
   ls "$ANDROID_HOME/emulator/emulator"
   test -x "$ANDROID_HOME/emulator/emulator" && echo "emulator binary: OK"
   ```
   Windows PowerShell:
   ```powershell
   if (Get-Command sdkmanager.bat -ErrorAction SilentlyContinue) { sdkmanager.bat --list }
   adb.exe version
   Test-Path "$env:ANDROID_HOME\emulator\emulator.exe"
   ```

6.5. **Configure Android 13+ runtime permissions (if testing API 33+ apps)**
   For apps targeting Android 13 (API 33) or higher, configure runtime permissions:
   macOS/Linux:
   ```bash
   # Check connected device/emulator API level
   adb shell getprop ro.build.version.sdk
   
   # For API 33+, grant POST_NOTIFICATIONS permission if needed
   # Replace <package> with your app's package name
   # adb shell pm grant <package> android.permission.POST_NOTIFICATIONS
   
   # List all runtime permissions for a package
   # adb shell dumpsys package <package> | grep permission
   ```
   Windows PowerShell:
   ```powershell
   # Check connected device/emulator API level
   adb.exe shell getprop ro.build.version.sdk
   
   # For API 33+, grant POST_NOTIFICATIONS permission if needed
   # Replace <package> with your app's package name
   # adb.exe shell pm grant <package> android.permission.POST_NOTIFICATIONS
   ```
   
   Note: Android 13+ introduced new runtime permissions including:
   - `POST_NOTIFICATIONS` for notification posting
   - Granular media permissions (`READ_MEDIA_IMAGES`, `READ_MEDIA_VIDEO`, `READ_MEDIA_AUDIO`)
   - `NEARBY_WIFI_DEVICES` for Wi-Fi device discovery

7. **Optional emulator instance preparation (if no physical device is connected and no emulator exists)**
   Skip step 7 when either of the following is true:
   - At least one device is already connected (`adb devices` shows a `device` entry)
   - At least one emulator instance already exists (`emulator -list-avds` is non-empty)

   Prepare an emulator instance using the latest stable system-image version and host-optimized architecture only when both are false.
   macOS/Linux (prefer native architecture first, then fallback):
   ```bash
   ARCH=$(uname -m)
   if [ "$ARCH" = "arm64" ] || [ "$ARCH" = "aarch64" ]; then
     PRIMARY_ARCH="arm64-v8a"
     FALLBACK_ARCH="x86_64"
   else
     PRIMARY_ARCH="x86_64"
     FALLBACK_ARCH="arm64-v8a"
   fi
   LATEST_API=$(sdkmanager --list | grep -o "system-images;android-[0-9]\+;google_apis;${PRIMARY_ARCH}" | sed 's/.*android-\([0-9]\+\).*/\1/' | sort -n | tail -1)
   IMAGE_ARCH="$PRIMARY_ARCH"
   if [ -z "$LATEST_API" ]; then
     LATEST_API=$(sdkmanager --list | grep -o "system-images;android-[0-9]\+;google_apis;${FALLBACK_ARCH}" | sed 's/.*android-\([0-9]\+\).*/\1/' | sort -n | tail -1)
     IMAGE_ARCH="$FALLBACK_ARCH"
   fi
   IMAGE="system-images;android-${LATEST_API};google_apis;${IMAGE_ARCH}"
   sdkmanager "$IMAGE"
   echo "no" | avdmanager create avd -n "api${LATEST_API}-google-${IMAGE_ARCH}" -k "$IMAGE"
   emulator -list-avds
   ```
   Windows PowerShell (prefer x86_64, fallback arm64-v8a):
   ```powershell
   $primaryArch = "x86_64"
   $fallbackArch = "arm64-v8a"
   $matches = sdkmanager.bat --list | Select-String "system-images;android-[0-9]+;google_apis;$primaryArch"
   $imageArch = $primaryArch
   if (-not $matches) {
     $matches = sdkmanager.bat --list | Select-String "system-images;android-[0-9]+;google_apis;$fallbackArch"
     $imageArch = $fallbackArch
   }
   $latestApi = ($matches | ForEach-Object { [int]([regex]::Match($_.Line, 'android-(\d+)').Groups[1].Value) } | Sort-Object)[-1]
   $image = "system-images;android-$latestApi;google_apis;$imageArch"
   sdkmanager.bat $image
   cmd /c "echo no| avdmanager.bat create avd -n api$latestApi-google-$imageArch -k $image"
   emulator.exe -list-avds
   ```
   Report version details in the task result:
   - macOS/Linux:
   ```bash
   emulator -version
   emulator -list-avds
   if command -v sdkmanager >/dev/null 2>&1; then sdkmanager --list | grep "system-images;android-" | head -n 20; fi
   ```
   - Windows PowerShell:
   ```powershell
   emulator.exe -version
   emulator.exe -list-avds
   if (Get-Command sdkmanager.bat -ErrorAction SilentlyContinue) { sdkmanager.bat --list | Select-String "system-images;android-" | Select-Object -First 20 }
   ```

7.5. **ARM64 emulator optimization (Apple Silicon Mac)**
   On Apple Silicon (M1/M2/M3) Macs, optimize emulator performance:
   ```bash
   # Verify Apple Silicon architecture
   if [ "$(uname -m)" = "arm64" ]; then
     echo "Apple Silicon detected - ARM64 emulator optimization available"
     
     # Start emulator with hardware acceleration
     # emulator @avd_name -accel on -gpu host
     
     # Configure memory and CPU cores for better performance
     # emulator @avd_name -memory 4096 -cores 4
     
     # For maximum performance, combine options:
     # emulator @avd_name -accel on -gpu host -memory 4096 -cores 4
     
     # Verify hardware acceleration is available
     emulator -accel-check
   fi
   ```
   
   Performance tips for ARM64 emulators:
   - Use ARM64 system images (`arm64-v8a`) for native performance
   - Enable hardware acceleration with `-accel on`
   - Use host GPU rendering with `-gpu host`
   - Allocate sufficient memory (4GB recommended for modern apps)
   - Assign multiple CPU cores based on host capabilities

8. **Completion criteria**
   Mark complete only when all are true:
   - `java -version` and `javac -version` succeed
   - `adb` is executable from `PATH`
   - Emulator binary exists under `ANDROID_HOME/emulator/emulator` (or `%ANDROID_HOME%\emulator\emulator.exe` on Windows)
   - Required SDK packages are installed (`platform-tools`, one platform, one build-tools version)
   - Android environment checks pass without requiring a connected device
   - Latest stable emulator/system-image version is prepared with host-optimized architecture only when no connected devices and no existing emulators are present; otherwise step 7 is skipped and current version details are reported in the task result

## Constraints
- Always use detect-first behavior and install only missing components.
- Re-run validation commands after each install/config change.
- Do not report success if `adb` is unavailable or emulator binary check fails.
- Treat optional dependencies and optional doctor warnings as non-blocking unless the user requests those features.
- Ask the user before installing optional dependencies; do not install them by default.
- If privileged commands are needed, pause and provide exact commands for user execution.
- Keep Android setup independent from Appium driver installation steps.
- Use shell-appropriate commands (`bash` for macOS/Linux, PowerShell/cmd for Windows).
- **Gradle Compatibility**: Ensure Gradle version matches AGP requirements (AGP 8.x requires Gradle 8.0+, AGP 7.x requires Gradle 7.0+).
- **Android 13+ Permissions**: For API 33+ apps, be aware of new runtime permissions (POST_NOTIFICATIONS, granular media permissions). Configure permissions via `adb shell pm grant` as needed.
- **ARM64 Emulator Optimization**: On Apple Silicon Macs, prioritize ARM64 system images and enable hardware acceleration (`-accel on -gpu host`) for optimal performance. ARM64 emulators run natively and significantly outperform x86_64 emulators under Rosetta 2 translation.
