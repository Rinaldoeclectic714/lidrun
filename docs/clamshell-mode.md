# Clamshell mode on a MacBook

**Short answer:** Clamshell mode on a MacBook keeps macOS running with the lid closed. There are two versions. The *classic* one is Apple-supported: plug in an external display, a keyboard/mouse, and power, and the Mac drives the external screen with the lid shut — no extra tools needed. The *headless* one — lid closed with **no** external display, for unattended AI/dev jobs — is not built into macOS (a MacBook with the lid down and no display goes to sleep), so it needs a keep-awake tool plus the documented `pmset` toggle.

Knowing which scenario you're in is the whole game. Below is the reference for both.

## The two clamshell scenarios

| | (A) Classic clamshell | (B) Headless clamshell |
|---|---|---|
| External display | Required | None |
| Keyboard / mouse | Required | Optional |
| Power adapter | Required | Recommended |
| Apple-supported | Yes | No (needs a tool) |
| Use case | Desk setup, laptop as desktop | Unattended AI agents, CI, long builds |

If you have a monitor on your desk, you want (A). If you want to close the lid and walk away while a job runs — and there's no monitor — you want (B).

## (A) Classic clamshell mode: MacBook + external display

This is the "use my MacBook as a desktop" setup. Apple designed for it. macOS calls it *closed-display mode*, most people call it clamshell mode on a MacBook.

Requirements, all at once:

1. **Power** — connect the Mac to its charger (USB-C / MagSafe). On battery, closed-display mode won't engage.
2. **An external display** — connected over HDMI, USB-C/Thunderbolt, or a dock.
3. **An input device** — an external keyboard or mouse/trackpad (USB or Bluetooth).

Steps:

```text
1. Plug in power.
2. Connect the external display and confirm you see the desktop on it.
3. Connect an external keyboard and/or mouse.
4. Close the lid. The internal screen turns off; the external stays live.
5. If the display sleeps, press a key or click to wake it.
```

Bluetooth gotcha: if you use a Bluetooth keyboard/mouse, enable **"Allow Bluetooth devices to wake this computer"** (System Settings → Bluetooth, or the trackpad/keyboard advanced options), or a closed lid may not wake from the external input.

This mode needs no third-party software and no `pmset` changes. If yours won't engage, it's almost always missing power or a missing/asleep external display.

For the full walkthrough with troubleshooting, see the canonical guide: [Clamshell mode on Mac](https://lidrun.com/blog/clamshell-mode-on-mac).

## (B) Headless clamshell: lid closed, no external display

Here's the part that trips developers up. Close the lid on a MacBook with **no external display attached**, and macOS treats it as "nothing needs the screen" and sleeps — pausing your build, your training run, your Claude Code agent, your `rsync`. This is a MacBook clamshell mode that Apple does not support out of the box, because there's no display to keep the machine awake.

You can prove the default with `pmset`:

```sh
# Watch what happens on lid close (leave running, close the lid)
pmset -g log | grep -i "lid\|sleep\|wake"
```

To keep a job alive with the lid shut and no monitor, you need two things:

1. A **power assertion** so the CPU/network stay awake (what `caffeinate` does).
2. The **lid-close override** — `sudo pmset -a disablesleep 1` — which stops the lid from forcing sleep.

`caffeinate` alone is not enough. It holds the system awake while the lid is *open*, but closing the lid still triggers sleep unless `disablesleep` is set:

```sh
# Keeps the Mac awake for one command — but the LID will still sleep it
caffeinate -dimsu -- your-long-job.sh

# The lid override (admin; remember to turn it back off)
sudo pmset -a disablesleep 1
# ... run your work ...
sudo pmset -a disablesleep 0
```

The manual combo works, but it's easy to forget the `disablesleep 0` and leave your Mac unable to sleep for the rest of the day. That's the exact pairing [LidRun](https://lidrun.com/download) automates: it holds an IOKit power assertion, toggles `disablesleep` on start, and — the important part — always pairs it back to `0` when the job ends, you quit, or a safety limit fires. Deeper background: [keep a Mac awake when the lid is closed](https://lidrun.com/blog/keep-mac-awake-when-lid-closed) and [what a closed-lid AI workflow looks like](https://lidrun.com/blog/what-is-closed-lid-ai-workflow).

## Headless clamshell for AI agents and long jobs

The common developer case: a coding agent (Claude Code, Cursor, Codex, Cline, Aider, Continue, Goose) or a local model (Ollama, LM Studio) that runs for hours, mostly waiting on an API. A naive keep-awake keeps the Mac up even after the work is done. A better setup releases when the work ends.

- **Scope it to one command.** The LidRun CLI holds the Mac awake for exactly the process you name, then releases:

  ```sh
  lidrun -- npm run build
  lidrun -- ollama run llama3 < prompts.txt
  ```

- **Let it track the agent, not CPU.** An idle agent waiting on a network call sits near 0% CPU — a CPU-threshold wake-lock would drop it. Auto Mode counts known agents as "working" by presence, so it stays awake while genuinely busy and releases when the agent exits.
- **Keep guardrails on.** Lid-closed, unattended, means nobody's watching the machine. Low-battery auto-stop, charging-only mode, thermal back-off, and a session timer (30 min–8 h) give you a hard stop. And keep it ventilated — a closed lid on a hot desk doesn't remove physics; guardrails reduce risk, they don't erase it.

More on this pattern: [MacBook lid-closed developer workflow](https://lidrun.com/blog/macbook-lid-closed-developer-workflow).

## FAQ

### Does clamshell mode work on a MacBook without an external display?

Not by default. Classic MacBook clamshell mode needs an external display, power, and an input device. With no display, closing the lid puts macOS to sleep. To run a headless clamshell — lid closed, no monitor — you need a keep-awake power assertion plus `sudo pmset -a disablesleep 1`, or a tool like LidRun that manages both and undoes them safely.

### How do I keep my MacBook awake with the lid closed?

Combine a power assertion with the lid override: `caffeinate -dimsu` to hold the system awake and `sudo pmset -a disablesleep 1` so the lid doesn't force sleep. Set `disablesleep` back to `0` when you're done. LidRun does exactly this pairing automatically and releases the Mac when the work ends or a safety limit is hit.

### Why does my Mac sleep when I close the lid even with caffeinate running?

Because `caffeinate` holds a power assertion but does not override the lid-close sleep trigger. Closing the lid with no external display is a separate signal to macOS. You also need `sudo pmset -a disablesleep 1` (admin) to stop the lid from sleeping the machine.

### Is closed-lid clamshell mode safe for my MacBook?

It's a documented capability — the same `pmset` toggle macOS ships — but it's not risk-free. A closed MacBook running a heavy job has less airflow, so keep it on a hard, ventilated surface, keep it plugged in, and use guardrails (low-battery stop, thermal back-off, a session timer). These reduce risk; they don't remove physics.

*[Download LidRun](https://lidrun.com/download) — free forever for simple sessions; Pro unlocks the full closed-lid AI workflow ([pricing](https://lidrun.com/pricing)).*
