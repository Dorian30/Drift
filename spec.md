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

## The focus budget (Phase 1.5)

Phase 1 answers "how often did I switch". That number is honest but flat. A day with 80 switches
between two files is not the same as a day with 80 switches into Slack. Phase 1.5 adds a model
that prices switching in the same unit as the work itself.

### What this model does not claim

Focus is not a battery. The theory that willpower drains like a tank, called ego depletion, failed
a preregistered replication across 23 labs with 2,141 participants. See
[research.md](research.md), claim 7. Drift never says "depleted" and never draws a battery.

Drift does something narrower and defensible. You set a daily budget. Drift spends that budget
against observed minutes, using coefficients you can see and change. The app applies a rule you
agreed to. It does not measure your brain.

One further limit, from [research.md](research.md) claim 2: Mark, Gudith and Klocke found that
interrupted work is completed *faster*, with no loss of quality, but with more stress, frustration
and effort. The real cost of switching does not show up in minutes. Drift counts minutes because
minutes are what macOS can observe. Say this in the interface, and never present the minute count
as a measure of output.

### App roles

The axis is not how "deep" an app is. The axis is whether a switch serves the task you are on or
leaves it. This matters because the same app plays different roles for different people. Slack
breaks concentration. Finder usually does not, because opening Finder is normally part of the
work in progress.

The research supports splitting the axis this way. Iqbal and Horvitz found that resumption time did
not differ significantly between fast and slow responses to an alert, but did differ by where the
user went (about 11 minutes for instant messaging, about 16 minutes for email). Where you go
predicts the cost better than how long you stay away.

So every bundle identifier carries one of three roles, editable by you:

| Role | Meaning | Effect on a block |
|---|---|---|
| **Anchor** | Where the work happens. VS Code, Figma, a writing app. | Starts and holds a block. |
| **Companion** | Serves the task in progress. Finder, Terminal, a documentation tab. | A short visit does not break the block. |
| **Interrupting** | Leaves the task. Slack, Mail, Messages. | Always breaks the block and charges the tax. |

A companion visit that runs past the grace period breaks the block anyway. Twenty minutes in
Finder is not a quick lookup, it is a different task.

Unknown apps default to interrupting. A false alarm you correct once is better than a silent
undercount.

### The model

```
budget          = 240 min/day   # your call, see research.md claim 5
warmup_enabled  = true          # toggle, see "Warm-up" below
warmup          = 15 min        # ramp-in time before a block counts as focused work
min_block       = 5 min         # used when warmup is off, so blips do not count
companion_grace = 2 min         # a visit under this does not break a block
idle_timeout    = 10 min        # no keyboard or mouse: the clock stops, no tax

reentry_cost(app):              # charged on the app you switched TO
    chat      -> 11 min         # Iqbal & Horvitz, IM resumption phase
    mail      -> 16 min         # Iqbal & Horvitz, email resumption phase
    other     -> 12 min         # midpoint default

# A block starts on an anchor app and holds through short companion visits.
# It ends on an interrupting app, on a long companion visit, or on idle.

focused_minutes(b):
    if warmup_enabled:
        return max(0, duration(b) - warmup)          # the ramp-in does not count
    else:
        return duration(b) if duration(b) >= min_block else 0

focused_minutes = sum(focused_minutes(b) for b in blocks)
switch_tax      = sum(reentry_cost(a) for each block that began after an interrupting switch)
spent           = focused_minutes + switch_tax
```

### Warm-up

Warm-up is the ramp-in time at the start of a block, before the work becomes real work. With the
toggle on, the first 15 minutes of every block earn nothing. A 20-minute block contributes 5
focused minutes. A 12-minute block contributes none.

This setting is a working assumption, not a research finding, and the interface says so. No study
establishes a time to reach flow. See [research.md](research.md), claim 6. The absence of a study
is not evidence that ramp-in does not exist. It only means Drift cannot cite a number, so it must
not present one as a fact.

The toggle is the honest way to hold both positions at once:

- **On** (default): ramp-in costs you, and fragmentation is punished hard, because a day of
  20-minute blocks earns very little.
- **Off**: a block counts from its first minute, and only `min_block` filters out blips.

Run a week each way and compare. If warm-up is real for you, the two numbers will disagree in a way
that matches how the weeks actually felt. The default of 15 minutes comes from the measured
resumption phase in claim 3, not from the folklore figure of 15 to 20 minutes to reach flow.

Rules that keep the number honest:

- The first block of the day pays no tax. You are not resuming anything yet.
- Idle time charges nothing. Lunch is not a context switch.
- `spent` is allowed to pass 100 percent of the budget. That is a real signal, not an error.
- Fragmentation costs even when no block qualifies. Six trips to Slack charge six re-entry costs
  whether or not the work between them ever reached 15 minutes. This is deliberate. Iqbal and
  Horvitz found that tasks abandoned after less than 5 minutes had a 10 percent chance of never
  being resumed within 2 hours.

### What it shows

One line in the dropdown:

> 2h 40m focused. 1h 05m switch tax. 3h 45m of your 4h budget.

The switch tax is the product. It puts a price on switching in the same unit as the work, which is
the only way to compare them.

### Honesty requirements

These are constraints on the feature, not suggestions:

- Every coefficient above is visible and editable in the interface. If the numbers feel wrong after
  a week, you change them instead of distrusting the app.
- The budget is labelled as a setting, not as a fact about you. The 4-hour figure is a
  generalization of a 1993 study of violin students who practiced 3.5 hours a day. It is a
  reasonable default and nothing more.
- Drift reports the number and the tax. It never draws the conclusion. An app that announces your
  focus is spent at 2pm becomes a permission slip to stop working, which inverts the point.
- Warm-up is never called "time to reach flow", and never presented as measured. It is labelled as
  your setting, it carries a link to [research.md](research.md) claim 6, and it can be switched off.

### Open questions for this phase

- Does the per-app re-entry cost feel right, or does the cost need to depend on how deep the broken
  block was? Iqbal and Horvitz measured alert-driven interruptions. Drift charges self-driven
  switches the same way, which the study does not cover.
- Does warm-up hold up against a week of your own data with the toggle off? This is the one
  coefficient with no measurement behind it, so it is the one most worth testing against your
  own experience.
- Companion apps are companions relative to a task, not absolutely. A per-project companion set is
  the obvious refinement, and the obvious complexity. Live with the flat list first.
- The standard deviations in the source study are as large as the means. Daily numbers will be
  noisy. Weekly totals are likely the honest unit of display.

## Phased roadmap

**Phase 1 — Mac tracker (this is the buildable "today" version)**
Menu bar app, local logging, live count in the menu bar, a dropdown with today's stats and top offenders. Ships as something you actually run daily.

**Phase 1.5 — Focus budget and switch tax**
Price the switching. See "The focus budget" above for the model, the coefficients, and the
honesty constraints that go with it.

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
