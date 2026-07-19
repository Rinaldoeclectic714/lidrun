# How to keep your Mac from sleeping

**Short answer:** to keep a Mac from sleeping you can (1) change **System Settings → Lock Screen / Battery** so the display and system never sleep, (2) run the built-in **`caffeinate`** command in Terminal for one session, or (3) use a menu-bar app like **[LidRun](https://lidrun.com/download)** that keeps the Mac awake only while your work is actually running — including with the **lid closed** — and then lets it sleep again. Which one you want depends on *why* you need it awake.

This page is a quick, practical map. Each method links to the full guide on **[lidrun.com](https://lidrun.com)**.

## 1. System Settings (no tools)

- **Ventura and later:** System Settings → **Lock Screen** → set *Turn display off* to a longer time; System Settings → **Battery** → *Options* → prevent automatic sleeping when the display is off (on power adapter).
- Good for a desk setup on mains power. The catch: it's a **global, always-on** change that's easy to forget — your Mac then never sleeps even when you *want* it to, which wears the battery and runs it hot.

## 2. `caffeinate` (built into macOS)

Keep the Mac awake for exactly one command:

```sh
caffeinate -i -- <your-long-command>
```

It's perfect for a single terminal job and releases the moment the command finishes. It does **not** cover a closed lid, a GUI app, or battery/thermal safety. → [The `caffeinate` command, explained](https://lidrun.com/blog/caffeinate-mac-command)

## 3. `pmset` (advanced — be careful)

`sudo pmset -a disablesleep 1` forces the Mac to stay awake even with the lid closed — but it's a **permanent, system-wide** override that's easy to leave on and can overheat a closed Mac. Prefer a tool that pairs every "stay awake" with an automatic "release". → [A safer alternative to `pmset disablesleep`](https://lidrun.com/blog/safe-alternative-to-pmset-disablesleep)

## 4. Keep it awake *with the lid closed*

None of the built-ins are built for closing the lid and walking away. If you need a build, a Docker job, or an AI agent to keep running after you shut the MacBook, see **[clamshell mode on a Mac](https://lidrun.com/blog/clamshell-mode-on-mac)**.

## When to reach for LidRun instead

The methods above keep the Mac awake **unconditionally** — you have to remember to turn them off. **[LidRun](https://lidrun.com/download)** is a [safe runtime layer](https://lidrun.com/blog/what-is-a-safe-runtime-layer), not a blind wake-lock: it holds the Mac awake **only while real work is running** (Claude Code, Cursor, Docker, Ollama, long terminal jobs), keeps it going with the **lid closed**, and **auto-releases** when the job ends or a battery/thermal limit is reached. It's free forever for simple keep-awake sessions; Pro unlocks the full closed-lid AI workflow. → [Download](https://lidrun.com/download) · [Pricing](https://lidrun.com/pricing)

## FAQ

**How do I stop my Mac from sleeping automatically?**
Adjust Lock Screen + Battery settings, or run `caffeinate -i` for a single session. For a job you want to leave running after closing the lid, use a workload-aware tool that releases the Mac when the work ends.

**How do I keep my Mac awake but still let it sleep when I'm done?**
Unconditional methods (`pmset`, Settings) can't do that. A tool that watches your workload — like [LidRun](https://lidrun.com/download) — keeps the Mac awake while work runs and sleeps it automatically afterward.

**Is it safe to prevent my Mac from sleeping overnight?**
On a ventilated, mains-powered desk it's usually fine. With the lid closed, cooling is reduced — keep the Mac on a hard surface, watch temperature and battery, and prefer a tool with a low-battery stop and thermal back-off. → [Safety](https://lidrun.com/safety)

---
*Full, up-to-date guides: [lidrun.com](https://lidrun.com) · Get the app: [lidrun.com/download](https://lidrun.com/download)*
