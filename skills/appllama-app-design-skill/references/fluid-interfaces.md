# Fluid interfaces — the physics of feel, materials and depth, and the principles behind them

An interface feels alive when motion **starts from the current on-screen
value, inherits the user's velocity, projects momentum forward, and can be
grabbed and reversed at any instant**. Springs make all of that natural,
because they are interruptible and velocity-aware by construction. This
reference is the *why* behind [motion.md](motion.md) and the feel-checks
behind the simulator pass — the things that separate "it animates" from "it
feels like an extension of the hand". Read it when building anything a
finger drags, anything translucent, or anything that gives feedback.

The frame: an interface serves four human needs — **predictability,
understanding, achievement, joy** — and behaves like the physical world:
things respond instantly, move continuously, carry momentum, resist at
boundaries, and can be redirected mid-motion.

## 1. Response — kill latency

The moment lag appears, directness falls off a cliff.

- **Respond on press-in, not on release.** Highlight the instant the finger
  lands. Waiting for touch-up to show anything feels dead.
- **Audit every latency on the input path** — debounces, artificial timers,
  "wait for the transition" gates, work scheduled before first feedback.
  Anything non-essential is a regression.
- **Feedback is continuous during the interaction, not only at the end.** A
  drag, slider or sheet updates 1:1 with the finger the whole way.

## 2. Direct manipulation — 1:1 tracking

Touch and content move together. When the user drags something it stays
glued to the finger **and respects the offset where they grabbed it** —
snapping to the element's center on grab breaks the illusion instantly.
Gesture Handler gives you `translationX/Y` and `velocityX/Y`; keep the grab
offset by capturing the current value in `onStart` and adding the
translation to it, never by assigning the absolute finger position.

## 3. Interruptibility — the most important principle

The thought and the gesture happen in parallel. Every animation must be
grab-able and reversible at any moment: a closing sheet the user catches
again follows the finger — it does not finish closing, then reopen.

- **Never lock out input during a transition.**
- **Animate from the presentation (current) value, never the target.**
  `onStart(() => { startY.set(translateY.get()) })` — reading the live value
  is what prevents the jump.
- **Springs, not fixed-duration timing, for anything gesture-driven** —
  timing restarts from zero; a spring retargets from where it is, carrying
  velocity.
- **When a gesture reverses, blend velocity — don't hard-cut it.** Replacing
  one animation with another at a reversal is a "brick wall"; a spring given
  the current velocity avoids it.
- **Decompose 2D motion into independent X and Y springs.** One spring on a
  2D distance desyncs when the axes have different velocities.

## 4. Behaviour over animation — think in springs

A pre-scripted animation can't respond to new input; a spring can — new
input just changes the target and motion stays continuous. Reason in the two
designer parameters, not mass/stiffness/damping:

- **Damping ratio** — controls overshoot. `1.0` = critically damped, no
  bounce; `< 1.0` overshoots; lower is bouncier.
- **Response / duration** — how quickly the value approaches the target. A
  spring has no fixed length; its settle time emerges.

| Interaction | Damping | Response |
|---|---|---|
| Move / reposition | `1.0` | `0.4 s` |
| Rotation | `0.8` | `0.4 s` |
| Drawer / sheet | `0.8` | `0.3 s` |

House style: damping `1.0` everywhere by default; bounce (`~0.8`) **only
when the gesture carried momentum**. Overshoot on a menu that faded in feels
wrong; on a card you flicked it feels right. (These are the `SETTLE` /
`SNAP` / `SHEET` constants in motion.md.)

## 5. Velocity hand-off — the seam between drag and animation

When a gesture ends the animation **continues at the finger's exact
velocity** so there is no visible seam. Pass `e.velocityY` into
`withSpring(..., { velocity })` every time a gesture releases. This is the
detail that most separates "fluid" from "fine".

## 6. Momentum projection — animate to where the gesture is *going*

Don't snap to the nearest boundary from the *release point*. Use velocity to
**project the resting position** — exactly like scroll deceleration — then
snap to the target nearest that projection. A flick then throws the element
instead of nudging it. The projection function (`project()` in
motion-recipes.md) is the exponential-decay form with
`decelerationRate ≈ 0.998` (0.99 for snappier) — not the physics-textbook
`v²/2a`. Decide **commit vs. revert from the projected resting position** —
release position + `project(velocity)`, tested against the threshold — never
from the release position alone. That is exactly `projected > HEIGHT * 0.4`
in the motion-recipes.md sheet, and it is why a short fast flick commits, a
slow drag past the threshold commits, and a release at rest short of it
snaps home.

## 7. Spatial consistency — symmetric paths, anchored origins

If something disappears one way, we expect it to return from there.

- **Enter and exit along the same path.** In from the right, out to the
  right. In-from-right / out-the-bottom reads as two unrelated things.
