# Data schemas

All content is JSON. Loaders reject anything malformed with a clear `DataError`.
Full working examples: `data/components.json` (real catalogue) and
`tests/fixtures/test_mission.json` (synthetic mission used only for tests).

Every number has one of the five truth labels (see `MODEL_CONTRACT.md` §1):
`NASA FACT` · `SCIENCE PRINCIPLE` · `SIMULATED` · `GAME VALUE` · `FICTIONAL`.
Units are real everywhere (`engine/units.py`): mass and propellant in **kg**, data in **GB**, money in
**US$** (millions, shown `$870M`, or thousands below $1 million, shown `$450K`). The values are GAME VALUES.
The currency is **US$ (millions)**. The JSON field names keep their original
`_cr` suffix (`cost_cr`, `budget_cr`) so existing saves, hashes and tests stay valid; every screen,
report and spoken text shows **US$**.

## Component

```jsonc
{
  "id": "power.radioisotope",            // stable, unique, never renamed once used in saves
  "name": "Radioisotope Power Unit (RTG-style)",
  "short_name": "RTG",
  "category": "power_generation",         // one of 11 categories (see below)
  "description": "Steady power day and night...",   // one line for cards
  "mass_mu": 60,                          // GAME VALUE (kg)
  "cost_cr": 140,                         // GAME VALUE, shown as US$ (millions)
  "reliability": 0.99,                    // 0..1, GAME VALUE, only affects event likelihood
  "power": {"generation_w": 110, "generation_type": "radioisotope"},
  //  generators: generation_w (rated at 1 AU from the Sun) + generation_type (solar | radioisotope)
  //  batteries:  capacity_wh ONLY (a battery stores, it never generates)
  //  consumers:  draw_w + duty ("always" | "when_active")
  "data": {"rate_du_per_h": 4},           // instruments; storage_du (storage); downlink_du_per_h (radios)
  "propulsion": {"isp_s": 310, "tank_capacity_mu": 300},   // propulsion only
  "launch": {"capacity_mu": 950},         // launch vehicles only
  "capabilities": ["radioisotope_power"], // what it provides
  "requires_capabilities": [],            // what must be provided by other parts
  "incompatible_with": [],                // component ids that cannot be combined
  "missions": ["mars_delta_rover", "europa_flyby"],   // availability
  "resilience": {"dust": 1, "cold": 1},   // -3..+3 per hazard key, GAME VALUE
  "max_per_config": 1,                    // >1 allows redundant copies
  "visual": {"asset_key": "power_rtg", "icon_key": "icon_rtg"},
  "truth_overrides": {"power.generation_w": "nasa_fact"},   // this one number is a NASA FACT
  "learning": {                                        // optional; see "Learning content"
    "fact_cards": ["mmrtg_power"],
    "fact_card_for": {"power.generation_w": "mmrtg_power"}
  },
  "source_note": {"basis": "sourced", "refs": ["REF-M2020", "REF-M2020-POWER"],
                  "note": "The 110 W output matches the real MMRTG (NASA FACT). Mass, cost and reliability are GAME VALUES."}
}
```

* `education` is the one-line note shown when a part is inspected. It may name a real mission
  only when `source_note.refs` cites it (a cited fact). The Academy's "Learn more" card adds the
  full beginner card for the part's category.
* The 110 W radioisotope value equals the real MMRTG output, so it is tagged **NASA FACT**
  through `truth_overrides` (basis `sourced`) instead of being labelled a game value. It is the only
  such exception (`tests/test_catalogue.py`).

Categories: `launch_vehicle, bus, propulsion, power_generation, battery, thermal,
avionics, communications, mobility_deployment, instrument, support`.

Hazard keys used by the catalogue: `dust, cold, radiation, terrain, navigation`.

## Mission definition (top level)

