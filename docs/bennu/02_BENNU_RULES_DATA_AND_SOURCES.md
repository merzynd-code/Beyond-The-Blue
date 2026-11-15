# BENNU: Rules, Data and Sources

**Purpose:** every number, formula and fact the Bennu mission uses, where it comes from, and how it is labelled.
**Rule zero:** `bennu_simulation_v3.json` is the canonical configuration and is **never edited**. Everything the game adds lives in a second file, `bennu_nasa_extensions.json`.

---

## 1. Four kinds of information (master document section 9)

| Label (text + shape) | Meaning | Examples |
|---|---|---|
| ✔ NASA FACT | Historical event or NASA-reported measurement, with a source | Launch 8 Sep 2016; TAG 20 Oct 2020; 121.6 g returned |
| ∿ SCIENCE PRINCIPLE | Derived from sourced data; physically supported but simplified | Surface gravity from GM/r²; phasing with the real rotation |
| ◇ GAME VALUE | Game-designed value or fictional scenario element | 450 kg limit, part masses, kg, sites A/B/C, alerts |
| ≈ SIMULATED | Output of the game's simplified rules | Arrival offset, hazard estimate, Q, sample mass, scores |

## 2. Bennu reference values (fixed, never randomised)

| Value | Used in game | Label | Source |
|---|---|---|---|
| Mean diameter | 492 m | NASA FACT | NASA Bennu facts; Lauretta et al. 2019 |
| Shape (equatorial × equatorial × polar) | 565 × 536 × 498 m (visual only) | NASA FACT | Lauretta et al. 2019 |
| Mass | 7.329 × 10¹⁰ kg | NASA FACT | Lauretta et al. 2019 |
| Bulk density | 1,190 kg/m³ (porous rubble pile) | NASA FACT | Lauretta et al. 2019 |
| Rotation period | 4.296 h | NASA FACT | Lauretta et al. 2019 |
| GM | 4.892 m³/s² | NASA FACT | Lauretta et al. 2019 |
| Surface gravity (spherical approx.) | ≈ 8.1 × 10⁻⁵ m/s² | SCIENCE PRINCIPLE | g = GM/r², r = 246 m |
| Equatorial effective acceleration | ≈ 4.1 × 10⁻⁵ m/s² | SCIENCE PRINCIPLE | gravity minus ω²r |
| Boulders up to ~22 m | visual hazard baseline | NASA FACT | NASA Bennu in depth |

## 3. Spacecraft parts (GAME VALUES from v3, not real OSIRIS-REx hardware)

| Part | Group | Mass kg | Power W | Effect |
|---|---|---|---|---|
| Small solar array | Power | 35 | 0 | +120 W in sunlight |
| Large solar array | Power | 70 | 0 | +240 W in sunlight |
| Battery pack | Power | 28 | 8 | Stores 500 Wh |
| Main thruster | Propulsion | 32 | 5 | Propulsion rating +20 |
| Propellant tank | Propulsion | 45 | 0 | Holds 100 kg |
| Attitude-control thrusters | Propulsion | 18 | 12 | Fine pointing for TAG |
| High-gain antenna | Comms | 16 | 18 | Comms +2; enables tracking passes |
| Star tracker | Navigation | 8 | 10 | Navigation accuracy +2 |
| Science camera (OCAMS) | Science | 12 | 15 | Hazard imaging +2 |
| Spectrometer (OVIRS/OTES) | Science | 18 | 35 | Composition +3 (optional) |
| Laser altimeter (OLA) | Science/navigation | 14 | 25 | Topography +3 (optional) |
| TAGSAM + sample return capsule | Sampling | 55 | 20 | Collects and contains the sample |

**Build rules:**
- All required parts plus exactly one solar array.
- Mass ≤ 450 kg; over the limit blocks the lock.
- Power margin ≥ 15% for a clean build. Below 0%, the spectrometer and then the laser altimeter are switched off at lock.
- Nominal margin assumes full sunlight and no battery discharge.

