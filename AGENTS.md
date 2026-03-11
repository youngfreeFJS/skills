# AGENTS Guide for Appium Skills

This file defines how AI agents should execute the skills in this repository.

## Execution Rules

- Execute skills one at a time in dependency order.
- Run commands step-by-step; avoid very long chained command blocks.
- Re-run checks after each fix.
- Use Appium doctor required fixes as the pass/fail gate:
  - Pass: `0 required fixes needed`
  - Optional warnings are non-blocking.
- Ask before installing optional dependencies.
- Avoid `sudo` in setup unless the user explicitly requests it.
- Prefer user-space installs and local project fallbacks when permissions are restricted.
- Use global npm/Appium commands by default (`npm -g`, `appium`).
- Use local execution (`npx appium`) only when the user explicitly asks for a local mode.
- Use `environment-setup-ffmpeg` as a shared optional dependency across drivers only when the user explicitly requests FFmpeg-related setup.
- Use `environment-setup-bundletool` as a shared optional dependency for UiAutomator2/Espresso only when the user explicitly requests bundletool setup.
- If output is incomplete/truncated, rerun only that step and capture logs.

## Recommended Skill Order

### Android + UiAutomator2

1. `environment-setup-node`
2. `environment-setup-android`
3. `environment-setup-uiautomator2`

### Android + Espresso

1. `environment-setup-node`
2. `environment-setup-android`
3. `environment-setup-espresso`

### Chrome/Chromium Browser

1. `environment-setup-node`
2. `environment-setup-chromium`

### Safari Browser (macOS/iOS)

1. `environment-setup-node`
2. `environment-setup-safari`

### macOS Application Automation (Mac2)

1. `environment-setup-node`
2. `environment-setup-mac2`

### Shared Optional Skill

1. `environment-setup-ffmpeg` (run only when user explicitly requests FFmpeg-related capabilities)
2. `environment-setup-bundletool` (run only when user explicitly requests bundletool setup for UiAutomator2/Espresso)

### iOS + XCUITest (macOS only)

1. `environment-setup-node`
2. `environment-setup-xcuitest`

## Completion Policy

A skill is complete only when its own completion criteria in `SKILL.md` are satisfied.

- Required doctor checks must pass.
- Optional doctor warnings do not block completion.
- Validate global command mode (`appium`) as the default completion path.
- Validate local command mode (`npx appium`) only when the user explicitly requests local execution.

## Prompt Templates

### Template: UiAutomator2

Use this as a starting prompt for an AI agent:

```text
Use this repository's skills to prepare Android + UiAutomator2.
Follow exactly, in order:
1) skills/environment-setup-node/SKILL.md
2) skills/environment-setup-android/SKILL.md
3) skills/environment-setup-uiautomator2/SKILL.md

Rules:
  1) Start Appium server in Terminal A (`appium server`) and keep it running.
  2) In Terminal B run `curl -s http://127.0.0.1:4723/status` and confirm success.
  3) In Terminal A logs confirm `Available drivers:` contains `uiautomator2`.
  4) In Terminal A stop Appium with `Ctrl+C`, then in Terminal B run `pgrep -fl "appium.*server" || echo "no appium server process"`.
```

### Template: Espresso

Use this as a starting prompt for an AI agent:

```text
Use this repository's skills to prepare Android + Espresso.
Follow exactly, in order:
1) skills/environment-setup-node/SKILL.md
2) skills/environment-setup-android/SKILL.md
3) skills/environment-setup-espresso/SKILL.md

Rules:
- Run one step at a time.
- Treat `appium driver doctor espresso` required fixes as blocking.
- Optional warnings are non-blocking.
- Ask before installing optional dependencies.
- Do not use sudo unless I explicitly ask.
- Show command output for each step.
- Smoke test sequence:
  1) Start Appium server in Terminal A (`appium server`) and keep it running.
  2) In Terminal B run `curl -s http://127.0.0.1:4723/status` and confirm success.
  3) In Terminal A logs confirm `Available drivers:` contains `espresso`.
  4) In Terminal A stop Appium with `Ctrl+C`, then in Terminal B run `pgrep -fl "appium.*server" || echo "no appium server process"`.