| Key | Purpose |
|---|---|
| `id`, `name`, `short_name`, `summary`, `scientific_question` | Briefing text. Titles are fictional (naming rule): GPS Satellite, Delta Rover, Europa Ice Probe. Flights get seeded names (`engine/naming.py`); `flight_name` is no longer used for Mars and Europa. |
| `inspired_by` | e.g. "Inspired by NASA's Europa Clipper. Parts and numbers here are GAME VALUES." One of the few places a real name may appear. |
| `card_text` | **New.** One short sentence for the main-page mission card. |
| `display_scale` | **New.** `{mass, cost, data, note}`: multipliers that turn internal numbers into this mission's real-world display sizes (kg, US$ million, GB). Display only (see MODEL_CONTRACT §2). |
| `launch_vehicles_real` | **New.** `{lv.light|lv.medium|lv.heavy: {liftoff_mass_kg, propellant_kg}}`: the team's rocket figures, shown on the launcher card. |
| `destination` | `{name, notes, sun_distance_au}`. **`sun_distance_au` is new**: distance from the Sun in AU (Earth = 1, Mars ≈ 1.52, Jupiter/Europa ≈ 5.2), used by the inverse-square law for solar power. |
| `theme` | `{palette_key, environment_art_key, name_theme}` → `design_tokens.mission_themes` (`navigation`, `mars`, `europa`). |
| `budgets` | `budget_cr` (unit US$), `ops_time_budget_h`, `mass_margin_target`, `budget_margin_target`, `propellant_reserve_target`. |
| `requirements` | `required_categories`, `required_capabilities`, `instrument_count {min,max}`, `forbidden_components`. |
| `compatible_components` | Optional explicit list; each must list this mission in its `missions`. |
| `options` | Player choices such as landing site; each choice may set `flags`. |
| `launch_profiles` | `{id, name, capacity_factor, cost_cr, manoeuvres:[{phase, dv_ms, label}]}` (cost in US$). |
| `phases` | Ordered `{id, kind, label, actions?, simulate_h?, active_capabilities?, environment?, on_enter?, on_exit?}`. Must start `briefing, design, readiness` and end `debrief`. |
| `hazards` | `{key, label, description}` — scored for design preparedness. |
| `science_targets` | `{id, name, instrument_capabilities, hours_required, value, priority (core/secondary), hidden?}`. |
| `objectives` | `{id, description, priority, requirement: <condition>}`. |
| `events` | Curated event cards (below). |
| `event_slots` | `{phase, at_h, count}` — when events are drawn. |
| `scoring.weights` | Exactly `science, objectives, data, efficiency, resilience`. |
| `outcome_thresholds` | `full_min_data_fraction, partial_min_data_fraction, recovery_min_data_fraction` (the cutoffs on mission return; must be full ≥ partial ≥ recovery). `partial_min_core_objectives` is still accepted but no longer decides the outcome. |
| `references`, `simplifications` | Shown in science notes / credits. |


### Fields added in the learning increment (Nov 2026)

* Currency shown as **US$** everywhere (screens, reports, speech, data text); field names unchanged.
* `destination.sun_distance_au` (and an optional phase `environment.sun_distance_au`); environment
  `illumination` now means **local** sunlight (see below).
* Mission `card_text`; component `truth_overrides` and `learning`.
* Leftover "lunar" placeholders removed from the docs and `tools/dev_harness.py`.

### Fields added in increment 5

* Mission: `return_label` (name of the cutoff number), `ops_hour_steps` (three hour buttons),
  `ops_info` (operations hint), `real_data` (`[{label, value, ref, game}]`, refs must exist in
  REFERENCES.md), `funding_scenarios` (`[{id, label, budget_cr (US$), ops_time_budget_h, text}]`,
  one is drawn per campaign), `scoring.labels`, `scoring.science_note`.
* `budgets.ops_time_budget_h: 0` means no time limit.
* Science target: `needs_downlink: false` (counts as soon as it is done, e.g. streaming),
  `progress_label` (e.g. "Streamed {h:g} / {req:g} h").
* Event slot: `only_if_reached: true` (skipped if the phase ends before `at_h`).
* Effect op: `ops_time_h {delta}` (adds/removes operations hours; ignored with no time limit).

### Fields added in increment 3

* Component: `short_name` (button label in operations).
* Mission: `short_name`, `difficulty`, `theme.name_theme` (navigation | mars | europa),
  `science_notes` (facts with `[REF-...]` tags, shown in briefing and report).
* Phase: `tab` (short label for the phase indicator), `label`, `narrative`.
* Science target: `verb` (Observe, Drive, Broadcast…), `requires` (condition, e.g. drive first),
  `requires_text` (shown when not yet possible). Any fitted part with a matching capability
  can work on a target (wheels drive, clocks measure, instruments observe).
* Objective: `why`, `tip` (used in the review sheet).
* Effect `disable_component` may target a `capability`.
* Metric `count_cap.<capability>` = number of working parts with that capability (redundancy).

### Environment with variants

```json
"environment": {"illumination": 1.0, "link_quality": 0.9,
                "variants": [{"when": {"flag": "risky_site"}, "illumination": 0.5}]}
"environment": {"illumination": 1.0, "sun_distance_au": 1.195}      // a cruise phase
```

`illumination` is **local** sunlight (0–1): night, shadow, dust, season. Distance from the
Sun is handled separately by the inverse-square law from `destination.sun_distance_au`
(see `MODEL_CONTRACT.md` §5). So a Mars midday with clear sky is `1.0`, not `0.43`.

### Event card

