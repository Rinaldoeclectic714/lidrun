# Install LidRun on macOS (and open it the first time)

**Short version:** download the `.dmg` from [lidrun.com/download](https://lidrun.com/download), drag **LidRun** into **Applications**, open it, and click **Open** on the first-launch prompt. LidRun is signed with an Apple Developer ID and notarized by Apple, so it's a single click — no Terminal, no Privacy & Security workaround.

Requirements: **macOS 13 (Ventura) or later**, Apple Silicon or Intel.

## Install

Pick one:

### 1. Direct download (recommended)
1. Open **[lidrun.com/download](https://lidrun.com/download)**.
2. Open the downloaded `LidRun.dmg`.
3. Drag **LidRun** onto the **Applications** folder shortcut.
4. Open **LidRun** from Applications (or Spotlight). Its icon appears in the menu bar.

### 2. Homebrew
```sh
brew install --cask aibrickai/lidrun/lidrun
```
This installs the same signed, notarized build. LidRun updates itself after that (built-in auto-update), so you don't need to `brew upgrade` for new versions.

### 3. GitHub Releases
Download `LidRun.dmg` from the [latest release](https://github.com/aibrickai/lidrun/releases/latest), then follow the drag-to-Applications steps above.

## If the app won't open

macOS shows a one-time check for **any** app downloaded from the internet. Because LidRun is Developer-ID-signed and notarized, you should only ever see the friendly prompt below — not the scary *"cannot be opened because the developer cannot be verified"* one.

- **"LidRun is an app downloaded from the Internet. Are you sure you want to open it?"** → click **Open**. Done.
- If you *do* see a blocked/verify message (rare — usually an old or partial download), **right-click** (or Control-click) the app in Applications → **Open** → **Open**. That approves it once, permanently.
- Still stuck? Re-download a fresh copy from [lidrun.com/download](https://lidrun.com/download) — a truncated download is the usual cause — or [open an issue](https://github.com/aibrickai/lidrun/issues/new/choose).

You should **not** need to disable Gatekeeper, run `xattr`, or change Privacy & Security settings. If a guide tells you to, you probably have a tampered copy — get the notarized build from the official [download page](https://lidrun.com/download).

## First run

1. LidRun lives in the **menu bar** (top-right), not the Dock.
2. Click the icon → choose a mode: **Keep Awake**, **Only When Charging**, **Timer**, or **Auto Mode** (holds the Mac awake only while your workload is actually working).
3. To keep a job running with the lid shut, turn on the closed-lid workflow, start your task, then close the lid.

Next: **[Keep Claude Code (or Cursor / Docker) running with the lid closed →](keep-claude-code-running.md)**

---
*Full documentation and up-to-date guides: [lidrun.com](https://lidrun.com) · Pricing (free forever + Pro): [lidrun.com/pricing](https://lidrun.com/pricing)*
