---
name: appllama-app-design-skill
description: Build native-feeling, benchmark-quality mobile app screens (Expo / React Native). Use when designing or implementing any mobile UI — screens, flows, onboarding, paywalls, tab bars, sheets, settings, empty states — or when polishing motion, gestures, navigation, typography, dark mode, or perceived performance. Enforces Apple HIG fidelity, semantic colors, native controls, anti-slop discipline, navigation semantics (push vs replace, modal vs sheet vs overlay, the one-way doors where back must not exist), a strict motion bar (frequency gate, exact springs/curves/durations, UI-thread discipline, haptics), a full-motion simulator-verified iteration loop, and a study-real-apps-first workflow (pairs with the Appllama MCP). Trigger on "build a screen", "make this screen better", "design the onboarding", "wire up this flow", "add a bottom sheet", "polish the UI", "make it feel native", "review the animations", or any mobile design/implementation task.
license: MIT
metadata:
  author: Appllama (appllama.io)
  version: 1.2.0
---

# Appllama App Design Skill

You are building screens that will sit on a phone next to the best-designed apps
in the world. The user will compare your output to those apps within seconds of
launching it. This skill defines the bar and the method for clearing it.

## The Prime Directive: study before you draw

Never design a screen from imagination when you can study how top apps solved
the same screen. Real, shipping, revenue-ranked apps encode thousands of hours
of design iteration and A/B testing. Your first move on any screen is research:

1. If the **Appllama MCP** is connected, pull real screens for the category and
   screen type you are building (see the `appllama-usage` skill for the exact
   research playbooks). Study 20–30 screens before writing a line of UI code.
2. Extract the **pattern, not the pixels**: layout skeleton, information
   hierarchy, control choices, spacing rhythm, where the primary CTA sits, what
   gets an illustration vs. plain text, how progress is communicated.
   Note: every Appllama image and video carries a small Appllama watermark in
   the top-left corner. It is provenance, not design — ignore it when reading
   a screen (it may sit over the status bar or a back button) and never
   reproduce it in anything you build.
3. Then design **your** screen: same proven skeleton, your product's voice.
   Copying a competitor's screen 1:1 is both lazy and legally risky; shipping a
   screen that ignores every convention users already know is worse.

## Platform baseline

Default stack assumptions (override only if the project already differs):

- **Expo + Expo Router** (native stack, `formSheet` routes, `NativeTabs`),
  React Native, TypeScript.
- `react-native-reanimated` 4 + `react-native-worklets` for motion,
  `react-native-gesture-handler` for gestures, `expo-haptics`,
  `react-native-keyboard-controller` for anything that tracks the keyboard,
  `@shopify/flash-list` (v2) for any list that can grow.
- `expo-image` for images (and SF Symbols via `source="sf:name"` on iOS),
  `expo-video` / `expo-audio` (never the deprecated `expo-av`).
- `react-native-safe-area-context` for insets. Never hard-code notch numbers.
- `process.env.EXPO_OS` over `Platform.OS` for compile-time platform checks.
- Install with `npx expo install <pkg>` so versions match the SDK; never
  hand-roll a component a listed package already solves
  ([references/native-controls.md](references/native-controls.md) has the picks).

## Native fidelity laws

These are the details that separate "web page in a wrapper" from "native app".
Violating any of them is a finding, not a style preference.

1. **Semantic colors, both themes, day one.** Use system/semantic color tokens
   (e.g. `Color` from `expo-router` on iOS: `Color.ios.label`,
   `Color.ios.secondarySystemBackground`; Material dynamic colors on Android).
   Every screen must render correctly in light AND dark before it is "done".
   Never pass semantic color objects into Reanimated animated styles — resolve
   to strings first.
2. **Native controls over rebuilt ones.** Switch, Slider, SegmentedControl,
   context menus, date pickers: use the native control or a faithful wrapper.
   A rebuilt toggle that animates 50 ms differently than iOS's reads as fake
   instantly.
3. **SF Symbols / Material Symbols for iconography.** On iOS prefer SF Symbols
   (`expo-image` with `sf:` sources, or `expo-symbols`); they inherit weight,
   optical size, and Dynamic Type behavior. Do not mix three icon families on
   one screen.
