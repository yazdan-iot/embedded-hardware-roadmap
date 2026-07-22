# Embedded Hardware Engineering Roadmap

![Status](https://img.shields.io/badge/status-work%20in%20progress-yellow)
![Notes Language](https://img.shields.io/badge/notes%20language-Persian%20(Farsi)-blue)
![Flashcards](https://img.shields.io/badge/flashcards-Anki-1E90FF)
![AI Assisted](https://img.shields.io/badge/drafted%20with-Claude%20(Anthropic)-8A2BE2)

Personal, self-taught study notes on the **hardware side of embedded engineering** — electronics fundamentals, components, communication protocols, PCB design, and everything in between. Written from scratch while learning, organized as a **roadmap**: broad coverage first, depth added topic by topic as I actually study each one.

The full mind map — all 25 topic areas with a short "what it is / why it matters" for each, plus the suggested study order — lives in **[ROADMAP.md](./ROADMAP.md)**. The table below mirrors that structure and tracks which sections actually have notes written so far.

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
| 01 | Electricity Fundamentals & Basic Laws | [notes.md](./01-electricity-fundamentals/notes.md) | [flashcards](./01-electricity-fundamentals/flashcards/) | 🔄 |
| 02 | Passive Components | [notes.md](./02-passive-components/notes.md) | [flashcards](./02-passive-components/flashcards/) | ⬜ |
| 03 | Discrete Active Components | [notes.md](./03-discrete-active-components/notes.md) | [flashcards](./03-discrete-active-components/flashcards/) | ⬜ |
| 04 | Power Electronics & Power Supply | [notes.md](./04-power-electronics/notes.md) | [flashcards](./04-power-electronics/flashcards/) | ⬜ |
| 05 | Protection Circuits | [notes.md](./05-protection-circuits/notes.md) | [flashcards](./05-protection-circuits/flashcards/) | ⬜ |
| 06 | Digital Logic & GPIO Concepts | [notes.md](./06-digital-logic-gpio/notes.md) | [flashcards](./06-digital-logic-gpio/flashcards/) | ⬜ |
| 07 | Serial Communication Protocols & Buses | [notes.md](./07-communication-protocols/notes.md) | [flashcards](./07-communication-protocols/flashcards/) | ⬜ |
| 08 | Memory Technologies | [notes.md](./08-memory-technologies/notes.md) | [flashcards](./08-memory-technologies/flashcards/) | ⬜ |
| 09 | Clock, Timing & Reset | [notes.md](./09-clock-timing-reset/notes.md) | [flashcards](./09-clock-timing-reset/flashcards/) | ⬜ |
| 10 | Analog↔Digital Conversion & Signal Conditioning | [notes.md](./10-adc-dac-signal-conditioning/notes.md) | [flashcards](./10-adc-dac-signal-conditioning/flashcards/) | ⬜ |
| 11 | Sensors | [notes.md](./11-sensors/notes.md) | [flashcards](./11-sensors/flashcards/) | ⬜ |
| 12 | Actuators & Motor Drivers | [notes.md](./12-actuators-motor-drivers/notes.md) | [flashcards](./12-actuators-motor-drivers/flashcards/) | ⬜ |
| 13 | PCB Design & Layout | [notes.md](./13-pcb-design-layout/notes.md) | [flashcards](./13-pcb-design-layout/flashcards/) | ⬜ |
| 14 | Signal Integrity & EMI/EMC | [notes.md](./14-signal-integrity-emc/notes.md) | [flashcards](./14-signal-integrity-emc/flashcards/) | ⬜ |
| 15 | Grounding & On-Board Power Distribution | [notes.md](./15-grounding-power-distribution/notes.md) | [flashcards](./15-grounding-power-distribution/flashcards/) | ⬜ |
| 16 | Thermal Management | [notes.md](./16-thermal-management/notes.md) | [flashcards](./16-thermal-management/flashcards/) | ⬜ |
| 17 | Battery & Power Management | [notes.md](./17-battery-power-management/notes.md) | [flashcards](./17-battery-power-management/flashcards/) | ⬜ |
| 18 | RF & Wireless Fundamentals | [notes.md](./18-rf-wireless-fundamentals/notes.md) | [flashcards](./18-rf-wireless-fundamentals/flashcards/) | ⬜ |
| 19 | Reading Datasheets, Schematics & Footprints | [notes.md](./19-datasheet-schematic-reading/notes.md) | [flashcards](./19-datasheet-schematic-reading/flashcards/) | ⬜ |
| 20 | Prototyping, Soldering & Lab Equipment | [notes.md](./20-prototyping-soldering-lab-equipment/notes.md) | [flashcards](./20-prototyping-soldering-lab-equipment/flashcards/) | ⬜ |
| 21 | Hardware Debugging | [notes.md](./21-hardware-debugging/notes.md) | [flashcards](./21-hardware-debugging/flashcards/) | ⬜ |
| 22 | Reliability, Testing & DFM/DFT | [notes.md](./22-reliability-testing-dfm-dft/notes.md) | [flashcards](./22-reliability-testing-dfm-dft/flashcards/) | ⬜ |
| 23 | Standards, Safety & Certification | [notes.md](./23-standards-safety-certification/notes.md) | [flashcards](./23-standards-safety-certification/flashcards/) | ⬜ |
| 24 | Hardware Project Management | [notes.md](./24-hardware-project-management/notes.md) | [flashcards](./24-hardware-project-management/flashcards/) | ⬜ |

> Section 25 of the roadmap ("Suggested Study Path") isn't a topic folder — it's a meta guide on *what order* to go through sections 1–24 in. It only lives inside [ROADMAP.md](./ROADMAP.md).
>
> This table is the single source of truth for progress. Every time a section is added or updated, this table and the changelog entry below get updated in the same commit.

---

## Changelog

A running log of what was added, newest first. Copy the line template below whenever you add or update a section.

```
- YYYY-MM-DD — Added notes.md for section 01 (Passive Components) + 20 interview flashcards
```

**Log:**

- 2026-07-22 — Added `ROADMAP.md`, the full 25-section mind map this README is now structured around.
- 2026-07-22 — Added `01-electricity-fundamentals/notes.md` (full notes) and `01-electricity-fundamentals/flashcards/` (20 interview-focused flashcards).

---

## Using this repo

- **Just want a map of what to study?** Skim the table above — each `notes.md` is written broad-first so you can see the full shape of a topic before deciding whether to go deep on it.
- **Want to actually study a section?** Open its `notes.md`.
- **Want to drill and retain it?** Import the matching `.apkg` from that section's `flashcards/` folder into [Anki](https://apps.ankiweb.net/).
- **Found something wrong or unclear?** Please open an issue — I'd rather fix it than let it sit here as a wrong note someone else learns from.

## License

Notes are shared as-is for personal/educational use. Feel free to reference or adapt with attribution; please don't republish wholesale as your own.
