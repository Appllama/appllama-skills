# Navigation — push, replace, sheets, overlays, and the doors that only open one way

A screenshot cannot show navigation, and users feel it inside ten seconds:
the modal that has no Close, the back swipe that lands on a login screen
they already passed, the detail page that stacks twice, the paywall that
re-opens when you leave the thank-you screen. This reference is the method
for getting every transition right. Read it before wiring any screen into a
flow, and again before the simulator pass.

Every transition answers three questions, in this order:

1. **What is the destination to here?** Deeper in the same hierarchy
   (*push*), a parallel self-contained task (*modal / sheet*), something that
   must sit on top of this screen while it stays visible (*overlay*), or a
   replacement of where the user is (*replace*)?
2. **Must the user be able to come back here?** If coming back would return
   them to a state the world has moved past (logged out, unpaid, unfinished
   session), the answer is no — and the transition has to make "no" true.
3. **What does back do afterwards** — the chevron, the iOS edge swipe, the
   Android hardware / predictive back, a re-tap on the active tab?

If you cannot answer all three for a screen, the screen is not designed yet.
The stack assumed below is Expo Router on the native stack
(`react-native-screens`); the reasoning holds for any navigator.

## The verbs

| Intent | Verb | Back afterwards |
|---|---|---|
| Go deeper; the user will want to return here | `router.push(href)` · `<Link href push>` | returns here |
| Swap where I am; never return here | `router.replace(href)` · `<Link href replace>` · `<Redirect href />` in render | returns to whatever was *below* this screen |
| "Go to X" from a place with no hierarchy (tab bar, deep link handler) | `router.navigate(href)` — Expo Router's general "go to": switches tabs when X lives in another tab, otherwise pushes. Its unwind-to-an-existing-copy behaviour has changed between versions — don't lean on it; say what you mean with `push` (go deeper) or `dismissTo` (go back to) | depends |
| Undo one step of navigation | `router.back()` (`router.canGoBack()` guard on web) | — |
| Leave this stack step / close this modal | `router.dismiss()` — pops the nearest stack; `router.dismiss(n)` pops n | — |
| Done with a flow; land on a known screen | `router.dismissTo(href)` — pops until `href` is reached, **replaces** with it if it was never in the stack. The verb for "finish and land on the list" | as if the flow never happened |
| Back to the first screen of this stack | `router.dismissAll()` — popToTop of the *nearest* stack. Inside a modal that owns its own Stack this lands on the modal's first step, not outside it; a multi-step modal's Cancel is `router.dismissTo('/(tabs)')` (or `router.dismiss()` from step one) | — |
| Warm the next screen before the tap lands | `<Link href prefetch>` prefetches as soon as the link renders on a focused screen; `router.prefetch(href)` when you want press-in / scrolled-into-view timing | — |
| Filter / sort / tab-within-screen changes | `router.setParams({...})` — the URL updates, history does not grow | unchanged |

Two flags that change what a verb *feels* like:

- `animationTypeForReplace: 'pop'` on the destination when a replace semantically goes *backwards* (sign-out, "return to start") — the screen slides out the way a pop would, instead of arriving like a push.
- `<Stack.Screen name="user/[id]" dangerouslySingular />` (or `router.push(href, { dangerouslySingular: true })`) so pushing the same profile twice collapses to one instance — the earlier copy is removed and the screen re-pushed on top (`getId` still works but has logged a deprecation since SDK 53). For screens where duplicates are legitimate (user → user → user) leave it off and let push stack.

## What the screen *is* — presentation

Pick the presentation from what the destination is, never from "what looks
nice". Users read presentation as meaning: a push says *you went deeper*, a
sheet says *short interruption, the world is still behind it*, a full-screen
modal says *focus on this one task*.

