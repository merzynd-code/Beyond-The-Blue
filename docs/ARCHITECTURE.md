# Architecture — common library

## Confirmed stack

| Item | Status |
|---|---|
| Language | **Python 3.10+** (confirmed by team). Engine uses the standard library only. |
| UI / rendering framework | **Ursina** (confirmed by team and mentor). Used only in `main.py` and `ui/`. |
| Tests | `unittest` (standard library) — no install needed. 358 tests. |
| Optional dev tools | Pillow, only for `tools/` image makers (`make_textures.py`, `make_icons.py`, `make_hub_art.py`, `render_style_guide.py`). The game does not import Pillow itself. |

Because the engine has no framework dependency, the UI could be swapped later without
changing any engine code.

## Module map

```
                    ┌──────────────────────────────────────────────┐
  data/*.json  ───▶ │ catalogue.py   mission.py   conditions.py      │  content loading + schema checks
                    └──────────────┬───────────────────────────────┘
                                   ▼
┌─────────────┐   ┌──────────────────────────────────────────────────────────────┐
│  UI layer   │   │                    session.py  (MissionSession)              │
│ (Ursina)    │──▶│ the ONLY object the UI calls; validates, logs every decision │
│ hub.py      │◀──│                                                              │
│ app.py      │   │  config.py ─ validation.py ─ resources.py   (design + physics) │
│ academy.py  │   │  phases.py                                   (state machine)  │
│ frontier/   │   │  rng.py ─ events.py                          (seeded events)   │
│ bennu_ui.py │   │  outcome.py ─ scoring.py                     (debrief)         │
│ design_     │   │  save.py                                     (save / replay)   │
│ system.py   │   └──────────────────────────────────────────────────────────────┘
└──────┬──────┘                ▲ reads results only (never writes)
       │          ┌────────────┴─────────────────────────────────────────────────┐
       └─────────▶│ education.py  (learning layer)   ◀── data/education/*.json     │
                  │ cards · numbers · facts · checks · evidence tracker · BENNU   │
                  │ bridge; also reads the BENNU glossary (engine/bennu/glossary)│
                  └──────────────────────────────────────────────────────────────┘
  engine/bennu/ + ui/frontier/ + multiplayer/: BENNU FRONTIER and Operation Nightingale (see docs/bennu/)
```

Arrows only go one way: the engine never imports the UI, and **the simulation never
imports `education.py`**. A UI reads `session.preview()`, `session.current_event()`,
`session.summary()`, `session.outcome`, `session.score`, calls action methods, and passes
those results to `education.py` to explain them.

## Responsibilities and why each module is reusable

