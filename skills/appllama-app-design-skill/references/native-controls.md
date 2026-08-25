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
| Contextual actions | Context menu (long-press/tap) | Popup menu | `zeego` (native menus on both) — never a JS dropdown for item actions |
| Destructive confirm | Action sheet | Bottom sheet / dialog | `ActionSheetIOS` via `@expo/react-native-action-sheet` |
| Bottom sheet content | Detented sheet | Bottom sheet | `@gorhom/bottom-sheet` (see notes) |
| Search | Nav-bar integrated search | SearchView | Expo Router `headerSearchBarOptions` |
| Pull to refresh | UIRefreshControl | SwipeRefreshLayout | `RefreshControl` on the ScrollView/list |
| Haptics | UIImpactFeedbackGenerator | Vibrator | `expo-haptics` |
| In-app browser | SFSafariViewController | Custom Tabs | `expo-web-browser` |

## Menus (zeego / native context menus)

Item-level actions (rename, share, delete) belong in a native context menu
anchored to the element, with SF Symbols on iOS:

```tsx
<ContextMenu.Root>
  <ContextMenu.Trigger>{card}</ContextMenu.Trigger>
  <ContextMenu.Content>
    <ContextMenu.Item key="share" onSelect={share}>
      <ContextMenu.ItemTitle>Share</ContextMenu.ItemTitle>
      <ContextMenu.ItemIcon ios={{ name: 'square.and.arrow.up' }} />
    </ContextMenu.Item>
    <ContextMenu.Item key="delete" destructive onSelect={confirmDelete}>
      <ContextMenu.ItemTitle>Delete</ContextMenu.ItemTitle>
      <ContextMenu.ItemIcon ios={{ name: 'trash' }} />
    </ContextMenu.Item>
  </ContextMenu.Content>
</ContextMenu.Root>
```

Destructive items: `destructive` role + a confirm step (action sheet), never a
bare tap-to-delete.

## Bottom sheets

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

## Navigation patterns

- Stack for drill-in, tabs for top-level destinations, modal for
  self-contained tasks. Do not put a back-navigable flow inside a modal deeper
  than 2 steps — use a stack inside the modal with its own header.
- iOS: swipe-back must always work (don't block the interactive pop gesture).
- Tab bars: 3–5 items, SF Symbols with the filled variant for the active tab,
  labels always on (icon-only tab bars fail recognition tests).
- Deep links: every screen reachable by URL via Expo Router's file routes.

## When you DO rebuild a control

Match the OS's numbers, not your instincts:
- iOS switch: thumb travel ~22 pt in ~0.2 s with a slight squish; haptic on
  toggle.
- Pressed states: opacity 0.4 for plain-text buttons, scale 0.97 + slight
  darken for filled buttons, spring back on release.
- Selection cells: checkmark animates in with a short fade+scale, row flashes
  the selection color for ~150 ms.
- Always add the platform haptic the real control would emit.
