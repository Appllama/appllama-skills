# Motion recipes — ready-to-build implementations

Start from the recipe, then adapt. Each one already makes the decisions in
[motion.md](motion.md) — frequency tier, purpose, tool, properties, spring
or curve, thread — so you inherit them instead of re-deriving them at 2 a.m.

## What every recipe assumes

```bash
npx expo install react-native-reanimated react-native-worklets react-native-gesture-handler expo-haptics
# + react-native-keyboard-controller for the keyboard recipe
```

`GestureHandlerRootView` wraps the app once (root `_layout`). Shared imports
and the app's single motion vocabulary:

```ts
import { useMemo, useState, useEffect } from 'react';
import Animated, {
  css, useSharedValue, useAnimatedStyle, useAnimatedScrollHandler, useAnimatedReaction,
  withSpring, withTiming, interpolate, Extrapolation, Easing,
  FadeInDown, FadeOutDown, LinearTransition,
} from 'react-native-reanimated';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import { scheduleOnRN } from 'react-native-worklets';
import * as Haptics from 'expo-haptics';
import { SETTLE, SNAP, SHEET, EASE_OUT, EASE_IN_OUT, EASE_SHEET, CSS_EASE_OUT } from '@/motion';  // see motion.md §5
```

Conventions, stated once: shared values are read/written with `.get()` /
`.set()`; `scheduleOnRN(fn, ...args)` replaces the deprecated `runOnJS`;
RNGH v2 gestures are wrapped in `useMemo` (v3's hooks — `usePanGesture({...})`
with `onActivate`/`onDeactivate` — manage their own identity, so drop the
memo there).

## Two worklets you will need everywhere

Momentum projection decides *where a flick was going*, so a short fast swipe
commits and a long slow one doesn't. Rubber-banding makes an edge resist
instead of stopping dead.

```ts
// Where the finger would come to rest if it kept decelerating —
// the exponential-decay form, not v²/2a.
export function project(velocity: number, decelerationRate = 0.998) {
  'worklet';
  return ((velocity / 1000) * decelerationRate) / (1 - decelerationRate);
}

// The further past the edge, the less the element follows.
export function rubberband(overshoot: number, dimension: number, constant = 0.55) {
  'worklet';
  return (overshoot * dimension * constant) / (dimension + constant * Math.abs(overshoot));
}
```

## Press feedback

Every pressable. It passes the frequency gate only because it is
near-imperceptible — 120 ms and 3% is the ceiling for something touched this
often. No gesture, no shared value: a CSS transition is the whole thing.

```tsx
import { Pressable } from 'react-native';

export function PressableScale({ onPress, children }) {
  const [pressed, setPressed] = useState(false);   // fires twice per press, not per frame — fine
  return (
    <Pressable onPress={onPress} onPressIn={() => setPressed(true)} onPressOut={() => setPressed(false)} hitSlop={12}>
      <Animated.View style={[styles.box, pressed && styles.pressed]}>{children}</Animated.View>
    </Pressable>
  );
}

// css.create (not StyleSheet.create) so the transition keys type-check;
// transitionTimingFunction takes a keyword or cubicBezier(), never a 'cubic-bezier(…)' string
const styles = css.create({
  box: {
    transform: [{ scale: 1 }],
    transitionProperty: 'transform',
    transitionDuration: '120ms',
    transitionTimingFunction: CSS_EASE_OUT,
  },
  pressed: { transform: [{ scale: 0.97 }] },
});
```

`hitSlop` brings a small icon up to the 44 pt target without growing it.
`Pressable` already tolerates a 20–30 pt finger drift before cancelling
(`pressRetentionOffset`); raise it for big, sloppy targets, never lower it.

## Bottom sheet you can drag to dismiss (in-screen)

First check [navigation.md](navigation.md): if the sheet is its own
destination, `presentation: 'formSheet'` gives you the real system sheet.
Build this only when the sheet belongs to the screen's own state.

