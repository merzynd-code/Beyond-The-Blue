# Fact check and consistency fixes — 15 November 2026

Everything below is in the game data (`data/missions/*.json` real data, `data/components.json`) and in
`data/education/fact_cards.json` (NASA FACT cards, each with sources and a "checked" date), and is
enforced by `tests/test_education.py`, `tests/test_catalogue.py` and `tests/test_cutoffs_streaming_funding.py`.

## Factual fixes

| Topic | Old | Now (game text) | Sources |
|---|---|---|---|
| GPS satellite count | "31 operational" (2023 figure) | "GPS needs at least 24 working satellites. About 31–32 are operational: the count reached 32 in April 2026, when the last GPS III satellite was launched." | REF-GPS, REF-GPS-SSC-2026 (21 Apr 2026) |
| GPS orbits | — (kept, already correct) | About 20,200 km, two orbits a day, 6 planes, about 55°. | REF-GPS |
| MOXIE | present tense | Past tense: made oxygen 16 times; **last run August 2023**; mission complete. | REF-MOXIE |
| Ingenuity | present tense | Past tense: **72 flights**; **mission ended January 2024** after rotor damage. | REF-INGENUITY-END |
| Perseverance samples | "for return to Earth" | "**cached for possible future return**". The earlier Mars Sample Return plan was ended: NASA proposed it in its 2025 budget request and Congress confirmed it in January 2026. | REF-MSR-END, REF-MSR-CONGRESS |
| Europa Clipper instruments | — | **9 science instruments plus a gravity and radio science investigation**. | REF-CLIPPER |
| Europa Clipper tour | — | Arrives **April 2030**; **49 flybys** of Europa. | REF-CLIPPER-TIMELINE, REF-CLIPPER-EPSC |
| Europa Clipper Earth flyby | — | **3 Dec 2026** gravity assist, about 3,200 km from Earth. Shown as a **live card** with a countdown ("Happening in 18 days" on 15 Nov). | REF-CLIPPER-TIMELINE, REF-CLIPPER-EPSC |
| Europa Clipper mass | varied | "**about 6,000 kg**" (NASA pages differ; cited). | REF-CLIPPER-MARS-GA |
| 110 W in the schema | labelled a game value | It **is** the real MMRTG output (about 110 W at the start of a mission). Tagged **NASA FACT** on the component with `truth_overrides`, linked to the `mmrtg_power` fact card. Game balance unchanged. | REF-M2020-POWER |
| Sunlight | flat `illumination` per phase | **Inverse-square law**: solar output × `1 / d²` (d in AU), then × local illumination. Mars ≈ **43 %**, Jupiter ≈ **3–4 %** of Earth's sunlight. Taught in cards, a knowledge check per mission, and the solar part's card at each destination. | REF-INVERSE-SQUARE |

Note on wording: GPS is operated by the **U.S. Space Force**, not NASA. The game keeps BENNU's
five labels, so GPS facts carry the **NASA FACT** label with the source shown on every card
("U.S. Space Force (GPS.gov)"); the label's definition says so. If the team prefers, renaming the
label to "REAL FACT" is a text change in `ui/frontier/common.py`, `engine/bennu/glossary.py` and
`data/education/truth_labels.json`.

## Consistency fixes

| Fix | Where |
|---|---|
| **One set of 5 truth labels** everywhere: NASA FACT · SCIENCE PRINCIPLE · SIMULATED · GAME VALUE · FICTIONAL, identical to BENNU FRONTIER. "GAME UNITS" text on the mission screens is now "GAME VALUES". | `truth_labels.json`, `engine/__init__.py`, mission screens, MODEL_CONTRACT §1 |
| **Currency CR → US$** on every screen, report, review, readiness message, briefing, funding text and spoken text. JSON field names keep `_cr` so saves stay valid. | `ui/hub.py`, `ui/app.py`, `engine/review.py`, `engine/validation.py`, `ui/speech.py`, mission data |
| **Leftover "lunar" placeholders removed** from the docs and `tools/dev_harness.py` (which loaded a mission that no longer exists). | ARCHITECTURE, DATA_SCHEMAS, DESIGN_SYSTEM, dev harness |
| **Real mission names rule**: mission titles are now GPS Satellite, **Delta Rover** and **Europa Ice Probe**, each with an "Inspired by …" line; flights get seeded names. Real names stay only in cited facts. | mission data, `naming_and_credits.json`, tests |
| **Credit line + no-endorsement disclaimer**: "Student-created educational game • Not a NASA product" on every screen; full disclaimer in Academy → Credits; **AI-assisted assets** button and "AI-generated" labels on the AI pictures; no NASA logo or NASA image. | main page, Academy credits, README, REFERENCES asset ledger |
| **Sunlight by the inverse-square law** in the engine (`sun_distance_au`), with balance kept. | `engine/mission.py`, mission data, MODEL_CONTRACT §5 |
