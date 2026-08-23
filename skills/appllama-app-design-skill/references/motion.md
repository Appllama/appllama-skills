# Motion — the decision sequence, the exact values, the Reanimated implementation

Motion is judged in the first ten seconds of using an app, and it is the
easiest surface to get wrong in both directions: too much of it and the app
feels like a toy; the wrong kind and it feels like a web page in a wrapper.
This reference turns a request for motion into an implementation that
survives a strict review on a real device. Decisions are made **in the order
below** — the first two steps exist to produce zero lines of code sometimes.

Mobile changes three things about animation, and everything here follows:

1. **There is no hover.** Every affordance the web puts in hover lives in
   press, position, or nowhere.
2. **There are two runtimes.** React renders and app logic run on the
   React Native runtime; Reanimated worklets run every frame on the UI
   runtime. Motion that touches the RN runtime stutters the moment the app
   does anything else. The whole craft is keeping motion on the UI runtime.
3. **The finger is on the element.** Gestures are the primary input, so
   interruptibility and velocity hand-off are the baseline, not polish.

## 1. Should it animate at all? (the frequency gate)

| How often a user meets it | Decision |
|---|---|
| 100+ times a day — tab switches, keyboard open/close, scrolling, toggles in settings, the back gesture | **No animation** beyond the platform default. Stop here. |
| Tens of times a day — press feedback, list navigation, row selection, pull to refresh | Near-imperceptible only: under 150 ms, or nothing |
| Occasional — sheets, modals, toasts, onboarding steps, filters | Standard animation |
| Rare / first time — success states, empty-state art, celebrations, the first run | The delight budget lives here, and only here |

**Tabs never slide.** Tabs are peers, not a hierarchy; a slide claims a depth
that isn't there and the user pays for it dozens of times a session.
**Screen transitions stay at the platform default** — never rebuilt in JS.
If the request fails this gate, say so and don't write it.

## 2. Name the purpose — in one word

**Feedback** (the interface heard you), **spatial consistency** (where it
came from / went), **state indication** (a change made legible),
**preventing a jarring change** (content that would teleport),
**explanation** (how a feature works — onboarding/marketing only),
**delight** (rare tier only). Can't name it? Don't build it. Data the user is
reading or acting on never moves for style.

## 3. Pick the tool — the cheapest that works

Walk down; stop at the first that fits.

| Need | Tool |
|---|---|
| A state-driven change with no gesture — press, toggle, color, a value flipping | **Reanimated CSS transition** (`transitionProperty` / `transitionDuration` / `transitionTimingFunction` in the style) |
| Loop, multi-stage, or plays on mount with no state | **Reanimated CSS animation** (`animationName` keyframes) |
| Mount / unmount / list reflow | **Layout animations** (`entering` / `exiting` / `layout` / `itemLayoutAnimation`) |
| Anything a finger touches; anything derived from scroll | **`useSharedValue` + `Gesture` + `useAnimatedStyle`** — the worklet path |
| Screen to screen | **Native stack `animation`** in Expo Router — see [navigation.md](navigation.md) |
| A sheet that is its own destination | **`presentation: 'formSheet'`** — the real system sheet, free |
| Tab bar | **`NativeTabs`** (`expo-router/unstable-native-tabs`, alpha since SDK 54) — the platform's bar, its transitions included |
| Context menu, press-and-hold preview | **`Link.Menu` / `Link.Preview`** (iOS, SDK 54+) or `@react-native-menu/menu` cross-platform — never rebuilt in JS |
| Header that collapses into a large title | **`headerLargeTitleEnabled`** on the native stack — not a scroll worklet |
| Pull to refresh | **`RefreshControl`** — hand-roll only for a signature interaction |
| UI that tracks the keyboard | **`react-native-keyboard-controller`** — the keyboard's real position, per frame, on the UI thread |
| Vector illustration, celebration, empty-state art | **Lottie** — illustration only, never UI state |
| A huge animated scene, freeform drawing | **`@shopify/react-native-skia`** |

Reach for a shared value only when the value is continuous or interruptible.
A press scale is a CSS transition; a drag is a shared value. Install with
`npx expo install <pkg>` so versions match the SDK:
`react-native-reanimated` + `react-native-worklets` (Reanimated 4 needs the
New Architecture), `react-native-gesture-handler`, `expo-haptics`,
`react-native-keyboard-controller`, `lottie-react-native`,
`@shopify/react-native-skia`.