| The destination is… | Present as | Expo Router | It dismisses by |
|---|---|---|---|
| Deeper in the same hierarchy — list → detail → sub-detail, settings → section | **Push** (`presentation: 'card'`, the default) | `router.push`, `Stack.Screen` default | chevron, iOS edge swipe, Android back. Never disable the edge swipe here |
| A self-contained task with its own steps that the user starts and finishes — compose a post, create an item, edit a profile, add a card | **Modal** with a stack inside | `presentation: 'modal'` on a route whose own `_layout` is a `Stack` | explicit **Cancel/Done** in its own header (always); iOS swipe-down from any step — it dismisses the *whole* modal, so guard it with `usePreventRemove` on the modal's root screen when dirty; Android back pops the inner stack, then closes the modal. Confirm before discarding unsaved work |
| A short interruption — pick one value, set a filter, pick a board or an in-app recipient to send to, quick-add, see options for an item | **Form sheet** with detents | `presentation: 'formSheet'`, `sheetAllowedDetents: 'fitToContents'` or `[0.5, 1]`, `sheetGrabberVisible: true` | drag down, tap the scrim, Android back. Single screen only — a sheet that grows a stack was a modal all along |
| Immersive content — camera, video, full-screen photo viewer, markup, a game level | **Full-screen modal** | `presentation: 'fullScreenModal'` | an explicit Close/X (mandatory — there is no swipe-down on a full-screen modal), Android back |
| Something that must sit *on top* while this screen stays visible — a custom confirm card, a lightbox, a coach-mark, "Saved to board" | **Overlay** | `presentation: 'transparentModal'`, `animation: 'fade'`, `contentStyle: { backgroundColor: 'transparent' }`; you draw the scrim. `containedTransparentModal` when it must stay inside a nested navigator (under the tab bar) | tap outside, its own button, Android back |
| A destructive or consequential choice — delete, leave, discard, sign out | **Action sheet / native alert** | `@expo/react-native-action-sheet` (iOS action sheet, Android dialog) or `Alert` | Cancel. Never a routed screen |
| Actions on one item — rename, share, move, delete | **Context menu** | `Link.Menu` (iOS only, SDK 54+, attached to a `Link.Preview`) or `@react-native-menu/menu` (native on both platforms) | tap outside |
| A task the OS already owns — share outside the app, open a web page, pick a photo or file, compose mail, buy, rate the app, sign in with a web provider | **System controller** — not a route | `Share.share({ message, url })` (`expo-sharing` `shareAsync` for a local file); `expo-web-browser` `openBrowserAsync` (`openAuthSessionAsync` for OAuth); `expo-image-picker` / `expo-document-picker`; `expo-mail-composer`; the store's own purchase sheet via your IAP SDK; `expo-store-review` `requestReview()` — the OS decides whether it shows, so never from a button and never gated on | its own chrome (Done / Cancel / X) and its own back — never rebuilt, never a `formSheet` / `modal` / WebView route, never `Linking.openURL` to Safari for a page the user reads and returns from. An in-app share picker (friends, chats, boards — the Instagram / TikTok / Spotify pattern) is a form sheet only when the app has recipients of its own and the winners you studied do it; its "More…" still ends in the system share sheet |
| Top-level destinations — 3–5 peers | **Tabs** | `NativeTabs` (`expo-router/unstable-native-tabs`) or `Tabs` | n/a — tabs are not a stack |
| A gate — auth wall, onboarding, forced update, splash | **Replace / guard** | `Stack.Protected guard={…}`, `<Redirect>`, `router.replace` | there is no back — see one-way doors |
| Sheet content that belongs to the *screen's own state* — a map's place card, a player's queue, a draggable panel that stays on screen | **In-screen sheet** (component, not a route) | `@gorhom/bottom-sheet` (see native-controls.md) | drag, scrim, Android back — wire it (see *Platform back*) |

Rules that fall out of the table:

- **If it has a URL, it is a route.** Anything a deep link or a notification
  could open (a sheet that shows a particular item, a filter state) must be a
  route with a presentation, not a component toggled by local state. Routes
  survive deep links, reloads, and back correctly; `useState` sheets do not.