- **Anchor interactions to their source.** A popover or menu grows from the
  element that triggered it (transform origin at the trigger, scale from
  `0.95`); a centered modal is the one exception — it is not anchored to
  anything. Native menus (`Link.Menu`, `@react-native-menu/menu`) get this for free; a JS
  popover must set its origin deliberately.
- **Mirror the easing on reversible transitions** so outbound and return
  paths match; `animationMatchesGesture: true` on custom stack animations.
- **Direction-aware**: forward slides one way, back slides the opposite way.

## 8. Hint in the direction of the gesture

People predict the final state from a trajectory. Intermediate frames should
*point at* the outcome — a card grows up and out toward the finger, a sheet
follows the drag direction — rather than interpolate blindly to the target.

## 9. Rubber-banding — soft boundaries

At an edge, resist progressively instead of stopping hard. A hard stop reads
as "frozen"; rising resistance reads as "responsive, but there's nothing
more here." Use `rubberband()` from motion-recipes.md past any boundary the
finger can cross; real things slow before they stop.

## 10. Gesture design details — the feel checklist

- **Tap**: highlight on touch-down, commit on touch-up; ~10 px of
  hysteresis / hit padding (`hitSlop`, `pressRetentionOffset`); cancel by
  dragging away, and un-cancel by dragging back.
- **Drag / swipe**: a small movement threshold before committing to a
  direction (`activeOffsetX/Y([-10, 10])`), then 1:1.
- **Recognize all plausible gestures in parallel from the first move**, then
  cancel the losers once intent is clear (`Gesture.Race` / `Simultaneous` /
  `requireExternalGestureToFail` on RNGH 2; `useCompetingGestures` /
  `useSimultaneousGestures` / `{ requireToFail }` on RNGH 3). Avoid
  recognizers that only report a final state — they throw away the
  continuous tracking feedback needs.
- **Minimize disambiguation delays.** Double-tap detection delays every
  single tap; pay that cost only where double-tap truly exists.
- **Multi-touch protection**: ignore extra touch points once a drag has
  begun, or switching fingers mid-drag teleports the element.

## 11. Frame-level smoothness

Smoothness is what's *in* the frames, not just the frame rate. Keep
per-frame position change below the perception threshold (avoid strobing);
for very fast motion a subtle stretch or blur encodes speed better than a
sharp streak; animate only `transform` and `opacity`; at 120 Hz the budget
is 8 ms per frame, which is why a UI-thread animation matters more on
mobile than on the web.

## 12. Materials and depth — translucency conveys hierarchy

Translucent materials are a floating functional layer that bring structure
without stealing focus.

- **Nav bars, tab bars, toolbars and sheets are translucent layers with
  content scrolling underneath** — not opaque strips that consume a fixed
  band. On the native stack: `headerTransparent` + `headerBlurEffect`; in
  custom chrome, `expo-blur`. On iOS 26 the system draws Liquid Glass for
  native bars — let it, and never fake glass under glass.
- **Material weight encodes hierarchy**: heavier, darker materials separate
  structural regions; lighter ones draw attention to interactive elements.
  **Never stack a light translucent surface on another** — legibility
  collapses.
- **Bigger surfaces read thicker**: stronger blur, deeper shadow than a small
  chip. Shadows are heavier over busy content (separation), lighter over
  plain backgrounds.
- **Dim to focus, separate to keep flow.** A modal task pairs the surface
  with a scrim and pushes the background back; a parallel, non-blocking
  panel uses translucency and offset *without* a scrim. For stacked sheets,
  progressively dim and push back each parent layer.
- **Vibrancy keeps text legible over changing backgrounds**: over a material,
  don't use flat grey text — higher contrast, slightly heavier weight, a
  touch more tracking. Put color on a solid layer, not on the translucent
  foreground.
- **Scroll-edge effects, not hairlines.** Where content meets floating
  chrome, a small blur/gradient mask — only where floating UI actually
  overlaps content.
- **Materialize, don't just fade.** A glass surface arrives with blur and
  scale together, so it reads as a material arriving rather than an opacity
  change (crossfade a static blur layer; never animate blur intensity per
  frame on Android).

## 13. Multimodal feedback — motion + sound + haptics

Three rules for combining senses:

1. **Causality** — it must be obvious what caused the feedback. Fire on the
   actual causal event (the toggle flipping, the item snapping home) and
   match the character to the action's physicality.
2. **Harmony** — visual, sound and haptic land on the **same frame**.
   Latency between them breaks the illusion.
3. **Utility** — feedback only where it earns its place (success, error,
   commit, snap). Over-feedback trains people to ignore all of it.

## 14. Reduced motion and accessibility signals

Reduced motion means a gentler, non-vestibular equivalent — not silence.
Honor three independent signals:

