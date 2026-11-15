# OPERATION NIGHTINGALE — round-based mission-design mode of BENNU FRONTIER

The **default** way to play BENNU FRONTIER (solo and team). The classic mission is unchanged
and one click away on the Briefing screen: **How to play → Classic mission** (in team play the
Flight Director chooses). The campaign overview screen, the three-mission
campaign, settings and the other missions are not touched.

## The mission flow (NASA Big Picture Process)

| Stage tab | Lifecycle step | What players do |
|---|---|---|
| Need & goals | 1 Need & goals, 2 Concept, 3 Requirements | Predict Bennu's surface (Science); choose 2-3 goals, each a measurable requirement; choose a concept of operations (Flight Director) |
| Design & build | 4 Design, 5 Build & verify | Choose a fictional launcher (Flight Director); fit parts and extras (Systems); compare Lean / Balanced / Thorough; Verify checklist; GO / NO-GO lock |
| Launch & cruise | 6 Implement | Unchanged hands-on cruise (aim burns, DSN pings) |
| Operate R1-R6 | 6 Implement | Unchanged hands-on global laser map, then **six operations rounds**, then commit a site |
| Commit & TAG | 6 Implement | Unchanged hands-on touch-and-go, parameterised by the rounds |
| Return | 6 Implement | Unchanged: sample leak stow-or-measure, return burn, capsule release |
| User & reflect | User segment | Requirements PASS/FAIL with evidence, data delivered, 4 score axes, 3 key decisions, hidden rules; then the full legacy debrief |

## Goals → requirements → verification (GAME VALUES)

| Goal | Requirement | Verified at | Needs |
|---|---|---|---|
| G1 Return a sample | ≥ 60 g (NASA FACT: the real requirement) | Spin measurement; if stowed unmeasured, the capsule weigh-in (verified late) | TAGSAM + capsule |
| G2 Characterise the surface | laser coverage ≥ 60 % at the committed site | Site commitment | OLA |
| G3 Determine composition | spectral coverage ≥ 50 % at the committed site | Site commitment | Spectral Instrument Package |
| G4 Context | imaging ≥ 30 % on all three sites | End of round 6 | OCAMS |
| G5 Usable evidence | ≥ 18 GB downlinked | End of round 6 | High-gain antenna |
| G6 Return margin | ≥ 10 kg **above** the 30 kg protected reserve (so ≥ 40 kg in the tank) | Return burn | Propellant |

A design may be locked with an unverifiable goal only through the GO / NO-GO poll: the Science
line shows a warning and calling GO is the explicit acknowledgement.

## Operations rounds

Six rounds of six hours (GAME VALUE). Each station sets its levers, presses **Ready**, and one
resolution applies everything together in this published order:

1. Safe mode from the last round (no science, no downlink, $10M recovery)
2. Navigation: band change (1.5 kg per step), Pass (+1 kg), Trim (0.8 kg); drift grows unless trimmed
3. Power: generation (sun, or 35 % in shadow) − bus load − player loads; battery-mode discharge cap;
   loads shed in this order: Cool mode, antenna boost, Spectral, OLA, OCAMS, antenna
4. Science: powered instruments add coverage and data packets
5. Particle event, if scheduled this round
6. Storage, following the Operations policy when full
7. Downlink through valid DSN windows
8. Battery under 20 % or drift over 70 %: safe mode next round
9. Evidence: laser map at 50 % laser coverage; spectral richness tier at 50 % spectral coverage

| Station | Levers |
|---|---|
| Flight Director | DSN windows 0-2, send highest value or oldest first, compression, overwrite-oldest or drop-new when full, Particle Watch |
| Systems | power OCAMS / OLA / Spectral / antenna, antenna boost, battery mode (normal / conserve / boost), thermal (standard / cool) |
| Navigation | band Survey 5 km / Recon 1 km / Pass 0.25 km, Hold or Trim, Pass target |
| Science | target (all of Bennu or a site), up to two built instruments |