- **A modal holds a stack; a sheet does not.** Multi-step inside a sheet is a
  smell — promote it to `modal`. (Android form sheets cannot host native
  headers or nested stacks anyway.)
- **Every modal and full-screen modal has a visible way out** in its own
  chrome — Cancel/Close top-left, Done/Save top-right. Swipe-down and
  hardware back are extra, never the only exit.
- **One modal at a time.** Don't present a modal over a modal; an action
  sheet or alert over a modal is fine.
- **Study the presentation, not just the pixels.** When you walk a flow on
  Appllama (`list_app_screens(flow=…)`, and the videos), note what each step
  *is*: the composer is a full modal, the filter is a sheet, the detail is a
  push. Winners are consistent about this; copy the grammar.

```tsx
// app/_layout.tsx — presentation lives in the navigator, once.
// No Reduce-Motion `animation` branch: the native stack honours the system setting itself (motion.md §9).
<Stack>
  <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
  <Stack.Screen name="post/[id]" />                                   {/* push */}
  <Stack.Screen name="compose" options={{ presentation: 'modal', headerShown: false }} />
  <Stack.Screen name="filters" options={{
    presentation: 'formSheet',
    sheetAllowedDetents: [0.5, 1],
    sheetGrabberVisible: true,
    sheetCornerRadius: 24,
  }} />
  <Stack.Screen name="viewer/[id]" options={{ presentation: 'fullScreenModal', headerShown: false }} />
  <Stack.Screen name="confirm-leave" options={{
    presentation: 'transparentModal',
    animation: 'fade',
    headerShown: false,
    contentStyle: { backgroundColor: 'transparent' },
  }} />
</Stack>
```

Form-sheet platform facts: Android caps detents at three and shows no
grabber; `fitToContents` needs explicitly sized content (a `flex: 1` root has
no intrinsic height); `sheetLargestUndimmedDetentIndex` lets a half sheet
leave the screen behind it interactive (maps, players). On iOS 26 (SDK 55+)
a form sheet defaults to a transparent background so the system glass shows
at partial detents — keep the sheet's root transparent rather than painting
it.

## One-way doors — when back must not go back

Back undoes *navigation*. It must never undo *events*. After the world has
moved — a session started, money moved, a flow completed — the screens that
belonged to the old world leave the stack. This is not "trapping" the user;
it is the opposite: they can never land somewhere that no longer makes sense.