## 4. Normative formulas (v3, used exactly as written)

```
mass_ratio        = total_mass_kg / 450
mass_penalty      = 0 if ratio ≤ 1, else min(30, 100 × (ratio − 1) × 2)
power_margin_pct  = 100 × (available_W − required_W) / max(required_W, 1)
available_W       = solar_W × illumination + battery_discharge_W
battery_next_Wh   = clamp(battery + (solar_W − required_W) × hours, 0, capacity)
manoeuvre_cost_GU = base × difficulty_multiplier × (1 + navigation_uncertainty_pct / 100)
optional_budget   = propellant − 30 (protected reserve)
site_hazard H     = clamp(0.40 boulders + 0.25 slope + 0.20 surface_uncertainty + 0.15 loose_regolith, 0, 1)
quality Q         = 0.30 alignment + 0.25 approach + 0.25 timing + 0.20 TAGSAM_readiness
sample_g          = 120 × (0.25 + 0.75 Q) × (1 − 0.60 H) × site_material_factor   (0 if no contact, max 150)
```

Manoeuvre base costs (kg): course correction 8, orbit insertion 18, approach adjustment 6, TAG retreat 10, return departure 20.

## 5. Rules the game adds (GAME VALUES in `bennu_nasa_extensions.json`)

The master document requires these behaviours but gives no numbers. Each is labelled and can be tuned.

| Area | Rule | Informed by |
|---|---|---|
| Starting propellant | 100 kg × resource modifier (0.9–1.1) × difficulty modifier (1.15 / 1.0 / 0.9) | - |
| Arrival model | 2D offset 20–60 units in a seeded direction. A burn of size 1.0 cancels the *known* offset. Execution error = 2.0 × burn size × uncertainty%. Max 2 corrections | GMAT Target/Vary/Achieve (concept only) |
| Corridor | 8 / 5 / 4 units. Outside the corridor, an approach adjustment (6 kg base) is charged | - |
| Tracking pass | Uncertainty × 0.7 (floor 1%) and reveals the true offset. Passes 3 / 2 / 2; needs the high-gain antenna | GMAT navigation; NASA Basics of Space Flight ch. 13 |
| Auto-target | Up to 5 deterministic iterations on burn size along the best direction | GMAT DifferentialCorrector (concept) |
| Observations | 7 / 6 / 5 slots. Each cuts that site's uncertainty by 0.12–0.18 (floor 0.05) and reveals one property. First estimates are off by a seeded amount that shrinks with initial knowledge | - |
| TAG readiness | Battery after 4.35 h descent at 0.5 illumination (arrays tilted back) must stay > 0. Propellant for retreat + return checked. Readiness score 1.0, or 0.6 with warnings | Real "Y-wing" array position (TAG poster) |
| Departure phasing | phase error = wrap(site longitude + 360/4.296 × (departure + 4.35)); score = 1 − \|error\| / (20° × window) | GMAT used to re-target OSIRIS-REx TAG incl. phasing |
| Alignment | Initial error 6–18°, ±1/±5° trims, 12 adjustments; score = 1 − \|error\| / (10° × window) | - |
| Approach | Score = 1 − \|v − 0.10\| / (0.05 × window); input 0.02–0.30 m/s | Real ~10 cm/s contact (Berry et al.) |
| Contact window | 6 s sweep; window centre seeded 2.2–3.8 s; ±0.7 s × window; contact made within 2× half-window | - |
| Timing sub-score | 0.5 × phasing + 0.5 × contact window | - |
| Unsafe contact | Approach score < 0.25 and H > 0.55, giving capsule integrity −0.3 | - |
| Second TAG | Allowed if propellant above the reserve covers approach adjustment + retreat | Real second attempt was planned for Jan 2021 |
| Return plans | Standard (accuracy 0.8) or + final correction (+6 kg, 1.0). Capsule recovered if integrity ≥ 0.5 | - |
| Risk register | POWER (< 15% medium, < 0 high), MASS, PROP (margin after required burns < 10 medium, < 0 high), NAV (uncertainty > 10% medium, > 14% high), SITE (estimated H > 0.4 medium, > 0.6 high), KNOWLEDGE (completeness < 0.7 medium), PHASE (score < 0.5 medium) | MDL technical risk assessment; OSIRIS-REx decision points |
| Approvals | One other connected player approves; host may override after 30 s | Master doc 6.3 |

