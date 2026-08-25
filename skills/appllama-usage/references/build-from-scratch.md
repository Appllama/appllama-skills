# Playbook: build an app from scratch

The user says "build me a habit tracker" (or any category). This is the full
loop. Do not stop early: the goal is a finished app where every screen is
verified better-than-reference in the simulator — not a scaffold.

## Phase 1 — Find the winners

1. Translate the user's need and priorities into 1–3 `search_apps` queries
   ("habit tracker", "habit tracker with streaks and social accountability").
   Add filters that encode the user's ambitions — e.g. `revenue_min` to bias
   toward proven money-makers, `launched_after` for modern design language.
2. From the results, shortlist the **top 5** by a blend of revenue,
   rating, downloads, and fit to the user's brief — not revenue alone.
   The flow lists in the results tell you a lot before you even open an
   app: one with `Paywall: 9` screens has a monetization story worth
   studying; one with `Onboarding: 25` is quiz-onboarding heavy.
3. `get_app` each shortlisted app. Record metrics + flows in
   `research/<category>/apps.md`.

## Phase 2 — Study every screen of the top 5

For each shortlisted app, walk `list_app_screens` page by page — ALL of it,
in journey order, and look at every image. Design language lives in the
pixels, not the metadata. As you go, build the app's `screens.md`: screen
id, name, flow, notable elements, palette — and download the screens so you
can read the journey side by side.

You are extracting the **category's design language**, so read across apps,
not just within one:

- What does the first-open moment look like in this category?
- How long is onboarding, and what does each step *earn* (permission,
  personalization data, commitment)?
- Where does the paywall sit, and what's on it (trial framing, price
  anchoring, feature grid, social proof)?
- What's the home-screen information hierarchy? What's one tap away?
- What do empty states, streaks/progress, and notifications look like?

Write the synthesis into `patterns.md`: the psychology of the category —
what every winner does (table stakes), what only the best do (edge), and
what all of them do badly (your opening).

## Phase 3 — Frame-by-frame on the top 3

Pick the 3 strongest apps and re-walk their decisive flows screen by screen
(`list_app_screens` with `flow=` filters; `get_screen` on pivotal screens to
see similar-screen alternatives across the library). For each screen answer:
what is its ONE job, what makes it work, what would you change. This is
where you stop being a catalog and start being a design director.

## Phase 4 — Design the feature set

From `patterns.md`, write the app's spec: the best features across all
studied apps, minus the bloat, plus the opening you found. Screen list with
flows, in journey order — and the **navigation map**: for every screen, what
it *is* (push, modal with its own stack, form sheet, full-screen modal,
overlay, tab root) and what back does from it, including the one-way doors
(sign-in, onboarding done, purchase, finished session) where back must not
re-enter the old state. The grammar comes from appllama-app-design-skill's
Navigation laws; the evidence comes from the flows you walked —
note whether each winner presents its composer as a modal, its filters as a
sheet, its detail as a push, and copy that consistency. Get user sign-off on
the spec if they're present; otherwise state your choices and proceed.

## Phase 5 — Build screen by screen

For EVERY screen, in journey order:

1. Re-open your references for that screen type (your local board first;
   `search_screens` for gaps — both modes, they surface different screens).
2. Build it following **appllama-app-design-skill** end to end (HIG fidelity,
   semantic colors, native controls, motion laws, state architecture).
3. Generate image assets with the best image model available (imagegen /
   Higgsfield MCP or CLI / whatever is present) at the highest quality — one
   style system for the whole app, per appllama-app-design-skill's
   references/image-assets.md.
4. **Simulator loop until flawless**: run on the iOS Simulator (or Android
   emulator), screenshot, actually look, exercise the motion frame by frame,
   check the Dynamic Island / safe areas / dark mode / Dynamic Type, fix,
   repeat. The checklist lives in appllama-app-design-skill's
   references/simulator-loop.md. A screen isn't done because it compiles —
   it's done when you can't find a flaw and it stands next to the top-3
   references without embarrassment.
5. State management stays boring and strong (server state / client state /
   ephemeral separation per the design skill). Motion is tracked and
   verified, not assumed.

## Phase 6 — The bar

Walk the whole app in the simulator as a new user, three times: happy path,
skeptic path (skip everything skippable), abuse path (bad input, offline,
interrupt mid-flow, and every back path — chevron, edge swipe, Android
hardware back, active-tab re-tap — especially right after sign-in,
onboarding, a purchase and a finished session, where back must fail to
re-enter the old state). Compare each flow against the best reference you
studied. If any screen of yours is worse than the best equivalent screen in
your research, it goes back into the loop. **You cannot declare the build
finished until every screen holds that comparison.**
