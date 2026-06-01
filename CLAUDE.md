# Pwn — context for any AI assistant working in this repo

If you're a human, just read [README.md](README.md). This file exists so a fresh Claude session boots into the right mental model without the user having to re-explain.

## Project in one paragraph

Pwn lets you drive macOS with a game controller. DualSense is the primary target (because of its haptics, adaptive triggers, and lightbar — affordances we plan to use semantically), but the design is device-agnostic: any HID macOS recognizes (Xbox, MFi, flight stick, MIDI macro pad) should work via remappable bindings. v0 does three things: (1) navigate windows / terminal panes / apps with sticks + D-pad, (2) push-to-talk dictation into the focused text field via local Whisper, (3) a HUD overlay that shows 4–8 LLM-curated options, selectable with face buttons. Full vision, the differentiated thesis vs. existing products, the landscape research, and the open design questions are in [README.md](README.md) — **read it before suggesting anything substantive**.

## What v0 is and isn't

v0 is intentionally narrow: **navigate + push-to-talk + LLM option-select.** It is *not* typing. The controller is a remote control + mic button + quick picker, not a keyboard substitute. If you catch yourself sketching designs or code that assume the controller types arbitrary text, stop and re-read the scope.

The user's actual day-to-day pattern that motivated this: "most of the time I don't have to type — I just have to switch windows or panes and then speak to the agent." Optimize for that.

## How the user wants to collaborate (important)

- **Design-first, not loop-first.** Foundational decisions get a deep upfront discussion before code. The user has been burned by greenfield rewrites caused by narrow early vision. When in doubt: pause, surface the trade-off, get alignment. Don't start scaffolding code in response to a vision-level question.
- **World-class quality bar, no leaky foundations.** SOLID, robust, scalable scaffolds. Refactor in a timely way rather than letting bloat accumulate. Don't ship "good enough" scaffolding that we'll have to gut later. No half-finished implementations.
- **Deep upfront vision conversation, then minimal involvement during execution.** The user clarifies thoroughly upfront, then steps back to testing/feedback mode. Don't pepper them with low-stakes decisions during execution — make a defensible call, note it in code or commit message, move on. Reserve interruptions for decisions that genuinely require their input.

## Where we are right now

v0. Just the README and this file. Nothing usable yet. **The next thread is the open design questions in the README** — mode design first, most likely, since it constrains nearly every downstream decision (where the HUD lives, what the loadout DSL needs to express, daemon-vs-app architecture, etc.).

The order we'll likely tackle them:
1. **Mode design** — how the controller cleanly separates navigate / talk / pick without overloading. Vim-like layers? Destiny-like radial menu? Fighting-game-like trigger holds? Solve on paper before code.
2. **Daemon-vs-app architecture** — Karabiner-style privileged daemon + virtual HID device, or single-process Swift app with Accessibility + Input Monitoring permissions? Pick the simplest thing that works for v0 without painting ourselves into a corner for v1.
3. **Loadout DSL shape** — config format that survives growing beyond DualSense.
4. **HUD substrate** — overlay window patterns, AX-tree traversal (Vimac is the open-source reference).

## Practical notes

- **Git account.** Repo is configured for the personal account (`v-i-s-h-a-l` / `vishalsingh2706@gmail.com`) at repo scope; pushes use a personal SSH key via local `core.sshCommand`. Don't change this. To verify: `git config --local --list | grep -E "user\.|core\.ssh"`.
- **Repo is public OSS.** License TBD — likely MIT.
- **Stack not yet committed.** Swift is the likely primary language given the macOS-native scope (GameController.framework, Core Haptics, Accessibility APIs are all Swift/Obj-C-first), but no code has been written, and no architectural choice is binding yet.
- **Naming history (for context, do not relitigate):** the project was briefly named "Glyph" on 2026-06-01 before being renamed "Pwn" the same day. The user wanted a Gen Z gamer-meets-dev culture amalgamation with some edge; Pwn — l33t-speak for "own," shared identically by both tribes for 20 years — won. Don't suggest renaming again unless explicitly asked.

## Skills worth using when relevant

These global skills are likely to come up:

- `swift-architecture-skill` — when planning module / package structure
- `swift-api-design-guidelines-skill` — when designing protocols, types, or argument labels
- `swiftui-pro` / `swiftui-view-refactor` — when writing or reviewing SwiftUI (for the HUD)
- `swift-concurrency-pro` — when writing async / actor / Sendable code
- `swift-testing-pro` — when writing tests
- `swiftui-liquid-glass` — if the HUD targets iOS 26+ visual language on macOS

Invoke them via the Skill tool when the task matches their description.