```jsonc
{
  "id": "ev_dust", "title": "Dust on panels", "hazard": "dust",
  "phases": ["operations"],
  "description": "Dust settles on exposed surfaces.",
  "warning": "Power output is dropping.",          // fairness: always shown with the event
  "condition": {"capability": "solar_power"},      // eligibility (optional)
  "base_weight": 1,
  "weight_modifiers": [{"when": {"capability": "solar_power"}, "multiply": 3,
                        "reason": "solar panels are exposed to dust"}],
  "fault_category": null,                          // e.g. "communications" -> reliability raises weight
  "max_occurrences": 1,
  "choices": [                                     // 2-3 choices
    {"id": "clean", "label": "Pause and clean", "tradeoff": "Costs 2 h.",
     "quality": 0.9, "recovery": true, "requires": null,
     "effects": [{"op": "time_h", "hours": 2}],
     "explanation": "Cleaning restores output at the cost of time."}
  ]
}
```

The learning layer turns each resolved event into a "What happened" card using the card's
own `warning`, `tradeoff` and `explanation`, the other choices, and the **actual** before/after
metrics from the session.

Effect ops: `set_flag, clear_flag, battery_wh, propellant_mu (≤0 only), manoeuvre,
lose_data, modifier (solar|radioisotope|downlink|data_rate), disable_component,
time_h, ops_time_h, unlock_target, note`. Any effect may carry a `"when"` condition.

### Conditions

```
{"all": [...]} {"any": [...]} {"not": {...}}
{"capability": "x"} {"category": "battery"} {"component": "id"} {"flag": "landed"}
{"option": "landing_site", "equals": "psr_edge"}
{"metric": "battery_fraction", "op": "<", "value": 0.3}
```
Metrics: `battery_wh, battery_fraction, propellant_mu, propellant_fraction, stored_du,
storage_fraction, time_h, phase_time_h, ops_remaining_h, resilience.<hazard>,
science.<target>.coverage | returned_fraction | generated_du`.

Engine-set flags: `entered.<phase>`, `recovery_action`, `power_critical`,
`manoeuvre_shortfall`. Missions may set `spacecraft_lost` to signal loss.

## Learning content (`data/education/`)

All files are plain JSON, schema-checked by `engine/education.py` (`Education.load()`
raises `DataError` with a clear message). No expressions or code.

| File | Contents |
|---|---|
| `truth_labels.json` | BENNU's 5 labels in order: `key, label, short, shape, rgb, meaning, player_hint`; plus `legacy_map` (old keys and BENNU display names). |
| `parameters.json` | One entry per player-facing number: `label, unit, unit_name, truth, kind (constraint/capacity/rate/state/balancing/derived), concept, plain, higher, lower, example, related, aliases?, format?, bands?`. `bands` give a status + sentence (`lt` may be `"target"` = the mission's target margin). |
| `concepts.json` | 13 concepts in 5 topics: `title, one_liner, plain, why_it_matters, analogy, misconceptions[{wrong,right}], related_parameters, fact_cards, bennu`. |
| `category_cards.json` | Beginner card for each of the 11 categories: `what_it_is, why_needed, how_it_works, if_none, more_or_less, tradeoffs, limitation, misconception, key_params, concepts`. A component's own `learning` block can override any field. |
| `fact_cards.json` | Cited real-world facts: `title, truth (fact/physics), text, missions, refs, as_of, source_agency, live_date?`. The **only** place real names appear in teaching text. |
| `references.json` | `REF-…` → `{title, publisher, url, checked}`. Merge into `REFERENCES.md`. |
| `readiness.json` | Explanation per readiness rule: `codes, keywords, concept, rule, why, try[]`; `fallback`. |
| `event_lessons.json` | Lesson per hazard and fault category; plain words for effect ops; metric→parameter map. |
| `knowledge_checks.json` | Scenario questions: `id, mission, milestone (design_review/debrief), concept, truth, question, options[{id,text,why}], answer, explanation, fact_card?`. |
| `curriculum.json` | Per mission: `sun_distance_au, stage_title, objectives, stages {concept: introduce/reinforce/apply}, fact_cards, next_mission_teaser`; `bennu_transfer`, `bennu_disclaimer`. |
| `naming_and_credits.json` | `naming_rule`, `real_names`, `allowed_fields`, `inspired_by` per mission, `credits` (disclaimer, AI-assisted assets note, team art, logo note). |

## Save file

```json
{"save_version": 1, "mission_id": "...", "mission_hash": "...", "catalogue_hash": "...",
 "seed": 2026, "phase": "debrief", "config": {...},
 "decisions": [{"action": "select_component", "args": {"cid": "t.bus"}}, ...],
 "event_history": [...], "outcome": {...}, "score": {...}}
```

The learning tracker is **not** part of the mission or campaign save (so replays are unaffected).
It is written to `saves/learning.json` (`LearningTracker.to_dict()`, `tracker_version` 1).
