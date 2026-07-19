# Clamshell mode on a MacBook (with or without an external display)

**Short answer:** *clamshell mode* is running a MacBook with the lid closed. The classic version needs an **external display, keyboard/mouse, and power** — close the lid and the Mac keeps working on the external screen. If you want the MacBook to keep running with the lid shut **and no external display** — for a build, a Docker job, or an AI agent — you need a keep-awake tool such as **[LidRun](https://lidrun.com/download)**, because macOS otherwise sleeps the moment you close the lid on battery or with no display attached.

## Classic clamshell mode (external display)

1. Plug the MacBook into power.
2. Connect an external display and a keyboard/mouse (or use them over Bluetooth).
3. Close the lid — the Mac stays awake and drives the external screen.

This is Apple's supported "desktop" setup. It does **not** keep the Mac awake if there's no display attached, and it's not designed for leaving a job running unattended.

## Clamshell mode *without* an external display

Closing the lid with nothing plugged in normally sends the Mac to sleep. To keep a workload running in that state you hold a power assertion (and, for a truly closed lid, toggle the documented `pmset` sleep setting) — which is exactly what **[LidRun](https://lidrun.com/download)** does, with safety guardrails around it. See the full walkthrough: **[Clamshell mode on a Mac](https://lidrun.com/blog/clamshell-mode-on-mac)** and **[keep a Mac awake when the lid is closed](https://lidrun.com/blog/keep-mac-awake-when-lid-closed)**.

## Why do this? The closed-lid AI/dev workflow

The modern reason to run clamshell without a monitor: you handed a long job to an AI agent or a build and want to close the MacBook and leave — without the run dying halfway. That's the [closed-lid AI workflow](https://lidrun.com/blog/what-is-closed-lid-ai-workflow). LidRun keeps the run alive with the lid shut and **auto-releases** the Mac when the work finishes or a battery/thermal limit is hit — so it's not a blind wake-lock. → [Lid-closed developer workflow](https://lidrun.com/blog/macbook-lid-closed-developer-workflow)

## Safety with the lid closed

A closed lid removes the Mac's main cooling path, so keep it on a hard, ventilated surface, don't run heavy jobs in a bag, and prefer a tool with a **low-battery stop** and **thermal back-off**. It reduces risk; it doesn't remove physics. → [Safety](https://lidrun.com/safety)

## FAQ

**What is clamshell mode on a MacBook?**
Running the MacBook with the lid closed. Traditionally it needs an external display and power; with a keep-awake tool you can also run closed-lid with no display for unattended jobs.

**How do I use clamshell mode without an external monitor?**
macOS sleeps when you close the lid with no display, so you need a tool that holds the Mac awake — see [keep a Mac awake when the lid is closed](https://lidrun.com/blog/keep-mac-awake-when-lid-closed) or get [LidRun](https://lidrun.com/download).

**Is clamshell mode safe overnight?**
On power and well ventilated it's generally fine; with the lid closed, watch heat and battery and use a tool that stops on low battery and backs off on heat. → [Safety](https://lidrun.com/safety)

---
*Full guides: [lidrun.com](https://lidrun.com) · Get the app: [lidrun.com/download](https://lidrun.com/download)*
