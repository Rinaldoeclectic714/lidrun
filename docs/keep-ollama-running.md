# Keep Ollama (a local LLM) running on a MacBook

**Short answer:** To keep Ollama running on a MacBook, wrap the command in `caffeinate -i -- ollama run <model>` for a single job, or run `caffeinate -is` alongside a persistent `ollama serve`. The Mac then won't idle-sleep mid-generation. To keep inference going with the **lid closed** or fully unattended, use [LidRun](https://lidrun.com/download) — its Auto Mode detects `ollama`/`llama-server` as live work and holds the Mac awake only while a job is actually running.

Local inference is bursty: a long `ollama run`, a batch loop against `ollama serve`, or an overnight generation can run for minutes to hours with the CPU/GPU near-idle between tokens or waiting on the next request. macOS reads that as "nothing's happening" and puts the machine to sleep. This is a dev recipe for keeping the model alive until the job is done — no more, no less.

## Option 1: `caffeinate` for a single `ollama run`

`caffeinate` ships with macOS. The `-i` flag prevents idle sleep, and `-- <command>` scopes the assertion to exactly that process — when the command exits, the wake-lock is released automatically.

```sh
# Keep the Mac awake for exactly one generation, then release
caffeinate -i -- ollama run llama3.1 "Summarize this 40-page transcript: $(cat transcript.txt)"

# A longer eval that pipes prompts in
caffeinate -i -- ollama run qwen2.5-coder < prompts.txt
```

The useful flags for local LLM work:

- `-i` — prevent **idle** system sleep (the one you almost always want)
- `-s` — prevent sleep on AC power
- `-d` — keep the **display** awake too (rarely needed for headless inference)
- `-u` — declare user activity (resets the idle timer)

```sh
# Serve a batch job: hold the machine awake while the API is up
caffeinate -is ollama serve
```

`caffeinate` covers the lid-open case cleanly. What it does **not** do: keep the Mac awake once you **close the lid** — macOS clamshell sleep overrides the idle assertion. For that you need the documented `pmset` clamshell toggle, which requires admin rights and has no automatic release.

## Option 2: LidRun for closed-lid and unattended jobs

If you want to close the lid and walk away — an overnight generation, a batch job feeding `ollama serve`, or a long eval you don't want to babysit — [LidRun](https://lidrun.com/download) is a menu-bar app built for exactly this.

- **Auto Mode** watches for known workloads and holds the Mac awake *only while they're actually working*. `ollama` and `llama-server` are recognized by presence, so a model sitting at ~0% CPU between requests (or blocked on the next prompt) is **not** treated as idle and dropped. When the job exits, LidRun releases the Mac on its own.
- **Closed-Lid workflow** keeps inference running with the lid shut. It's guarded, not a blind wake-lock — the safety guardrails below still apply.
- **Session timer** caps a run from 30 minutes to 8 hours if you'd rather bound it explicitly.

Auto Mode also recognizes coding agents (Claude Code, Cursor, Codex, Cline, Aider, Continue, Goose) and LM Studio, so a workflow that pairs a local model with an agent stays awake as one unit.

For the CLI-scoped pattern — awake for exactly one command, then released, same shape as `caffeinate --` — LidRun ships a `lidrun` command:

```sh
lidrun -- ollama run llama3.1 "Generate 500 test cases as JSON"
```

The [full guide to keeping Ollama running](https://lidrun.com/blog/keep-ollama-running-on-macbook) walks through the closed-lid setup in more detail.

## Thermal reality of closed-lid inference

Sustained inference is one of the heavier things you can ask a MacBook to do, and running it with the lid shut traps heat against the panel. LidRun applies thermal back-off — it eases off when the machine gets too hot — but keeping the machine cool is still on you. This reduces risk; it doesn't remove physics.

Practical steps for an overnight or closed-lid run:

- Keep the Mac on a hard, ventilated surface — not a bed, couch, or closed bag.
- Prefer AC power; long generations drain a battery fast.
- If you're feeding `ollama serve` a batch, smaller/quantized models finish sooner and run cooler than a large one you're paging to disk.

## Safety guardrails

The reason to use a guarded tool rather than a raw `pmset disablesleep` is the auto-release behavior. LidRun's guardrails:

- **Low-battery stop** — if you're on battery and it runs low, LidRun stops holding the Mac awake and lets it sleep, so an unattended job doesn't drain to zero.
- **Charging-only mode** — only stay awake while plugged in.
- **Auto-release when work ends** — when `ollama` exits, the assertion is dropped; the Mac isn't left awake forever.
- **Why Awake / Why Stopped** — plain-language transparency on what's holding the machine and why it stopped.

## FAQ

**How do I stop my MacBook from sleeping during a long Ollama generation?**
Run it under `caffeinate -i -- ollama run <model>`. The `-i` flag blocks idle sleep and the assertion is released when the command finishes. For a persistent server, `caffeinate -is ollama serve`.

**Can I keep Ollama running with the MacBook lid closed?**
Not with `caffeinate` alone — clamshell sleep overrides it. Use LidRun's Closed-Lid workflow, and keep the Mac on a ventilated surface because closed-lid inference traps heat.

**Does Ollama keep running when I close the lid by default?**
No. macOS clamshell sleep pauses everything unless something explicitly holds the Mac awake through it. That's what LidRun's Closed-Lid mode is for.

**Will an unattended overnight job drain my battery flat?**
It can. Run on AC power, or rely on LidRun's low-battery stop / charging-only mode so the Mac is allowed to sleep before the battery is exhausted.

*[Download LidRun](https://lidrun.com/download) — free forever for simple sessions; Pro unlocks the full closed-lid AI workflow ([pricing](https://lidrun.com/pricing)).*
