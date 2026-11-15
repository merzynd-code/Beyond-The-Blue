# BENNU in MISSION CONTROL (Ursina): Integration Plan

**Goal:** bring the Bennu mission into the existing Ursina MISSION CONTROL game as the **team-mode mission**, replacing Europa Clipper in team mode, without breaking anything that works today.
**This is a plan only. No code until the team approves it.**

---

## 1. What changes and what stays the same

| Stays exactly as it is | Changes |
|---|---|
| Solo campaign: GPS, Perseverance, Europa Clipper (missions, scores, report, unlock order) | The team-mode box on the hub shows **Bennu** instead of Europa Clipper |
| Cosmo, hints, read-aloud, settings window, reduced motion, sound | Team mode gets Bennu's roles (see D1) and Bennu's screens |
| Manual landing and drive for Perseverance | A Bennu TAG screen reuses the same manual-sim style |
| Engine, data files and all 218 existing tests | New Bennu engine package + its own tests |
| LAN team networking (TCP, lobby, vote, chat, version check) | Team protocol gains a "Bennu" mission type |

**Rule:** Bennu runs on its **own rules and its own config** (`bennu_simulation_v3.json`, unchanged, plus `bennu_nasa_extensions.json`). It does **not** use Mission Control's 42-part catalogue, credits or game units. This keeps the approved Bennu values exact. It also means Bennu cannot break GPS, Mars or Europa.

## 2. Decisions to confirm before coding