| Moment | What happens to the stack | How |
|---|---|---|
| **Sign-in demanded by one action** — save, follow, comment, buy, a Pro feature — **in an app you can browse signed out** | the user keeps their place: the auth screen is a parenthesis over the current screen, not a door. On success it vanishes and the action completes where it was tapped; on cancel they are exactly where they were | present the sign-in route as `presentation: 'modal'` (or `formSheet`) *over* the screen, carrying the intent (`/sign-in?returnTo=…&action=save&id=…`); on success flip `isSignedIn` — the `guard={!isSignedIn}` around the auth group removes the sheet (or `router.dismiss()` if that route is unguarded) — then run the pending action in place. Never `replace('/(tabs)')` here: it throws away the product page and the intent. Don't wrap the browsable group in `guard={isSignedIn}`; guard (or inline-prompt) only the screens that mean nothing signed out — a Profile tab says "Sign in to see your profile" inside the tab, not a redirect to a wall |
| **"Skip" / "Continue without an account"** from onboarding or the welcome screen | onboarding is gone exactly as if they had signed in; relaunch lands in the app as a guest, never back in onboarding or on a wall | persist `onboardingCompletedAt` (and guest state) and `router.replace` to the same destination Continue reaches; sign-in stays available later through the row above |
| **Sign in / sign up succeeds — wall apps** (the product is unusable signed out; auth is the first screen) | the auth screens vanish from history; the app's root is the new bottom; Android back from home exits the app — it never shows Login again | `Stack.Protected guard={isSignedIn}` around the app group and `guard={!isSignedIn}` around the auth group — flipping the guard removes the guarded screens. Land explicitly with `router.replace(pendingHref ?? '/(tabs)')` and clear `pendingHref`: Protected routes send a blocked deep link to the anchor and forget it, so if the link must be honoured, record the intended href yourself (the launch URL from `expo-linking`) and replay it once after sign-in |
| **Sign out** | the app screens vanish; the auth root is the bottom | flip the guard; give the auth screen `animationTypeForReplace: 'pop'` so the swap reads as leaving, not arriving |
| **Onboarding completed** | onboarding is gone; relaunch never shows it | `router.replace('/(tabs)')`, persist `onboardingCompletedAt`, and let the root `_layout` `<Redirect>` on state. Inside onboarding, back *between steps* stays enabled — edits are cheap and state carries forward |
| **Purchase / payment / submit in flight** | for the seconds *your* request is irreversible, nothing leaves the screen (the store's own purchase sheet is a system modal — you freeze only your post-purchase confirm, never the app behind StoreKit / Play Billing) | `usePreventRemove(inFlight, () => {})` + `gestureEnabled: false` + `headerBackVisible: false` + disabled buttons + visible progress. Seconds, not minutes: if it can take longer, let them leave and notify |
| **Subscription bought from inside the app** — a paywall opened from a feature gate ("Export PDF" → Pro) or from Settings → Upgrade | the paywall leaves the stack; the user is exactly where they were, now unlocked — never thrown to a tab root | the paywall is a `modal` / `fullScreenModal` over the current screen and keeps its Close (the door is one-way only *after* the purchase). On success: confirm in place (a short in-modal "you're in" beat, `notificationAsync(Success)`), then `router.dismiss()` (`back()` if it was pushed from a Settings row) and the gated action proceeds where it was. No success route, no `dismissTo('/(tabs)')` |
| **Subscription bought — or declined — at the end of onboarding** | onboarding and the paywall both leave history whether or not they paid; relaunch shows neither | the paywall is the last onboarding step: onboarding `replace`s into it, and Close and the purchase CTA both end in `router.replace('/(tabs)')` with `onboardingCompletedAt` persisted (the onboarding row above) |
| **Checkout — a one-off order placed** | the checkout cannot be re-entered by going back | `router.replace('/checkout/success')` from the checkout (never push); the success CTA does `router.dismissTo(origin)` — the cart's parent or `/(tabs)` — and "View order" is a `push` from there |
| **Session finished** — workout done, quiz submitted | the live session screen is gone; back from the summary goes *home*, not into the finished session | summary `replace`s the session route; "Done" → `dismissTo` the origin |
| **Expired / deleted / unauthorized target** — from a link, a notification, a stale list | the user lands on the parent with an inline notice, never on an error screen they can "go back" from | `<Redirect href="/items" />` (or `replace`) + a toast/inline message |
| **Cold start from a deep link / notification** | back has a real screen underneath | `export const unstable_settings = { initialRouteName: 'index' }` (`anchor: 'index'` is the equivalent key on newer SDKs and takes precedence) in the nested stack's `_layout`; `<Link withAnchor>` when linking into a stack from outside it. Never open a detail with nothing behind it |
| **Link / notification tapped while signed out** | the auth door still closes behind them — and the intent survives it: after sign-in they land on the target (with its stack underneath), never on home | `Stack.Protected` bounces a guarded href to the anchor and never replays it, so capture it yourself: when the guard is down, stash the initial URL (`Linking.useLinkingURL()`) or the tapped notification's href (`useLastNotificationResponse()`) as `pendingHref`; once signed in, `router.replace(pendingHref ?? '/(tabs)')` and clear it. A stale target still obeys the expired-target row |
| **Notification / link tapped while the app is warm** — on another screen, another tab | the user's place is kept; the target lands *on top* of it, once; back returns to where they were | in the *response* listener (`addNotificationResponseReceivedListener` / `useLastNotificationResponse` — never `addNotificationReceivedListener`, which is arrival) `router.push(href, { dangerouslySingular: true })`, or `router.navigate(href)` when the target lives in another tab; never `replace` / `dismissAll` on a tap. A notification that *arrives* in the foreground is a banner (`setNotificationHandler` → `shouldShowBanner: true`), not a navigation — navigate only when it is tapped. If a modal is open, don't push over it: dismiss a clean one (`dismissTo` the origin) or run the dirty-work prompt first, then push |
| **Splash / boot gate** | the gate is never in history — and the first painted frame is already the right screen | `Stack.Protected` / `<Redirect>` by session state in the root, never `router.push('/splash')`. Session and onboarding state load asynchronously, so hold the native splash until they have resolved: `SplashScreen.preventAutoHideAsync()` at module scope in the root `_layout` (`expo-splash-screen`, re-exported by `expo-router` — global scope, not inside a component, or it runs too late), render nothing (or the splash image) while the state is still loading, compute every guard from a *resolved* value only, then `SplashScreen.hideAsync()` once the real destination has rendered — the same render in which the guard flips, never before. Login-then-Home, or an empty tab bar before content, is the one-frame flash the full-motion pass catches, and "kill → relaunch lands by state" is not true without this |
| **Unsaved work in a modal** | back and Cancel *ask* before discarding; they never silently lose work | `usePreventRemove(isDirty, ({ data }) => confirm → navigation.dispatch(data.action))` on the modal's **root** screen (the inner stack's `index`) — removing that route *is* leaving the modal, and React Navigation propagates a nested route's prevention up to the modal route in the root stack, so swipe-down, Cancel (`dismiss` / `dismissTo`) and Android back at step one all ask, while back *between* steps stays free. Never put it in a step: the hook blocks every removal of the route it is called in, a plain pop included. If the swipe-down must also be physically disabled while dirty, that is `gestureEnabled: !isDirty` on the `compose` route in the root `_layout`, driven from the draft store — a `<Stack.Screen options>` inside a step only reaches the inner stack and would kill the between-steps edge swipe instead |

