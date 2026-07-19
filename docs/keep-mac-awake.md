# How to keep your Mac awake

**Short answer:** To keep your Mac awake, the simplest built-in option is `caffeinate` in Terminal — it holds the display or system awake until you stop it. But an unconditional wake-lock drains the battery when nothing is running. The better pattern for long jobs is *workload-aware* keep-awake: stay awake only while there's actual work, then let the Mac sleep. This guide covers both, and when to pick which.

## The real question: awake *always* or awake *while working*?

Most "keep macbook awake" advice hands you a switch that pins the machine on until you flip it back. That's fine for a demo or a download you're watching. It's the wrong default for anything long-running, because the Mac stays fully awake for hours after the work finished — burning battery, and sometimes heat, for nothing.

Two mental models:

- **Unconditional** — "stay awake, period." You are responsible for turning it off.
- **Workload-aware** — "stay awake *only while my job is doing something*, then release." The tool is responsible for turning it off.

Pick based on whether you'll be there to flip the switch back.

## Quick method: `caffeinate` (built in)

`caffeinate` ships with macOS. No install.

```sh
# Keep the system awake until you press Ctrl-C
caffeinate

# Keep it awake while a specific command runs, then release automatically
caffeinate -i make build

# Keep the *display* awake too (as if you nudged the mouse)
caffeinate -d

# Awake for exactly 2 hours (7200 seconds), then stop on its own
caffeinate -t 7200
```

The `caffeinate <command>` form is already workload-aware in the crudest sense: it releases the assertion the moment the command exits. That's the cleanest built-in pattern — bind the wake-lock to a process, not to wall-clock time.

Flags worth knowing: `-i` prevent idle sleep, `-d` prevent display sleep, `-s` prevent sleep on AC power, `-m` prevent disk idle, `-u` declare user activity, `-t` timeout in seconds.

## Quick method: `pmset` (persistent, needs sudo)

`pmset` changes power settings rather than holding a temporary assertion:

```sh
# See current power assertions and what's holding sleep off
pmset -g assertions

# Never sleep on AC power (0 = never). Persists until you change it back.
sudo pmset -c sleep 0

# Restore a sane default (e.g. 10 minutes) when you're done
sudo pmset -c sleep 10
```

`pmset -c` only affects AC (charger) power, which is often exactly what you want on a desk. The catch: it's a *setting*, not a session — easy to set and forget, so you leave a machine that never sleeps. Prefer `caffeinate` for anything temporary.

## The gap: idle drain and the closed lid

Both built-ins have two blind spots for real dev work:

1. **They don't know if your job is idle.** `caffeinate` (bare) keeps a laptop awake whether your build is compiling or finished an hour ago. On battery, that's wasted charge.
2. **They don't survive a closed lid on their own.** Close the lid and a MacBook sleeps regardless of `caffeinate -i`; keeping a job alive with the lid shut needs a separate, deliberate mechanism.

For a long agent run or overnight job you walk away from, you usually want: *stay awake while the work is genuinely active, tolerate the lid being closed, and stop the moment it's safe.*

## Workload-aware keep-awake (Auto Mode)

[LidRun](https://lidrun.com/download) is a macOS menu-bar app built around this pattern. Its **Auto Mode** holds the Mac awake only while your workload is actually working, then releases it — so an idle machine goes back to sleep instead of draining.

The nuance that matters for AI/dev work: known agents (Claude Code, Cursor, Codex, Cline, Aider, Continue, Goose, Ollama, LM Studio, and similar) count as "working" *by presence*. An agent sitting at ~0% CPU while it waits on an API response isn't mistaken for idle and dropped mid-task — a naive CPU-threshold tool would kill the wake-lock right when the agent is waiting on the model.

For the concept in depth, see [what "keeping your Mac awake" actually does](https://lidrun.com/blog/what-is-keeping-your-mac-awake), and [why an agent should keep the Mac awake only while working](https://lidrun.com/blog/auto-keep-awake-only-while-working) — the full guides.

It also has a CLI that mirrors the `caffeinate <command>` idea — bind the wake-lock to one command, release when it exits:

```sh
# Keep the Mac awake for exactly this job, then release
lidrun -- ./train.sh
```

Beyond Auto Mode, LidRun adds a guarded **closed-lid workflow** (keep a job running with the lid shut — not a blind wake-lock), a **session timer** from 30 minutes to 8 hours, and safety guardrails: low-battery stop, charging-only mode, thermal back-off, and auto-release when work ends. It also shows *why* it's awake and *why* it stopped. A closed lid traps heat, so keep the machine ventilated — guardrails reduce risk, they don't remove physics. For a comparison of options for this audience, see [the best Mac keep-awake app for developers](https://lidrun.com/blog/best-mac-keep-awake-app-for-developers).

## Which one should I use?

- **Keep it simple, you're at the keyboard** → `caffeinate` (or `caffeinate <command>` to auto-release).
- **Leave a long job and walk away** → a workload-aware tool (Auto Mode) so an idle machine sleeps and a closed lid doesn't kill the run.
- **Desk on power, want it to never nap** → `sudo pmset -c sleep 0` (and remember to set it back).

## FAQ

**How do I keep my MacBook awake with the lid closed?**
The built-ins don't do this on their own — a MacBook sleeps when the lid shuts. You need a tool that explicitly supports a closed-lid workflow. LidRun offers a guarded closed-lid mode; keep the machine ventilated since a shut lid traps heat.

**Does keeping my Mac awake drain the battery?**
An unconditional wake-lock does — it holds the machine on even when nothing is running. Workload-aware keep-awake avoids most of that by releasing when the job goes idle, letting the Mac sleep normally.

**How do I keep my Mac awake for a specific command only?**
Use `caffeinate ./your-command` or `lidrun -- ./your-command`. Both hold the Mac awake for exactly that process and release the instant it exits — no leftover wake-lock.

**How do I stop keeping my Mac awake?**
For `caffeinate`, press Ctrl-C (or the process exits on its own). For `pmset`, set the value back, e.g. `sudo pmset -c sleep 10`. Check what's still holding sleep off with `pmset -g assertions`.

*[Download LidRun](https://lidrun.com/download) — free forever for simple sessions; Pro unlocks the full closed-lid AI workflow ([pricing](https://lidrun.com/pricing)).*