| # | Question | Options | Recommendation |
|---|---|---|---|
| D1 | Roles in Bennu team mode | **A.** 4 Bennu roles, 1–4 players (master doc) · **B.** keep today's 3 roles (designer → Systems, science head → Science, head engineer → Navigation + Operations) | **A**: matches the Bennu design and gives each role a full job. Raises the team limit from 3 to 4 for Bennu only |
| D2 | Can Bennu be played solo? | **A.** team mode only · **B.** also a 4th card in the solo campaign | **B**: one player holds all four roles. Lets you demo Bennu with one laptop. The campaign average stays over the first three missions unless you decide otherwise |
| D3 | How players stay in sync | **A.** today's method: every computer replays the same decisions · **B.** the host runs Bennu and sends the full mission state after each action | **B**: Bennu has approvals, a host override after 30 s and the timed contact press, which are simpler and safer if only the host decides. The state is small (~15 KB) |
| D4 | How fancy the 3D is | **A.** 2D panels + simple 3D Bennu · **B.** detailed 3D spacecraft, descent and capsule entry | **A** first; extra 3D only if time remains (master doc feature-cut order) |
| D5 | Who keeps the "sees everything + system alerts" power | **A.** Mission Operations (closest to today's head engineer) · **B.** nobody special; everyone sees everything | **A** |

## 3. How Bennu plugs into the existing game

```
mission_control/
  data/bennu/bennu_simulation_v3.json        NEW  canonical config (byte-identical copy)
  data/bennu/bennu_nasa_extensions.json      NEW  game values, NASA facts, sources, labels
  engine/bennu/                              NEW  pure Python, no Ursina (testable on its own)
      formulas.py       v3 formulas exactly as written
      navigation.py     tracking, burn preview, auto-target, phasing, approach score
      scenario.py       seeded scenario + solvability check
      mission.py        BennuMission: 7 stages, roles, approvals, alerts, TAG, return
      scoring.py        debrief: 7 categories, 3 key decisions, advice
      roles.py          4 roles, allocation by player count, permissions
  multiplayer/core.py                        CHANGE  team protocol: mission type "bennu", 4 roles, state snapshots (D3)
  ui/bennu/                                  NEW  Bennu screens (Ursina)
      bennu_screens.py  briefing, build, navigation, exploration, return, debrief
      bennu_tag.py      7-step TAG incl. full-screen descent + contact sweep (manual-sim style)
      bennu_scene.py    3D Bennu (procedural rubble-pile "spinning top" + boulders), spacecraft, capsule
  ui/hub.py                                  CHANGE  team box -> Bennu card; optional solo Bennu card (D2)
  ui/team_ui.py                              CHANGE  lobby shows 4 roles + Bennu card; make_team builds a Bennu game
  ui/app.py                                  CHANGE  route team mode (and solo Bennu) to the Bennu screens
  engine/companion.py, data/companion.json   CHANGE  Cosmo: Bennu tips and sourced fun facts
  tools/make_card_art.py                     CHANGE  render assets/cards/bennu.png
  tools/screen_tour.py                       CHANGE  add Bennu screens to the automatic screenshot tour
  tests/test_bennu_*.py                      NEW  rules, scenarios, approvals, team sync, governance
  REFERENCES.md, README.md                   CHANGE  Bennu sources, acknowledgements, how to play
```

**Reused, not rewritten:**
- **Bennu's rules:** the browser build's engine (already tested by 66 tests) moves into `engine/bennu/` with only import changes.
- **Ursina toolkit:** `ursina_kit` (panels, text, buttons, meters, icons, fonts) and the navy dashboard style.
- **Manual-sim pattern:** the full-screen layout, Cosmo bubble and result overlay from `ui/manual_sim.py`.
- **Team mode:** `team_ui` lobby, chat panel, vote buttons and the network link.

## 4. Screens in Ursina (one per stage)

Layout follows today's mission screens: top bar (stage, roles, players), stage tracker, main area, and a right column with **Next step**, alerts, the waiting decision, the risk register and the team chat (team mode) or Cosmo (solo).

| Stage | Main area | Right column extras |
|---|---|---|
| Briefing | Objective, constraints, scenario card, Bennu facts with labels, role cards | - |
| Build | Part cards by group (Fit/Remove), live equipment list with mass, power and propellant meters, requirement checklist, trade study table (3 slots), 3D spacecraft that grows as parts are fitted | Risk register (mitigate/accept), *Propose: Lock build* |
| Navigation | 2D map (Sun, Earth, Bennu, gravity assist) + arrival target plane (corridor, position, uncertainty ring, preview arrow); direction and size sliders, Preview, Auto-target with iteration table, Tracking pass, Arrive | Burn proposal / approval |
| Exploration | Rotating 3D Bennu with 3 site markers; comparison table (± estimates, measured ✓, confidence, H); observe grid (site × OCAMS/OLA/OVIRS/OTES); slots left; real-mission card (Nightingale/Osprey) | Site risk mitigate/accept, site proposal |
| TAG | Step tracker 1–7. Readiness checklist → departure-timing dial (top-down Bennu, arrival point, site position) + alignment needle (±1°/±5°) → **full-screen descent** (altitude ladder Orbit → Checkpoint → Matchpoint → surface, contact-speed slider, Wave-off) → **contact sweep** (Space) → fire nitrogen → back-away → assessment (sample g, Q bars, H, Accept / Try again) | TAG GO decision, alerts |
| Return | Stow sample · return plan (2 choices) · departure burn · capsule release; short 3D capsule entry with parachutes (reduced motion: still images) | Capsule release decision |
| Debrief | Total + 7 category bars, the 3 key decisions with advice, what happened, capsule sequence, revealed truth, real-mission comparison, seed/versions, earlier runs; host: Replay (new seed / same seed) | - |

**About & sources** (from the settings window and the debrief) shows the NASA acknowledgement, the disclaimer, the label legend and all sources.

## 5. Team mode for Bennu

- **Joining (unchanged):**
  1. Turn on multiplayer in Settings.
  2. Press Host on one laptop; others Join with the host's address.
  3. Same Wi-Fi network; the default port stays 50607.
- **Lobby:**
  - Shows the Bennu card and **4 roles** (D1).
  - Players vote and ties are random.
  - With fewer than 4 players, roles are shared as in the role table of document 01.
  - The host can reassign roles if someone leaves.
- **Sync (D3 = B):**
  - Every button press is sent to the host.
  - The host checks the role and the rules, applies the action once, and sends the new mission state to everyone.
  - Two players acting at the same moment are handled one after the other, so nobody can spend the same fuel twice.
- **Decisions:**
  - A player proposes; everyone else sees an *Approve / Reject* card.
  - The proposer cannot approve; the host may approve for the team after 30 s.
- **Team chat** replaces Cosmo in team mode, as today. Mission Operations gets **system alerts** and sees everyone's activity (D5).
- **Leaving and rejoining:** the game continues, and the host reassigns the leaver's roles. A rejoining player receives the current state.

## 6. Honest limits of the Ursina version

| Master document asks for | Ursina version | Why |
|---|---|---|
| Private **internet** rooms with invite link + password, cloud server, HTTPS/WSS | **Same-network (LAN) only**, host address + port | A desktop game on one laptop cannot be reached over the internet without router port-forwarding or a relay server: not realistic in the time left. The master doc's fallback (14.2) allows a reliable two-device co-op demo plus solo/hot-seat, stated openly |
| Players join in a browser, no install | **Every player needs Python + Ursina + the game** on their own laptop | Ursina is a desktop app |
| Phones/tablets as players | Not supported | Same reason |

If internet play becomes essential, the browser build already made does it. The two versions share the same Bennu rules.

## 7. Build plan (small increments, each tested before the next)

| # | Increment | Done when (gate) |
|---|---|---|
| 1 | Port the Bennu engine + configs into `engine/bennu/` and `data/bennu/` with its tests | Bennu tests pass on Python 3.10; all 218 existing tests still pass |
| 2 | Team protocol for Bennu: 4 roles, host-run state snapshots, approvals, reassign, rejoin | Automated test: host + 3 bot players complete a mission; simultaneous spends cannot overspend |
| 3 | Bennu screens part 1: Briefing + Build (incl. 3D spacecraft, trade study, risk register) | Solo playable to Lock build; screenshots in the screen tour |
| 4 | Navigation + Exploration screens (2D map, sliders, 3D Bennu, site table) | Solo playable to site selection |
| 5 | TAG (7 steps, full-screen descent, contact sweep with Space, retry) + Return | Solo playable to capsule release; missed-contact and retry paths shown |
| 6 | Debrief, replay, About/sources, Cosmo Bennu tips and facts | Solo full mission end to end; failure run still debriefs |
| 7 | Team-mode wiring: hub card, lobby with 4 roles, chat, proposal cards, alerts for Operations | Two windows on one PC and two laptops on one Wi-Fi complete a mission together |
| 8 | Polish + docs: README, REFERENCES, NASA integration doc, test log, screen tour | Fresh-folder start on the team laptop; demo run rehearsed |

Each increment is a separate commit, so you can always go back to the last working version.

**Time estimate (honest):** about one long working day of building on my side across several turns, plus your testing on the real laptops. With the hackathon running now, increments 1–6 give a demo-ready **solo** Bennu. Increment 7 adds team play. If time runs short, cut in this order: detailed 3D → animations → extra alerts → learning-level depth. **Never cut** the full mission loop, the TAG interaction, the debrief or team play once it works.

## 8. Acceptance checklist (adapted from the master doc's definition of done)

- [ ] A new user starts the game with the README steps on the team laptop
- [ ] One player completes Bennu from briefing to debrief
- [ ] Two laptops on one Wi-Fi complete Bennu together (four if claimed)
- [ ] Every role has meaningful actions; joint decisions need a second player
- [ ] Part choices change mass, power and propellant in ways that matter later
- [ ] Navigation decisions change fuel use and arrival accuracy
- [ ] Observations change site confidence and the choice
- [ ] TAG is a hands-on 7-step operation with explained outcomes
- [ ] Sample containment, return and capsule recovery are playable
- [ ] The debrief explains performance and gives replay advice
- [ ] Seeds reproduce runs; every scenario passes the solvability check
- [ ] Facts are accurate and every number carries its label
- [ ] GPS, Perseverance and Europa Clipper solo missions still work exactly as before (218 tests pass)
- [ ] Sources, acknowledgements and run instructions are included
- [ ] The team can explain the architecture, the simplifications and what was actually tested
