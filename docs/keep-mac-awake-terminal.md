# Keep your Mac awake from the Terminal

**Short answer:** To keep your Mac awake from the Terminal, use the built-in `caffeinate` command — `caffeinate -i` prevents idle sleep, and `caffeinate -- <command>` keeps the Mac awake for exactly as long as `<command>` runs, then releases it. `caffeinate` is perfect for lid-open, plugged-in jobs. When you need the lid closed or safety guardrails around a long AI/dev run, `lidrun -- <command>` does the same one-command wrapping with a closed-lid workflow and low-battery/thermal auto-stop on top.

This is a command reference for both tools: what each flag does, real examples, and when to graduate from one to the other.

## `caffeinate` — the built-in tool

`caffeinate` ships with every macOS install. With no arguments it holds off idle sleep until you press `Ctrl-C`. Its power is in the flags, which each block a specific kind of sleep.

```sh
caffeinate            # block idle sleep until Ctrl-C
caffeinate -i         # prevent idle system sleep
caffeinate -d         # prevent the display from sleeping
caffeinate -m         # prevent the disk from idle-sleeping
caffeinate -s         # prevent system sleep (only takes effect on AC power)
caffeinate -u         # declare user activity — wakes the display, keeps it up briefly
```

Flags combine. A common "stay fully awake, screen on" invocation:

```sh
caffeinate -dimsu
```

### Wrap a command with `-- <cmd>`

The most useful form. `caffeinate` stays alive only while the wrapped command runs, then exits and releases every assertion — no dangling process to remember to kill.

```sh
# Keep awake for exactly this build, then release
caffeinate -i -- ./gradlew assembleRelease

# A long data job
caffeinate -i -- python train.py --epochs 50

# Keep the display awake too while a script runs
caffeinate -dimsu -- ./run-benchmark.sh
```

### Time-box it with `-t <seconds>`

```sh
caffeinate -i -t 3600     # stay awake for one hour, then release
caffeinate -dimsu -t 1800 # display + system awake for 30 minutes
```

### Tie it to an existing process with `-w <PID>`

Already started a job? Attach `caffeinate` to its PID and it releases when that process exits.

```sh
./long-job.sh &          # start the job in the background
caffeinate -i -w $!      # keep awake until it finishes ($! = last PID)

# Or target a process you already have running
caffeinate -i -w 45210
```

### What `caffeinate` does *not* do

`caffeinate` is a thin wrapper over IOKit power assertions, so it inherits IOKit's limits:

- **It won't keep a job running with the lid closed.** Close the lid on a laptop and the Mac still sleeps — `caffeinate` has no way around clamshell sleep.
- **No safety guardrails.** It will happily hold the machine awake at 3% battery until it dies. There's no low-battery stop, no charging-only mode, no thermal back-off.
- **No workload awareness.** It holds the assertion for the whole timer/command regardless of whether real work is happening.
- **No GUI, no visibility.** You track it by PID and remembering it's running.

For a quick lid-open, plugged-in job, that's exactly enough. The [full `caffeinate` guide](https://lidrun.com/blog/caffeinate-mac-command) covers more edge cases.

## The `lidrun` CLI — same ergonomics, closed lid + safety

[LidRun](https://lidrun.com/download) is a menu-bar app, but it ships a `lidrun` CLI that mirrors the `caffeinate -- <cmd>` pattern and adds the things `caffeinate` can't do: a guarded closed-lid workflow, low-battery/charging/thermal guardrails, and workload detection so it releases when work actually ends.

### Wrap one command

```sh
# Keep the Mac awake for exactly this command, then release
lidrun -- npm run build

# Run an overnight test suite; the Mac stays up until it's done
lidrun -- pytest -q

# A long agent session
lidrun -- claude
```

Same mental model as `caffeinate -- <cmd>`: awake for the duration of the command, released the moment it exits.

### Other subcommands

The CLI also drives the app's session state directly:

```sh
lidrun status            # is the Mac being held awake, and why
lidrun start             # start a keep-awake session
lidrun stop              # end it
lidrun timer 7200        # keep awake for 7200 seconds (2h), then release
lidrun autowatch on      # auto-hold only while a known workload is active
lidrun autowatch off
lidrun watch "ffmpeg"    # hold awake while a process matching this pattern runs
```

`autowatch` is the piece `caffeinate` has no equivalent for: known agents (Claude Code, Cursor, Codex, Cline, Aider, Continue, Goose, Ollama, LM Studio, and more) count as *working by presence*, so an agent sitting at ~0% CPU waiting on an API response is never dropped — but the Mac still releases when the work genuinely ends.

### Scripting examples

Because `lidrun -- <cmd>` returns the wrapped command's exit code, it drops into scripts and Makefiles cleanly:

```sh
#!/usr/bin/env bash
# ci-local.sh — keep the Mac awake through a full local CI run
set -e
lidrun -- bash -c '
  npm ci &&
  npm run lint &&
  npm test &&
  npm run build
'
```

```makefile
# Makefile
release:
	lidrun -- ./scripts/build_and_notarize.sh
```

## When to graduate from `caffeinate` to `lidrun`

Reach for `caffeinate` when the job is short, the lid stays open, and you're on power. Reach for `lidrun` when any of these is true:

- **You want the lid closed.** Shut the laptop and walk away while a build or agent keeps running (guarded — it's not a blind wake-lock).
- **You want safety.** Low-battery stop, charging-only mode, thermal back-off, and auto-release when the work ends — so a forgotten session doesn't drain the battery flat or cook a machine in a bag. Keep it ventilated; guardrails reduce risk, they don't remove physics.
- **You want workload detection.** Hold awake *only* while real work is happening, including idle-but-waiting AI agents.
- **You want to see why.** A "Why Awake / Why Stopped" readout instead of hunting for a PID.

The [LidRun CLI guide](https://lidrun.com/blog/lidrun-cli-keep-awake-terminal) is the full reference for the command-line workflow.

## FAQ

**How do I keep my Mac awake in the Terminal without installing anything?**
Use the built-in `caffeinate`. `caffeinate -i` blocks idle sleep until you hit `Ctrl-C`, and `caffeinate -i -- <command>` keeps the Mac awake only while that command runs.

**Does `caffeinate` work with the lid closed?**
No. `caffeinate` can't prevent clamshell sleep — close the lid and a laptop still sleeps. For a closed-lid workflow you need something like LidRun's guarded closed-lid mode.

**How do I keep my Mac awake for a specific command and then let it sleep?**
Wrap the command: `caffeinate -i -- ./build.sh` or `lidrun -- ./build.sh`. Both stay awake exactly as long as the command runs, then release automatically.

**Will keeping my Mac awake overnight drain or overheat it?**
`caffeinate` has no safety net, so on battery it can run the charge flat. `lidrun` adds low-battery stop, charging-only mode, and thermal back-off to reduce that risk — but keep the machine ventilated regardless; software guardrails don't override physics.

*[Download LidRun](https://lidrun.com/download) — free forever for simple sessions; Pro unlocks the full closed-lid AI workflow ([pricing](https://lidrun.com/pricing)).*