- **Reduce Motion** (`useReducedMotion()` / `AccessibilityInfo.isReduceMotionEnabled`)
  → short cross-fades and static transitions for slides, springs, parallax;
  drop overshoot; keep the opacity/color changes that aid comprehension;
  custom transitions become crossfades; native stack / tab / sheet
  transitions stay the system's (iOS crossfades pushes only when the user
  also chose Prefer Cross-Fade Transitions —
  `AccessibilityInfo.prefersCrossFadeTransitions()` if custom chrome mirrors
  it).
- **Reduce Transparency** (`AccessibilityInfo.isReduceTransparencyEnabled`,
  iOS) → translucent surfaces go frosty or solid: raise background opacity,
  drop the blur.
- **Increase Contrast** → near-solid surfaces with a defined border.

Also: no full-viewport moving backgrounds; no slow looping oscillation (~one
cycle per 5 s); no abrupt brightness jumps (ease light↔dark theme changes);
large moving objects go semi-transparent while they travel.

## 15. Typography — optical sizing, tracking, leading

- **Tracking is size-specific, never one value.** Large display text wants
  *negative* tracking (`letterSpacing` around `-0.02em` equivalents); body
  sits near `0`; small captions slightly positive. A fixed letterSpacing is
  wrong somewhere.
- **Leading tracks size inversely**: tight on large headings, looser on body;
  looser for scripts with tall ascenders/descenders; tighter for dense UI.
- **Hierarchy is weight + size + leading as a set**, not size alone; weight
  adds presence without taking space.
- **Respect Dynamic Type.** Layout scales *with* the text; spacing follows the
  type; `maxFontSizeMultiplier` only where a layout genuinely cannot grow,
  never as a blanket cap.
- **Default to the platform system font** (SF / Roboto) — it ships optical
  sizing and tracking tables. Override only with a reason, and then carry the
  same discipline yourself.

## 16. The eight principles — the names you reason with

1. **Purpose** — decide what *not* to build; every feature spends the user's
   attention and trust.
2. **Agency** — keep people in control; easy undo for slips; confirmation
   dialogs only for genuinely destructive, irreversible actions (overuse
   trains click-through).
3. **Responsibility** — act in the user's interest: ask for permissions at
   the moment they're needed, only for what's needed; anticipate misuse.
4. **Familiarity** — build on what people already know; metaphors neither too
   literal nor too abstract; things that look the same behave the same and
   live in the same place. Break a convention only with proof, then test it.
5. **Flexibility** — adapt to context, device, ability; let people
   personalize when no single layout fits everyone.
6. **Simplicity, not minimalism** — strip the unnecessary so the core shines;
   plain words, fewer steps, hierarchy through order/spacing/contrast;
   sometimes *adding* context simplifies; common path first, advanced one
   level deeper.
7. **Craft** — nothing is random: every spacing, timing and alignment is a
   choice you can defend; jittery scroll, misaligned icons and layouts that
   break on rotation read as carelessness.
8. **Delight** — the result of the other seven, not confetti on top. Decide
   the emotion (calm, confident, excited) and reinforce it everywhere.

Tactical rules that serve them:

- **Feedback comes in four kinds** — status, completion, warning, error.
  Confirm meaningful actions, expose ongoing status, warn before problems,
  validate inline.
- **Wayfinding**: every screen answers *Where am I? Where can I go? What's
  there? How do I get out?* Never trap the user (see navigation.md).
- **Grouping and mapping**: proximity implies relationship; a control sits
  near what it affects and is arranged to mirror what it changes. A control
  that needs a label to explain it has weak mapping.
- **Specific labels beat safe ones**: "Progress", "Library" — not "Home",
  "More". Specificity creates predictability.

## 17. Process

- **Prototype interactively** — a working interaction is worth a thousand
  stills; you discover the interface by playing with it, and it sets a bar
  the final implementation can't slip under (see variant-lab.md).
- **Design interaction and visuals together** — motion is not a layer added
  after the pixels.
- **Review in slow motion and frame by frame**, with fresh eyes the next day —
  which is exactly what the full-motion simulator pass does.

## Quick reference

| Need | Technique | Value |
|---|---|---|
| Default UI spring | critically damped | damping `1.0`, response `0.3–0.4 s` |
| Momentum / flick spring | slight bounce | damping `~0.8`, response `0.3–0.4 s` |
| Gesture → spring | hand off release velocity | `withSpring(to, { ..., velocity })` |
| Flick landing point | project momentum | `current + project(v)`, `d ≈ 0.998` |
| Interrupt cleanly | start from the live value | capture in `onStart` |
| Reverse vs commit | projected position vs. the threshold | `current + project(v)` > threshold |
| 1:1 drag | translation added to the grab-time value | never the absolute finger position |
| Boundary | rubber-band | `rubberband(over, dimension, 0.55)` |
| Feedback | on press-in, continuous | never only at the end |
| Translucent chrome | material + content scrolls under | `headerTransparent` + blur |
| Tracking | size-specific | tighten display, body ~0 |
| Reduced motion | custom motion: cross-fade, not slide/spring; native transitions: the system's | keep comprehension cues |
