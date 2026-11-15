# BENNU: Mission Design (what the players experience)

**Part of:** MISSION CONTROL (Ursina) · **Replaces:** the Europa Clipper team mode · **Status:** draft for team review
**Companion documents:** `02_BENNU_RULES_DATA_AND_SOURCES.md` (numbers, formulas, sources) · `03_BENNU_IN_MISSION_CONTROL_URSINA.md` (how it is built into the game)

---

## 1. The mission in one paragraph

Inspired by NASA's **OSIRIS-REx**, the team designs a spacecraft and flies it to the near-Earth asteroid **Bennu**. They study three candidate sampling sites, choose one from the evidence, and perform a hands-on **Touch-and-Go (TAG)**: a brief contact with the surface, a burst of nitrogen gas, then a quick back-away. They then fly home and release the sample capsule. The debrief explains what went well, the three decisions that mattered most, and how the team's run compares with the real mission.

Every stage builds on the one before: mass and power choices limit later operations, navigation sets the fuel left for TAG, and the site choice sets the sample hazard. Players never just click "Collect sample".

## 2. Goals for the player

| Goal | How the game checks it |
|---|---|
| Build a spacecraft that can do the whole mission | All required parts fitted, mass within the limit, power margin ≥ 15% |
| Arrive at Bennu accurately | Arrival inside the navigation corridor |
| Choose a safe, valuable sampling site from evidence | At least two sites observed; hazard and confidence compared |
| Collect a sample with a controlled TAG | Execution quality (alignment, approach speed, timing, readiness) |
| Bring the sample home | Sample stowed, capsule released, capsule recovered |
| Work as a team | Roles act, decisions approved, risks handled at decision points |

The game's sample target is **120 g**, a **game value**. The real mission returned **121.6 g** (NASA fact).

## 3. Players and roles

Bennu is played by **1–4 players** on one network (team mode) or by **one player alone** (all roles). There are four roles, modelled on how real mission teams split the work:

| Role | Short | Main job | Unique actions |
|---|---|---|---|
| Systems Engineer | SYS | Spacecraft build and health | Fit or remove parts, save or load trade designs, propose *Lock build*, answer the battery alert |
| Flight Dynamics & Navigation | NAV | Trajectory, burns, TAG pointing | Tracking passes, burn preview and auto-target, propose burns, arrive, departure timing, alignment, contact speed, return plan |
| Science & Exploration | SCI | Instruments and site choice | Observe sites with each instrument, propose the site, fire the nitrogen gas, accept the sample, answer the particle alert |
| Mission Operations | OPS | Readiness and sequencing | Propose *TAG GO*, descend to Checkpoint, press CONTACT, wave-off, depart for Earth, propose *capsule release*, answer the antenna alert |

**Roles by number of players** (a decision for the team; see document 03, D1):

| Players | Roles |
|---|---|
| 1 | All four roles |
| 2 | Systems + Science / Navigation + Operations |
| 3 | Systems + Operations / Navigation / Science |
| 4 | One role each |