## 6. Randomness and fairness (master doc V3)

- **Seeding and storage:**
  - One seed per game, generated by the host (or a fixed seed for testing).
  - The scenario snapshot stores the seed, generator version and config versions, and the debrief shows them.
- **What varies:**
  - From v3: navigation uncertainty (2–15%), initial observation completeness (0.55–1.0), resource modifier (0.9–1.1), and three sites (boulders 0.15–0.85, slope 0.05–0.75, surface uncertainty 0.10–0.80, loose regolith 0.10–0.80, material 0.75–1.10). Alerts: 0–2, never more than the difficulty allows.
  - Added by the game: arrival offset and direction, error direction per burn, site longitudes, estimate bias, initial alignment error and contact-window centre. These come from a **separate seeded stream**, so the v3 values for a seed never change.
- **Solvability check before play:**
  - The reference build passes mass and power.
  - The reference navigation plan (track, burn, track, burn) plus insertion, retreat and return fits the propellant.
  - At least one site has H ≤ 0.6 and gives at least 60 g at good execution.
  - A departure slot scores at least 0.5.
  - Failing scenarios are regenerated in a fixed order (up to 20 attempts) and logged.
- **Determinism:** same seed + same actions = same results.

## 7. Scoring (debrief)

| Category | Weight | Scored from |
|---|---|---|
| Engineering | 15 | Mass penalty, power margin, trade study used |
| Navigation | 15 | Inside corridor and how close; tracking used |
| Science | 20 | Sites and instrument types observed, true hazard of the chosen site vs the best, sample vs 120 g |
| Operations | 15 | Q, readiness clean, alerts answered, contact made |
| Resources | 10 | Propellant left vs reserve, no approach penalty, no damage or stranding |
| Sample Return | 15 | Sample stowed, capsule integrity, recovery × landing accuracy |
| Teamwork | 10 | Roles that acted, approvals and decisions, risks handled |

The **top three decisions** are the categories that lost the most weighted points, each with one piece of advice.

## 8. NASA resources: which one informed what

**Primary: NASA Mission Design Lab (MDL)**

| Feature | MDL idea |
|---|---|
| Briefing as a concept study | Concept of Operations, resources summary |
| Master Equipment List with live totals | MDL point-design products |
| Trade study (Baseline / Alt A / Alt B) | Baseline vs alternative designs, trade tree |
| Requirement checklist with "how we check it" | Requirements with a verification plan (also the NASA science-mission webinar) |
| Risk register and Key Decision Points | Technical risk assessment (plus the real OSIRIS-REx Operational Key Decision Points) |
| Debrief "study report" | Study products reviewed at the end |

**Secondary: NASA General Mission Analysis Tool (GMAT)**

| Feature | GMAT idea |
|---|---|
| Burn preview and auto-target iterations | Target / Vary / Achieve; DifferentialCorrector with a maximum number of iterations |
| Tracking passes | Navigation / orbit determination (plus NASA Basics of Space Flight) |
| Instant burns, propellant in kg | Impulsive burns |
| Departure phasing with Bennu's rotation | GMAT was used to re-target the OSIRIS-REx TAG sequence, including phasing with the asteroid's rotation |
| Checkpoint → Matchpoint descent | Propagate to a stopping condition, then burn |

**Supporting:**
- **NASA webinar "Designing and Developing Science Missions":** mission segments and the six-step process.
- **Webb "Build It Yourself" game:** greyed-out options with reasons, and comparison with a real mission.
- **Mission Viz:** studied only. It has no licence file, so nothing is reused.

