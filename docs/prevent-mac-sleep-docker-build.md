# Prevent your Mac from sleeping during a Docker build

**Short answer:** To prevent your Mac from sleeping during a Docker build, wrap the build in `caffeinate`: `caffeinate -i -- docker build .`. That holds the Mac awake for exactly the length of the build, then lets it sleep again. For unattended runs — closing the lid, or firing off several builds back to back — use [LidRun](https://lidrun.com/download), which detects `docker`/`docker-compose` and holds the Mac awake only while the build is actually running.

Long multi-stage builds, `buildx` bake jobs, and cold `docker compose build` runs can take 20–40 minutes. If your Mac dims and sleeps in the middle, the Docker daemon pauses, network pulls stall, and you come back to a half-baked image cache. Here's how to keep the machine awake for the build and only the build.

## The one-liner: `caffeinate -i`

`caffeinate` ships with macOS. The `-i` flag prevents *idle* sleep, and `--` runs a command and releases the assertion the moment that command exits:

```sh
# Keep the Mac awake for exactly this build, then let it sleep
caffeinate -i -- docker build -t myapp:latest .

# Same for Compose
caffeinate -i -- docker compose build

# A slow multi-stage / buildx bake
caffeinate -i -- docker buildx bake
```

Because `caffeinate` exits when the build exits, there's nothing to remember to turn off — the Mac goes back to its normal sleep behavior automatically. This is the cleanest option when you're sitting at the machine with the lid open.

A couple of things `caffeinate -i` does **not** do:

- It does **not** keep the Mac awake with the **lid closed**. Close the lid on a laptop and the display-sleep/clamshell path takes over regardless of the idle assertion.
- It's tied to that one terminal invocation. If you want to run five builds in sequence, or babysit a CI-style loop overnight, you're back to managing it by hand.

## Closing the lid, or running builds unattended: LidRun

When you want to shut the lid and let a long build (or a queue of them) finish, [LidRun](https://lidrun.com/download) is built for that. It's a macOS menu-bar app that holds the Mac awake — including with the lid closed — and then releases it when the work ends.

**Auto Mode** is the piece that matters for Docker. It watches for known dev workloads and holds the Mac awake only while they're *actually working*. `docker` and `docker-compose` are recognized, so LidRun keeps the machine up through the build and stops on its own once the build process exits — no leftover wake-lock draining the battery afterward.

For a scripted, terminal-first flow, LidRun also has a **CLI** that mirrors the `caffeinate --` pattern:

```sh
# Hold the Mac awake for exactly this command, then release
lidrun -- docker build -t myapp:latest .

# Chain a few builds in one awake window
lidrun -- sh -c 'docker compose build && docker buildx bake'
```

The difference from `caffeinate` is the **Closed-Lid workflow**: LidRun's clamshell mode is guarded, not a blind wake-lock. It pairs with safety guardrails — a low-battery stop, a charging-only mode, thermal back-off, and an auto-release when the work ends — plus a *Why Awake / Why Stopped* readout so you can see exactly why the Mac is up. You can also set a **session timer** (30 minutes to 8 hours) as a hard ceiling for a build queue.

For the full walkthrough, see the canonical guides: [Prevent Mac sleep during a Docker build](https://lidrun.com/blog/prevent-mac-sleep-during-build) and the more general [Stop your Mac sleeping during a build](https://lidrun.com/blog/stop-mac-sleeping-during-build).

## Let the Mac sleep once the build finishes

The goal isn't to keep the Mac awake forever — it's to keep it awake *for the build*. Both approaches release automatically:

- `caffeinate -i -- <cmd>` drops the assertion when `<cmd>` returns.
- LidRun's Auto Mode stops when the Docker process exits; the CLI's `lidrun -- <cmd>` releases on the same exit.

That auto-release is what keeps this from turning into a dead battery. Avoid `caffeinate` with no command and no timeout (`caffeinate &`) — that holds the Mac awake indefinitely until you remember to kill it.

## Heat during heavy builds — be honest about it

A cold multi-stage build with parallel `buildx` stages will peg your cores, and a laptop running flat-out with the lid closed traps heat against the keyboard deck and battery. Keeping the Mac awake doesn't change the thermodynamics.

Practical steps that actually help:

- Keep the machine on a **hard, ventilated surface** — not a bed, couch, or lap — so the intakes aren't blocked.
- Prefer running lid-closed builds while **plugged in**; charging plus a full CPU load is where things get hottest.
- macOS will thermally throttle on its own, and LidRun adds a **thermal back-off** guardrail, but neither removes physics. If the room is warm and the build is long, give it airflow.

This reduces risk; it doesn't make a sustained full-load build "100% safe." Treat a warm laptop as expected during a heavy build and plan ventilation accordingly.

## FAQ

**Does `caffeinate` keep a MacBook awake with the lid closed?**
No. `caffeinate -i` prevents idle sleep while the lid is open, but closing the lid triggers clamshell sleep regardless. For a lid-closed build, use LidRun's guarded Closed-Lid workflow.

**How do I stop my Mac sleeping during a long `docker compose build`?**
Run it under `caffeinate`: `caffeinate -i -- docker compose build`. The Mac stays awake for the build and sleeps again when it finishes. For multiple builds or lid-closed runs, use LidRun.

**Will the Mac sleep if my agent is waiting on Docker at 0% CPU?**
Under plain idle-timeout logic it might. LidRun's Auto Mode treats known workloads as *working by presence*, so a build (or a coding agent) that's idle waiting on I/O or an API call isn't dropped.

**Do I have to turn keep-awake off after the build?**
No. `caffeinate -- <cmd>` and `lidrun -- <cmd>` both release automatically when the command exits, and LidRun's Auto Mode stops when the Docker process ends.

*[Download LidRun](https://lidrun.com/download) — free forever for simple sessions; Pro unlocks the full closed-lid AI workflow ([pricing](https://lidrun.com/pricing)).*