## 4. Pick the properties

- **`transform` and `opacity` are free.** Everything else re-runs layout:
  `width`, `height`, `margin`, `padding`, `flex`, `top`, `left`, `gap` re-lay
  out the node *and its siblings* every frame.
- **The one exception**: an absolutely positioned element with no children
  (a tab pill, a progress fill). Out of flow, nothing else moves, and
  animating `width` keeps a corner radius that `scaleX` would smear.
- **Never `scale(0)`.** Enter from `scale(0.9–0.97)` + `opacity: 0`. Nothing
  real appears from nothing.
- **`transform` order matters** — `[{ translateY }, { scale }]`; reversed, the
  translate gets scaled too.
- **Android shadows are `elevation`**, and animating it re-renders the shadow
  each frame — fade a pre-shadowed layer instead. **Never animate `BlurView`
  intensity** — crossfade a static blur.
- **Percentages in `translate`** are relative to the element's own size:
  `translateY('100%')` hides a sheet whatever its height.

## 5. Spring or timing — and the exact values

**If a finger was involved, use a spring.** Springs carry velocity through
an interruption; timing restarts. Everything else uses timing.

Reanimated's spring takes the two designer parameters directly — use this
form, not mass/stiffness/damping, and define the vocabulary once per app:

```ts
// motion/springs.ts — one vocabulary, imported everywhere
export const SETTLE = { duration: 400, dampingRatio: 1 };            // default, no overshoot
export const SNAP   = { duration: 400, dampingRatio: 0.8 };          // reposition after a drag (+ velocity)
export const SHEET  = { duration: 300, dampingRatio: 0.8 };          // sheets, drawers (+ velocity)
// add overshootClamping: true when the element must not pass a hard edge
// `duration` here is PERCEPTUAL — Reanimated lets the spring micro-settle ~1.5× longer.
// The sub-300 ms rule below is about what the eye reads as finished, not the last sub-pixel.
```

Bounce **only when the gesture carried momentum** — a flicked card may
overshoot; a menu that faded in never does.

**Easing**, for everything without a finger on it. Built-in curves are too
weak; use these and nothing hand-rolled:

```ts
import { Easing, cubicBezier } from 'react-native-reanimated';
// for withTiming and layout-animation builders
export const EASE_OUT    = Easing.bezier(0.23, 1, 0.32, 1);    // entering / exiting — the default
export const EASE_IN_OUT = Easing.bezier(0.77, 0, 0.175, 1);  // moving or morphing on screen
export const EASE_SHEET  = Easing.bezier(0.32, 0.72, 0, 1);   // the iOS sheet curve
// CSS twins for transitionTimingFunction — it takes a keyword or cubicBezier(), NEVER a 'cubic-bezier(…)' string
export const CSS_EASE_OUT    = cubicBezier(0.23, 1, 0.32, 1);
export const CSS_EASE_IN_OUT = cubicBezier(0.77, 0, 0.175, 1);
```

| Situation | Easing |
|---|---|
| Entering or exiting | `EASE_OUT` |
| Moving / morphing on screen | `EASE_IN_OUT` |
| Constant motion (progress, marquee) | `Easing.linear` |
| Default | `EASE_OUT` |

**Never `ease-in` on UI.** It starts slow, delaying the exact moment the user
is watching.

| Element | Duration |
|---|---|
| Press feedback | 100–150 ms |
| Toggle, chip, small state change | 150–200 ms |
| Sheet, modal, drawer | spring, ~300 ms perceived |
| Toast | ≤ 300 ms in, ~20% faster out |
| Screen transition | the platform default (iOS push is 350 ms) — don't override |

UI animation stays **under 300 ms**. Match the platform for navigation, beat
it everywhere else. **Exits are faster than entrances** (~0.7–0.8×) and an
element **leaves the way it arrived** — in from the bottom, out through the
bottom.

## 6. Keep it off the JS thread

This is where most React Native motion dies.

- **Never `setState` from a gesture or scroll handler.** One React render per
  frame is the single biggest cause of RN jank. Shared value →
  `useAnimatedStyle`; React never re-renders.
- **Never schedule back to the RN runtime inside `onUpdate` or a scroll
  handler.** `scheduleOnRN(fn, ...args)` from `react-native-worklets`
  (the Reanimated 4 replacement for the deprecated `runOnJS`) belongs in
  `onEnd`, or in a `useAnimatedReaction` that fires when a value crosses a
  threshold.
