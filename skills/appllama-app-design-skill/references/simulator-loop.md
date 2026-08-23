# The simulator loop — verification checklist & device matrix

A screen is finished when it survives this checklist on a real simulator, not
when the code compiles. Budget as many loop iterations as it takes; the goal
is "cannot find a flaw", not "looks fine".

## Loop mechanics

1. Launch on the iOS Simulator (primary) — `npx expo start` + `i`, or your
   dev build. Android emulator second.
2. Screenshot the screen (simctl: `xcrun simctl io booted screenshot s.png`,
   or your agent's screenshot tool). **Open the screenshot and study it** —
   do not trust memory of what you wrote.
3. Interact: tap every control, type overlong text, background/foreground the
   app, rotate if supported.
4. For motion: screen-record the ENTIRE flow
   (`xcrun simctl io booted recordVideo m.mov`) — not just the hero
   transition. Watch it at full speed for feel, then scrub frame by frame.
   Stills cannot catch a one-frame flash, a dropped spring, or a keyboard
   jump-cut; only the recording can.
5. Fix → relaunch → re-verify. Never batch more than a few fixes between
   looks; regressions hide in batches.

If a UI-testing tool (e.g. Maestro) is available, script the flow's happy
path once it stabilizes — taps, assertions, screenshots — so later changes
re-verify for free.

## Per-screen checklist

**Layout**
- [ ] Nothing clipped by the Dynamic Island / status bar; scrolled content
      passes *under* it with the intended fade/blur, not a hard edge
- [ ] Bottom CTA clears the home indicator (safe-area inset respected)
- [ ] Optical alignment: icons vs text baselines, centered things actually
      look centered (check at 2× zoom)
- [ ] Spacing rhythm consistent (no rogue 13px gaps in an 8pt system)
- [ ] Long text: 2× length titles truncate/wrap by design, not by accident
- [ ] Empty state, loading state, error state each verified by forcing them

**Theming & type**
- [ ] Dark mode AND light mode screenshots taken and inspected
- [ ] Dynamic Type at XL: no overlap, no clipped labels
- [ ] Contrast: secondary text still readable in both themes

**Motion (evaluated on the full-flow recording, never on stills)**
- [ ] Entrance plays once, correctly, on first mount (and NOT again on
      back-navigation)
- [ ] Gesture follows the finger 1:1; release springs with velocity;
      cancelling mid-gesture settles cleanly
- [ ] Frame-by-frame: no pop at animation start/end, no double-render flash,
      no one-frame white/wrong-theme/wrong-color frames during transitions
- [ ] Every modal/sheet cycle recorded: present, drag, dismiss, cancel —
      smooth in both directions
- [ ] Keyboard appear AND dismiss recorded: layout glides with it, focused
      input stays visible, nothing jump-cuts or reflows after settling
- [ ] Sustained 60 fps through every transition of the flow (measure with
      the perf monitor / Instruments, don't eyeball) — on a **release build**;
      Expo Go and a dev build hide exactly the jank you're hunting
- [ ] Gesture motion judged on a device (the slowest you support), not only
      the simulator: velocity hand-off, haptic timing and rubber-banding
      cannot be read from a recording alone
- [ ] Reduce Motion enabled → spatial animations become fades, screen
      transitions become `fade`, feedback (opacity/color) survives
- [ ] Dynamic Type at XL: no animation targets a height measured at default
      type; keyboard-synced UI still clears the focused input

**Interaction**
- [ ] Every tappable ≥ 44pt; press feedback on press-*in* (scale 0.97,
      100–150 ms); haptics where native controls would have them — same
      frame as the visual, never alone
- [ ] Keyboard: appears with the right type, doesn't cover the focused input,
      dismisses sensibly
- [ ] Gestures feel-checked by hand, not just watched: flick it, interrupt
      it mid-flight, reverse it, drag past the boundary (rubber-band, not a
      wall); release continues at the finger's speed
- [ ] Rapid double-taps don't double-navigate or double-submit

**Navigation & back stack** (the audit in
[navigation.md](navigation.md) — write the answer for every screen)
- [ ] Chevron, iOS edge swipe and Android hardware back all do the same
      thing: the previous screen in *this* hierarchy
- [ ] Every one-way door holds: after sign-in, onboarding, purchase, a
      finished session — back cannot re-enter the old state; Android back
      from home exits the app, never shows Login
- [ ] Every modal has Cancel/Done in its own header; swipe-down / back
      dismiss it; unsaved work asks before discarding
- [ ] Every sheet drags, tap-on-scrim dismisses, keyboard up → still
      dismissible; no sheet has grown a second step
- [ ] Tabs: no slide, each tab keeps its stack, re-tap pops to root (and,
      at the root, scrolls to top); full-attention screens (composer,
      player, checkout) cover the tab bar
- [ ] Cold start from a deep link / notification lands with a real stack
      underneath; kill → relaunch lands by state
- [ ] Only the two sanctioned cases block back (request in flight, dirty
      modal) — and both via `usePreventRemove`

**State**
- [ ] Background the app mid-flow → return: state intact
- [ ] Kill and relaunch: persisted state restores, ephemeral state resets
- [ ] Offline: actions queue or fail loudly — never silently

## Device matrix (minimum)

| Profile | Why |
|---|---|
| Latest iPhone Pro (Dynamic Island) | Primary design target |
| iPhone SE-class (small, no island) | Layout compression + button reachability |
| Latest Pixel (Android) | Material behaviors, back gesture, font metrics |
| One tablet/iPad IF the app claims support | Otherwise explicitly letterbox |

Run the full checklist on the primary; on the others, verify layout,
safe areas, and the hero flow.