```tsx
const translateY = useSharedValue(0);
const startY = useSharedValue(0);

const pan = useMemo(() => Gesture.Pan()
  .activeOffsetY([-10, 10])                       // let horizontal swipes win; require intent
  .onStart(() => { startY.set(translateY.get()); })   // continue from where the eye last saw it
  .onUpdate((e) => {
    const next = startY.get() + e.translationY;
    translateY.set(next >= 0 ? next : rubberband(next, HEIGHT));   // down is free; up resists
  })
  .onEnd((e) => {
    const projected = translateY.get() + project(e.velocityY);
    if (projected > HEIGHT * 0.4) {
      translateY.set(withSpring(HEIGHT, { ...SETTLE, duration: 300, velocity: e.velocityY, overshootClamping: true },
        (finished) => { if (finished) scheduleOnRN(onClose); }));
    } else {
      translateY.set(withSpring(0, { ...SHEET, velocity: e.velocityY }));
      scheduleOnRN(Haptics.impactAsync, Haptics.ImpactFeedbackStyle.Light);   // it snapped home
    }
  }), [onClose]);

const sheetStyle = useAnimatedStyle(() => ({ transform: [{ translateY: translateY.get() }] }));
const backdropStyle = useAnimatedStyle(() => ({
  opacity: interpolate(translateY.get(), [0, HEIGHT], [1, 0], Extrapolation.CLAMP),
}));
```

The four details that separate this from a bad drag: `onStart` captures the
current value (grabbing a sheet mid-animation must not teleport it);
**velocity decides, not distance** (a flick from a few pixels down
dismisses); the **velocity is handed to the spring** (no seam between finger
and animation — the single detail that most separates fluid from fine);
`overshootClamping` on dismissal so the sheet never springs past the screen
edge and flashes a gap. The backdrop derives from the same value, so it is
always in sync for free.

## Swipe to delete a row

Gesture-handler ships `ReanimatedSwipeable` for swipe-to-reveal actions
(thresholds, overshoot, open/close, on the UI thread) — use it when the row
reveals buttons. Build the gesture yourself only for swipe-to-commit with
momentum, like this:

```tsx
const x = useSharedValue(0);
const startX = useSharedValue(0);

const pan = useMemo(() => Gesture.Pan()
  .activeOffsetX([-10, 10])                 // declare the axis or it fights the vertical scroll
  .onStart(() => { startX.set(x.get()); })
  .onUpdate((e) => { x.set(Math.min(0, startX.get() + e.translationX)); })
  .onEnd((e) => {
    const projected = x.get() + project(e.velocityX);
    if (projected < -SWIPE_THRESHOLD) {
      x.set(withTiming(-WIDTH, { duration: 200, easing: EASE_OUT }, (f) => { if (f) scheduleOnRN(onDelete, id); }));
    } else {
      x.set(withSpring(0, { ...SETTLE, duration: 300, velocity: e.velocityX }));
    }
  }), [onDelete, id]);
```

Closing the gap is the list's job, not the row's:

```tsx
const ROW_CLOSE = LinearTransition.duration(200);   // module scope
<Animated.FlatList data={items} keyExtractor={(i) => i.id} itemLayoutAnimation={ROW_CLOSE} … />   // single-column lists only
```

`activeOffsetX` is the mobile-specific part: a pan inside a scroll view with
no axis declared steals vertical scrolls, and the bug looks like a scrolling
bug rather than a gesture bug.

## Collapsing header on scroll

```tsx
const scrollY = useSharedValue(0);
const onScroll = useAnimatedScrollHandler((e) => { scrollY.set(e.contentOffset.y); });

const titleStyle = useAnimatedStyle(() => ({
  opacity: interpolate(scrollY.get(), [0, 60], [1, 0], Extrapolation.CLAMP),
  transform: [{ translateY: interpolate(scrollY.get(), [0, 60], [0, -12], Extrapolation.CLAMP) }],
}));

<Animated.ScrollView onScroll={onScroll} scrollEventThrottle={16} />
```

Fixed-height container, translate the content inside, `overflow: 'hidden'`.
Never animate the header's `height` — that is a layout pass on everything
below it on every scroll frame, competing with the scroll itself. (And for
the standard iOS large title, `headerLargeTitleEnabled` — no worklet at all.)

## List entrances