4. **Typography is hierarchy.** Use the platform type ramp (Large Title / Title
   / Headline / Body / Footnote on iOS). One display size per screen. Tabular
   numerals (`fontVariant: ['tabular-nums']`) for anything that counts, times,
   or prices. `Text selectable` on data users may want to copy.
5. **Continuous corners.** `borderCurve: 'continuous'` on every rounded
   rectangle. Squircles are the single cheapest "feels iOS" win that exists.
6. **Shadows via CSS `boxShadow`**, not legacy `shadow*`/`elevation` props.
   Shadows are for elevation logic, not decoration — one elevation system per
   app.
7. **Spacing rhythm.** Pick a base unit (4 or 8) and never leave it. Prefer
   flexbox `gap` over margin stacking. ScrollView padding goes in
   `contentContainerStyle`, never on the ScrollView itself.
8. **Safe areas and the Dynamic Island are part of the design.** Screens must
   be verified with content scrolled under the island / status bar (does the
   blur/fade treatment hold?), with the home indicator (does the bottom CTA
   clear it?), and in landscape if supported.
9. **Navigation titles belong to the navigator.** Use the stack's native title
   (and large-title collapse behavior on iOS) rather than a hand-rolled header
   whenever possible.
10. **Haptics are punctuation.** `selectionAsync` when a value ticks past a
    step, light impact when something snaps home or a drag commits,
    notification success/error for outcomes. Same frame as the visual, one
    per user action, never the only feedback — and never on scroll, never
    per frame, never on an entrance the user didn't cause.
11. **Format numbers like a product, not a database**: 1.4M, 38k, $4.99. Trim
    trailing zeros. Localize dates.
12. **Root scroll behavior**: screens that can ever overflow wrap content in a
    ScrollView (first component in the route) with
    `contentInsetAdjustmentBehavior="automatic"`. Use `useWindowDimensions`,
    never `Dimensions.get()`.

## Navigation laws

Navigation is the part of a screen you can't screenshot, and users feel it
within ten seconds. Every transition answers three questions: *what is the
destination to here* (deeper → push; a self-contained task → modal; a short
interruption → form sheet; must see through → overlay; a replacement of
where I am → replace), *must the user be able to come back*, and *what does
back do afterwards* — chevron, edge swipe, Android hardware back, active-tab
re-tap. The full method, verb by verb and case by case, is
[references/navigation.md](references/navigation.md). The laws:

1. **Push goes deeper, replace moves on.** `router.push` when the user will
   want to return here; `router.replace` / `<Redirect>` when coming back
   would land in a state the world has moved past. Don't rely on
   `navigate` to unwind — `dismissTo(href)` says "finish and land on X".
2. **Presentation is meaning.** A self-contained task with steps is a
   `modal` with its own stack and its own Cancel/Done; a short interruption
   is a `formSheet` with detents; immersive content is a `fullScreenModal`
   with an explicit Close; something that must sit on top of a visible
   screen is a `transparentModal` overlay; destructive confirms are action
   sheets; item actions are context menus. A sheet that grows a second step
   was a modal all along. If a link could open it, it is a route.
3. **One-way doors remove themselves from history.** Sign-in / sign-up,
   onboarding completion, a successful purchase, a finished session, an
   expired target: `Stack.Protected` guards and `replace` so back can never
   re-enter the old state — Android back from home exits the app, never
   shows Login; leaving a thank-you screen never re-opens the paywall.
4. **Back is blocked in exactly two cases** — an irreversible request in
   flight (seconds, with visible progress) and unsaved work in a modal
   (ask first) — both through `usePreventRemove`, never a bare
   `BackHandler`, never to keep someone in a funnel. Everywhere else the
   iOS edge swipe and Android hardware back work, always.
5. **Tabs are peers.** No slide, each tab keeps its stack, re-tapping the
   active tab pops to root (and, at the root, scrolls to top); full-attention screens
   (composer, player, checkout) live in the root stack *above* the tabs so
   the tab bar gets out of the way.
6. **Deep links land with a stack underneath** (`initialRouteName`,
   `withAnchor`) so back has somewhere to go; cold start lands by state.
7. **Study the grammar, not just the pixels.** When you walk a winning flow
   on Appllama, note what each step *is* — push, modal, sheet — and copy
   that consistency.

