# BENNU FRONTIER: the hands-on, single-screen Bennu mission

**Start it:** campaign screen → **BENNU FRONTIER** card → pick Beginner / Explorer / Expert → **Launch BENNU ›**
(or `python main.py --frontier`). **Esc** returns to the campaign at any time.

**Operation Nightingale (round mode):** on the Briefing screen choose *How to play → Operation
Nightingale* for mission goals and requirements, a launcher and $250M budget, and six team
operations rounds at Bennu before the touch-and-go. See `OPERATION_NIGHTINGALE.md`.

**Team play (LAN):** the same mission on up to four laptops at Beginner level; see README
"Team mode". Every change is a small command (`engine/bennu/frontier_net.py`) that the host puts
in one order for everyone; the host flies the real-time descent and streams snapshots.

On one laptop: one player can run all four stations, or a family can play side by side: each station has its own keys, and the glowing tile in the station bar shows whose
move it is.

| Station | Colour | Hands-on job |
|---|---|---|
| Flight Director (FD) | gold | runs the GO / NO-GO polls, flies descent speed (W / S), wave-off (X), releases the capsule |
| Systems (SYS) | orange | builds the spacecraft, keeps the battery alive during descent (1 / 2 / 3 switch loads), stows the sample |
| Navigation (NAV) | blue | aims burns, pings the Deep Space Network (P), times the orbit departure, holds attitude (A / D) |
| Science (SCI) | green | predicts the surface, fires the laser altimeter, points instruments at sites, calls hazards (H), fires the nitrogen (Space) |

## The mission, scene by scene

| # | Scene | What your hands do | Real mission link |
|---|---|---|---|
| 1 | Briefing | Click one of three 3D surface samples you expect; drag "how sure are you" | Telescopes suggested a sandy, beach-like surface |
| 2 | Build | Click parts on the shelf; they fly onto the turning spacecraft; watch mass, power and propellant gauges; lock the design in a GO / NO-GO poll | Mission Design Lab equipment list and design review |
| 3 | Cruise | Drag from your position on the arrival target to aim a burn; **P** pings the Deep Space Network; **Space** fires | GMAT-style target / vary / achieve; Earth gravity assist 22 Sep 2017 |
| 4 | Laser scan | Hold the mouse (or Space) over spinning Bennu to fire the laser altimeter; **W / S** fly closer (low-energy laser under 1 km). At ~22% coverage the smooth "expected" Bennu turns into the real boulder field | OLA (Canadian Space Agency): two lasers, ~7 cm global maps, ~3 cm at sites |
| 5 | Site choice | Click a glowing site; point OCAMS / OLA / OVIRS / OTES at it; choose it in a poll | Nightingale was a safe patch only ~16 m wide |
| 6 | Touch-and-Go | Drag Bennu's rotation to time the orbit departure; then fly the last kilometre in real time: Checkpoint (~125 m), Matchpoint (~50 m), contact, nitrogen burst, back-away | ~4 h 21 min descent, ~10 cm/s contact, ~6 s contact, head sank ~48 cm |
| 7 | Sample leak | Rocks drift out of the sampler head; a 20 s countdown: **1** stow now or **2** measure first | Oct 2020: NASA cancelled the mass measurement and stowed early |
| 8 | Return & debrief | Click a return path, **Space** for the departure burn, release the capsule in a poll; watch the fiery entry and parachutes; open the capsule: real Bennu discoveries unlock by grams returned | Capsule landed in Utah, 24 Sep 2023; 121.6 g returned |

## What is real and what is a game rule

Every number on screen carries a label: **NASA FACT**, **SCIENCE PRINCIPLE**, **GAME VALUE** or **SIMULATED**.
The rules, formulas and scoring are the unchanged BENNU engine (`engine/bennu/mission.py`, `formulas.py`,
`scoring.py`, configs in `data/bennu/`). The hands-on layer is `engine/bennu/frontier.py`; its added
numbers are listed in `GAME_VALUES` at the top of that file:

| Game value | Value | Why |
|---|---|---|
| Battery at orbit departure | 40-60% (seeded) | makes the Systems power job real during descent |
| Descent time compression | coast 18 s; x25 below Checkpoint | the real 4 h 21 min fits in about a minute |
| Attitude drift | 0.25-0.6 deg/s (seeded) | Navigation has something to hold |
| Hazard warnings | 1-2 per descent, 3 s to call | Science's real-time job (Natural Feature Tracking) |
| Leak: measure first | lose 10-25% (seeded); timeout loses 30% | the real stow-or-measure trade |
| Discovery thresholds | 1 / 40 / 70 / 95 / 115 g | turn grams into rewards |

## Sources added for BENNU FRONTIER

1. NASA SVS 13565, *Selecting Site Nightingale*: https://svs.gsfc.nasa.gov/13565
2. NASA, *Bennu's surface is like a plastic ball pit* (2022): https://www.nasa.gov/missions/surprise-again-asteroid-bennu-reveals-its-surface-is-like-a-plastic-ball-pit
3. NASA SVS 14179, *Bennu's surprising surface*: https://svs.gsfc.nasa.gov/old/14179
4. NASA release, 23 Oct 2020 (sample head overflowing, early stow): https://www.nasa.gov/press-release/nasa-s-osiris-rex-spacecraft-collects-significant-amount-of-asteroid
5. University of Arizona / NASA, 29 Jan 2025 (amino acids, nucleobases, evaporites): https://lpl.arizona.edu/news/spotlight/asteroid-bennu-comes-long-lost-salty-world-ingredients-life
6. McCoy et al. 2025, *Nature* 637: https://doi.org/10.1038/s41586-024-08495-6
7. Glavin et al. 2025, *Nature Astronomy* (ammonia, organics): https://doi.org/10.1038/s41550-024-02472-9
8. Canadian Space Agency, OSIRIS-REx game and OLA: https://www.asc-csa.gc.ca/eng/satellites/osiris-rex/activities/game.asp
9. Daly et al. 2017, *The OSIRIS-REx Laser Altimeter*: https://ui.adsabs.harvard.edu/abs/2017SSRv..212..899D

All other sources: `02_BENNU_RULES_DATA_AND_SOURCES.md`.

## For developers

```
engine/bennu/frontier.py        FrontierGame, ScanModel, DescentModel, polls, emergency, discoveries (no Ursina)
engine/bennu/frontier_pilot.py  a scripted good player (tests + screenshot tour)
ui/frontier/                    the eight scenes, procedural 3D models, vertex-colour shader
tests/test_bennu_frontier.py    19 tests: determinism, wave-off, safe mode, polls, emergency, full runs
tools/frontier_tour.py          python main.py --frontier-screens out/frontier   (saves 27 screenshots)
```

Everything 3D is generated in code (no downloaded models): the asteroid mesh, boulders, terrain and spacecraft.