```tsx
// A multi-step composer: free back between steps, a guarded exit.
// The hook lives ONLY on the modal's root screen (compose/index.tsx). Removing that
// route is leaving the modal; steps push on top of it inside the modal's own Stack
// and pop freely. The hook never goes in a step — it would block back between steps.
import { usePreventRemove } from 'expo-router/react-navigation'; // SDK 56+ (56.2.10+ for TS types); '@react-navigation/native' before
import { useNavigation } from 'expo-router';
import { Alert } from 'react-native';

export default function ComposeRoot() {
  const navigation = useNavigation();
  const isDirty = useDraftStore((s) => s.isDirty);

  // Fires for swipe-down, Cancel (dismiss / dismissTo) and Android back at step one —
  // React Navigation propagates a nested route's prevention to the modal route above it,
  // so the native swipe-down is cancelled too. No gestureEnabled needed here.
  usePreventRemove(isDirty, ({ data }) => {
    Alert.alert('Discard this post?', 'Your edits will be lost.', [
      { text: 'Keep editing', style: 'cancel' },
      { text: 'Discard', style: 'destructive', onPress: () => navigation.dispatch(data.action) },
    ]);
  });

  return /* step one … */;
}
```

(Version note: the `navigation.dispatch(data.action)` Discard above is the
documented pattern on every released expo-router — SDK 57 and below. The next
major makes re-dispatching `data.action` while the flag is still set
re-prevent it — the alert comes back and the modal never closes. There,
Discard is `() => { clearDirty(); router.back(); }`. Whichever you ship,
prove it in the simulator pass: dirty modal → back → Discard closes it in one
tap.)

