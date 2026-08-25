# Performance — measure, fix, re-measure

Perceived quality is half design, half frame rate. This reference is the
method; never optimize on vibes.

## The loop

1. **Measure** a baseline for the exact interaction that feels bad (FPS during
   the transition, TTI from cold start, commit counts during typing).
2. **Optimize** the one thing the measurement indicts.
3. **Re-measure** the same way.
4. **Validate** with a number: 45 → 60 fps, TTI 3.2 s → 1.8 s. No number, no
   claim.

Do not recommend `useMemo`/`useCallback`/`React.memo` without profiler
evidence of wasted renders. Do not flag "stale closure risk" without a repro.
Measure the target interaction, not tree depth.

## FPS & re-renders (highest impact)

- Long or growable list on ScrollView → replace with FlashList. This single
  swap fixes more RN jank than everything else combined.
- Re-render storms: profile with React DevTools; the classic causes are a
  broad context/store consumed by leaves, inline object/array props into
  memoized children, and parent state that should be local.
- Prefer atomic state (Zustand selectors, Jotai atoms) so a change re-renders
  only its consumers.
- React Compiler: enable once profiling shows cascading re-renders; it
  replaces most manual memoization. Watch for bailouts (mutations, non-plain
  patterns) — a bailed-out hot component silently loses the win.
- `useDeferredValue` for expensive derived UI (filter results, search
  highlighting) behind fast-changing inputs.

## Typing performance

Controlled TextInputs re-render the tree per keystroke. For search bars and
forms: uncontrolled inputs (`defaultValue` + `onChangeText` into a ref or
store), commit to state on submit/debounce.

## Startup / TTI

- Measure only cold starts, with `react-native-performance` markers.
- Native navigation (`react-native-screens`) enabled.
- Defer everything not needed for first paint: heavy SDK init, analytics,
  below-the-fold data.
- Hermes: check bundle compression guidance for your RN version (mmap).
- Inspect the bundle when it grows: `npx react-native bundle … --dev false`
  then `source-map-explorer`. Barrel imports (`import { x } from '@/components'`)
  are the classic silent bloat — import from the source file.

## Memory

- Symptoms: growing RSS while navigating back and forth. Hunt JS leaks with
  heap snapshots (retained listeners, intervals, subscriptions in effects
  missing cleanup) before blaming native.
- Lists: `recyclingKey` on `expo-image` items, no inline closures capturing
  huge parent scopes in `renderItem`.

## Animations

- Anything janky mid-gesture: confirm the animation runs as a worklet on the
  UI thread; a single `runOnJS` in `onChange` is enough to ruin it.
- Heavy screens committed during a transition stall the JS thread and hitch
  even UI-thread animations — defer the destination screen's expensive work
  until `InteractionManager.runAfterInteractions` / after the transition ends.

## Budgets to hold

| Metric | Budget |
|---|---|
| Transition/gesture FPS | 60 (no dropped frames in the hero flow) |
| Cold-start TTI (mid-tier device) | < 2 s |
| Keystroke → echo | < 50 ms |
| List scroll (FlashList) | blank-cell-free at fling speed |
| JS bundle (initial) | watch the trend; investigate any +10% jump |