**What is not claimed:** the game does not use, contain or reproduce GMAT, MDL tools or any NASA software. No GMAT outputs are used. The numbers are simplified game rules, not NASA calculations. No NASA logos are used.

## 9. Real-mission facts shown in the game (all NASA FACT unless marked)

| Stage | Fact | Source |
|---|---|---|
| Briefing | OSIRIS-REx launched 8 Sep 2016 on an Atlas V 411 | NASA mission page; NTRS 20170002015 |
| Briefing | Ground system: Mission Support Area, Flight Dynamics System, Science Processing and Operations Center, plus the Deep Space Network | NTRS 20170002015 |
| Briefing | GMAT was used for OSIRIS-REx launch-window analysis and to check the TAG sequence (the game does not use GMAT) | GMAT NASA usage; NASA Spinoff |
| Navigation | Earth gravity assist 22 Sep 2017 tilted the orbit to match Bennu's | NASA mission page |
| Navigation | Navigation repeats track → plan → burn → track (SCIENCE PRINCIPLE) | Basics of Space Flight ch. 13 |
| Navigation | Arrived 3 Dec 2018; orbit insertion 31 Dec 2018, the smallest object ever orbited by a spacecraft | NASA mission page |
| Exploration | Near Bennu, navigation moved from star-based to landmark-based optical navigation | NTRS 20170002015 |
| Exploration | Nightingale primary site, Osprey backup | NASA mission page |
| Exploration | Bennu ejects particles from its surface | NASA Bennu in depth |
| Exploration | Five instrument families: OCAMS, OLA, OTES, OVIRS, REXIS | NASA OSIRIS-REx in depth |
| TAG | TAG on 20 Oct 2020 at Nightingale: contact, nitrogen-gas burst, back-away | NASA release 20-103 |
| TAG | Checkpoint (~125 m) and Matchpoint (~50 m) burns, ~10 cm/s contact | Berry et al. (NTRS 20200000774); TAG poster |
| TAG | Natural Feature Tracking compared images with landmarks and adjusted both burns | TAG poster; Berry et al. |
| TAG | Orbit departure to contact took about 4 h 21 min, roughly one Bennu rotation | TAG poster; Lauretta 2019 |
| TAG | Big operational choices were made at Operational Key Decision Points | Burns & Lauretta, SpaceOps 2025 |
| TAG | A second TAG attempt was planned for Jan 2021 if needed | NASA/UA mission news, 20 Oct 2020 |
| Return | Departure 10 May 2021 with a ~7-minute full-throttle main-engine burn | NASA mission page |
| Return | Capsule landed at the Utah Test and Training Range on 24 Sep 2023 | NASA mission page; NASA blog |
| Debrief | 121.6 g returned, more than twice the mission's sample requirement | NASA blog, 15 Feb 2024 |

## 10. Known issues in v3 (reported, not changed)

1. **Reserve:** the protected reserve (30 kg) equals the *base* cost of retreat + return. With navigation uncertainty those cost up to 34.5 kg. The game applies the reserve to optional burns only, and the solvability check guarantees the full path is affordable. Suggested fix: reserve 35.
2. **Mass:** one of every part totals 351 kg, so the 450 kg limit never binds; the live trade-off is power. Suggested fix: allow two tanks or arrays, or lower the limit on Mission Expert.

## 11. Acknowledgement and disclaimer (exact in-game text)

> BENNU is an educational mission inspired by NASA's OSIRIS-REx asteroid sample-return mission. Its mission-design gameplay (equipment list, trade studies, risk register, design reviews) was informed by public information about NASA Goddard's Mission Design Lab. Its course-correction, tracking and TAG-timing mechanics were informed by concepts documented for NASA's open-source General Mission Analysis Tool (GMAT). Mission facts come from NASA and OSIRIS-REx team publications. Made for the NASA Space Apps Challenge 2026.
>
> *This game does not use, contain or reproduce NASA software, and its numbers are simplified game rules, not NASA calculations or operational analyses. It is not affiliated with or endorsed by NASA. Values marked GAME VALUE or SIMULATED are not NASA data.*

