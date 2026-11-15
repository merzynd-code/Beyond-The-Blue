# BENNU ACADEMY for the three learning missions

Implements the team's *Learning Framework for the Three Preparatory Space Missions* without
changing the core game: same rules, missions, events, scores and saves. BENNU FRONTIER's own
learning layer is described in `docs/bennu/LEARNING_LAYER.md`; both share the five truth labels
and meet in one place, **BENNU ACADEMY**.

## Files

```
engine/education.py          content, explanations, quick checks, evidence tracker (stdlib only, read-only)
data/education/*.json        all learning text: numbers, concepts, part cards, facts + references,
                             readiness rules, event lessons, quick checks, curriculum, naming + credits
ui/academy.py                the Academy panel, "?" buttons, card renderer, quick-check flow
ui/academy_theme.py          colours (WCAG AA tested)
ui/learning_views.py         results -> display blocks (no Ursina; unit-tested)
tests/test_education.py, tests/test_learning_views.py
```

## The player's loop

Understand a part or number → decide → see what happens → read why → do better next mission.

| Moment | What the player gets |
|---|---|
| Briefing | **Learning plan ?**: goals for this mission, concepts with their stage (introduce / reinforce / apply), sunlight at this distance, cited fact cards, what comes next. |
| Design | **?** on every number in the left rail; **Learn more** on any part. |
| Readiness | **Why?** on each error/warning; optional **Quick check** before launch. |
| Problems | **Learn: this hazard** in the event window; **What happened** after the choice (real before/after numbers, only causes the engine shows). |
| Operations | **?** on numbers and on the sunlight line ("Sunlight 4 % of Earth's (5.2 AU)"). |
| Debrief | **What you learned**: mission performance, learning progress, getting ready for BENNU; quick check after the mission. |
| Anytime | **L** or **BENNU Academy (L)** in the bottom bar or the main-page card. |

## Learning framework acceptance checklist (section 14)

| Criterion | Status |
|---|---|
| Every player-facing technical parameter has a plain explanation and unit/label | ✅ 42 entries; tests cover every MODEL_CONTRACT variable and HUD metric |
| Every component has a beginner learning card | ✅ all 11 categories + the part's own numbers, notes and facts |
| GAME VALUES visibly distinct from real units and facts | ✅ five labels with shapes on every value |
| Descriptions explain purpose and trade-offs | ✅ trade-offs, if-missing, mix-ups on every card |
| Readiness warnings explain rule, why, what to do | ✅ every validation code mapped (13 rules) |
| Events/consequences use actual simulation results | ✅ before/after numbers from the session; causes only from numbers/flags |
| Previews match the engine | ✅ power walkthrough uses the contract rules and prefers engine results (tested) |
| Glossary available in design and operations | ✅ Academy (L) on every screen |
| Debrief separates performance from learning | ✅ separate columns (tested) |
| Knowledge checks explain answers and don't block | ✅ 18 questions, every option explained, skip/retry |
| Introduce → reinforce → apply across missions | ✅ `curriculum.json` (tested) |
| BENNU transfer explained | ✅ bridge on Start here, in every debrief |
| No change to the engineering model | ✅ read-only; the 320 existing tests pass unchanged except label text |
| Sourced claims traceable | ✅ every fact card cites `REFERENCES.md` IDs with a checked date |

## Editing content

Everything is JSON in `data/education/`. Run `python -m unittest discover -s tests -t .` after
editing: tests catch unknown references, missing explanations, wrong quiz answers (every number
is recomputed), real names outside cited facts and colour contrast.
