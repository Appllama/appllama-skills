# Native controls — use the platform's, wire them right

A native-feeling app is mostly assembled from controls the OS already ships.
Rebuild a control only when the design genuinely diverges — and then match the
platform's timing and haptics so it still reads as native.

## Control selection table

| Need | iOS | Android | Package |
|---|---|---|---|
| Toggle | Switch | Material Switch | `react-native` `Switch` (renders native on both) |
| Single choice, 2–5 options | Segmented control | Tabs / segmented buttons | `@react-native-segmented-control/segmented-control` |
| Value in a range | Slider | Material Slider | `@react-native-community/slider` |
| Date / time | Wheel or inline calendar | Material pickers | `@react-native-community/datetimepicker` (`display="inline"` for calendars on iOS) |
| Contextual actions | Context menu (long-press/tap) | Popup menu | `Link.Menu` / `Link.Preview` (expo-router, iOS, SDK 54+); `@react-native-menu/menu` cross-platform — never a JS dropdown for item actions |
| Destructive confirm | Action sheet | Bottom sheet / dialog | `ActionSheetIOS` (built in) on iOS; `@expo/react-native-action-sheet` for a cross-platform wrapper |
| Bottom sheet content | Detented sheet | Bottom sheet | `@gorhom/bottom-sheet` (see notes) |
| Search | Nav-bar integrated search | SearchView | Expo Router `headerSearchBarOptions` |
| Pull to refresh | UIRefreshControl | SwipeRefreshLayout | `RefreshControl` on the ScrollView/list |
| Haptics | UIImpactFeedbackGenerator | Vibrator | `expo-haptics` |
| In-app browser | SFSafariViewController | Custom Tabs | `expo-web-browser` |

## Menus (native context menus)

Item-level actions (rename, share, delete) belong in a native context menu
anchored to the element, with SF Symbols on iOS — never a JS dropdown. When
the element links somewhere, Expo Router's `Link.Menu` (iOS, SDK 54+) gives
you the system menu and a press-and-hold preview for free:

```tsx
<Link href={`/post/${id}`}>
  <Link.Trigger>{card}</Link.Trigger>
  <Link.Preview />
  <Link.Menu>
    <Link.MenuAction title="Share" icon="square.and.arrow.up" onPress={share} />
    <Link.MenuAction title="Delete" icon="trash" destructive onPress={confirmDelete} />
  </Link.Menu>
</Link>
```

For a menu that must work on both platforms, or on an element that isn't a
link, use `@react-native-menu/menu` (`MenuView`, native on iOS and Android)
directly. `zeego` wraps it with a nicer API but has not shipped since early
2025 and needs its peers pinned forward on current SDKs — check before
adopting.

Destructive items: `destructive` role + a confirm step (action sheet), never a
bare tap-to-delete.

## Bottom sheets

First decide what the sheet *is* (see [navigation.md](navigation.md)): if it
is a destination — anything a link could open, a picker, a filter, options
for an item — it is a **route** with `presentation: 'formSheet'`, and the
platform's own sheet (detents, grabber, drag, scrim, keyboard) is free and
correct. Build an in-screen sheet (`@gorhom/bottom-sheet`) only when the
sheet belongs to the screen's own state and stays on screen: a map's place
card, a player's queue, a draggable panel.

- Detents should be content-derived (`enableDynamicSizing`) or the platform
  set (medium/large) — arbitrary 37%/63% detents feel arbitrary.
- The sheet's drag must hand off to inner scroll correctly: use the
  library's provided `BottomSheetScrollView`/`BottomSheetFlashList`, never a
  plain ScrollView inside.
- Backdrop: fade in with sheet position (`interpolate` on `animatedIndex`),
  tap-to-dismiss, and dim to the platform's standard (~40% black).
- Keyboard: `keyboardBlurBehavior="restore"`, and test with the keyboard up —
  half the bottom-sheet bugs in the wild are keyboard interactions.

## Forms and inputs

- Labels above fields, not placeholders-as-labels.
- `keyboardType`, `autoComplete`, `textContentType` on every input — enables
  autofill and the right keyboard. `textContentType="oneTimeCode"` for OTPs.
- Return-key chaining: `returnKeyType="next"` + focus the next field;
  final field submits.
- Validate on blur or submit, never on keystroke; errors appear beneath the
  field in the platform's error color and *stay* until fixed.
