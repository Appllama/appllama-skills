# Research methods: flows, elements, and how to study like a director

## Flow research — study journeys, not screenshots

Flows are where conversion and retention actually live. Use them when the
question is "how do winners structure X?" rather than "what does X look
like?".

1. `list_flows(query="onboarding")` — or browse the taxonomy — to find the
   exact flow names and their footprint (screens/apps counts tell you how
   universal a flow is).
2. `get_flow_apps(flow="Paywall")` — the apps that contain it, top revenue
   first, each with its screen count *inside that flow*. A 9-screen paywall
   flow from a $2M/mo app is a masterclass; read it.
3. `list_app_screens(app_id=…, flow=…)` — the flow itself, in order. Study
   3–5 apps' versions of the same flow side by side and chart the common
   spine: step count, what each step asks vs. gives, where friction is
   deliberately placed, where it's removed — and what each step *is*
   (a pushed screen, a modal with its own steps, a sheet, an overlay),
   which the videos show better than stills. Winners are consistent about
   presentation; that grammar is part of the spec.

High-value flow studies for almost any category: Onboarding (length,
personalization, permission timing), Paywall (placement, trial framing,
price anchoring), Welcome (first 5 seconds), plus the category's signature
flows (Food Logging, Workout Session, Habit Check-in, …).

## Element research — how winners build one component

When the question is component-level ("how should our tab bar / progress
indicator / CTA look?"):

1. `list_ui_elements()` once — the ~38 family catalog with counts and top
   variants (Bottom Tab Bar, Rounded Primary CTA, Top Progress Bar, Skip
   Text Link, …).
2. `get_element_screens(element=…)` — real screens carrying that family.
   Pin a `variant` to narrow to one specific treatment.
3. Also usable as a filter: `search_screens(query=…, element=…)` combines
   content and component ("stats screens that use a Floating Bottom Nav").

Extract the numbers, not the vibe: sizes, placements, label conventions,
active-state treatments, how many items, what gets an icon vs. text.

## Study discipline (what separates research from tourism)

- **Question first.** Every pass answers a named question from your current
  task — "how long is onboarding here", "what does a winning paywall carry".
  Knowing the question is what turns screens into a spec.
- **See everything, in order.** Walk the full screen list of an app you're
  studying and look at every image; the journey is the unit of design, and
  a sampled journey lies. Notes outlive links: media URLs die in ~1 h, but
  screen ids (`app_id/screen_id`) keep any screen one `get_screen` away.
- **Cross-app before in-app.** One app tells you its taste; five apps tell
  you the category's grammar. Divergence between winners = a real choice;
  convergence = a convention you break knowingly or not at all.
- **Revenue is context, not truth.** A $5M/mo app's paywall is evidence
  about paywalls; its settings screen might still be lazy. Weight evidence
  by whether that surface plausibly drives the app's success.
- **Stop at saturation, not before.** A research question is answered when
  new screens stop changing your spec. Until then, keep looking.

## Working with the member's own curation

`list_my_boards()` shows boards the member built on appllama.io;
`get_board(board_id)` reads any of them — screens boards return full screen
payloads with media, apps boards return app profiles, flows boards return
(app, flow) pairs ready for `list_app_screens`. An apps board also works as
a `search_apps(board_id=…)` base. Their taste is a requirement, not a
suggestion — when a board exists for the task, start from it.