- **Never read or write a shared value during render.** A read is a stale
  snapshot; a write replays on re-renders you didn't cause. Touch them only
  in worklets, handlers, and effects — and use **`.get()` / `.set()`**, the
  form the React Compiler can see through (`sv.set((v) => v + 1)` works).
- **Functions called from a worklet start with `'worklet'`** (or are
  auto-workletized by the Babel plugin) — otherwise the UI runtime throws
  "Tried to synchronously call a non-worklet function".
- **Gestures (RNGH v2 builder API) are wrapped in `useMemo`** — an
  unmemoized gesture forces a handler config update on every render (wasted
  work, stale closures); the docs recommend memoizing. (v3's hook API —
  `usePanGesture({...})` — manages its own identity; drop the memo there.)
- **Layout-animation builders live at module scope or in `useMemo`** — an
  inline `FadeInDown.duration(…)` chain in JSX rebuilds every render.

## 7. Press, not hover

- **Feedback on press-in, commit on press-out.** Showing nothing until the
  tap completes feels dead — this is the latency users actually perceive.
- **`scale: 0.97` in 100–150 ms** on any pressable (`Pressable` + a CSS
  transition). Scale takes label and icon with it, which is what reads as
  physical. Plain-text buttons: opacity 0.4 instead.
- **44×44 pt minimum target** (48 dp Android). Smaller visual → `hitSlop`,
  never a bigger visual. `Pressable`'s `pressRetentionOffset` already lets a
  finger drift 20–30 pt before cancelling — only ever raise it, never set it
  lower.
- **Android ripple only in a Material-styled app.** In a custom design, the
  same scale on both platforms is more coherent than a ripple on one.

## 8. Haptics

Mobile has a sense the web lacks. Sparingly it makes the app feel expensive;
everywhere and users turn it off.

| Moment | Call |
|---|---|
| A value ticks past a step — picker, slider detent, segmented control | `Haptics.selectionAsync()` |
| Something snaps home, a sheet detent catches, a drag commits | `Haptics.impactAsync(ImpactFeedbackStyle.Light)` |
| A heavy object lands, a destructive action fires | `Haptics.impactAsync(ImpactFeedbackStyle.Medium)` |
| Operation succeeded or failed | `Haptics.notificationAsync(NotificationFeedbackType.Success / Error)` |

Three absolute rules: **same frame as the visual** (fire at the causal
moment — the detent catching — not when the animation ends); **one per user
action** (never on scroll, never per frame, never on an entrance the user
didn't cause); **never the only feedback** (haptics are off system-wide for
many people and silent on most Android hardware). From a worklet:
`scheduleOnRN(Haptics.selectionAsync)`.

## 9. Reduced motion and text size

```ts
import { useReducedMotion, ReduceMotion, withSpring } from 'react-native-reanimated';
const reduced = useReducedMotion();
// per animation:
withSpring(0, { ...SHEET, reduceMotion: ReduceMotion.System });
// per screen transition: animation: reduced ? 'fade' : 'default'
```

Reduced motion means **fewer and gentler, not zero**: keep the opacity and
color changes that explain a state change; drop translation, scale, parallax,
overshoot. **Text scales** — `allowFontScaling` is on by default, so a
height measured at default type is wrong at 200%. Never animate to a
hard-coded height: measure with `onLayout` or animate a transform.

## Entrances, exits, layout

```tsx
// module scope
const ROW_IN   = FadeInDown.duration(220).easing(EASE_OUT);
const ROW_OUT  = FadeOut.duration(150);
const REFLOW   = LinearTransition.duration(200);

<Animated.View entering={ROW_IN} exiting={ROW_OUT} layout={REFLOW} />
```

- **Stagger 30–80 ms per item**, cap at ~8 items; beyond that enter as a
  block. Stagger is decorative — it never blocks interaction.
- **Never `entering` on a row inside FlashList / FlatList / any recycled
  list** — rows re-fire as they recycle and the list flickers while
  scrolling. Animate the container once, or use `itemLayoutAnimation` for
  reflow only.
- Entrances are for content the user asked for and is waiting on. A list they
  scroll past all day should already be there.
- `layout` transitions on containers whose children reorder or resize are
  what make filter chips, expanding cards and reordering lists feel
  expensive. `itemLayoutAnimation` on `Animated.FlatList` is single-column
  only.

