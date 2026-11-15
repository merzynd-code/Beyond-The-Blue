# BENNU ACADEMY — the learning layer of BENNU FRONTIER

Goal (from the team's *Learning & Scientific Knowledge Layer* guide): a player should finish BENNU
FRONTIER knowing more about how a real space mission works than when they started.

## What is built

| Guide requirement | Where it is in the game |
|---|---|
| LEARN before a decision | A **?** after every lever label (all four station panels), on every resource chip (click the chip), on the site evidence, the rotation-phase dial, the combo preview, the launcher, mass, budget, extras and Verify on the Build screen, and the goals and Big Picture on the Briefing. Each opens the term in a side panel without leaving the screen. |
| Parameter panel contents | Each entry has: full name and abbreviation; plain definition; the principle (labelled NASA FACT or SCIENCE PRINCIPLE); why it matters; unit and what the unit means; how it works in BENNU FRONTIER (labelled SIMULATED, GAME VALUE or FICTIONAL); what changing it does; connected terms (clickable); an example; sources with an Open button. Two short pages ("Learn" / "In this game"), so nothing is cut off. |
| Knowledge library for all stations and phases | **? Library** in the top bar on every screen (both modes) lists all 51 terms by phase; the **?** on a station panel lists that station's terms. |
| EXPERIMENT | The combo preview and the live chips (Power plan, Manoeuvre cost, New data) show the effect of a lever before the round is resolved. Solo play and **Fly again** with the same seed let players repeat a mission with different choices. |
| DISCOVER after each round | The round summary lists every resolution step in order, with why loads were shed, data lost, or safe mode triggered. |
| REFLECT at the end | **What you learned** (after the Mission outcome): sound engineering judgement (combos, requirements met), what was misunderstood or wasted with evidence (anti-combos, by round), failures and losses with the reason (safe modes, particle hits, lost GB, failed requirements with measured vs needed values), hidden-rule discoveries with their evidence and truth label, next-time tips, key concepts used, and sources. Every line has a **?** to its glossary entry. |
| Solo and multiplayer | The panels are local to each computer and never change the mission, so every teammate can read while others play. |
| No duplicate values | `data/bennu/glossary.json` holds text only. Numbers are placeholders such as `{ext:rounds_mode.bands.survey.alt_km}` or `{cfg:manoeuvres.protected_reserve_gu}`, filled from the game's own configuration when it loads (`engine/bennu/glossary.py`); a test fails if a placeholder points nowhere or if a number is copied. |
| Source-grounded, no endorsement | Every entry cites ids from the existing `source_register` (NASA OSIRIS-REx pages, NASA Basics of Space Flight, the NASA science-mission webinar, MDL, GMAT, Webb Build It Yourself, peer-reviewed papers). Each panel states the project is independent and not endorsed by NASA. |

## Truth labels (one set everywhere)

| Label | Meaning | Example |
|---|---|---|
| **NASA FACT** | A real, sourced statement about the mission or Bennu | TAG on 20 Oct 2020; the 60 g requirement; particle ejections were observed |
| **SCIENCE PRINCIPLE** | An established scientific or engineering idea, or a value derived from one | photovoltaics; v = sqrt(GM/r); 1 GB = 10^9 bits |
| **SIMULATED** | An output of this game's simplified model | drift %, coverage, sample grams, the event log |
| **GAME VALUE** | An invented number that makes the game work | kg, US$, battery-mode limits, the 120° particle window, score weights |
| **FICTIONAL** | An invented situation or object | Sites A/B/C, the Kestrel/Condor/Albatross launchers, the thermal camera noise rule (D) |

The guide's names map as follows: GAME SIMULATION → **SIMULATED**, FICTIONAL SCENARIO → **FICTIONAL**.
Older internal codes in the data files (SCIENCE_BASED, SIM_OUTPUT) are always shown with these five names.
Units are real everywhere (kg, GB, US$); the **numbers** are GAME VALUES and are labelled so in every entry.

## Acceptance checklist (from the guide)

- [x] Every visible parameter, acronym and unit has an explanation (51 entries; a test checks every lever and chip).
- [x] Every control explains its effect and trade-offs ("If you change it").
- [x] Learning content opens without leaving the gameplay screen (side panel).
- [x] Round feedback explains outcomes, not only numbers (round summary).
- [x] NASA facts are distinguished from simulations, game values and fiction (five labels on every entry).
- [x] The library covers all stations and mission phases.
- [x] The final debrief identifies learning outcomes with evidence (What you learned).
- [x] Solo and multiplayer both expose the learning system.
- [x] Authoritative references, no implied endorsement.

## Future work
* **Experiment mode**: a sandbox copy of the current round to try settings without risk.
* **Searchable Mission Dictionary**: a search box over the same glossary file.
* **Learning profile**: track which concepts a player has met across missions.
