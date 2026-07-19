# Keep a Cursor agent running on a Mac

**Short answer:** To keep a Cursor agent running on a Mac through a long task, you need to stop the machine from sleeping — an open editor alone does not hold a Mac awake. Turn on Cursor's Auto Mode so the agent keeps working without manual approvals, then hold a wake assertion for the duration. The one-liner is `caffeinate -disu`, but it won't survive a closed lid; for a lid-shut run, use [LidRun](https://lidrun.com/download)'s Auto Mode, which detects Cursor as a working agent and releases the Mac when the task finishes.

When you hand Cursor a big refactor — "migrate this module to the new API across 40 files" — it can churn for a long time: reading, editing, running the terminal, waiting on model responses. The problem isn't Cursor. It's that macOS decides your Mac looks idle (no keyboard, no mouse) and sleeps it out from under the agent. Below are the practical ways to keep the run alive.

## Why a merely-open editor doesn't keep the Mac awake

macOS sleep is driven by *user idle*, not by whether an app is open. Cursor sitting on screen with an agent mid-task counts as idle to the power manager if you're not touching the machine. When the display sleeps and then the system follows, the agent's terminal jobs and any local model calls stall.

Two things actually hold a Mac awake:

- **Real work** — sustained CPU, disk, or network activity from a process.
- **A wake assertion** — an explicit "don't sleep" flag held by a tool.

An agent waiting on an API response sits near 0% CPU, so "real work" alone is an unreliable signal — the exact moment the agent pauses to think is when a naive CPU check would let the Mac drop.

## Option 1: `caffeinate` for a lid-open run

If your lid stays open and you just want to block idle sleep while Cursor works, the built-in tool is enough:

```sh
# Prevent idle + display + system + disk sleep, indefinitely
caffeinate -disu
# Ctrl-C to release when the agent finishes
```

Bound it to a wall-clock limit instead:

```sh
# Keep awake for 4 hours (14400 seconds), then release automatically
caffeinate -disu -t 14400
```

Or tie the assertion to a specific command so it releases the instant that command exits — handy if you drive Cursor's agent from a script or run the heavy build yourself:

```sh
caffeinate -disu -- npm run build
```

`caffeinate` is fine at a desk. Its limits: it does nothing once you **close the lid** (the Mac still sleeps on clamshell), and it holds the assertion whether or not the agent is actually doing anything — you have to remember to kill it.

## Option 2: Closed lid — LidRun Auto Mode + Cursor

The common real-world case is: start a long Cursor task, close the laptop, walk away. That's where a plain wake-lock isn't enough, because clamshell sleep is a separate path from idle sleep.

1. Open Cursor and start your agent task (the refactor, the test sweep, the migration).
2. **Turn on Auto Mode** in Cursor so the agent runs the whole task without stopping to ask for approval on each step. A closed-lid run is pointless if the agent halts on the first confirmation.
3. In LidRun, enable **Auto Mode** and **Closed-Lid** mode.
4. Close the lid. LidRun holds the Mac awake and keeps the run going.

The reason this pairs well with agents: LidRun's Auto Mode treats **known agents as working by presence**. Cursor — along with Claude Code, Codex, Cline, Aider, Continue, Goose, Ollama, and LM Studio — counts as "working" even when it's idling at ~0% CPU waiting on a model response. So the assertion isn't dropped during the agent's think-time. When the task genuinely ends and the agent goes quiet, LidRun **auto-releases** and lets the Mac sleep — you're not left holding a wake-lock overnight.

Closed-lid mode is guarded, not a blind wake-lock: low-battery stop, charging-only mode, and thermal back-off can end the run early, and LidRun shows a **Why Awake / Why Stopped** reason so you know what happened. It reduces the risk of a Mac cooking in a bag — it doesn't remove physics, so keep it on a hard, ventilated surface, ideally plugged in.

Full walkthrough: [Keep a Cursor agent running on a Mac](https://lidrun.com/blog/keep-cursor-agent-running-on-mac).

## Option 3: A bounded session timer

If you know the job fits in a window — say a two-hour migration — set a **session timer** instead of an open-ended hold. LidRun supports timed sessions from 30 minutes to 8 hours; the Mac stays awake for that window, then releases on its own even if something wedges. It's the belt-and-suspenders version of `caffeinate -t`, but it survives a closed lid.

## Why "by presence" matters for agents

Long agent tasks are bursty: a spike of tool calls, then a quiet stretch waiting on the model, then another spike. A wake tool that only watches CPU will release during the quiet stretch and sleep the Mac mid-task. Recognizing the agent by presence is what keeps a Cursor run alive across those pauses — the same idea behind [AI agent continuity](https://lidrun.com/blog/what-is-ai-agent-continuity): keep the environment alive for as long as the agent is on the job, and only for that long.

## FAQ

**Does closing the lid stop a Cursor agent?**
By default, yes — closing the lid triggers clamshell sleep and the agent's jobs stall. `caffeinate` won't help here because it only blocks idle sleep. A guarded closed-lid tool like LidRun keeps the run going with the lid shut and releases when the task ends.

**Will Cursor keep working if my Mac stays awake but I walk away?**
It will, provided the agent doesn't stop for a confirmation. Enable **Auto Mode** in Cursor so it runs the full task unattended, and hold a wake assertion so the machine doesn't sleep while you're gone.

**Does an open Cursor window keep my Mac awake on its own?**
No. macOS sleeps on user-idle regardless of which apps are open. You need either sustained activity or an explicit wake assertion — an editor sitting idle provides neither reliably.

**How do I keep the Mac awake for just one command?**
Use `caffeinate -disu -- <command>`, or LidRun's CLI: `lidrun -- <command>` keeps the Mac awake for exactly that command and releases the moment it exits.

*[Download LidRun](https://lidrun.com/download) — free forever for simple sessions; Pro unlocks the full closed-lid AI workflow ([pricing](https://lidrun.com/pricing)).*
