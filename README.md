# Embedded Hardware Engineering Roadmap

![Status](https://img.shields.io/badge/status-work%20in%20progress-yellow)
![Notes Language](https://img.shields.io/badge/notes%20language-Persian%20(Farsi)-blue)
![Flashcards](https://img.shields.io/badge/flashcards-Anki-1E90FF)
![AI Assisted](https://img.shields.io/badge/drafted%20with-Claude%20(Anthropic)-8A2BE2)

Personal, self-taught study notes on the **hardware side of embedded engineering** — electronics fundamentals, components, communication protocols, PCB design, and everything in between. Written from scratch while learning, organized as a **roadmap**: broad coverage first, depth added topic by topic as I actually study each one.

I'm not doing this for a bootcamp or a course. I'm doing it because I already have a solid software background (C/C++, FreeRTOS, NVS, OTA, driver-level firmware work) but wanted to properly understand the hardware side — why a wire goes to a specific pin, why a specific component is chosen, how to read a datasheet and actually reason about a schematic instead of just copying it. This repo is the trail I'm leaving behind while closing that gap.

If you're also starting out in embedded hardware, feel free to use this as a roadmap for what to study and in what order. If you spot a mistake, please open an issue or a PR — I'd genuinely rather be corrected than have wrong notes sitting here.

---

## How these notes are written

I use **Claude (Anthropic's AI)** heavily while writing these notes — and I want to be upfront and specific about that, because I think it's worth explaining rather than hiding.

My process, roughly:
1. I pick a topic from the roadmap below.
2. I use Claude to pull together explanations, mental models, and practical examples from solid electronics fundamentals — the kind of synthesis that would normally mean digging through multiple textbooks, datasheets, and app notes.
3. I go back and forth with it: asking for simpler explanations where something isn't clicking, asking for real numeric examples, asking "why" until the reasoning actually makes sense to me, not just the conclusion.
4. I verify anything I'm unsure about against real datasheets, and test it hands-on on a breadboard/PCB whenever possible.
5. Only after that does it become a note in this repo.

I see this as a skill, not a shortcut. Knowing how to direct an AI tool toward a genuinely well-structured, technically accurate, and clearly explained set of notes — and knowing when to double-check it — is itself something I had to learn. The notes here are the output of that process, not a copy-paste of a chat log.

---

## Repository structure

Each topic lives in its own numbered folder, matching the roadmap order below:

```
section-folder/
├── notes.md              → the reference notes for that topic
└── flashcards/
    └── section-name.apkg → Anki deck for that topic (import into Anki)
```

- `notes.md` is meant to be read and referenced — it's the actual explanation.
- The `.apkg` file is meant to be **imported into Anki** for spaced-repetition review, or used to drill for interview prep. Notes and flashcards are kept separate on purpose: one is for understanding, the other is for retention.

---

## Roadmap & progress

Legend: ✅ done · 🔄 in progress · ⬜ not started yet

| # | Section | Notes | Flashcards | Status |
|---|---|---|---|---|
| 00 | Prerequisites (basic electrical physics) | [notes.md](./00-prerequisites/notes.md) | [flashcards](./00-prerequisites/flashcards/) | ⬜ |
| 01 | Passive Components (R, L, C) | [notes.md](./01-passive-components/notes.md) | [flashcards](./01-passive-components/flashcards/) | 🔄 |
| 02 | Semiconductors & Basic Analog | [notes.md](./02-semiconductors-analog/notes.md) | [flashcards](./02-semiconductors-analog/flashcards/) | ⬜ |
| 03 | Digital Logic Circuits | [notes.md](./03-digital-logic/notes.md) | [flashcards](./03-digital-logic/flashcards/) | ⬜ |
| 04 | Power Management | [notes.md](./04-power-management/notes.md) | [flashcards](./04-power-management/flashcards/) | ⬜ |
| 05 | Circuit Protection & Stability | [notes.md](./05-protection-stability/notes.md) | [flashcards](./05-protection-stability/flashcards/) | ⬜ |
| 06 | Clocks & Timing | [notes.md](./06-clock-timing/notes.md) | [flashcards](./06-clock-timing/flashcards/) | ⬜ |
| 07 | MCU Interfaces (GPIO/ADC/DAC/PWM) | [notes.md](./07-mcu-interfaces/notes.md) | [flashcards](./07-mcu-interfaces/flashcards/) | ⬜ |
| 08 | Wired Communication Protocols | [notes.md](./08-wired-protocols/notes.md) | [flashcards](./08-wired-protocols/flashcards/) | ⬜ |
| 09 | Wireless Communication & RF | [notes.md](./09-wireless-rf/notes.md) | [flashcards](./09-wireless-rf/flashcards/) | ⬜ |
| 10 | Sensors | [notes.md](./10-sensors/notes.md) | [flashcards](./10-sensors/flashcards/) | ⬜ |
| 11 | Actuators & Motors | [notes.md](./11-actuators-motors/notes.md) | [flashcards](./11-actuators-motors/flashcards/) | ⬜ |
| 12 | PCB Design | [notes.md](./12-pcb-design/notes.md) | [flashcards](./12-pcb-design/flashcards/) | ⬜ |
| 13 | Signal Integrity & EMI/EMC | [notes.md](./13-signal-integrity-emc/notes.md) | [flashcards](./13-signal-integrity-emc/flashcards/) | ⬜ |
| 14 | Assembly & Soldering | [notes.md](./14-assembly-soldering/notes.md) | [flashcards](./14-assembly-soldering/flashcards/) | ⬜ |
| 15 | Test & Debug Tools | [notes.md](./15-test-debug-tools/notes.md) | [flashcards](./15-test-debug-tools/flashcards/) | ⬜ |
| 16 | Datasheet Reading & Component Selection | [notes.md](./16-datasheet-component-selection/notes.md) | [flashcards](./16-datasheet-component-selection/flashcards/) | ⬜ |
| 17 | Connectors, Cables & Wiring | [notes.md](./17-connectors-wiring/notes.md) | [flashcards](./17-connectors-wiring/flashcards/) | ⬜ |
| 18 | Thermal Management & Mechanical Enclosure | [notes.md](./18-thermal-mechanical/notes.md) | [flashcards](./18-thermal-mechanical/flashcards/) | ⬜ |
| 19 | Standards, Safety & Regulatory | [notes.md](./19-standards-safety/notes.md) | [flashcards](./19-standards-safety/flashcards/) | ⬜ |
| 20 | Advanced Specialized Topics | [notes.md](./20-advanced-topics/notes.md) | [flashcards](./20-advanced-topics/flashcards/) | ⬜ |

> This table is the single source of truth for progress. Every time a section is added or updated, this table and the entry below get updated in the same commit.

---

## Changelog

A running log of what was added, newest first. Copy the line template below whenever you add or update a section.

```
- YYYY-MM-DD — Added notes.md for section 01 (Passive Components) + 20 interview flashcards
```

**Log:**

- 2026-07-22 — Added `01-passive-components/notes.md` (full notes) and `01-passive-components/flashcards/` (20 interview-focused flashcards).

---

## Using this repo

- **Just want a map of what to study?** Skim the table above — each `notes.md` is written broad-first so you can see the full shape of a topic before deciding whether to go deep on it.
- **Want to actually study a section?** Open its `notes.md`.
- **Want to drill and retain it?** Import the matching `.apkg` from that section's `flashcards/` folder into [Anki](https://apps.ankiweb.net/).
- **Found something wrong or unclear?** Please open an issue — I'd rather fix it than let it sit here as a wrong note someone else learns from.

## License

Notes are shared as-is for personal/educational use. Feel free to reference or adapt with attribution; please don't republish wholesale as your own.