## Scroll-linked effects

```ts
const y = useScrollOffset(scrollRef);   // Reanimated 4 name (useScrollViewOffset is deprecated); or useAnimatedScrollHandler
const titleStyle = useAnimatedStyle(() => ({
  opacity: interpolate(y.get(), [0, 60], [1, 0], Extrapolation.CLAMP),
  transform: [{ translateY: interpolate(y.get(), [0, 60], [0, -12], Extrapolation.CLAMP) }],
}));
```

- Large title collapses into the bar between ~0 and ~52 pt of scroll — but
  use `headerLargeTitleEnabled` rather than rebuilding it.
- **Never animate a header's `height` to collapse it** — that is a layout pass
  on the header and everything below it on every scroll frame, competing with
  the scroll itself. Fixed-height container, translate the content, clip with
  `overflow: 'hidden'`.
- `Extrapolation.CLAMP` is not optional; without it a long list drives the
  value past the range and the header reappears inverted.
- Content scrolling under a translucent bar gets a fade/blur mask, not a hard
  clip. Overscroll stretch on hero images: interpolate negative offsets into
  scale.

## Shared-element feel without shared elements

Keep the tapped thumbnail's position stable while the detail fades in over
it (`measure()` in a worklet); match corner radius and aspect ratio between
origin card and destination hero so the eye reads one object. True shared
elements are still niche; fake the continuity.

## Setup that silently breaks motion

- `GestureHandlerRootView` must wrap the app (root `_layout`) or gestures do
  nothing, with no error.
- In Expo, `babel-preset-expo` configures the worklets plugin — no manual
  Babel step. A missing plugin throws `Failed to create a worklet` at runtime.
- **Expo Go and the simulator are not performance environments.** Judge feel
  on a release build on the slowest device you support; a dev build's JS
  thread is slow enough to hide exactly what you're looking for.
- **120 fps**: on ProMotion iPhones third-party animation is capped at 60
  unless `CADisableMinimumFrameDurationOnPhone` is set — recent Expo SDKs set
  it in the prebuild template; confirm in the generated
  `ios/<App>/Info.plist` (or add it under `ios.infoPlist` in `app.json` if
  you override the plist). Then the frame budget is 8 ms.

## Never ship

| Never | Instead |
|---|---|
| `PanResponder` | `Gesture.Pan()` from gesture-handler |
| `setState` in a gesture or scroll handler | shared value + `useAnimatedStyle` |
| `runOnJS` (deprecated in Reanimated 4) | `scheduleOnRN` from `react-native-worklets` |
| `scheduleOnRN` per frame | `onEnd`, or `useAnimatedReaction` at a threshold |
| Reading / writing a shared value during render | `.get()` / `.set()` in worklets, handlers, effects |
| Core `Animated` for anything a finger touches | Reanimated |
| Animating `height` / `width` / `margin` / `flex` / `top` | `transform` + `opacity` (absolute, childless elements exempt) |
| Animating `BlurView` intensity or Android `elevation` | crossfade a static layer |
| `entering` on a virtualized list row | animate the container, or `itemLayoutAnimation` |
| A screen transition rebuilt in JS | native stack `animation` |
| Sliding between tabs | `animation: 'none'` |
| `Easing.in(...)` on a UI element | `EASE_OUT` |
| `scale(0)` entrance | `scale(0.95)` + `opacity: 0` |
| Distance-only dismissal threshold | velocity **or** distance — a flick is enough |
| Hard stop at a boundary | rubber-band resistance |
| A haptic per frame, or as the only feedback | one per commit, always with a visual |
| Exit slower than, or along a different path from, the entrance | faster, same path |
| Judging feel in Expo Go or the simulator | release build, slowest supported device |

## Where to go next

- Ready-to-build implementations (press, drag-to-dismiss sheet,
  swipe-to-delete, collapsing header, list entrances, keyboard-synced UI,
  tab indicator, toast, threshold reactions):
  [motion-recipes.md](motion-recipes.md).
- The physics of *feel* — response, interruptibility, velocity hand-off,
  momentum projection, rubber-banding, materials and depth:
  [fluid-interfaces.md](fluid-interfaces.md).
- Reviewing motion, auditing a codebase, and deciding what *not* to animate:
  [motion-review.md](motion-review.md).
- Naming motion precisely in specs and briefs:
  [motion-vocabulary.md](motion-vocabulary.md).
