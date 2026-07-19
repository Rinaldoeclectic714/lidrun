# Keep Claude Code running when your MacBook is closed

**Short answer:** To keep Claude Code running when your MacBook is closed, you need something that holds a power assertion *and* handles the lid-close event — plain `caffeinate` won't survive a shut lid. [LidRun](https://lidrun.com/download) does both: start your Claude Code task, turn on **Auto Mode**, close the lid, and the Mac stays awake until the agent finishes, then releases on its own. Because Auto Mode counts Claude Code as working *by presence*, a session sitting at ~0% CPU while it waits on the API is never dropped.

## Why a long Claude Code run dies on lid-close

A long agent run — a multi-file refactor, a test-and-fix loop, an overnight batch — spends most of its wall-clock time *waiting*: on an API round-trip, on a tool call, on a subprocess. During those stretches CPU sits near idle.

That breaks two common approaches:

- **Naive keep-awake tools** that watch CPU usage decide "nothing's happening" and let the machine sleep — right in the middle of your run.
- **`caffeinate`** holds an assertion, but on a MacBook, closing the lid triggers clamshell sleep regardless of the assertion (unless an external display + power + keyboard are attached). Your terminal freezes with the lid shut.

So the two things you actually need are: *don't* judge "working" by CPU alone, and *do* keep running through a closed lid.

## The recipe: Claude Code + Auto Mode + closed lid

1. **Start your long task** in the terminal as usual:

   ```sh
   claude
   # e.g. "refactor the auth module and make all tests pass"
   ```

2. **Turn on Auto Mode** from the LidRun menu-bar icon. Auto Mode holds the Mac awake only while a workload is actually working — and it recognises known agents by presence. Claude Code, Cursor, Codex, Cline, Aider, Continue, Goose, Ollama and LM Studio all count as *working* whenever the process is alive, so a Claude session parked at ~0% CPU waiting on a response won't be mistaken for idle.

3. **Enable the closed-lid workflow**, then **close the lid.** LidRun keeps the job alive with the lid shut. This is guarded, not a blind wake-lock — the safety guardrails below still apply.

4. **Walk away.** When Claude Code exits (or Auto Mode sees no more work), LidRun **auto-releases** the assertion and lets the Mac sleep normally. You don't have to remember to turn anything off.

Auto Mode is the piece that makes this reliable for agents specifically: presence-based detection is the difference between "kept awake for the whole run" and "dropped during a long API call."

## The CLI alternative: `lidrun -- <command>`

If you'd rather script it — a cron job, a CI-style batch, a Makefile target — LidRun ships a CLI. Wrap any command and the Mac stays awake for exactly that command, then releases the instant it returns:

```sh
# Keep the Mac awake for one scripted agent run, then release
lidrun -- claude -p "run the full test suite and fix failures"
```

```sh
# Works for any long job, not just Claude Code
lidrun -- ./scripts/nightly-build.sh
```

The assertion is scoped to the process lifetime, so there's nothing to clean up and no assertion left dangling if the command crashes. This is the closest analogue to `caffeinate -s <command>`, but tied to LidRun's release logic instead of a raw wake-lock.

For comparison, the stock-macOS building blocks look like this — useful to know, but neither survives a closed lid on battery:

```sh
# Keep awake while a command runs (lid must stay OPEN)
caffeinate -s claude -p "..."

# Inspect what's currently holding the system awake
pmset -g assertions
```

## Safety with the lid closed

Running with the lid shut concentrates heat and drains battery with no screen to warn you, so LidRun keeps guardrails on the whole time:

- **Low-battery stop** — it releases and lets the Mac sleep before the battery bottoms out.
- **Charging-only mode** — optionally refuse to run a closed-lid session unless the Mac is on power.
- **Thermal back-off** — it eases off when the machine runs hot.
- **Auto-release when work ends** — the assertion drops the moment the job finishes.
- **Why Awake / Why Stopped** — plain-language transparency so you can always see *why* the Mac stayed up or shut down.

None of this removes physics: a closed MacBook running a heavy job still needs airflow. Keep it on a hard, ventilated surface rather than a bed or a bag. The guardrails **reduce risk**; they don't make an overnight run risk-free.

For the deeper background on why presence-based detection matters for agents, see [what is AI agent continuity](https://lidrun.com/blog/what-is-ai-agent-continuity), and for the full walkthrough read the canonical guide, [keep Claude Code running when the MacBook is closed](https://lidrun.com/blog/keep-claude-code-running-when-macbook-closed).

## FAQ

**Can I close my MacBook and keep Claude Code running?**
Yes — with a tool that both holds a power assertion and handles the lid-close event. Turn on LidRun's Auto Mode and closed-lid workflow, then shut the lid; the run continues and releases when it's done.

**Why does Claude Code stop when my Mac sleeps?**
Once macOS sleeps, your terminal and the agent process are suspended. Long agent runs are mostly idle-waiting on API calls, so CPU-based keep-awake tools misread them as "done" and allow sleep. Presence-based Auto Mode avoids that.

**Does `caffeinate` keep a MacBook awake with the lid closed?**
No. `caffeinate` holds an assertion but doesn't override clamshell sleep — close the lid on battery and the Mac sleeps anyway. You need lid-aware handling for a closed-lid run.

**How do I keep the Mac awake for just one scripted command?**
Use the CLI: `lidrun -- <command>`. The Mac stays awake for exactly that command and releases as soon as it exits — no leftover assertion.

*[Download LidRun](https://lidrun.com/download) — free forever for simple sessions; Pro unlocks the full closed-lid AI workflow ([pricing](https://lidrun.com/pricing)).*