The exceptions are the whole list above, plus one that removes no screen:
**transient in-screen state consumes back first.** Selection / edit mode on
a list, an expanded search field, an open in-screen sheet or panel (the
`@gorhom` row above — a map's place card, a player's queue): the first back
clears that state, the next back leaves the screen. That is the Android
convention (contextual action bar, search view and modal bottom sheet all
behave this way) and it is `BackHandler` inside `useFocusEffect`, returning
`true` only while the transient state is up — the React Navigation-documented
pattern, Android-only by nature: on iOS the state keeps its own exit
(Cancel/Done, the sheet's drag and scrim) and the edge swipe still pops.
Reach for `usePreventRemove(isTransient, () => clearTransient())` instead
only when iOS should hold too — selection mode whose header has replaced the
chevron with Cancel/Done. A press handled in JS can't play the
predictive-back preview; acceptable for a sheet or a selection, never for a
whole screen. **Blocking back anywhere else is a defect** — never to keep
someone on a paywall, a rating prompt, or a "before you go" screen. Apple's
own wayfinding test applies to every screen: *Where am I? Where can I go?
What's there? How do I get out?* A screen with no answer to the last
question ships with a bug.

## Tabs, stacks inside tabs, and what covers the tab bar

- **Bottom tabs are peers, not a hierarchy.** Switching tabs in the tab bar
  never slides (`animation: 'none'`; a Material-styled app on JS `Tabs` may
  cross-fade with `'fade'`), never pushes, and each tab keeps its own stack
  where the user left it. Re-tapping the active tab pops that tab to its
  root; re-tapping again while at the root scrolls to top (`NativeTabs`
  does both, Android from SDK 55; opt out per tab with `disablePopToTop` /
  `disableScrollToTop` only with a reason). Swipe-paged top tabs *inside* a
  screen (`react-native-pager-view` / material-top-tabs) are a pager, not
  the tab bar — their lateral slide is the finger's, and stays.
- **Each tab that drills down owns a `Stack`** (`(tabs)/feed/_layout.tsx`)
  with `unstable_settings = { initialRouteName: 'index' }` so a deep link into
  `/feed/123` still has the feed under it.
- **Decide, per screen, whether the tab bar stays.** It stays when the user
  is *browsing* inside a tab's hierarchy and will bounce between tabs
  (App Store, Music: category → list → detail). It goes when the screen wants
  full attention — composer, player, checkout, viewer, onboarding, anything
  full-bleed. To hide it, put the route in the **root** stack *above*
  `(tabs)` instead of inside a tab. Back then returns to whichever tab you
  came from; a detail routed inside a tab *switches to that tab and pops*,
  which surprises people when the link came from elsewhere.
- **Shared routes** (`(tabs)/(feed,search)/users/[username]`) when the same
  detail legitimately belongs under more than one tab; back stays in the
  tab the user was in.
- **Android**: by default back from another tab returns to the first tab
  before exiting the app (`NativeTabs` `backBehavior` defaults to
  `initialRoute`, JS `Tabs` to `firstRoute`). Leave the default.

## Platform back, in full

- **iOS**: the chevron and the edge swipe are one gesture — never disable
  the swipe (`gestureEnabled`) outside the in-flight freeze and the
  dirty-modal case. `fullScreenGestureEnabled` is on by default on iOS 26;
  on iOS 18 and below enabling it swaps the native edge-swipe transition for
  `simple_push`, so leave the default there. Custom `animation` values come with
  `animationMatchesGesture: true` so dragging back runs the same transition
  in reverse.
- **Android**: hardware and predictive back *are* the back button. Predictive
  back peeks at the previous screen; a JS-rebuilt transition or a swallowed
  back breaks the system animation. Modals and sheets dismiss on back —
  including in-screen sheets, selection mode and an expanded search field,
  which consume the first back (`BackHandler` in `useFocusEffect`, returning
  `true` only while that state is up). Beyond that, intercept back only
  through the two sanctioned `usePreventRemove` cases — never a
  `BackHandler` that returns `true` to "keep them here".
- **iOS 26 / Liquid Glass**: the `NativeTabs` bar is glass by default and
  minimizes on scroll only if you opt in
  (`<NativeTabs minimizeBehavior="onScrollDown">`); the form sheet shows the
  system glass at partial detents. Let native chrome draw it; don't fake
  glass under glass.
- **Web**: modals are routes; `router.canGoBack()` before `back()`; a
  dismiss link falls back to the parent URL.

## The back-stack audit (part of the simulator loop)

Walk every screen and write the answer for each:

| Action | Expected |
|---|---|
| Chevron / `back()` | previous screen in *this* hierarchy — never a gate, never a finished session |
| iOS edge swipe | identical to the chevron (or, in the two exceptions, the prompt / nothing) |
| Android hardware back | identical to the chevron; on a root tab → home tab → exit |
| Re-tap the active tab | pop to root; a second tap at the root scrolls to top |
| Rapid double tap on a row / button | one push, not two (disable on press, or `dangerouslySingular`) |
| Background → return | same screen, same scroll, same form values |
| Kill → relaunch | lands by *state* (signed in → home; signed out → the wall in a wall app, home as a guest in a browsable one; mid-onboarding → that step) |
| Cold start from a deep link | target screen **with a real stack underneath**; back lands somewhere sensible |
| Link / notification tapped while signed out | sign in → lands on the target with a stack underneath, not on home |
| Notification tapped while warm (another screen / tab) | target on top, once; back returns to where I was; a foreground arrival shows a banner and does not navigate |
| Every modal | Cancel/Done visible; swipe/back dismiss; dirty → asks first, and Discard closes it in one tap |
| Every sheet | drags; tap-scrim dismisses; keyboard up → still dismissible |
| After each one-way door | back cannot re-enter the old state (login, paywall, session) |

## Never / Instead

| Never | Instead |
|---|---|
| `push('/home')` after login / onboarding / purchase | `replace` + `Stack.Protected` so the gate leaves history |
| `replace('/(tabs)')` after a sign-in that one action demanded | the auth sheet closes (guard flip / `dismiss()`), the action runs where it was tapped |
| A success screen pushed on top of checkout | `replace` the checkout with success; CTA → `dismissTo` |
| `router.navigate` assumed to "go back to" an existing route | `dismissTo(href)` for go-back-to; `push` for go-deeper |
| A sheet that grows steps | `presentation: 'modal'` with a stack |
| A `useState` bottom sheet for something a link could open | a `formSheet` route |
| A hand-built share sheet, link list, photo/file picker, or an in-app WebView for reading a page | the system controller — `Share.share`, `expo-web-browser`, `expo-image-picker` / `expo-document-picker`; an in-app recipient picker only when the app has recipients of its own |
| A modal with no Close because "swipe down works" | Cancel/Done in the modal's own header |
| `gestureEnabled: false` to keep users in a funnel | leave it; only the in-flight and dirty cases may block |
| A `BackHandler` returning `true` to keep users on a screen | `usePreventRemove` in the two sanctioned cases; `BackHandler` only to clear transient in-screen state (selection, search, in-screen sheet) — the next back leaves |
| Sliding between bottom tabs, or pushing a tab | `animation: 'none'` (`'fade'` in a Material-styled app); tabs are peers — a swipe-paged top-tab pager keeps its slide |
| A detail with an empty stack behind it from a deep link | `initialRouteName` / `withAnchor` |
| `replace` / `dismissAll` in a notification-tap handler, or `router.push` in the *received* listener | `push(href, { dangerouslySingular: true })` / `navigate` from the *response* listener; foreground arrival = banner |
| Tab bar visible on a composer / player / checkout | route it in the root stack above `(tabs)` |
| Modal over modal | action sheet / alert over the modal, or `dismissTo` and re-present |
| Blocking back for longer than a request takes | let them leave; notify on completion |
| `dismissTo('/(tabs)')` (or a success route) after a paywall opened from a feature | confirm in place, `dismiss()` — land them back on the feature, unlocked |
