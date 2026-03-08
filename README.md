# Appium Agent Skills

This repository contains AI Agent skills for preparing Appium driver environments.

## Available Skills

| Skill | Description |
|---|---|
| [appium-node-environment-setup](skills/appium-node-environment-setup/SKILL.md) | Prepares Node.js and npm environment for Appium CLI commands |
| [appium-android-environment-setup](skills/appium-android-environment-setup/SKILL.md) | Prepares Android SDK, Java, and ADB prerequisites for Appium Android drivers |
| [appium-environment-setup-uiautomator2](skills/appium-environment-setup-uiautomator2/SKILL.md) | Prepares and validates an Android + UiAutomator2 Appium environment |
| [appium-environment-setup-xcuitest](skills/appium-environment-setup-xcuitest/SKILL.md) | Prepares and validates a macOS + XCUITest Appium environment |

## Reliable Execution Notes

- Run skill commands step-by-step rather than as one very long chained command.
- For long-running checks, prefer an isolated background terminal and capture output after completion.
- If output appears incomplete, rerun only the affected step and collect logs from that step.
- Treat Appium doctor as the source of truth for pass/fail (`0 required fixes needed`).
