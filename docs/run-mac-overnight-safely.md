# Running your Mac overnight for AI/dev jobs, safely

**Short answer:** To run your Mac overnight for AI/dev jobs safely, keep it on a hard, ventilated surface, leave it on the charger (or use a charging-only mode), set a session timer, and rely on a low-battery stop, thermal back-off, and auto-release when the job ends. These guardrails reduce risk — they don't remove physics. Closing the lid cuts your Mac's main cooling path, so treat lid-closed overnight runs as the case that needs the most care.

Long training runs, overnight builds, and coding agents grinding through a backlog are exactly the workloads you want to leave running while you sleep. The problem: macOS sleeps aggressively, and the obvious workaround (close the lid, walk away) traps heat. Below is a practical checklist you can apply tonight, whether you automate it or do it by hand.

## The overnight safety checklist

Work through this before you leave a job running unattended:

1. **Hard, ventilated surface.** Put the Mac on a desk, a stand, or a metal tray — never a bed, couch, blanket, or inside a bag. Soft surfaces block the bottom vents and the case can't shed heat.
2. **Stay on the charger.** A workload that runs to 0% and dies mid-job wastes the whole night. Plug in, or gate the run so it only proceeds while charging.
3. **Set a session timer.** Decide the maximum time the Mac should stay awake (e.g. 4 hours) so a stuck job doesn't hold the machine awake until morning.
4. **Keep a low-battery stop.** If you're not plugged in, have something that force-stops and lets the Mac sleep before the battery hits a dangerous floor.
5. **Allow thermal back-off.** If the chip gets too hot, the safe response is to let the Mac sleep, not to keep pushing. Don't defeat that.
6. **Confirm auto-release.** When the job finishes, the wake-lock should drop so the Mac sleeps normally instead of idling awake and hot all night.
7. **Prefer lid-open when you can.** Lid-closed (clamshell) is convenient but removes the keyboard-deck cooling path. If the room allows it, leave the lid open a crack or fully.

## Doing it by hand with built-in tools

macOS ships two tools that cover the basics — they keep the Mac awake but do **not** give you the safety layer above.

Keep the Mac awake for one command, then release automatically:

```sh
# Stay awake until `make train` finishes, then let the Mac sleep again
caffeinate -i make train
```

Keep the display and system awake for a fixed window (here, 4 hours = 14400s):

```sh
caffeinate -dimsu -t 14400
```

Inspect what's currently holding your Mac awake — useful when it *won't* sleep:

```sh
pmset -g assertions
```

`caffeinate` is great for lid-**open** runs. What it doesn't do: keep going with the lid **closed** on battery, stop on low battery, back off on heat, or tell you *why* the Mac stayed awake. On a stock Mac, closing the lid still sleeps unless you also disable clamshell sleep — and doing that blindly is how people cook a laptop in a backpack. For the reasoning behind clamshell heat, see the full guides: [why your MacBook gets hot while coding](https://lidrun.com/blog/macbook-gets-hot-while-coding) and [auto-sleep on low battery](https://lidrun.com/blog/macbook-auto-sleep-low-battery).

## Automating the guardrails with LidRun

[LidRun](https://lidrun.com/download) is a macOS menu-bar app that turns the checklist above into defaults you don't have to remember. It holds the Mac awake while work runs, then releases it — with the safety layer built in.

- **Auto Mode** holds the Mac awake only while your workload is actually working. Known coding agents (Claude Code, Cursor, Codex, Cline, Aider, Continue, Goose, Ollama, LM Studio…) count as working *by presence*, so an agent sitting at ~0% CPU while it waits on an API call is never dropped — but a genuinely finished job lets the Mac sleep.
- **Closed-Lid workflow** keeps a job running with the lid shut, but as a *guarded* mode, not a blind wake-lock — the safety stops still apply.
- **Session timer** from 30 minutes to 8 hours, so an overnight run has a hard ceiling.
- **Safety guardrails:** low-battery stop, charging-only mode, thermal back-off, and auto-release when work ends.
- **Why Awake / Why Stopped** transparency — you can always see the reason the Mac is up or why it went to sleep.

For CLI-driven jobs, LidRun mirrors the `caffeinate` pattern but with its own release semantics:

```sh
# Keep the Mac awake for exactly this command, then release
lidrun -- make train
```

The honest caveat: these guardrails **reduce** risk, they don't eliminate it. A closed lid still cuts the main cooling path, and no software repeals thermodynamics. Keep the Mac ventilated and plugged in; treat the safety stops as a floor, not a license to run a hot job on a pillow. Read LidRun's [safety page](https://lidrun.com/safety) and the [closed-lid mode safety guide](https://lidrun.com/blog/closed-lid-mode-safety-guide) for the full picture.

## FAQ

**Is it safe to leave a MacBook running with the lid closed overnight?**
It can be reasonable *if* the Mac is on a hard, ventilated surface, plugged in, and has a session timer plus low-battery and thermal stops. It's never risk-free — closing the lid removes the main cooling path, so keep it cool and don't run a heavy job buried in a bag.

**How do I keep my Mac awake for a long job without changing system settings?**
Use `caffeinate -i <command>` for a single job, or a menu-bar tool like LidRun that holds the Mac awake only while the work is running and releases automatically when it finishes.

**Will my Mac stop the job if the battery gets low?**
Not on its own — plain `caffeinate` will hold the Mac awake until the battery dies. Use charging-only mode or a low-battery stop (LidRun ships both) so the Mac sleeps safely before the battery hits a dangerous floor.

**Why won't my Mac sleep after a job finished?**
Something is still holding a power assertion. Run `pmset -g assertions` to see what. If you used auto-release (LidRun's Auto Mode), the lock drops when the workload ends so the Mac sleeps on its own.

*[Download LidRun](https://lidrun.com/download) — free forever for simple sessions; Pro unlocks the full closed-lid AI workflow ([pricing](https://lidrun.com/pricing)).*
