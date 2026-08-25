# Motion — Reanimated patterns that read as native

Motion quality is judged in the first 10 seconds of using an app. This file is
the working reference for gesture-driven and system-driven animation.

## The two families of motion

| Family | Driver | Curve | Examples |
|---|---|---|---|
| **Responsive** (user is touching it) | Gesture position/velocity | Spring, seeded with gesture velocity | Sheet drag, swipe-to-dismiss, pull-to-refresh, card pan |
| **Narrative** (system initiated) | Time | `withTiming`, ease-out, 150–350 ms | Screen entrances, fades, reveals, toasts |

Mixing them up is the #1 tell of non-native motion: a sheet that closes on a
fixed 300 ms timing after a fling feels dead; a button that springs for 800 ms
on tap feels like a toy.

## Springs

```ts
// The designer form (duration is perceptual): critically damped, no oscillation
const SNAP = { duration: 400, dampingRatio: 1 };
// Playful: one soft overshoot — use sparingly (celebrations, mascots)
const POP = { duration: 400, dampingRatio: 0.8 };

offset.set(withSpring(dest, { ...SNAP, velocity: event.velocityY }));
```

- Always pass the **gesture velocity** into the spring when a gesture releases.
- One spring vocabulary per app. Define `SNAP`/`POP` once, import everywhere.

## Gesture → animation, all on the UI thread

```ts
const pan = Gesture.Pan()
  .onChange((e) => { offset.set(offset.get() + e.changeY); })   // worklet
  .onEnd((e) => {
    const dismiss = offset.get() > H * 0.3 || e.velocityY > 800;
    offset.set(withSpring(dismiss ? H : 0, { ...SNAP, velocity: e.velocityY }));
    if (dismiss) scheduleOnRN(onClose);   // react-native-worklets; replaces the deprecated runOnJS
  });
```

Rules:
- Never read/write React state inside `onChange`. `scheduleOnRN` only at
  gesture end, for navigation/effects — and read/write shared values with
  `.get()`/`.set()`, never during render.
- Thresholds combine **distance OR velocity** — a fast flick from 10 px away
  must dismiss.
- Interruptible: starting a new gesture mid-spring must grab the current
  animated value, not the destination.

## Entrances, exits, layout

```tsx
<Animated.View
  entering={FadeInDown.duration(220).springify().damping(30)}
  exiting={FadeOut.duration(150)}
  layout={LinearTransition.springify().damping(30)}
/>
```

- Stagger list entrances by index (`delay(index * 40)`), cap the stagger at
  ~8 items — beyond that, enter as a block.
- Exits are always faster than entrances (~0.7×).
- `layout` transitions on containers whose children reorder/resize — this is
  what makes filter chips, expanding cards, and reordering lists feel expensive.

## Shared-element feel without shared elements

True shared-element transitions are still niche; fake the continuity:
- Keep the tapped thumbnail's position stable while the detail screen fades in
  over it (measure with `measure()` in a worklet).
- Match corner radius and aspect ratio between the origin card and the
  destination hero so the eye reads them as the same object.

## Scroll-linked effects

```ts
const y = useScrollViewOffset(scrollRef);
const headerStyle = useAnimatedStyle(() => ({
  opacity: interpolate(y.value, [0, 64], [0, 1], Extrapolation.CLAMP),
  transform: [{ translateY: interpolate(y.value, [-100, 0], [-50, 0], Extrapolation.CLAMP) }],
}));
```

Standard native behaviors worth reproducing exactly:
- Large title collapses into the nav bar between ~0 and ~52 pt of scroll.
- Content scrolling under a translucent bar gets a fade/blur mask, not a hard
  clip.
- Overscroll stretch on hero images (`interpolate` negative offsets into scale).

## Reduce Motion

```ts
const reduceMotion = useReducedMotion(); // react-native-reanimated
// spatial → opacity-only
entering={reduceMotion ? FadeIn.duration(150) : SlideInDown.springify()}
```

Every spatial animation needs its cross-fade fallback. This is an accessibility
requirement, not a nice-to-have.

## Performance guardrails

- Animate only `transform` and `opacity` where possible; animating layout
  props (width/height/padding) forces layout every frame.
- One `useAnimatedStyle` per animated node; don't share a giant style across
  20 list items.
- Never allocate inside a worklet's hot path (no `.map`, no object spread per
  frame).
- If a transition stutters: profile first (see performance.md) — the usual
  culprits are a JS-thread stall from a heavy render committed mid-animation,
  or an image decode on the UI thread.
