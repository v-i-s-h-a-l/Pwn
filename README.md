# Pwn

> Pilot your Mac with a game controller. Talk to it. Make work feel like play.

## What this is

Most of what I do at a Mac all day isn't typing. It's switching apps, jumping between terminal panes, navigating browser tabs, asking an LLM something, dictating a quick message, picking from a list of choices. The keyboard and trackpad are doing a lot of work for what could be a few buttons and a stick.

**Pwn** is an experiment: drive your Mac with a game controller. Move between windows and panes. Switch apps. Push a trigger to talk to an agent — dictate, ask a question, pick from options it offers. Make it feel like playing a game while you're actually getting work done.

The primary target is the DualSense (because of its haptics, adaptive triggers, and lightbar — affordances begging to be used semantically). But anything macOS recognizes as a HID — Xbox controller, MFi pad, flight stick, MIDI macro pad — should work once bindings are remappable.

## Why

- Switching contexts on a Mac is a hand-eye dance of Cmd-Tab, Cmd-\`, Cmd-{, mouse, trackpad, repeat. A D-pad and two sticks can do most of this in fewer motions, with less context-switching for the brain.
- Most "talk to AI" interactions are short: ask a question, dictate a message, pick from a list. A trigger-to-talk + face-button-to-select model is faster and more pleasant than alt-tabbing into a chat window.
- It should feel good. Haptics, adaptive trigger resistance, sound, modal layers — these are the language of video games. There's no reason work has to feel sterile when the same input device can make it tactile.

## Where the name comes from

**Pwn** — gamer/hacker l33t-speak for "own," coined by a mistype in the early 2000s and never let go of by either tribe since. To pwn something is to dominate it, take it over, make it yours. That's the project: pwn your Mac with a controller. The fact that gamers and devs have been using the same word with the same meaning for two decades is the amalgamation Pwn is aiming at.

## What v0 wants to do (narrow on purpose)

1. **Navigate** — switch apps, move between window panes (iTerm, tmux, Ghostty, Warp), focus windows. D-pad and sticks.
2. **Push-to-talk** — hold a trigger, speak, the focused text field receives the transcript. Local Whisper, no round trip.
3. **Pick from LLM options** — when an agent surfaces a small list of choices (1–8), they appear as a HUD overlay and the face buttons / D-pad pick one.

**Explicitly not in v0:** trying to replace the keyboard for arbitrary typing. The controller isn't a keyboard. It's a remote control + a mic button + a quick picker.

## What it could grow into

- DualSense affordances used **semantically**: adaptive trigger resistance = active mode indicator, lightbar color = active layer, haptics = action confirmation. Apple's `GameController.framework` + Core Haptics support all of this natively, no third-party drivers.
- Per-app and per-user **loadouts** (à la Steam Input action sets).
- Support for arbitrary HIDs — flight sticks, MIDI controllers, racing wheels — wired through the same loadout DSL.

---

## Landscape & prior art

Before starting, I researched what already exists. Short version: **the project is novel as an integrated stack, not as any single component.** Every piece exists somewhere; nobody composes them.

### Closest existing products

- **[ConMapper](https://conmapper.app/)** — closed, macOS. Controller → keyboard / mouse / Apple Shortcuts / layers / gyro. Productivity-framed. No LLM HUD, no dictation, no DualSense haptics or adaptive-trigger semantics. This is the closest existing product.
- **Joystick Mapper, [Enjoyable](https://yukkurigames.com/enjoyable/), ControllerMate, [AntiMicroX](https://github.com/AntiMicroX/antimicrox)** — all "translate gamepad → key events." None treat the pad as a primary OS driver.
- **Steam Input** — best-in-class action-set / community-profile UX, but only works for Steam-launched processes. Worth studying for UX, not for architecture.

### Apple's own prior art (important)

- **Voice Control "numbered grid"** ([docs](https://support.apple.com/guide/mac-help/use-item-number-and-grid-overlays-with-voice-control-mchl26854b08/mac)) — Apple already overlays numbers on every actionable UI element on screen and lets you pick by saying the number. This is the structural twin of the LLM HUD idea. The difference: *exhaustive* (every element) vs. *curated* (4–8 relevant actions).
- **[Switch Control](https://support.apple.com/guide/mac-help/use-switch-control-mh43607/mac)** — Apple explicitly supports joysticks-as-switches today. Scan-based, designed for severe motor disability — slow UX, but it proves the API surface exists for gamepad-driving-the-OS.

### Open-source reference architectures

- **[Karabiner-Elements](https://karabiner-elements.pqrs.org/)** — privileged daemon + virtual HID device (DriverKit). Right backbone shape for synthesizing OS events from controller input.
- **[Hammerspoon](https://github.com/Hammerspoon/hammerspoon)** — `hs.hotkey.modal` + Spoons plugin model. Right template for the loadout / extension surface.
- **[yabai](https://github.com/koekeishiya/yabai)**, **[skhd](https://github.com/koekeishiya/skhd)** — how far you can push macOS window management; where the entitlement walls are.
- **[Vimac](https://github.com/nchudleigh/vimac)** (archived) and **[Homerow](https://www.homerow.app/)** (closed) — AX-tree overlay with hint characters. Closest precedent for the HUD substrate.
- **[whisper.cpp](https://github.com/ggml-org/whisper.cpp)** — local push-to-talk STT, Metal-accelerated. The STT problem is solved; the work is the UX shell.
- **Apple `GameController.framework`** — [`GCDualSenseGamepad`](https://developer.apple.com/documentation/gamecontroller/gcdualsensegamepad), `GCDualSenseAdaptiveTrigger`, Core Haptics. Gyro, adaptive triggers, dual-motor + actuator haptics, lightbar — natively, wired or Bluetooth, no third-party driver.

### Adjacent validation

- **Stream Deck, [Monogram](https://monogramcc.com/), MIDI macro pads** — purpose-built productivity HIDs are a real category. Users want this; Mac just lacks a flexible enough software layer.
- **Joy-Cons for Photoshop, flight sticks for video editing** — hobbyist patterns prove the "any HID for productivity" thesis empirically.

### Differentiated thesis

1. **LLM-curated HUD** — Voice Control's exhaustive numbered grid, inverted into 4–8 actions relevant to current intent, button-mappable.
2. **DualSense affordances used semantically** — trigger resistance as mode indicator, lightbar as active layer, haptics as confirmation. Apple's first-party APIs make this tractable; nobody does it.
3. **Unified loadout DSL** tying controller + dictation + window management + LLM HUD into one coherent modal system.

---

## Open design questions

- **Mode design.** How do you cleanly separate "navigate windows" from "push-to-talk" from "pick an LLM option" on one controller without overloading it? Vim-like modal layers, Destiny-like radial menus, fighting-game-like trigger holds? Worth solving on paper before code.
- **Where does the HUD live?** Floating overlay panel, menu-bar dropdown, accent on the focused window? Borrow from Voice Control / Homerow / Raycast?
- **Loadout DSL.** Lua (Hammerspoon-style)? Native Swift config? JSON + UI? Steam Input action-set imports?
- **Daemon vs. app.** Karabiner-style privileged daemon + virtual HID device, or a single-process Swift app with Accessibility + Input Monitoring permissions? What's the simplest thing that could work for v0?
- **Permissions story.** Accessibility, Input Monitoring, Microphone, possibly Screen Recording. How to onboard without scaring people off.
- **Scope of "any HID."** Is the loadout DSL device-agnostic from day one, or DualSense-first with the abstraction extracted later once we know what we actually need?

## Status

v0. Just this README. Nothing usable yet. The next conversations live in this repo.

## License

TBD — likely MIT.
