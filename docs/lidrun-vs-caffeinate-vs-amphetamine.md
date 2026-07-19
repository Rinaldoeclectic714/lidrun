# LidRun vs caffeinate vs Amphetamine: which Mac keep-awake tool?

**Short answer:** For most one-off jobs, the built-in `caffeinate` is the right call for a single Mac keep-awake command — it's already installed and needs no download. **Amphetamine** is the best free app for general-purpose keep-awake with flexible triggers and schedules. **LidRun** is purpose-built for AI/dev runtime continuity: it keeps the Mac awake only while your workload is actually working, survives a closed lid, and auto-releases with battery/thermal guardrails. Pick the one that matches the job, not the marketing.

All three are legitimate tools. They just optimize for different problems.

## `caffeinate` — the built-in you already have

`caffeinate` ships with macOS. Zero install, scriptable, and perfect for wrapping a single command:

```sh
# Keep the Mac awake for the duration of one command, then release
caffeinate -i make deploy

# Keep the *display* awake too (-d), for a 2-hour render
caffeinate -dis -t 7200

# Keep awake while a specific PID is alive
caffeinate -w 12345
```

Flags worth knowing: `-i` prevents idle sleep, `-d` prevents display sleep, `-s` asserts on AC power, `-m` prevents disk sleep, `-u` simulates user activity, `-t` sets a timeout in seconds.

Its limit is the lid. `caffeinate` holds a standard idle-sleep assertion, which macOS overrides on **clamshell (lid-closed)** sleep unless the Mac is on AC power with an external display attached. It also has no notion of *whether your work is still running* beyond a raw PID or a wrapped command — it holds the assertion for exactly as long as you told it to. Great for scripts, not for a long agent run you walk away from.

## Amphetamine — the flexible free keep-awake app

[Amphetamine](https://apps.apple.com/app/amphetamine/id937984704) is a genuinely excellent free menu-bar app. It does one category of thing very well: general keep-awake with rich triggers. You can keep the Mac awake indefinitely, on a timer, while a specific app runs, while a drive is connected, or on custom conditions you script. If your problem is "stop my Mac sleeping while I do X," Amphetamine is a strong, well-maintained answer and costs nothing.

Where it stops is deep AI/dev workflow: it isn't designed to detect a coding agent's presence, and closed-lid operation on modern Macs is constrained by the same macOS clamshell rules everything else hits. It keeps the Mac awake; it doesn't reason about whether your build, training run, or agent is still doing work.

## LidRun — built for AI/dev runtime continuity

LidRun is a signed, notarized menu-bar app (macOS 13+) focused on one job: keep a long-running AI or dev workload alive without babysitting it, then get out of the way.

- **Auto Mode** holds the Mac awake only while your workload is *actually working*. Known agents — Claude Code, Cursor, Codex, Cline, Aider, Continue, Goose, Ollama, LM Studio — count as working *by presence*, so an agent idling at ~0% CPU while it waits on an API call is never dropped.
- **Closed-Lid workflow** keeps a job running with the lid shut. It's guarded, not a blind wake-lock.
- **CLI** mirrors the `caffeinate` ergonomics you already know:

```sh
# Keep the Mac awake for exactly this command, then release
lidrun -- pytest -q
lidrun -- npm run build
```

- **Safety guardrails**: low-battery stop, charging-only mode, thermal back-off, and auto-release when work ends. **Why Awake / Why Stopped** tells you exactly which assertion is holding and why it let go.

The tradeoff: it's a separate app, and the full closed-lid AI workflow is a Pro feature. Simple keep-awake sessions are free forever.

## Comparison table

| | caffeinate | Amphetamine | LidRun |
|---|---|---|---|
| Built-in / no download? | Yes (ships with macOS) | No (free app) | No (free tier) |
| Closed-lid without external display | No (clamshell override) | Constrained by macOS rules | Yes (guarded Pro workflow) |
| Workload-aware auto-release | No (timeout / PID only) | Trigger-based | Yes (Auto Mode) |
| AI-agent presence detection | No | No | Yes (by presence) |
| Battery / thermal guardrails | No | Partial (custom triggers) | Yes (built-in) |
| Price | Free | Free | Free tier; one-time Pro |

## Pick this if…

- **Pick `caffeinate`** if you want to wrap one command in a script and never think about it again. It's the correct default for CI-style, terminal-bound tasks.
- **Pick Amphetamine** if you want a free, flexible menu-bar app for everyday keep-awake with triggers and schedules, and you don't need closed-lid AI runtime.
- **Pick LidRun** if you leave long AI-agent or dev jobs running (often lid-closed, often unattended) and want the Mac to stay awake *only while work is happening*, then release on its own with battery and thermal limits.

Full write-ups: [LidRun vs caffeinate](https://lidrun.com/blog/lidrun-vs-caffeinate), [LidRun vs Amphetamine](https://lidrun.com/blog/lidrun-vs-amphetamine), an [Amphetamine alternative for Mac](https://lidrun.com/blog/amphetamine-alternative-for-mac), and the [best Mac keep-awake app for developers](https://lidrun.com/blog/best-mac-keep-awake-app-for-developers).

## FAQ

**Is `caffeinate` enough to keep my Mac awake with the lid closed?**
Usually no. `caffeinate` holds an idle-sleep assertion, but macOS still triggers clamshell sleep when the lid closes unless the Mac is on AC power with an external display. For reliable lid-closed runs, you need a tool that manages the clamshell path.

**Can Amphetamine keep a coding agent alive overnight?**
It can keep the Mac awake on a timer or while a chosen app runs, but it doesn't detect coding-agent presence or auto-release when the job finishes. If the agent stalls or completes, Amphetamine keeps holding until its trigger ends.

**Do I have to pay for LidRun?**
No — simple keep-awake sessions are free forever. Pro unlocks the full closed-lid AI workflow as a one-time purchase, no subscription.

**Will keeping my Mac awake with the lid closed overheat it?**
Running with the lid shut reduces airflow, so LidRun's thermal back-off and auto-release reduce risk — but they don't remove physics. Keep the Mac ventilated and on a hard surface, not buried in a bag.

*[Download LidRun](https://lidrun.com/download) — free forever for simple sessions; Pro unlocks the full closed-lid AI workflow ([pricing](https://lidrun.com/pricing)).*