| Module | Responsibility | Reusable because |
|---|---|---|
| `engine/catalogue.py` | Load components; enforce per-category fields (battery can't generate, launcher can't draw power…). | All three missions draw from one catalogue; `missions` field controls availability. |
| `engine/mission.py` | Load & validate a mission definition file; resolve each phase's environment, including **sunlight by the inverse-square law** from `sun_distance_au`. | Missions are data; GPS / Mars / Europa are three JSON files, not three code paths. |
| `engine/conditions.py` | Safe JSON condition language (no `eval`). | Used by events, objectives, environment variants and effect guards in every mission. |
| `engine/config.py` | The player's choices. | Same shape for satellite, rover, flyby. |
| `engine/validation.py` | Readiness review: errors block launch, warnings explain risk. Each issue has a `code`. | Rules read from mission `requirements` and `budgets`. |
| `engine/resources.py` | Mass, budget, rocket equation, power/battery step, data store, reliability. | Pure functions that implement `MODEL_CONTRACT.md`. |
| `engine/phases.py` | Phase order, legal transitions, legal actions. | Phase *kinds* are shared; each mission lists its own phases. |
| `engine/rng.py` + `events.py` | Seeded selection from curated events, weighted by design. | Event cards are data; weighting uses capabilities/resilience/reliability. |
| `engine/outcome.py` + `scoring.py` | Four outcome classes; five transparent score dimensions. | Thresholds and weights come from mission data. |
| `engine/save.py` | Save seed + decisions; replay and verify. | Works for any mission because every change is a logged action. |
| `engine/naming.py` | Seeded mission/campaign names; blocks real mission names. | Theme words come from the mission's `theme.name_theme`. |
| `engine/review.py` | Strengths / improvements with reasons and tips. | Rules read only the finished session and mission data. |
| `engine/campaign.py` | Three-mission campaign, averaged score, rank, save/load. | Mission list is data (`CAMPAIGN_MISSIONS`). |
| `engine/report.py` | Self-contained HTML report sheet. | Built from campaign results only. |
| `engine/companion.py` | Cosmo the robot: tips, warnings, hints, fun facts with sources. | Lines come from `data/companion.json` and the session. |
| **`engine/education.py`** | **Learning layer: number explanations, part learning cards, concepts, cited fact cards, readiness and event/action "What happened" explanations, quick checks, evidence tracker (introduced → practised → demonstrated), learning debrief, BENNU bridge, naming check, credits.** | **Read-only; all content is JSON in `data/education/`; works for any mission and for BENNU.** |
| `engine/bennu/*` | BENNU FRONTIER rules, Operation Nightingale rounds, hidden rules, the BENNU glossary. | See `docs/bennu/`. |
| `ui/hub.py` | **Main page "Beyond the Blue"**: banner, campaign stats, BENNU FRONTIER flagship (Play Solo, level, Play as a Team), three learning-mission cards, campaign progress, BENNU ACADEMY card, navigation bar. | Reads campaign state only; pictures from `assets/hub/`. |
| `ui/app.py` | Mission screens; calls the session; hooks for the Academy ("?" on numbers, Learn more, Why?, What happened, What you learned). | One app object for every mission. |
| **`ui/academy.py`, `ui/academy_theme.py`, `ui/learning_views.py`** | **BENNU ACADEMY learning centre (tabs: Start here, Concepts, Numbers, Real facts, BENNU terms, Quick checks, Credits) and the card renderer.** | **`learning_views.py` has no Ursina import and is unit-tested; colours are contrast-tested.** |
| `ui/scenes3d.py`, `ui/sun_shader.py` | 3D planets, spacecraft, rocket, scenes; simple lighting shader. | Spacecraft models are assembled from the capabilities of the chosen parts. |
| `ui/design_system.py` | Tokens, scaling, HUD regions, contrast checks, motion/audio settings. | Framework-neutral numbers and rules. |

Deliberately **not** added: plugin systems, ECS, dependency injection, per-mission
subclasses, a scripting language.

## Learning layer guarantees

* The engine stays the single source of truth. `education.py` never changes mass, power,
  propulsion, data, events, scoring or outcomes (tested with a fake session; the 320 engine,
  mission and BENNU tests are unchanged and pass).
* "What happened" explanations only claim a cause that the engine's numbers or flags show;
  otherwise they say "several factors".
* Learning progress is saved in `saves/learning.json`, **separately** from mission and campaign
  saves, so replays and hashes are unaffected. Mission score and learning are shown apart.
* Quick checks never block progress: skip and retry are always available.

## Truth labels (one set everywhere)

`NASA FACT` · `SCIENCE PRINCIPLE` · `SIMULATED` · `GAME VALUE` · `FICTIONAL` — the same five in the
three learning missions, BENNU FRONTIER, BENNU ACADEMY, the data and the docs
(`data/education/truth_labels.json`, `ui/frontier/common.py`).

## Naming rule and credits

* Game mission titles are fictional (**GPS Satellite, Delta Rover, Europa Ice Probe, BENNU
  FRONTIER**) and each card shows an **"Inspired by …"** line. Real mission and spacecraft names
  appear only in cited facts (fact cards, real data, science notes, cited part notes), the
  `inspired_by` line, references and credits (`tests/test_education.py`; `engine/naming.py`
  blocks real names in generated flight names).
* The main page carries **"Student-created educational game • Not a NASA product"** and an
  **"AI-assisted assets (images/illustrations)"** button. The banner, BENNU illustration and mission
  pictures are AI-generated and labelled "AI-generated" on screen; the 3D models, textures and
  icons are made by the team's scripts. **No NASA insignia, logotype, seal or NASA image is used.**

## Missions (data only)

`data/missions/gps_navigation.json`, `mars_delta_rover.json`, `europa_flyby.json`.
Each passes `tests/test_missions.py`: the balanced sample design succeeds with good decisions
on every tested seed; with the risky design, poor decisions score clearly lower and cost
outcomes; replays are exact. Their learning plan (introduce → reinforce → apply) lives in
`data/education/curriculum.json`.
