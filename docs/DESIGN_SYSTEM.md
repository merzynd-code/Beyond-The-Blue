# Design system

Source of truth: `data/design_tokens.json`. Helpers: `ui/design_system.py`.
Main page: `ui/hub.py` (pictures in `assets/hub/`, made by `python tools/make_hub_art.py`).
Learning centre: `ui/academy.py` (colours in `ui/academy_theme.py`, contrast-tested).
Preview: `python tools/render_style_guide.py` → `docs/style_guide_preview.png`;
every screen, including the Academy: `python main.py --screens out`.

**Direction:** a modern mission-operations console. Calm dark surfaces, warm off-white
text, one periwinkle accent, generous spacing, numbers in a monospaced face. The main page
("Beyond the Blue") is the one bold, poster-like screen: deep-space navy, glowing gold for the
BENNU FRONTIER flagship and blue for the learning missions and BENNU ACADEMY. No neon glow on
mission screens, no scanlines or decorative cockpit brackets.

| Area | Rule |
|---|---|
| Resolution | Designed at 1280×720; uniform scale + letterbox for 1366×768 and 1920×1080. |
| Type | Inter (UI, bundled Bold/SemiBold) + JetBrains Mono (telemetry), SIL OFL. Scale: 40/28/22/18/16/14/13. Body text never below 13 px; small captions (badges, chips) 11–12 px only where a word is also shown elsewhere. Slanted headline art (BENNU FRONTIER, mission titles, BENNU ACADEMY) is pre-rendered from Inter Bold. |
| Spacing | 4 px grid: 4, 8, 12, 16, 24, 32, 48. 12-column grid, 24 margin, 16 gutter. |
| Colour | All text colours and all status colours pass WCAG AA (≥ 4.5:1) on every surface — checked by tests (`tests/test_design_system.py`, `tests/test_learning_views.py::TestContrast`). |
| Status | nominal / caution / warning / critical / offline / info. Each has a distinct icon **shape** and an upper-case label. Colour is never the only signal. |
| Truth labels | NASA FACT ◆ diamond · SCIENCE PRINCIPLE ▲ triangle · SIMULATED ⬢ hexagon · GAME VALUE ■ square · FICTIONAL ● circle. Same five as BENNU FRONTIER; shape + word, never colour alone. |
| HUD | Top bar (mission, phase indicator, clock+seed) · left rail (spacecraft status, each number with a **?**) · stage · right panel · bottom bar (disclaimer, **BENNU Academy (L)**, keys). |
| Components | Button (primary, secondary, ghost, danger; hover, pressed, focus, disabled), card (selected, invalid), panel, tooltip (400 ms delay), modal (one decision per modal), meter (always with number + unit), status chip, phase indicator (done ✓ / current ● / upcoming / blocked 🔒). |
| Motion | 120 / 200 / 320 ms; 700 ms for skippable scenes. Motion only explains state change. Reduced-motion → 0 ms, no parallax/particles. Low-performance → no particles/blur, 30 fps cap. |
| Audio | Mute (M), master/music/sfx sliders. Fully playable muted. Read aloud (V) uses the computer's own voice. |
| Themes | neutral, gps (charcoal/cream), mars (rust/ochre), europa (ice white/blue). |

## Main page ("Beyond the Blue")

| Region | Content |
|---|---|
| Banner | AI-generated art: team logo "TEAM BEYOND DELTA V", "BEYOND THE BLUE — three learning missions, one asteroid", four worlds. |
| Header | Mission Control emblem, "Game Hub" + campaign name, tagline, Credits saved (US$), Campaign score, Missions. |
| Flagship | BENNU FRONTIER: badge, slanted title, one-line pitch, **Play Solo**, **Level** drop-down, **Play as a Team (same network)**, 1–4 players, "Illustration (AI-generated)" label, ALWAYS OPEN feature list. |
| Mission cards | Picture ("AI-generated" tag), state badge (ready / locked / completed), mission number and difficulty, slanted title, "Inspired by …", one sentence, budget (US$), time limit, difficulty bars, Launch / "Complete Mission N to unlock" / score + Fly again. |
| Right column | Campaign progress (Earth → Mars → Europa, status, next mission); BENNU ACADEMY card. |
| Navigation | Home · Campaign report · New campaign · "Student-created educational game • Not a NASA product" · AI-assisted assets (opens credits) · Settings · Quit. |

## BENNU ACADEMY (learning components)

| Component | Where | Behaviour |
|---|---|---|
| **?** | Next to every left-rail number, the risk index, the sunlight line in operations, "Learning plan" on the briefing. | Opens the number's card: plain meaning, unit, truth label, higher/lower, live status sentence ("It will launch, but the margin is below target"), example. |
| **Learn more** | Part inspector in design. | Full beginner card: what it is, why needed, how it works, its numbers with labels, trade-offs, if missing, a common mix-up, sunlight at this destination for solar parts, cited fact. |
| **Why?** | Every readiness error/warning. | The rule, why it matters, what to try. |
| **Learn: this hazard** | Mission event window. | The hazard's concept card. |
| **What happened** | Opens after a mission problem is answered. | Your decision, what changed (real before/after numbers), why (only causes the engine shows), impact, other options you had. |
| **Quick check** | Readiness (optional) and debrief; Quick checks tab. | 3 questions; every option explained; CORRECT / NOT QUITE + best answer; Try again, Skip, Next. Never blocks. |
| **What you learned** | Debrief. | Three columns: mission performance · learning progress · getting ready for BENNU. |
| Academy panel | L key, bottom bar, main-page card. | Tabs: Start here (learning loop, five labels, learning path) · Concepts · Numbers · Real facts (live countdown for the Europa Clipper Earth flyby) · BENNU terms (the 51-term BENNU glossary) · Quick checks · Credits. Long cards turn pages (arrow keys); Esc goes back. |

Do not use the NASA insignia ("meatball"), logotype ("worm"), seal, mission patches or NASA
images anywhere. AI-generated pictures carry an "AI-generated" label where they appear.

Fonts: Inter Bold and SemiBold are bundled in `assets/fonts/` with the OFL licence. JetBrains
Mono is not bundled yet; until then telemetry uses the engine's monospaced font.
