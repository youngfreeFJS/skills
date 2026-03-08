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
- If a maintained Node version manager exists (`nvm`, `fnm`, `asdf`): use it as the primary way to install/switch Node versions.
- If no Node version manager is available: ask the user to adopt a Node version management tool before continuing with persistent setup.
- If `npm` is unavailable or unhealthy: repair npm environment before Appium install.
- If current Node does not satisfy Appium's required `engines.node` range: switch Node version (prefer version manager).
- If global install is blocked by permissions policy: switch to project-local install and use `npx appium`.
- If `npm doctor` reports recommendations only: treat them as advisory unless they block Appium install/run.

## Instructions
1. **Detect Node version management tools**
   ```bash
   command -v nvm || true
   command -v fnm || true
   command -v asdf || true
   ```
   Prefer this order when available: `fnm` / `nvm` / `asdf`.
   If none is available, ask the user whether to proceed with a version manager setup first (recommended) or continue with current system Node.

2. **Verify Node and npm availability**
   ```bash
   node -v
   npm -v
   npm config get prefix
   npm config get cache
   ```

3. **Validate npm health (advisory)**
   ```bash
   npm ping
   npm doctor
   ```
   Treat recommendations as non-blocking unless npm cannot install/run Appium commands.

4. **Check Appium-required Node version**
   ```bash
   node -p 'process.versions.node'
   npm view appium@latest engines --json
   ```
   If Appium is already installed, verify against the installed Appium version:
   ```bash
   APPIUM_VERSION=$(appium -v)
   npm view "appium@${APPIUM_VERSION}" engines --json
   npx -y semver "$(node -p 'process.versions.node')" -r "$(npm view \"appium@${APPIUM_VERSION}\" engines.node)"
   ```
   If semver check fails, switch Node version using the detected version manager.

5. **Prepare npm install strategy**
   Choose one strategy and keep it consistent:
   - **Global Appium CLI strategy** (default)
   - **Project-local Appium CLI strategy** (fallback for restricted global installs)

6. **Install Appium via npm (global strategy)**
   ```bash
   npm install -g appium
   appium -v
   npm ls -g --depth=0 appium
   ```

7. **Install Appium via npm (project-local strategy)**
   In a project directory:
   ```bash
   npm init -y
   npm install --save-dev appium
   npx appium -v
   npm ls appium
   ```

8. **Verify Appium npm commands used by downstream skills**
   Validate both command styles that other skills may use:
   ```bash
   appium driver list --installed
   npx appium driver list --installed
   ```

9. **Agent completion criteria**
   Mark complete only when all are true:
   - `node -v` and `npm -v` succeed
   - current Node version satisfies Appium `engines.node` requirement
   - At least one Appium execution mode works (`appium` or `npx appium`)
   - `appium driver list --installed` or `npx appium driver list --installed` succeeds

## Constraints
- Prefer Node.js LTS versions only.
- Prefer maintained version-managed Node installations (`fnm`, `nvm`, `asdf`) when available.
- Do not assume global npm install privileges; provide local fallback.
- Avoid using `sudo` in setup commands; prefer user-space tooling and local npm installs when permissions are restricted.
- Treat `npm doctor` recommendations as advisory unless they block Appium install/run.
- Always validate command availability immediately after installation.
- Do not proceed to driver-specific setup until npm/Appium command checks pass.
- Use deterministic terminal checks, not assumptions.
