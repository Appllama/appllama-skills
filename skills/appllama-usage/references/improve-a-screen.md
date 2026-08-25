# Playbook: make an existing screen better

The user has a screen (code, screenshot, or a live app) and wants it better.
"Better" means: measurably closer to the best equivalent screens shipped by
top-grossing apps — in hierarchy, motion, and feel — verified in the
simulator, not eyeballed in code.

## 1. Diagnose before you search

Run the screen in the simulator and study it against appllama-app-design-skill's
definition-of-done checklist. Name the top 3 deficits precisely ("no visual
hierarchy — three same-weight text rows", "dead motion — modal pops with no
transition", "non-native segmented control"). The research pass is aimed at
these deficits, not at "inspiration".

## 2. The 30 + 30 reference board

Two searches, three pages each (10 per page) — and keep paging past three
if the results are still strong:

- `search_screens(query="<screen type / content words>", mode="keyword")` —
  finds screens NAMED like yours (e.g. "workout summary", "streak stats").
  Add filters that sharpen it: `screen_type`, `flow`, `element`.
- `search_screens(query="<what it should feel like>", mode="semantic")` —
  finds screens that LOOK like the goal ("calm dark stats dashboard with a
  hero number and weekly bars"). Different screens will surface — that's the
  point of running both.

Save all ~60 into a local board (see SKILL.md's structure). Pick the **best
5–8** and say why each earns its place. If one is nearly perfect, pull
`get_screen` on it — its `similar_screens` often surfaces an even better
sibling from another app.

## 3. Extract the pattern

From the picks, write the target: layout skeleton, hierarchy order, control
choices, spacing rhythm, palette role-mapping, motion moments (entrance,
press feedback, data reveal). This is a spec, not a mood board — every line
should be checkable in a screenshot.

## 4. Rebuild and iterate (the loop)

1. Implement against the spec using **appllama-app-design-skill** (typography
   ramp, semantic colors both themes, native controls, Reanimated motion
   with the platform's curves, state kept boring).
2. Asset gaps (illustration, empty-state art, icons beyond the symbol set):
   generate with the best available image model at max quality, one style
   system, per the design skill's image-assets reference.
3. **Simulator loop until perfect**: screenshot → compare side-by-side with
   your top references → fix → repeat. Record the motion and scrub it
   frame by frame. Verify Dynamic Island/safe areas, dark + light, Dynamic
   Type XL, Reduce Motion, 60fps on the interaction.
4. Do not stop at "better than before". Stop when the honest side-by-side
   with the best reference reads **at least as good** — hierarchy, motion,
   font discipline, state handling, everything. If it doesn't, name the gap
   and go around again.

## 5. If the screen is part of a flow

Screens live in journeys. After the screen passes, walk one step before and
one step after it in the simulator: entrance transition, exit transition,
state carried across — and what back does from it on iOS and Android
(appllama-app-design-skill's Navigation laws).
If the screen is a sheet, a modal or a step behind a one-way door, verify
its presentation matches what it *is*, not just how it looks. Use `get_flow_apps` + `list_app_screens(flow=…)` if
you need to see how winners chain the surrounding steps
(references/research-methods.md).