Players vote for roles in the lobby (as in today's team mode), and ties are decided at random. The host can reassign roles if someone drops out. Every role has something to do in every stage, so nobody is a spectator.

**Joint decisions.** Five critical decisions are *proposed* by one player and *approved* by another: lock build, each course correction, site selection, TAG GO and capsule release. A player cannot approve their own proposal. If the proposer is the only player connected, the decision goes ahead at once. If a teammate is away, the host can approve for the team after 30 seconds, and the log shows it.

## 4. The seven stages

### Stage 1: Briefing (a "mission concept study")
- **Shown:**
  - **Objective:** a sample on Earth.
  - **Constraints:** 450 kg mass limit, ≥ 15% power margin, propellant in abstract **kg**, a 30 kg protected reserve.
  - **This scenario's conditions:** navigation uncertainty, initial knowledge of the sites, resource modifier, number of alerts.
  - **Bennu facts:** about 490 m wide, 7.329 × 10¹⁰ kg, a porous rubble pile, rotation 4.296 h.
  - **Real mission milestones.**
- **Framing:** start from the end goal, then requirements, design and operations, the way a NASA Mission Design Lab study works.
- **Action:** *Begin spacecraft build* (anyone).

### Stage 2: Spacecraft build (point design)
- **Parts:** 12 parts in 7 groups: power, propulsion, comms, navigation, science, science/navigation, sampling and return.
  - Required: battery, main thruster, propellant tank, attitude thrusters, high-gain antenna, star tracker, science camera, TAGSAM + capsule, and **one** solar array (small or large).
  - Optional: spectrometer and laser altimeter.
- **Master Equipment List:** live mass, power generated vs used, power margin, and propellant left after the required manoeuvres.
- **Requirement checklist:** each item says how it is checked ("inspection: parts list", "analysis: sum of masses").
- **Trade study:** save the design to *Baseline*, *Alt A* or *Alt B*, change it, and compare side by side.
- **Risk register:** appears once the build is complete. At **Key Decision Point 1** (lock build), every MEDIUM or HIGH risk must be *mitigated* or *accepted*.
- **The real trade-off is power:**
  - A small array plus both optional instruments gives a power deficit.
  - The large array fixes it but adds mass.
  - With a deficit at lock, the optional instruments are switched off, so the team loses those observations later.
- **Decision:** SYS proposes *Lock build*, a teammate approves.

### Stage 3: Deep-space navigation (flight dynamics)
- **Views:**
  - A schematic Sun–Earth–Bennu map with the Earth gravity assist (Sep 2017).
  - An **arrival target plane**: the aim point, a green corridor, your estimated position and an orange "not yet measured" ring.
- **Loop:** *track → plan → burn → track again*, as real navigators do.
  - **Tracking pass** (Deep Space Network): measures where you really are and lowers navigation uncertainty, which makes every later burn cheaper. Limited passes: 3 / 2 / 2 by difficulty.
  - **Plan a burn:** set direction (5° steps) and size (1.00 = cancel the known offset), then *Preview* (free). On Beginner and Explorer, *Auto-target* shows a few iterations converging on a good burn.
  - **Burn:** NAV proposes and a teammate approves. It costs kg. The true result includes an execution error you only see after the next tracking pass.
  - **Arrive:** orbit insertion is charged. Outside the corridor, an extra approach burn is charged. This is never a game over.
- **Decision:** spend tracking passes before a burn (accuracy and cheaper burns) or after it (know the real result)? Burning blind misses the corridor far more often.

### Stage 4: Bennu exploration (site characterisation)
- **Visual:** a rotating, top-shaped, boulder-covered Bennu with three **fictional** candidate sites (A, B, C) and a site-comparison table.
- **Site properties:**
  - Boulders, slope, loose regolith and science value (material factor) are shown as **estimates ± error** until measured.
  - Confidence (%) and estimated hazard **H** are shown for each site.
- **Instruments** (only the ones fitted and powered can be used):
  - **OCAMS** (camera) measures boulders.
  - **OLA** (laser altimeter) measures slope.
  - **OVIRS** (spectrometer) measures science value.
  - **OTES** (spectrometer) measures loose regolith.
  - **REXIS** is shown as information only.
- **Observation slots:** limited to 7 / 6 / 5 by difficulty. Each observation raises that site's confidence and reveals one property. **Observations lower uncertainty; they never remove boulders.**
- **Rule:** at least **two different sites** must be observed before one can be proposed.
- **Key Decision Point 2:** a site with MEDIUM or HIGH hazard needs the team to mitigate or accept the risk.
- **Decision:** SCI proposes the site, a teammate approves.
- **Real-mission card:** Nightingale (primary) and Osprey (backup); sites in the game are fictional.

### Stage 5: Touch-and-Go (the signature sequence)
Seven steps, each with its own controls and feedback:

| Step | Who | What happens |
|---|---|---|
| 1 Readiness | OPS (+ all) | Checklist: TAGSAM fitted, attitude thrusters, site chosen, battery after the 4.35 h descent with arrays tilted back, propellant for retreat + return, no unanswered alert. Blockers vs warnings. **Key Decision Point 3**: handle risks, then OPS proposes **TAG GO** |
| 2 Alignment & timing | NAV | Pick the **orbit-departure time** so Bennu's rotation brings the site under the arrival point (a top-down diagram shows where the site will be). Trim the attitude error with ±1° / ±5° adjustments (limited) |
| 3 Descent | OPS + NAV | Descend to **Checkpoint (~125 m)**, set the contact speed for the **Matchpoint (~50 m)** burn (real reference ~0.10 m/s), fire Matchpoint, or **wave-off** safely at Checkpoint |
| 4 Contact window | OPS | A 6-second descent sweep: press **CONTACT** (Space) while the marker is inside the green window. With reduced motion: pick the moment on a slider |
| 5 Collection | SCI/OPS | Fire the nitrogen gas: the sample estimate is calculated |
| 6 Retreat | NAV/OPS | Back-away burn. Without enough propellant the retreat is incomplete: spacecraft damage, but the collected sample is kept |
| 7 Assessment | SCI/OPS | Sample mass, quality score Q and its four parts, site hazard H. *Accept* the sample, or *Try TAG again* (one more attempt if propellant allows) |

**Different outcomes, explained:**
- **Success.**
- **Low yield.**
- **Missed contact:** recoverable with a second attempt.
- **Unsafe contact:** too fast on a hazardous site, which damages the capsule.
- **Incomplete retreat.**
- **Recoverable warning:** sample collected with warnings.

### Stage 6: Sample return (end-of-mission planning)
1. **Stow the sample** in the capsule (required before leaving if there is a sample).
2. **Choose a return plan:**
   - *Standard*: cheaper, 80% landing accuracy.
   - *Standard + final correction*: +6 kg, 100% accuracy.
3. **Fire the departure burn.** If there isn't enough propellant, the spacecraft is stranded. The mission still goes to the debrief and explains why.
4. **OPS proposes capsule release**, a teammate approves. The sequence: entry behind the heat shield, parachutes, landing in the Utah desert, recovery. Capsule integrity drops after an unsafe contact or incomplete retreat.

### Stage 7: Debrief
- **Total score** and seven category scores (weights 15/15/20/15/10/15/10): Engineering, Navigation, Science, Operations, Resources, Sample Return, Teamwork.
- **Explanation:**
  - The **three decisions that mattered most**, each with the points it cost and advice for next time.
  - What happened, category by category.
- **Results:**
  - Achievements and problems.
  - The capsule return sequence.
  - **Revealed truth:** the real hidden values of the fictional sites and the true arrival offset.
  - Comparison with the real OSIRIS-REx (121.6 g, sampled at Nightingale on 20 Oct 2020).
- **Replay:** the seed and versions for this run. The host can replay with a **new seed** or the **same seed**, and earlier runs are listed for comparison.
- **A failed mission still gets a full, useful debrief.**

## 5. Operational alerts (bounded, fair surprises)

Each run has 0–2 alerts (Beginner at most 1). Each one warns first, belongs to one role, offers two clear responses, and blocks only that stage's critical action until it is answered.

| Alert | Stage | Owner | Choices |
|---|---|---|---|
| Deep Space Network antenna maintenance | Navigation | OPS | Backup antenna (tracking slightly less precise) / Wait (lose one tracking pass) |
| Particles leaving Bennu's surface (real phenomenon) | Exploration | SCI | Spend an observation slot to map it (site more certain) / Note it (site less certain) |
| Star tracker reset before descent | TAG | NAV | Recalibrate (lose one alignment adjustment) / Proceed (alignment starts 4° worse) |
| Battery cell temperature warning | Return | SYS | Isolate the cell (20% less battery) / Keep monitoring (small capsule risk) |

## 6. Difficulty (changes the challenge, never the science)

| | Beginner | Explorer | Mission Expert |
|---|---|---|---|
| Propellant | +15% | baseline | −10% |
| Initial site knowledge | higher | baseline | lower |
| Arrival corridor | 8 units | 5 | 4 |
| Tracking passes / observation slots | 3 / 7 | 2 / 6 | 2 / 5 |
| TAG windows (alignment, timing, contact, approach) | ×1.2 | ×1.0 | ×0.85 |
| Alerts per run | 0–1 | 0–2 | 0–2 |
| Hints | step-by-step (incl. recommended departure time) | contextual | minimal, no auto-target |

## 7. Help, accessibility and honesty

- **Next step card:** always says the single next action and which role does it.
- **Cosmo:** gives Bennu tips and sourced fun facts in solo play. In team mode the team chat replaces Cosmo, as today.
- **Labels:** every number has a label, in text and shape, never colour alone:

  | Label | Meaning |
  |---|---|
  | ✔ NASA FACT | Real mission fact or measurement |
  | ∿ SCIENCE PRINCIPLE | Derived from real data |
  | ◇ GAME VALUE | Game design or fiction |
  | ≈ SIMULATED | Game calculation |

- **Controls and reading:**
  - Keyboard: Space = CONTACT.
  - Reduced motion turns off the rotation and sweep animations.
  - Read-aloud and settings work as in the rest of the game.
- **About screen:** NASA acknowledgements, a disclaimer (the game does not use or reproduce NASA software) and the full source list.

## 8. Replayability

- **Each run is different:** every run is seeded. Navigation uncertainty, site properties and positions, arrival offset, alerts, alignment error and the contact moment all change.
- **Each run is fair:** every scenario is checked for a winnable path before play, and replaying the same seed with the same moves gives the same result.
- **Learning:** the debrief turns each run into advice for the next.