- Wrap forms in a keyboard-avoiding strategy you have actually tested
  (`react-native-keyboard-controller` is the current best answer).

## Navigation

Which verb (push / replace / dismissTo), which presentation (push, modal,
form sheet, full-screen modal, overlay), and where back must not exist are
decided in [navigation.md](navigation.md) — read it before wiring a screen
into a flow. The control-level rules that live here:

- Tab bars: 3–5 items, SF Symbols with the filled variant for the active tab,
  labels always on (icon-only tab bars fail recognition tests). Prefer
  `NativeTabs` for the platform's real bar (Liquid Glass, minimize on scroll,
  pop-to-top on re-tap).
- Every modal and full-screen modal carries its own Cancel/Done in its own
  header; swipe-down and hardware back are extras, never the only exit.
- iOS swipe-back always works; Android hardware back always does what the
  chevron does — the two exceptions (request in flight, unsaved work) go
  through `usePreventRemove`, never a bare `BackHandler`.
- Deep links: every screen reachable by URL via Expo Router's file routes,
  with a real stack underneath it (`initialRouteName`, `withAnchor`).

## Library picks — don't hand-roll a solved component

Check what's installed first and use it. When a need below appears, reach
for the listed package instead of rebuilding it; a hand-rolled toast or
dropdown is how an app ends up with a `<View>` that has no focus management,
no haptics, and no platform behaviour. If the project already uses a
competitor, flag it and don't churn the dependency without being asked.

| Need | Package |
|---|---|
| Motion | `react-native-reanimated` (+ `react-native-worklets`) |
| Gestures | `react-native-gesture-handler` (`ReanimatedSwipeable` for swipe-to-reveal rows) |
| Navigation, sheets, native tabs, menus, link previews | `expo-router` (`formSheet`, `NativeTabs`, `Link.Menu` / `Link.Preview`) |
| Context / dropdown menus | `Link.Menu` / `Link.Preview` (iOS, SDK 54+); `@react-native-menu/menu` for both platforms (see Menus above) |
| In-screen bottom sheet | `@gorhom/bottom-sheet` |
| Keyboard-following UI | `react-native-keyboard-controller` |
| Lists that grow | `@shopify/flash-list` (v2) — never a `ScrollView` + `.map` |
| Images | `expo-image` (placeholders, `recyclingKey`, SF Symbol sources) |
| Icons | `expo-symbols` / `expo-image` `sf:` sources on iOS; Material Symbols on Android |
| Haptics | `expo-haptics` |
| Toasts | `sonner-native` (active; Reanimated 4 + RNGH peers) — never a custom absolute `<View>`. `burnt` gives true native toasts but its last release predates the SDK 56 iOS-16.4 floor; only with its podspec patched |
| Native toggles, sliders, segmented controls, date pickers | `react-native` `Switch`, `@react-native-community/slider`, `@react-native-community/datetimepicker`; `@react-native-segmented-control/segmented-control` (still SDK-bundled, low activity — prefer the platform control via `@expo/ui` where available) |
| Action sheets | `ActionSheetIOS` (built in) on iOS; `@expo/react-native-action-sheet` cross-platform (no release since early 2025, known iOS 26 layout issues — verify) |
| OTP inputs | `input-otp-native` (or `textContentType="oneTimeCode"` on a plain input) |
| Forms | `react-hook-form` + `zod` |
| Server state | `@tanstack/react-query` |
| Client state | `zustand` (atomic selectors) |
| Fast key-value persistence | `react-native-mmkv` (v4 also needs `react-native-nitro-modules`; New Architecture only) |
| Blur / materials | `expo-blur` (static layers; crossfade, never animate intensity) |
| Vector illustration, celebration | `lottie-react-native` |
| Custom drawing, huge animated scenes | `@shopify/react-native-skia` |
| SVG | `react-native-svg` |
| In-app browser | `expo-web-browser` |

Install with `npx expo install <pkg>` so versions match the SDK.

## When you DO rebuild a control

Match the OS's numbers, not your instincts:
- iOS switch: thumb travel ~22 pt in ~0.2 s with a slight squish; haptic on
  toggle.
- Pressed states: opacity 0.4 for plain-text buttons, scale 0.97 + slight
  darken for filled buttons, spring back on release.
- Selection cells: checkmark animates in with a short fade+scale, row flashes
  the selection color for ~150 ms.
- Always add the platform haptic the real control would emit.