## 12. Sources

1. NASA GSFC: Mission Design Lab. https://etd.gsfc.nasa.gov/capabilities/integrated-design-center/mission-design-lab/
2. NASA Software Catalog: GMAT R2025a, GSC-19468-1. https://software.nasa.gov/software/GSC-19468-1
3. GMAT on SourceForge (Apache License 2.0). https://sourceforge.net/projects/gmat
4. GMAT User Guide: DifferentialCorrector / Target / Vary / Achieve (R2018a mirror). https://documentation.help/GMAT/DifferentialCorrector.html
5. GMAT Wiki: NASA Usage. https://gmat.atlassian.net/wiki/spaces/GW/pages/380273244/NASA+Usage
6. NASA Spinoff: GMAT. https://spinoff.nasa.gov/node/10892
7. NASA SE webinar: Designing and Developing Science Missions (26 Mar 2025). https://www.nasa.gov/wp-content/uploads/2025/04/250323-design-and-develop-science-mission.pdf
8. NASA Science: Webb Build It Yourself. https://science.nasa.gov/mission/webb/game-build-it-yourself/
9. NASA Mission Viz. https://nasa.github.io/mission-viz/
10. NASA Science: OSIRIS-REx mission and milestones. https://science.nasa.gov/mission/osiris-rex/
11. NASA OSIRIS-REx in depth. https://science.nasa.gov/mission/osiris-rex/in-depth/
12. NASA Release 20-103 (20 Oct 2020). https://nssdc.gsfc.nasa.gov/planetary/text/osiris_rex_pr_20201020.txt
13. NASA/UA mission news, 20 Oct 2020. https://www.asteroidmission.org/?p=26436
14. NASA blog: bulk sample mass 121.6 g (15 Feb 2024). https://science.nasa.gov/blogs/osiris-rex/2024/02/15/nasa-announces-osiris-rex-bulk-sample-mass/
15. OSIRIS-REx TAG Technical Poster. https://www.asteroidmission.org/wp-content/uploads/2020/10/TAG-Technical-Poster.pdf
16. Berry et al., Revisiting OSIRIS-REx TAG performance (NTRS 20200000774). https://ntrs.nasa.gov/api/citations/20200000774/downloads/20200000774.pdf
17. Burns & Lauretta, OSIRIS-REx Operational Key Decision Points (SpaceOps 2025, NTRS 20250002700). https://ntrs.nasa.gov/api/citations/20250002700/downloads/Burns_SpaceOps2025_paper_2strives.pdf
18. OSIRIS-REx mission design and ground system overview (NTRS 20170002015). https://ntrs.nasa.gov/api/citations/20170002015/downloads/20170002015.pdf
19. NASA/JPL: Basics of Space Flight ch. 13. https://science.nasa.gov/learn/basics-of-space-flight/chapter13-1/
20. NASA Science: Bennu facts. https://science.nasa.gov/solar-system/asteroids/101955-bennu/facts/
21. NASA: Bennu in depth. https://solarsystem.nasa.gov/asteroids-comets-and-meteors/asteroids/101955-bennu/in-depth.amp
22. Lauretta et al. (2019), The unexpected surface of asteroid (101955) Bennu. https://heliophysics.gsfc.nasa.gov/sed/content/uploadFiles/publication_files/Lauretta2019.pdf
23. NASA PDS: Bennu shape model. https://pds.nasa.gov/ds-view/pds/viewProfile.jsp?dsid=EAR-A-I0037-5-BENNUSHAPE-V1.0
24. NASA Space Apps 2026 Participant Terms (all resources must be cited). https://www.spaceappschallenge.org/legal/
