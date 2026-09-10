# Context Switch Tracker — Project Plan

## The idea

You want something that tells you, honestly, how often you break your own focus while working — every time you flip from your editor to Slack, from Slack to a browser tab, from the browser back to email. The Even Realities glasses page you linked is the inspiration: a wearable HUD that could flash a live counter or a quiet nudge in your peripheral vision. But since you don't have the hardware yet, the plan below starts as a native Mac app that does the tracking and the nagging on its own, structured so a glasses HUD can be bolted on later as a display layer rather than a rewrite.

## Why Mac-only first is the right call

I looked at both current Even Realities glasses to see what a "push my laptop's context-switch count to the glasses" integration would actually take, and the two models are quite different:

The **G1** has an official phone-first app, but the community has reverse-engineered its Bluetooth LE protocol. A Python library called `even_glasses` (by emingenc) can talk to the glasses directly and push text/notifications to the display — no phone in the loop. That's the model that would let a Mac app write straight to the lenses. The catch is you'd need to actually own a G1, and BLE pairing on macOS with a device that expects two independent connections (one per arm) can be finicky.

The **G2** works differently: Even Realities ships an official SDK (`even_hub_sdk`) where you write small HTML/CSS/JS "apps" that live on and run from your *phone*; the glasses only render output and capture touchpad/mic input over BLE. There's no path for a Mac to talk to G2 glasses directly — your Mac app would need to relay data to a companion phone app first, which then pushes to the glasses. More moving parts, more places for lag and bugs to creep in.

Since you don't have either yet, building the Mac side first means the core value (an honest count of how often you interrupt yourself) exists and is useful on day one, regardless of which glasses you eventually buy — or whether you buy any at all.

## What counts as a "context switch"

Per your call: **any change of the frontmost application** (VS Code → Slack → Chrome → Slack counts as three switches). This is trackable on macOS without Accessibility permissions or invasive hooks — the OS already broadcasts "the active app changed" as a system notification, so the tracker is a passive listener, not something that inspects window contents or keystrokes. Tab-level tracking (e.g. counting Chrome tab changes as separate switches) is a natural Phase 2 addition once app-level tracking feels right, but it needs a browser extension to see tab changes, which is a separate moving part.

## Architecture (Phase 1 — Mac only)

**Menu bar app**, not a Dock app — it should be present but never demand attention, which fits the theme of the whole project.

- **Event source**: macOS's `NSWorkspace` posts a notification every time the frontmost app changes. The app subscribes to this once at launch; it's cheap, reliable, and built into the OS (no polling, no Accessibility permission prompt).
- **Local log**: each switch gets written to a small local database (SQLite is plenty) with a timestamp and the app's bundle identifier — `2026-09-10 14:32:07, com.tinyspeck.slackmacgap`. Nothing leaves the machine.
- **Derived stats**, computed from that log on demand:
  - switches per hour / per day
  - longest unbroken "focus streak" (time between switches)
  - top "offender" apps — which apps you keep bouncing back to
  - a rough "focus score" for the day (e.g. switches per hour of active use, so a light day doesn't look artificially good)
- **Menu bar display**: the icon itself can show a live number (today's switch count) so the ambient-awareness effect starts immediately, with a dropdown for the hour/day/week breakdown.
- **Optional gentle nudge**: if switches spike above some rate in a short window (say, 6+ switches in 5 minutes), a single low-key notification — not a lecture, just a mirror.

### Tech stack options

| Option | Pros | Cons |
|---|---|---|
| **Swift + SwiftUI/AppKit** (native menu bar app) | Best OS integration, smallest footprint, no runtime to install, `NSWorkspace` is a first-class API | Slower to prototype if you're not in Swift day-to-day |
| **Python + `rumps`** (menu bar framework) | Very fast to prototype, easy to iterate on stats logic, `pyobjc` gives access to `NSWorkspace` too | Feels less "native," needs Python packaged or installed, slightly heavier |
| **Electron/web-based menu bar** | Reuse web charting libraries for the stats view | Overkill for something this lightweight; more memory for zero real benefit here |

Given this is a personal daily-driver tool, I'd lean Swift for the long-term version but Python+`rumps` is the faster way to get a working prototype to actually live with for a week before committing to native.

## Phased roadmap

**Phase 1 — Mac tracker (this is the buildable "today" version)**
Menu bar app, local logging, live count in the menu bar, a dropdown with today's stats and top offenders. Ships as something you actually run daily.

**Phase 2 — Better visualization**
A small daily/weekly view — trend of switches over time, which hours of the day are worst, correlation with calendar meetings if you want to get fancy (reading your calendar locally, not syncing anywhere). This is where a proper chart (not just numbers) starts to matter.

**Phase 3 — Glasses HUD (once you have hardware)**
If you get a **G1**: extend the Mac app to also push the live count (or a small glyph/color cue) to the lenses via the `even_glasses` BLE library, so the "ambient awareness" moves from a menu bar icon you have to glance at to something literally in your field of view.
If you get a **G2**: build a small EvenHub web-app companion and a thin phone relay that receives switch-count updates from the Mac (over local network) and forwards them to the glasses — more infrastructure, but doable.

**Phase 4 — polish / share** (optional): export weekly reports, maybe open-source it if it turns out to be genuinely useful — "how many times did you switch context today" is a fun, slightly uncomfortable stat a lot of people would want to see about themselves.

## Open questions before Phase 1 starts

A few decisions worth pinning down before writing code:

- **Swift vs. Python** for the prototype — my lean is Python+`rumps` for speed unless you specifically want to end up with a native, installable app.
- **Menu bar number vs. icon-only**: showing a live count in the menu bar is more useful but takes up more menu bar real estate, which is contested territory on most people's Macs.
- **Idle handling**: should switches while you're away from the keyboard (e.g. a meeting, lunch) count? Probably not — likely worth pairing `NSWorkspace` tracking with idle-time detection so a long stretch away doesn't get miscounted as a "focus streak."
- **Nudge threshold**: whether you want the gentle notification at all, or just want the passive stats without the app ever interrupting you.

## Next step

Whenever you're ready, I can build the Phase 1 Mac menu bar app directly — happy to start with the Python+`rumps` version for a fast prototype, or go native Swift if you'd rather live with the real thing from day one. Just say the word and which stack you'd prefer.

---

### Sources

- [Even Realities Developer Docs](https://hub.evenrealities.com/docs) — G2 SDK architecture (phone-hosted web apps, BLE-only connectivity, display/sensor specs)
- [even-realities/EvenDemoApp](https://github.com/even-realities/EvenDemoApp) — official G1 demo app, BLE protocol details, display/text-push capabilities
- [galfaroth/awesome-even-realities-g1](https://github.com/galfaroth/awesome-even-realities-g1) — community project index, including the `even_glasses` Python BLE library
- [Even Realities smart glasses product page](https://www.evenrealities.com/smart-glasses)

---

### Research grounding

The claims behind this plan were source-checked. Some of them did not hold up, including the
widely-quoted "23 minutes to refocus" figure. See [research.md](research.md) for each claim,
its verdict, and what it changes in the design.
