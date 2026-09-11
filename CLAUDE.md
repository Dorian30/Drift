# CLAUDE.md

Working rules for Drift. Read [spec.md](spec.md) for the design and
[research.md](research.md) for the evidence behind it.

## What this repo is

Drift is a macOS menu bar app that counts how often you change the frontmost application, and
prices that switching in minutes. It is a personal daily-driver tool. The repository currently
holds the plan and the research, and no code yet.

## Writing

Use the `simple-english` skill for every document in this repo: commit messages, pull request
descriptions, and all Markdown.

This rule is here for agents that do not load the skill automatically, such as a cloud session or a
different machine. If the skill is already active, follow it and change nothing.

## Research rules

These matter more than anything else in this file. The whole point of `research.md` is that
plausible productivity numbers are usually wrong.

1. **Never add a number to `spec.md` without a source in `research.md`.** If you want to cite a
   figure that is not in `research.md`, check it against the primary paper first, then add it to
   `research.md` with a verdict, then use it.
2. **Never cite "23 minutes" for interruption recovery.** No published paper contains that figure.
   It is the most repeated statistic in this field and it does not survive a source check. See
   `research.md` claim 1. An agent adding "research context" to this project will reach for it
   almost every time. Do not.
3. **Prefer the measured number to the popular one.** The resumption figures from Iqbal and Horvitz
   (10 to 16 minutes) are the defaults in this project because they come from a logged field study.
4. **Mark paywalled sources as paywalled.** Do not present a figure read from an abstract as if it
   came from the full text.

## Product honesty rules

These are requirements, not preferences. They exist because the subject matter invites false
precision.

- **Drift informs. It never concludes.** Report the number and the switch tax. Do not tell the user
  that their focus is spent, or that a day was good or bad. An app that draws the conclusion becomes
  a permission slip to stop working.
- **Label every unmeasured assumption as a setting.** The daily budget and the warm-up time have no
  measurement behind them for knowledge work. The interface presents them as the user's settings,
  with a link to the relevant `research.md` claim, and both can be changed.
- **Every coefficient is visible and editable.** Drift applies a rule the user agreed to. It does
  not measure anyone's brain.
- **Minutes are not output.** Interrupted work is completed faster, with more stress. See
  `research.md` claim 2. Drift counts minutes because macOS can observe minutes, and says so.

## Vocabulary

Use these words. Do not introduce synonyms.

| Use | Not |
|---|---|
| focus budget | energy, battery, reserves |
| switch tax | penalty, cost of distraction |
| block | session, sprint, focus period |
| warm-up | time to reach flow |
| anchor / companion / interrupting app | deep / shallow app |

Never write "depleted" or draw a battery. Ego depletion failed replication. See `research.md`
claim 7.

App roles answer one question: does this switch serve the current task, or leave it? The question
is not how demanding the app is. Slack breaks concentration. Finder usually does not, because
opening Finder is normally part of the work.

## Scope

The MVP is a switch counter with a local log and a menu bar count. Keep it that way. Good ideas that
are not the MVP go in the "Further exploration" section of `spec.md` with a note on why they are
deferred. Do not grow Phase 1.

Nothing leaves the machine. No telemetry, no sync, no accounts. Tracking is passive: Drift listens
for the `NSWorkspace` notification that the frontmost app changed. It does not read window contents
or keystrokes, and it does not request Accessibility permission.

## Commits

Keep the `Co-Authored-By` and `Claude-Session` trailers on commits and pull requests. The user
confirmed this deliberately for this public repository, after checking that the session link returns
403 to anyone who is not signed in.