```

### Template: Chromium

Use this as a starting prompt for an AI agent:

```text
Use this repository's skills to prepare Chrome/Chromium browser automation.
Follow exactly, in order:
1) skills/environment-setup-node/SKILL.md
2) skills/environment-setup-chromium/SKILL.md

Rules:
- Run one step at a time.
- Treat `appium driver doctor chromium` required fixes as blocking.
- Optional warnings are non-blocking.
- Ask before installing optional dependencies.
- Do not use sudo unless I explicitly ask.
- Show command output for each step.
- Smoke test sequence:
  1) Start Appium server in Terminal A (`appium server`) and keep it running.
  2) In Terminal B run `curl -s http://127.0.0.1:4723/status` and confirm success.
  3) In Terminal A logs confirm `Available drivers:` contains `chromium`.
  4) In Terminal A stop Appium with `Ctrl+C`, then in Terminal B run `pgrep -fl "appium.*server" || echo "no appium server process"`.
```

### Template: Safari

Use this as a starting prompt for an AI agent:

```text
Use this repository's skills to prepare Safari browser automation on macOS/iOS.
Follow exactly, in order:
1) skills/environment-setup-node/SKILL.md
2) skills/environment-setup-safari/SKILL.md

Rules:
- Run one step at a time.
- Treat `appium driver doctor safari` required fixes as blocking.
- Optional warnings are non-blocking.
- Run `safaridriver --enable` with administrator password (mandatory one-time setup).
- For iOS Real Device: manually enable Remote Automation in Settings → Safari → Advanced.
- Ask before installing optional dependencies.
- Do not use sudo unless I explicitly ask.
- Show command output for each step.
- Smoke test sequence:
  1) Start Appium server in Terminal A (`appium server`) and keep it running.
  2) In Terminal B run `curl -s http://127.0.0.1:4723/status` and confirm success.
  3) In Terminal A logs confirm `Available drivers:` contains `safari`.
  4) In Terminal A stop Appium with `Ctrl+C`, then in Terminal B run `pgrep -fl "appium.*server" || echo "no appium server process"`.
```

### Template: Mac2

Use this as a starting prompt for an AI agent:

```text
Use this repository's skills to prepare macOS application automation with Mac2.
Follow exactly, in order:
1) skills/environment-setup-node/SKILL.md
2) skills/environment-setup-mac2/SKILL.md

Rules:
- Run one step at a time.
- Treat `appium driver doctor mac2` required fixes as blocking.
- Optional warnings are non-blocking.
- Requires macOS 11+ and Xcode 13+.
- Manually enable Xcode Helper Accessibility access (drag & drop to System Preferences).
- xcode-select must point to Xcode.app/Contents/Developer.
- Ask before installing optional dependencies.
- Do not use sudo unless I explicitly ask.
- Show command output for each step.
- Smoke test sequence:
  1) Start Appium server in Terminal A (`appium server`) and keep it running.
  2) In Terminal B run `curl -s http://127.0.0.1:4723/status` and confirm success.
  3) In Terminal A logs confirm `Available drivers:` contains `mac2`.
  4) In Terminal A stop Appium with `Ctrl+C`, then in Terminal B run `pgrep -fl "appium.*server" || echo "no appium server process"`.
```

### Template: XCUITest

Use this as a starting prompt for an AI agent:

```text
Use this repository's skills to prepare macOS + XCUITest.
Follow exactly, in order:
1) skills/environment-setup-node/SKILL.md
2) skills/environment-setup-xcuitest/SKILL.md

Rules:
- Run one step at a time.
- Treat `appium driver doctor xcuitest` required fixes as blocking.
- Optional warnings are non-blocking.
- Ask before installing optional dependencies.
- Do not use sudo unless I explicitly ask.
- Show command output for each step.
- Smoke test sequence:
  1) Start Appium server in Terminal A (`appium server`) and keep it running.
  2) In Terminal B run `curl -s http://127.0.0.1:4723/status` and confirm success.
  3) In Terminal A logs confirm `Available drivers:` contains `xcuitest`.
  4) In Terminal A stop Appium with `Ctrl+C`, then in Terminal B run `pgrep -fl "appium.*server" || echo "no appium server process"`.
```

## Notes for Tooling Integrations

- If your agent platform supports repository-level instruction files, prioritize this file before running skill commands.
- If your platform does not auto-load this file, copy one prompt template above and provide it manually.
