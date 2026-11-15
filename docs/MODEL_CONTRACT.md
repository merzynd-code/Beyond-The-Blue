# MISSION CONTROL — Engineering Model Contract

This contract is written **before** the formulas are coded. Every calculation in
`engine/resources.py`, `engine/session.py` and `engine/validation.py` must follow it.
If code and contract disagree, the contract wins and the code is a bug.

The learning layer (`engine/education.py`) explains these rules to the player; it never
calculates mission results itself (§13).

## 1. Five truth labels

One set of labels is used everywhere: the three learning missions, BENNU FRONTIER, BENNU ACADEMY,
the HUD, fact cards, the data and the report (`data/education/truth_labels.json`,
`ui/frontier/common.py`). In the Academy each label also has its own icon shape, so colour is
never the only signal.

| Label | Shape | Meaning | Example |
|---|---|---|---|
| **NASA FACT** | diamond | A real, sourced statement about a real mission, spacecraft, planet or system, with its reference and checked date. (GPS facts come from the U.S. government's GPS.gov; the card shows that source.) | "GPS needs at least 24 working satellites"; the radioisotope unit's 110 W |
| **SCIENCE PRINCIPLE** | triangle | An established law or engineering idea, or a value derived from one. | energy = power x time; the rocket equation; sunlight = 1/d² |
| **SIMULATED** | hexagon | An output of this game's simplified model. | battery charge, data on board, propellant left, coverage |
| **GAME VALUE** | square | An invented number that makes the game work (real unit names such as W, Wh, s, m/s are used where they help learning). | kg, US$, GB, part ratings, reliability, resilience, targets |
| **FICTIONAL** | circle | An invented situation, object or picture. | mission names, events, launchers, 3D models, AI-generated illustrations |

Mapping from the old three kinds: *sourced fact* → NASA FACT; *real physics, game values* →
SCIENCE PRINCIPLE for the law and GAME VALUE for the inputs; *pure game units* → GAME VALUE.

No component number in `data/components.json` is a real spacecraft specification, **except
where a component's `truth_overrides` tags it NASA FACT.** The only such value is the
radioisotope unit's **110 W**, which matches the real MMRTG at the start of a mission
(`REF-M2020-POWER`). Every other component has `"source_note": {"basis": "game_units", ...}`.

## 2. Variables and units

**Display units (all four missions, `engine/units.py`):** mass and propellant in **kg**, data in
**GB** (data rates in GB/h), money in **US$** shown as `$870M` (millions) or `$450K` (thousands, below
$1 million), power in W, energy in Wh, speed change in m/s, Isp in s, time in h. The units are real;
the values are GAME VALUES unless a NASA FACT tag says otherwise. JSON field names keep their
historical suffixes (`mass_mu`, `cost_cr`, `*_du`, `*_gu`, `*_mc`, `*_gb`) so saves, hashes and the
canonical BENNU file stay valid.

**Real-world sizes per mission (`display_scale`).** Each learning mission shows real-world sizes by
multiplying the simulation's internal numbers at display time only (`engine/units.py`):

| Mission | Mass | Cost | Data | Team reference values |
|---|---|---|---|---|
| GPS Satellite | x 6.0166 | x 0.425 | x 0.08 | satellite ~4,350 kg, $255M, 24 GB storage |
| Delta Rover | x 4.7264 | x 2.8684 | x 0.00667 | spacecraft ~4,060 kg (rover 1,025 kg), $2,725M, 2 GB flash |
| Europa Ice Probe | x 6.7614 | x 5.9770 | x 0.7143 | ~6,065 kg (3,241 kg dry + 2,750 kg propellant), $5,200M, ~500 GB |

The balanced sample design of each mission therefore shows the reference mass, budget and storage;
individual parts are scaled estimates (GAME VALUES). Margins, the rocket equation, power and scores
are ratios or use unscaled quantities, so game balance, saves and replays are unchanged. Each
mission also lists its three rockets' lift-off and propellant masses (`launch_vehicles_real`), shown
on the launcher card; the payload a rocket can lift is the scaled game capacity.

| Symbol / field | Unit | Label | Meaning |
|---|---|---|---|
| `mass_mu` | kg | GAME VALUE | Dry mass of one component. |
| `propellant_mu` | kg | GAME VALUE (loaded) / SIMULATED (in flight) | Propellant currently on board. |
| `cost_cr` | US$ (millions) | GAME VALUE | Cost of a component, launch vehicle or launch profile. The field name keeps `_cr` so saves stay valid; the unit shown everywhere is **US$**. |
| `budget_cr` | US$ (millions) | GAME VALUE | Mission budget (field name kept; unit shown as US$). |
| `capacity_mu` | kg | GAME VALUE | Launch vehicle base payload capacity. |
| `capacity_factor` | dimensionless 0–1 | GAME VALUE | Share of base capacity available for a launch profile (harder trajectory → less). |
| `generation_w` | W | GAME VALUE | Electrical power a **generator** produces at 100 % conditions; for solar, **at 1 AU** from the Sun. |
| `draw_w` | W | GAME VALUE | Electrical power a component **consumes** while powered. |
| `capacity_wh` | Wh | GAME VALUE | Energy a **battery** can store. A battery never has `generation_w`. |
| `battery_wh` | Wh | SIMULATED | Energy currently stored. |
| `sun_distance_au` | AU | NASA FACT (rounded) | Distance from the Sun: mission `destination` (Earth 1, Mars 1.524, Jupiter 5.2) or a phase's `environment` (cruise averages: Mars 1.195, Europa 1.58). |
| `sunlight_fraction` | 0–1 | SCIENCE PRINCIPLE | `1 / sun_distance_au²` (inverse-square law). |
| `illumination` | 0–1 | GAME VALUE | **Local** sunlight in the current phase/environment: night, shadow, dust (1 = full local Sun). |
| `rate_du_per_h` | GB/h | GAME VALUE | Data an instrument produces per hour of observation. |
| `storage_du` | GB | GAME VALUE | Onboard data storage capacity. |
| `downlink_du_per_h` | GB/h | GAME VALUE | Data a radio can send per hour at link quality 1.0. |
| `link_quality` | 0–1 | GAME VALUE | Fraction of the radio's rate achievable in the current contact window. |
| `isp_s` | s | GAME VALUE | Specific impulse of an engine. |
| `dv_ms` | m/s | GAME VALUE | Velocity change a manoeuvre needs. |
| `reliability` | 0–1 | GAME VALUE | Balancing value for "chance this unit works without a fault over the mission". Never used to destroy a spacecraft directly. |
| `resilience` | integer −3…+3 per hazard key | GAME VALUE | How much a component protects against (+) or exposes to (−) a hazard. |
| `t`, `dt` | h (hours) | SIMULATED | Simulation time step `dt = 1 h`. |

Every one of these has a beginner explanation in `data/education/parameters.json`
(test: `test_every_contract_parameter_explained`).

## 3. Time step

* Design, readiness and briefing are **untimed**.
* Timed phases (cruise, arrival, operations, downlink…) advance in whole hours.
* Each power/data step is exactly **1 h**. An action lasting `n` hours runs `n` steps.
* Phases may declare `simulate_h` (time that passes automatically on entry) and the
  mission may declare an `ops_time_budget_h` that limits player-chosen operation hours
  (0 = no limit; events with `ops_time_h` can change the limit during the mission).
* Phases may also declare `background_h`: travel time compressed into the phase (weeks or
  months of cruise). On entry, the power balance runs for these hours with always-on loads
  only; they do not move the mission clock, use operations time or trigger events. With
  generation this barely matters; a battery-only spacecraft runs flat, as it would in reality.

## 4. Mass and budget

```
dry_mass_mu      = Σ mass_mu  of every spacecraft component (launch vehicle excluded)
wet_mass_mu      = dry_mass_mu + propellant_load_mu
launch_capacity  = launch_vehicle.capacity_mu × profile.capacity_factor
mass_margin      = (launch_capacity − wet_mass_mu) / launch_capacity

total_cost_cr    = Σ cost_cr of components + launch_vehicle.cost_cr + profile.cost_cr     (unit: US$)
budget_margin    = (budget_cr − total_cost_cr) / budget_cr
```
* `wet_mass_mu > launch_capacity` → **error** (readiness blocked).
* `total_cost_cr > budget_cr` → **error** (readiness blocked).
* Margin below the mission's target margin → **warning** only.
* Duplicate components (redundancy) count once per copy for mass and cost.

## 5. Power and battery

For one step of `dt = 1 h`:
```
sunlight_fraction = 1 / sun_distance_au²                       (inverse-square law)
solar_w   = Σ generation_w(solar units, not disabled)
              × sunlight_fraction × illumination × modifier["solar"]
nuclear_w = Σ generation_w(radioisotope units, not disabled) × modifier["radioisotope"]
gen_w     = solar_w + nuclear_w
base_w    = Σ draw_w of powered "always" components (avionics, thermal, standby radio…)
active_w  = Σ draw_w of components the current action switches on (instrument, transmitter…)

net_wh    = (gen_w − base_w − active_w) × dt
```
Sunlight relative to Earth: Earth orbit 100 %, Mars (≈ 1.52 AU) ≈ 43 %, Jupiter/Europa
(≈ 5.2 AU) ≈ 3–4 %. Radioisotope output does not depend on distance or illumination.

Implemented in `engine/mission.py` (`MissionDefinition.environment`): when a mission gives
`destination.sun_distance_au` (or a phase overrides it), the resolved environment carries
`sun_distance_au`, `sunlight_fraction`, `local_illumination` and the combined `illumination`
that `resources.generation_w` uses. Missions without a distance behave exactly as before.

**Balance note.** Earlier data held the distance effect inside `illumination` (Mars 0.43, Europa
0.04, cruises 0.7 / 0.4). Those values are now local sunlight 1.0 with distances 1.524 AU (Mars),
5.2 AU (Jupiter/Europa) and cruise averages 1.195 / 1.58 AU, so the old balance is kept (Europa
solar power is about 7 % lower, from 4 % to 3.7 % of Earth's sunlight). All mission balance tests pass.

Battery rules (`0 ≤ battery_wh ≤ capacity_wh`, enforced every step):
1. If `net_wh ≥ 0`: `battery_wh = min(capacity_wh, battery_wh + net_wh)`.
   Energy above capacity is recorded as `curtailed_wh` (wasted), never stored.
2. If `net_wh < 0` and the battery can cover it: `battery_wh += net_wh`.
3. If the battery cannot cover the active load: the **active load is shed** for that
   hour (the instrument/transmitter does nothing that hour) and this is recorded as
   `load_shed_h`.
4. If the battery cannot even cover the base load: battery goes to 0, the missing energy
   is recorded as `deficit_wh`, and flag `power_critical` is set.

A battery **stores** energy; it never creates it. A design with zero generators can
only spend its initial charge. The battery starts each mission full (`battery_wh = capacity_wh`)
— this is a stated simplification.

## 6. Data

```
generated_du (per hour) = rate_du_per_h of the observing instrument (0 if load shed)
free_du                 = storage_du − stored_du
stored  += min(generated, free_du);   overflow_du += generated − stored_now
sent_du (per hour)      = Σ downlink_du_per_h (radios not disabled) × link_quality × modifier["downlink"]
```
* Data is kept as packets tagged with `target`, `priority` (`core` before `secondary`)
  and creation order. Downlink sends core packets first, then secondary, oldest first.
* `transmitted_du ≤ stored_du` always: you cannot send data you do not have.
* Data generated when storage is full is **lost** and counted as `overflow_du`.
* If the transmitter's power is shed for an hour, nothing is sent that hour.

## 7. Science return

For each science target:
```
coverage          = min(1, observed_h / hours_required)
returned_fraction = transmitted_du(target) / generated_du(target)   (0 if nothing generated)
science_value     = target.value × coverage × returned_fraction
```
Observing a target requires an instrument with at least one of the target's
`instrument_capabilities`. Science counts only once data reaches Earth.

## 8. Propulsion (simplified)

Uses the real **Tsiolkovsky rocket equation** with game-balanced inputs:
```
g0 = 9.80665 m/s²
propellant_needed = m0 × (1 − exp(−dv_ms / (isp_s × g0)))      m0 = current wet mass
```
* `isp_s` = best Isp of non-disabled components with capability `main_propulsion`.
* If `propellant_needed > propellant_mu`: all propellant is used, flag
  `manoeuvre_shortfall` is set, and the shortfall is recorded. Propellant never goes negative.
* Mass units cancel in the equation, so kg is acceptable.
* Readiness predicts the total propellant for all profile manoeuvres
  (sequentially, mass decreasing). Not enough → **error**. Reserve below the mission's
  `propellant_reserve_target` fraction → **warning**.
* Simplifications: no gravity losses, no staging, no trajectory dynamics, no gravity
  assists. Profiles are menu choices with a capacity factor, a cost and a list of Δv
  manoeuvres.

## 9. Reliability and risk

```
unit_r(category)  = 1 − Π over copies (1 − reliability)      (redundant copies help)
system_r          = Π unit_r(category)  over categories present
risk_index        = 1 − system_r                               (displayed as %, GAME VALUE)
```
Reliability **only** changes event weights (an unreliable unit makes a related fault event
more likely). It never removes a spacecraft by dice roll, so failures always come with a
warning and a choice.

## 10. Resilience

`resilience[h]` = Σ resilience values for hazard `h` over all components. Exposed as
metric `resilience.<h>` for event conditions and weight modifiers.

## 11. Rounding and invalid state

* Internally all values are floats; nothing is rounded during simulation.
* Display rounding: mass/cost/data to 1 decimal, percentages to whole numbers.
* Clamps are explicit (`battery_wh`, `propellant_mu`, `stored_du`, link/illumination 0–1).
* Negative input values in data files are rejected by the loader (except `resilience`).
  `sun_distance_au` must be > 0.
* Unknown IDs, illegal phase transitions and actions in the wrong phase raise
  `MissionRuleError` — the engine never silently "fixes" an impossible request.

## 12. Known model limitations (simplifications)

* No orbital mechanics, gravity losses, gravity assists or real trajectories.
* Power generation is constant within an hour; no battery efficiency losses; no ageing
  (the real MMRTG loses a few percent per year).
* Thermal behaviour is represented by hazard resilience, not heat-flow equations.
* Sunlight uses the mean distance of the destination; real distances change along an orbit.
* Communication delay is not part of the shared power/data model. The Mars manual landing and
  drive apply a short command delay (2 s high-gain / 5 s low-gain, GAME VALUE); the learning
  layer explains the real 3–22 minute one-way delay as a SCIENCE PRINCIPLE.

## 13. Learning layer (read-only)

* `engine/education.py` reads session results and data; it never changes mass, power,
  propulsion, data, events, scoring or outcomes.
* Previews and power-step walkthroughs use the engine's reported values. If a value is
  missing, a preview may be computed with **these same rules** and is marked "preview".
* An explanation may only state a cause that the engine's metrics or flags show (for
  example `load_shed_h` increased → "the active part was switched off"). Otherwise it says
  "several factors".
