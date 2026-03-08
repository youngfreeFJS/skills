---
name: "appium-node-environment-setup"
description: "Prepare and validate Node.js and npm for Appium CLI usage"
metadata:
  model: "GPT-5.3-Codex"
  last_modified: "Sun, 08 Mar 2026 00:00:00 GMT"

---
# appium-node-environment-setup

## Goal
Prepares a stable Node.js and npm environment for Appium by validating the active Node runtime, configuring npm behavior, installing Appium through npm, and confirming both global and project-local Appium command execution paths.

## Decision Logic
- If `node` is missing or major version is `< 20`: install/upgrade Node.js to active LTS.
- If `npm` is unavailable or unhealthy: repair npm environment before Appium install.
- If global install is blocked by permissions policy: switch to project-local install and use `npx appium`.
- If `npm doctor` reports issues: resolve reported issues before moving to Appium driver setup.

## Instructions
1. **Verify Node and npm availability**
   ```bash
   node -v
   npm -v
   npm config get prefix
   npm config get cache
   ```

2. **Validate npm health**
   ```bash
   npm ping
   npm doctor
   ```
   If checks fail, fix npm config/network/auth issues and repeat until healthy.

3. **Prepare npm install strategy**
   Choose one strategy and keep it consistent:
   - **Global Appium CLI strategy** (default)
   - **Project-local Appium CLI strategy** (fallback for restricted global installs)

4. **Install Appium via npm (global strategy)**
   ```bash
   npm install -g appium
   appium -v
   npm ls -g --depth=0 appium
   ```

5. **Install Appium via npm (project-local strategy)**
   In a project directory:
   ```bash
   npm init -y
   npm install --save-dev appium
   npx appium -v
   npm ls appium
   ```

6. **Verify Appium npm commands used by downstream skills**
   Validate both command styles that other skills may use:
   ```bash
   appium driver list --installed
   npx appium driver list --installed
   ```

7. **Agent completion criteria**
   Mark complete only when all are true:
   - `node -v` and `npm -v` succeed
   - `npm doctor` has no blocking failures
   - At least one Appium execution mode works (`appium` or `npx appium`)
   - `appium driver list --installed` or `npx appium driver list --installed` succeeds

## Constraints
- Prefer Node.js LTS versions only.
- Do not assume global npm install privileges; provide local fallback.
- Avoid using `sudo` in setup commands; prefer user-space tooling and local npm installs when permissions are restricted.
- Always validate command availability immediately after installation.
- Do not proceed to driver-specific setup until npm/Appium command checks pass.
- Use deterministic terminal checks, not assumptions.