```tsx
function Row({ item, index }) {
  const entering = useMemo(() => FadeInDown.duration(250).easing(EASE_OUT).delay(Math.min(index, 8) * 40), [index]);
  return <Animated.View entering={entering}>{/* … */}</Animated.View>;
}
```

Stagger 30–80 ms; cap the stagger around eight items. **Never on rows inside
a virtualized list** — they re-fire on recycle and the list flickers while
scrolling; animate the container once, or `itemLayoutAnimation` for reflow.

## Keyboard-synced UI

```bash
npx expo install react-native-keyboard-controller
```

```tsx
// root _layout, next to GestureHandlerRootView
<KeyboardProvider><Stack /></KeyboardProvider>

// the footer / composer bar
const { height } = useReanimatedKeyboardAnimation();   // 0 → -keyboardHeight, on the UI thread
const footerStyle = useAnimatedStyle(() => ({ transform: [{ translateY: height.get() }] }));
```

Never `Keyboard.addListener` + a timing animation: the keyboard rides a
private system curve, the event reaches JS after it has started moving, and
any duration you pick visibly lags or leads it. Drive the UI from the
keyboard's actual position, frame by frame.

## Tab / segmented indicator

Measure once with `onLayout`, then animate transforms.

```tsx
const [layouts, setLayouts] = useState<Record<string, { x: number; width: number }>>({});
const x = useSharedValue(0);
const w = useSharedValue(0);

useEffect(() => {
  const l = layouts[active]; if (!l) return;
  x.set(withTiming(l.x, { duration: 250, easing: EASE_IN_OUT }));
  w.set(withTiming(l.width, { duration: 250, easing: EASE_IN_OUT }));
}, [active, layouts]);

const pillStyle = useAnimatedStyle(() => ({ transform: [{ translateX: x.get() }], width: w.get() }));
```

This is the sanctioned `width` animation: the pill is absolute and childless,
so nothing else re-lays-out, and its corners survive (`scaleX` would smear
them). `EASE_IN_OUT` because it moves across the screen rather than entering.
`Haptics.selectionAsync()` on the press, not when the pill lands.

## Toast

```tsx
const TOAST_IN  = FadeInDown.duration(300).easing(EASE_OUT);    // module scope
const TOAST_OUT = FadeOutDown.duration(250).easing(EASE_OUT);

<Animated.View entering={TOAST_IN} exiting={TOAST_OUT}
  style={{ position: 'absolute', bottom: insets.bottom + 16, left: 16, right: 16 }} />
```

Uninvited UI is quicker and quieter than motion the user asked for; it
leaves the way it came, ~20% faster; and it sits above the home indicator
(safe-area insets, always). Before building one: a native toast/alert
library is usually the right answer — see the library picks in
[native-controls.md](native-controls.md).

## Fire once at a threshold

Detents, snaps, pull-to-refresh arming — never poll from JS and never
`scheduleOnRN` every frame:

```tsx
const armed = useSharedValue(false);
useAnimatedReaction(
  () => pullDistance.get() > REFRESH_THRESHOLD,
  (isArmed, wasArmed) => {
    // wasArmed is null on the first run — guard it, or a haptic fires on mount with no user action
    if (wasArmed !== null && isArmed !== wasArmed) {
      armed.set(isArmed);
      scheduleOnRN(Haptics.impactAsync, Haptics.ImpactFeedbackStyle.Light);
    }
  },
);
```

The comparison runs on the UI thread every frame; the JS call happens twice
per pull (arm, disarm). This is the pattern for every "do something when the
value reaches X".

## Screen transitions

Configured on the native stack, never rebuilt in JS — the native transition
keeps the interactive back gesture and matches every other app on the
device:

```tsx
<Stack screenOptions={{ animation: reduced ? 'fade' : 'default' }}>
  <Stack.Screen name="settings" options={{ animation: 'slide_from_right', animationMatchesGesture: true }} />
  <Stack.Screen name="compose"  options={{ presentation: 'modal' }} />
  <Stack.Screen name="filters"  options={{ presentation: 'formSheet', sheetAllowedDetents: 'fitToContents', sheetGrabberVisible: true }} />
</Stack>
```

Which presentation a screen gets — and when back must not exist — is
[navigation.md](navigation.md).
