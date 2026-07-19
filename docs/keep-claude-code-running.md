# Keep Claude Code (and Cursor, Codex, Docker) running with the lid closed

**Short version:** start your agent, turn on LidRun's closed-lid workflow, then close the MacBook. LidRun keeps macOS awake so Claude Code, Cursor, Codex, Docker or Ollama keep running — and releases the Mac automatically when the job finishes or a battery/thermal limit is hit.

The frustrating part is never that the agent crashed. It's that **macOS went to sleep** the moment you closed the lid, freezing the run halfway. LidRun exists to fix exactly that.

## The 30-second setup

1. Start your long-running job — e.g. a Claude Code refactor, a Cursor agent task, a Docker build, an Ollama eval.
2. Open the **LidRun** menu-bar icon → turn on **Auto Mode** (recommended) or **Keep Awake**.
   - **Auto Mode** watches your workload and only holds the Mac awake while it's actually working, then lets the Mac sleep when it's done.
3. Turn on the **closed-lid** workflow, then **close the lid** and walk away.
4. LidRun keeps the run alive with the lid shut, and releases the Mac when the work ends or a safety limit is reached.

## Why an agent at "0% CPU" isn't dropped

A coding agent often sits near 0% CPU while it waits on an API call or a long tool run. Naive keep-awake tools that watch CPU would let the Mac sleep right then. LidRun treats **known agents** (Claude Code, Cursor, Codex, Cline, Aider, Continue, Goose, Ollama, LM Studio and more) as *working by presence* — so a waiting agent is never dropped mid-task.

## Safety, honestly

Closing the lid removes the Mac's main cooling path, so LidRun keeps guardrails on:

- **Session timer** — cap the run from 30 minutes to 8 hours.
- **Charging-only mode** — stay awake only on mains power.
- **Low-battery stop** — release before the battery drains flat.
- **Thermal monitoring** — back off when the chip reports heat pressure.
- **Auto-release** — sleep the moment the workload finishes.

It reduces risk; it doesn't remove physics. Keep the Mac on a hard, ventilated surface, and don't run heavy jobs with the lid closed inside a bag. More: [Safety](https://lidrun.com/safety) · [MacBook gets hot while coding](https://lidrun.com/blog/macbook-gets-hot-while-coding) · [Low-battery auto-sleep](https://lidrun.com/blog/macbook-auto-sleep-low-battery).

## Prefer the terminal?

Keep the Mac awake for exactly one command:

```sh
lidrun -- <your-long-command>
```

LidRun holds the assertion only for that command and releases it when the command exits. More in the [CLI guide](https://lidrun.com/blog/lidrun-cli-keep-awake-terminal).

## Full, up-to-date guides (English)

- 📖 [Keep Claude Code running when the MacBook is closed](https://lidrun.com/blog/keep-claude-code-running-when-macbook-closed)
- 📖 [Keep a Cursor agent running on a Mac](https://lidrun.com/blog/keep-cursor-agent-running-on-mac)
- 📖 [Prevent Mac sleep during a Docker build](https://lidrun.com/blog/prevent-mac-sleep-during-docker-build)
- 📖 [Lid-closed developer workflow](https://lidrun.com/blog/macbook-lid-closed-developer-workflow)

---
*[Download LidRun](https://lidrun.com/download) — free forever for simple sessions; Pro unlocks the full closed-lid AI workflow ([pricing](https://lidrun.com/pricing)).*