## Anti-slop laws

AI-built apps share a look, and users file it under "template" within seconds.
Each of these is a *default ban* — there is always an override when the brand
explicitly asks for the thing AND you can articulate why it fits this product.

1. **No AI-default styling.** Purple/indigo gradient CTAs with a glow,
   glassmorphism on every card, mesh-gradient heroes, confetti for minor
   events, sparkles in headings — that is the model's house style, not
   design. Your palette, materials, and layout come from the reference
   screens you studied, never from the priors you'd reach for unprompted.
2. **One accent, locked.** Pick one accent color and it is THE accent on
   every screen — no blue CTA on one screen and teal on the next, no new hue
   appearing in screen seven. Neutrals carry the app; the accent is spent
   where the money is (primary action, active state, progress).
3. **One grey family.** Warm greys or cool greys — never both in one app.
4. **Shape lock.** One corner-radius scale, stated as a rule ("actions are
   pills, cards 16, inputs 8") and never violated. Mixed radii without a
   stated rule read as assembled-from-parts.
5. **No emoji as iconography.** Icons are SF Symbols / Material Symbols
   (fidelity law 3). Emoji appear only when the product's voice is genuinely
   chat-native or playful — sparingly, in content, never in chrome.
6. **One label per intent.** "Get started", "Start now", and "Begin" are the
   same intent — pick one phrasing and use it everywhere it appears.
7. **Emphasis stays in the family.** Emphasize a word with weight or italic
   of the same typeface; injecting a serif word into a sans headline (or vice
   versa) for visual interest is amateur.
8. **Ship full state cycles, not the happy path.** Static-successful-state-
   only is the default failure mode: skeletons must match the final layout's
   shape, empty states are composed (and say how to fill them), errors are
   inline and specific.
9. **The slop pre-flight is mechanical.** Before any flow reaches the
   simulator pass, count: distinct accent hues (must be 1), distinct corner
   radii (all from the stated scale), emoji in UI chrome (0), gradients
   without a brand reason (0), duplicate labels for one intent (0). A failed
   count is a fix, not a judgment call.

## Motion laws

Motion is the highest-leverage polish surface and the easiest to overdo.
Decisions are made in order — the method, the exact values and the
implementation are in [references/motion.md](references/motion.md); the
physics of *feel* in [references/fluid-interfaces.md](references/fluid-interfaces.md).

- **The frequency gate comes first.** Something met 100+ times a day (tab
  switch, keyboard, scroll, back) gets nothing beyond the platform default;
  tens a day gets near-imperceptible (<150 ms); occasional (sheets, modals,
  toasts) gets standard motion; the delight budget is spent only on rare,
  first-time moments. **Tabs never slide. Screen transitions stay at the
  platform default.** Passing this gate with zero lines of code is a
  success.
- **Name the purpose in one word** — feedback, spatial consistency, state
  indication, preventing a jarring change, explanation, delight — or don't
  build it. Data the user is reading never moves for style.
- **Cheapest tool that works**: a Reanimated CSS transition for state
  changes, layout animations for mount/unmount/reflow, shared values +
  gestures only for what a finger touches or scroll drives, the native stack
  for screens, `formSheet` for sheets, `NativeTabs` for tabs, native menus
  for menus. `transform` and `opacity` only (an absolute, childless element
  may animate `width`); never `scale(0)`.
- **If a finger was involved, it's a spring** — from the live value, with the
  release velocity handed in, interruptible at any instant, rubber-banded at
  boundaries, committed by momentum (a flick is enough). One spring
  vocabulary per app: `SETTLE { duration: 400, dampingRatio: 1 }`,
  `SNAP { 400, 0.8 }`, `SHEET { 300, 0.8 }`; bounce only after momentum.
- **Everything else is timing, under 300 ms, strong ease-out** —
  `Easing.bezier(0.23, 1, 0.32, 1)` to enter/exit, `(0.77, 0, 0.175, 1)` to
  move on screen; never ease-in on UI. Press 100–150 ms at scale 0.97,
  feedback on press-*in*. Exits faster than entrances, along the same path.
- **Off the JS thread.** No `setState` in a gesture or scroll handler, no
  `scheduleOnRN`/`runOnJS` per frame, no shared-value reads in render, no
  `PanResponder`, no `entering` on recycled list rows, no animated `height`
  to collapse a header, no JS-rebuilt screen transition.
- **Haptics** follow fidelity law 10 — same frame, one per action, never
  alone.
- **Respect Reduce Motion** — fewer and gentler, not zero: spatial motion
  collapses to cross-fades, screen transitions to `fade`, feedback survives;
  and no animation targets a height measured at default text size.
- The bar: 60 fps on a mid-tier device, zero dropped frames through the hero
  flow — measured on a release build, not vibed in Expo Go
  ([references/performance.md](references/performance.md)). Ready-to-build
  recipes live in [references/motion-recipes.md](references/motion-recipes.md);
  reviewing and auditing motion — and deciding what *not* to animate — in
  [references/motion-review.md](references/motion-review.md).

## State architecture

Screens that feel great are screens whose state is boring:

- **Server state** in TanStack Query (or the project's equivalent): caching,
  retries, optimistic updates. Never `useEffect`+`fetch`.
- **Client state** in a small atomic store (Zustand/Jotai). Broad "app state"
  contexts cause the re-render cascades that make UIs feel heavy.
- **Ephemeral UI state** (open/closed, focus, scroll) stays local to the
  component.
- **Optimistic by default**: taps reflect instantly, reconcile in the
  background, roll back loudly on failure.
- Uncontrolled `TextInput`s for high-frequency typing surfaces; controlled
  inputs are a top-3 cause of typing jank.
- Persist tiny client state in MMKV, not AsyncStorage, when latency shows.

## Perceived performance

- Skeletons only for content whose shape you know; otherwise progressive
  reveal. Never a full-screen spinner for a partial update.
- FlashList for every list; give stable keys.
- Preload the next screen on press-in (`router.prefetch` / `<Link prefetch>`
  plus its data), not on navigation-complete.
- Images: right-size sources, `expo-image` with `recyclingKey` in lists,
  thumbhash/blurhash placeholders.
- Cold-start TTI and bundle discipline live in
  [references/performance.md](references/performance.md) — apply the
  measure → optimize → re-measure loop, never blind memoization.

## Image & illustration assets

When a screen calls for illustration, empty-state art, hero imagery, or icons
beyond the symbol set:

- Generate assets with the **best image model available to you** (e.g. an
  imagegen tool or the Higgsfield MCP/CLI if connected) at the **highest
  quality settings**, then downscale to @1x/@2x/@3x. Never upscale.
- One visual language per app: pick a style (gradient-mesh, flat-duotone,
  3D-clay, hand-drawn, mascot style) and generate ALL assets in that same style, same
  palette, same lighting. A mixed-style asset set reads as template slop.
- Prompt for **transparent or solid-flat backgrounds** matched to your surface
  color; composite artifacts (white halos, wrong-color mattes) are an
  automatic redo.
- Full asset pipeline and prompt patterns:
  [references/image-assets.md](references/image-assets.md).

## The simulator loop (non-negotiable)

A screen does not exist until you have seen it running. The loop:

1. Implement → launch in the iOS Simulator (or Android emulator).
2. Screenshot and **actually look**: alignment, optical centering, spacing
   rhythm, truncation with long content, dark mode, Dynamic Type at XL.
3. Run the **full-motion pass** below — screenshots prove layout; they prove
   nothing about motion.
4. Fix, relaunch, re-verify. Repeat until you cannot find a defect — then run
   the checklist in [references/simulator-loop.md](references/simulator-loop.md)
   once more.

Do not declare a screen finished from code review alone. Do not stop at "looks
fine" — stop at "cannot find a flaw at 100% zoom".

### The full-motion pass (mandatory, per flow)

Every flow is evaluated as **moving pictures in the simulator, never as
stills**. Screen-record the entire flow end to end
(`xcrun simctl io booted recordVideo flow.mov`), exercising ALL of it:

- every screen transition, push/pop, tab switch
- every modal and sheet: present, drag, dismiss — and cancel mid-drag
- every back path: chevron, edge swipe, Android hardware back, active-tab
  re-tap — and, after each one-way door (sign-in, onboarding done,
  purchase, finished session), an attempt to go back that must fail to
  re-enter the old state
- the keyboard, both directions: appear (does the layout glide, is the
  focused input visible?) and dismiss (does anything jump-cut?)
- every user interaction: press states, gesture follow-through, interrupted
  gestures, rapid taps, scroll flings at the extremes

Watch the recording **twice**: once at full speed for feel, once scrubbing
frame by frame. You are hunting:

- dropped or stuttered frames — the bar is a sustained **60 fps** through
  every transition, measured, not vibed
- one-frame flashes: white/unstyled first paint, wrong-theme frames mid-
  transition, color pops where a surface briefly renders the wrong token
- layout jumps, double-render pops, springs that clip or overshoot into
  content, elements that reflow after appearing

The whole recording must play like one native piece — smooth end to end,
zero UX glitches. One glitchy frame means the flow is not done.

## Definition of done, per screen

- [ ] Studied 10+ real reference screens for this screen type (via Appllama
      MCP when available) and can name the pattern you adopted
- [ ] Navigation answered: what this screen *is* (push / modal / sheet /
      overlay / replace), what back does from it on iOS and Android, and —
      if it sits behind a one-way door — that back cannot re-enter the old
      state; modals carry Cancel/Done; deep links land with a stack
- [ ] Light + dark mode verified in the simulator
- [ ] Safe areas / Dynamic Island / home indicator verified
- [ ] Long-content, empty, loading, and error states designed — not defaulted
- [ ] Motion passed the gate (frequency tier + named purpose for every
      animation; nothing slides between tabs; screen transitions native),
      uses the app's one spring/easing vocabulary, runs off the JS thread,
      and every gesture hands its velocity to a spring
- [ ] Motion: the full flow screen-recorded and scrubbed — entrances,
      presses, transitions, modals, keyboard — native feel, zero glitch or
      wrong-color frames; Reduce Motion respected; 60 fps measured on a
      release build on the target device profile
- [ ] Dynamic Type XL doesn't break layout; text is selectable where useful
- [ ] All tap targets ≥ 44pt; contrast passes in both themes
- [ ] Assets: single style family, crisp at @3x, no compositing halos
- [ ] List surfaces virtualized; no controlled-input jank; no re-render storms
      (profiled, not guessed)

## References

| File | Load when |
|---|---|
| [references/navigation.md](references/navigation.md) | Wiring any screen into a flow: push vs replace vs dismissTo, modal vs form sheet vs overlay, tabs, deep links, and the one-way doors where back must not exist — plus the back-stack audit |
| [references/native-controls.md](references/native-controls.md) | Choosing/wiring iOS+Android native controls, menus, pickers, sheets, forms; the library picks |
| [references/motion.md](references/motion.md) | Any motion: the decision sequence (frequency gate → purpose → tool → properties → spring/curve → thread), exact values, haptics, reduced motion, the never-ship list |
| [references/motion-recipes.md](references/motion-recipes.md) | Building a press, a drag-to-dismiss sheet, swipe-to-delete, a collapsing header, list entrances, keyboard-synced UI, a tab indicator, a toast, a threshold haptic |
| [references/fluid-interfaces.md](references/fluid-interfaces.md) | Anything a finger drags, anything translucent, anything that gives feedback: response, interruptibility, velocity hand-off, projection, rubber-banding, materials & depth, the design principles |
| [references/motion-review.md](references/motion-review.md) | Reviewing a diff's motion, auditing an app's motion into plans, or hunting for (and rejecting) places that could animate |
| [references/motion-vocabulary.md](references/motion-vocabulary.md) | Decoding a loose brief ("make it bouncy") or writing a motion spec with exact terms |
| [references/variant-lab.md](references/variant-lab.md) | An open brief or a hero screen where direction matters: build 3 divergent variants behind a dev-only switcher and let the user pick |
| [references/performance.md](references/performance.md) | Jank, slow TTI, big bundles, memory leaks, profiling method |
| [references/image-assets.md](references/image-assets.md) | Generating illustrations/icons/hero art with image models |
| [references/simulator-loop.md](references/simulator-loop.md) | Final verification checklist (layout, theming, motion, interaction, navigation & back stack, state) + device matrix |