Solo: one player sets all four panels, timer off by default. Team (LAN): each player sets only
their own stations, a 180 s timer is on by default, and the host resolves when everyone is
ready or the timer ends (stations that did not plan keep last round's levers).

### Combos and anti-combos (GAME VALUES)
C1 Full Sync (data ×3) · C2 Laser Sweep (laser coverage ×2) · C3 Optical Nav Fix (exact drift
vector) · C4 Cool Science (cancels Rule D) · C5 Relay Burst (4.5 GB per window) · C6 Storm
Shelter (no particle penalty). Anti-combos: X1 Dark instrument, X2 Idle power, X3 Dead air,
X4 Blind pass. The combo preview says which supporting action is missing but never reveals a
hidden rule before the team has seen evidence of it.

### Hidden rules
| Rule | Evidence | Truth label |
|---|---|---|
| A Weak-gravity drift | drift chip; compare Hold and Trim rounds; C3 gives the vector | weak gravity = NASA FACT; rates = SIMULATED |
| B Ball-pit surface | images look smooth; laser coverage reveals boulders and loose material | boulder-rich, loose surface = NASA FACT; site maps = GAME VALUE |
| C Particle events | event log + rotation-phase dial (red dots) | particle ejections = NASA FACT; the 120° phase window = invented GAME VALUE puzzle, **not** a NASA finding |
| D Thermal camera noise | OCAMS marked noisy with the Spectral Package in Standard mode | entirely FICTIONAL, disclosed |

## How the existing architecture is reused (not duplicated)

| Need | Reused module |
|---|---|
| Parts, mass, power totals, required parts, one solar array | `engine/bennu/mission.py` build rules and lock poll |
| Battery step | `engine/resources.power_step` (MODEL_CONTRACT §5: no efficiency losses, surplus curtailed) |
| Storage and downlink | `engine/resources.DataStore` (MODEL_CONTRACT §6); round mode adds the overwrite-oldest policy |
| Propellant | `MissionState._spend` with the 30 kg protected reserve |
| Site evidence | `MissionState.a_observe` / `site_view` (rounds feed observations in) |
| TAG, leak, return, debrief | unchanged; a narrow adapter sets battery, attitude error from drift, and one extra boulder over a site that was never laser-mapped |
| Legacy scoring | `engine/bennu/scoring.py` 7 categories, still shown in the full debrief |
| Team play | `engine/bennu/frontier_net.py` command layer (one new command kind, `ng`) |

New code: `engine/bennu/nightingale/` (requirements, build, hidden_rules, combos, rounds,
round_scoring), `ui/frontier/ops.py`. Tuning: `data/bennu/bennu_nasa_extensions.json → rounds_mode`.
`bennu_simulation_v3.json` is unchanged (hash checked by a test). No component IDs, saved-game
formats or public interfaces changed (`FrontierGame(..., mode="nightingale")` is optional).

## NASA challenge resources and how they are used

| Resource | Use in the game |
|---|---|
| NASA SSSVI webinar *Design and Develop Science Missions* (J. Levri, 2025) | The six-step Big Picture Process is the stage tracker and the Briefing checklist; launch / space / ground / user segments structure the mode |
| NASA Mission Design Lab | Concurrent engineering: four station owners edit one shared design and see cross-subsystem effects (design method only; no MDL software or data) |
| NASA GMAT | Target–vary–achieve in cruise and in the band choice with a cost preview (concepts only; no GMAT computation) |
| JPL Small-Body Database | Reference panel in Help: published Bennu elements and physical data, with source and epoch; a local copy, not a live connection, and game motion is not propagated from it |
| Webb "Build it yourself" pattern | Constrained part cards, live mass / power / cost meters and a Verify checklist |

Before submission, compare these titles with the official 2026 challenge Resources tab and
refresh the SBDB values.

## Honest limitations
* All US$ costs, fictional launchers (Kestrel, Condor, Albatross), component costs, kg costs, round
  durations, thresholds, score weights, coverage gains and risk numbers are GAME VALUES.
  OSIRIS-REx flew on an Atlas V 411 (NASA FACT).
* Orbit bands and the two-body estimates in Help are educational abstractions: no trajectory
  propagation, no irregular gravity field, no solar radiation pressure.
* Particle phase windows, drift percentages, shadow schedules and site attributes are seeded game
  mechanics, not OSIRIS-REx telemetry. Sites A/B/C are fictional; Nightingale and Osprey were the
  real sites.
* The Spectral Instrument Package abstracts two real instruments, OVIRS and OTES.
* DSN windows, data volumes and downlink are abstracted, not a link budget. Power is counted for
  one hour of each six-hour round.
* Sample grams come from the existing game model, not a physical model of TAGSAM.
* The leaderboard lives in memory for the session only.
* Keyboard: Tab / Shift+Tab / Enter reach every button on every BENNU screen, and the mouse gestures
  have keys: 1/2/3 and the arrows set the surface prediction, Auto-target / P / Space fly the cruise,
  holding Space sweeps the laser, the arrows time the departure, and the descent is all keys.
* MISSION CONTROL / BENNU FRONTIER is an independent educational project, not affiliated with or
  endorsed by NASA.

## Tests and developer tools
`tests/test_bennu_nightingale.py` (build arithmetic and limits, power in sun/shadow with every
battery mode and the shed order, storage overflow and no double counting, 100-seed determinism,
schedules, discoverability, every combo and anti-combo, requirements at their verification
moments, the TAG adapter, the classic mode unchanged, lever ownership, the v3 hash) and
`tests/test_bennu_frontier_team.py` (levers over a real socket).

    python main.py --frontier-screens out/ng --only nightingale       # solo screenshot tour
    MC_TEAM_MODE=nightingale python main.py --team-screens out/t --team-role host   # + join in a 2nd window

## One name per station
The four stations are **Flight Director**, **Systems**, **Navigation** and **Science** everywhere:
on the screens, in the lobby, in this guide and in the README. (The Flight Director runs the
ground link and onboard data in the rounds; NASA's "mission operations" work belongs to that station.)

## Truth labels
NASA FACT · SCIENCE PRINCIPLE · SIMULATED · GAME VALUE · FICTIONAL — the same five labels on every
screen, in `bennu_nasa_extensions.json → provenance_labels.public_set`, in the glossary and in the README.

## Learning layer (BENNU ACADEMY)
See `LEARNING_LAYER.md`: a "?" on every lever opens its glossary entry, the top bar has a Library,
and the debrief adds **What you learned**.

## Future work
* Experiment mode (try settings on a copy of the mission without risk). Solo play and "Fly again"
  with the same seed already let players experiment.
* A searchable Mission Dictionary (the Library lists every term by phase today).
* A per-player learning profile across missions.
