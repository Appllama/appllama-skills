# Motion vocabulary — say exactly what you mean

Briefs describe motion by sensation ("springy", "slides off", "draws itself
in") and specs written that way produce guesses. This glossary turns the
sensation into the term, so a spec reads "**pop in** from the trigger,
damping `0.8`, 250 ms" instead of "make it bouncy". Use it in three
directions: decode a loose brief, write an exact spec, and name a defect in
a review ("the sheet has no **velocity hand-off**").

How to use it: read for intent, not keywords; lead with the best match, then
one or two close alternates with how they differ; when nothing fits, say
it's an approximation and describe the effect in these words. Stay inside
this list — if a term isn't here, explain the idea with the words that are.

## Entrances and exits
- **Fade in / out** — opacity only. The floor; usually paired with a transform.
- **Slide in** — enters from off-screen (top, bottom, left, right).
- **Scale in** — grows from smaller to full size, paired with a fade; starts at 0.9–0.97, never 0.
- **Pop in** — scale-in with a slight overshoot that settles.
- **Reveal** — uncovered progressively (a mask or clip), not faded.
- **Enter / exit** — the pair an element plays when mounted / unmounted (`entering` / `exiting`).
- **Asymmetric enter/exit** — exit faster than entry, same path.

## Sequencing and timing
- **Stagger** — several items one after another with a small delay (30–80 ms).
- **Orchestration** — multiple animations timed to read as one motion.
- **Delay / duration / fill mode** — when it starts, how long, what sticks after.
- **Stepped** — discrete steps, no interpolation (a countdown).
- **Keyframes** — defined points the system fills between (Reanimated CSS animations).

## Movement and transforms
- **Translate / scale / rotate / skew** — the free properties; everything else is layout.
- **3D tilt / flip** — rotateX / rotateY with perspective.
- **Transform origin** — the anchor a scale or rotation grows from.
- **Origin-aware** — an element animates out of its trigger (a menu from its button), not from its own center. Centered modals are the exception.
- **Transform order** — translate first, then scale, unless you want the multiplication.

## Transitions between states
- **Crossfade** — one element fades out as another fades in, in place.
- **Continuity transition** — a change that keeps the user oriented by visually connecting before and after.
- **Morph** — one shape becomes another (Dynamic Island).
- **Shared-element transition** — an element travels and transforms from one position to another (thumbnail → hero).
- **Layout animation** — size or position changes animate instead of snapping (`layout`, `itemLayoutAnimation`).
- **Accordion / collapse** — a section grows or shrinks its height (the rare sanctioned layout animation; keep it short).
- **Direction-aware transition** — forward slides one way, back the opposite way.
- **Push / modal / sheet / overlay** — the navigation presentations (navigation.md); each has a transition the platform owns.

## Scroll
- **Scroll-driven** — progress tied to scroll offset (`useAnimatedScrollHandler`).
- **Collapsing header / large title** — chrome that condenses as content scrolls.
- **Parallax** — layers moving at different speeds.
- **Overscroll stretch** — a hero that scales as you pull past the top.
- **Scroll-edge effect** — the fade/blur where content meets floating chrome.

## Feedback and interaction
- **Press feedback** — a subtle scale-down on press-in (0.97), release on press-out.
- **Hold to confirm** — a fill that progresses while the finger holds; linear, slow in, snappy release.
- **Drag** — moving by grabbing; momentum on release.
- **Drag to reorder** — items shift to make room.
- **Swipe to dismiss** — dragging off-screen to close (a toast, a sheet).
- **Swipe to reveal** — a row slides to expose actions.
- **Rubber-banding** — rising resistance past a boundary, then snap-back.
- **Pull to refresh / arming** — the threshold at which a pull will trigger (one haptic, at the threshold).
- **Shake / wiggle** — a short jitter for rejected input.
- **Ripple** — Material's expanding circle from the touch point.
- **Haptic** — selection tick, light/medium impact, success/error notification.

## Easing
- **Ease-out** — fast start, slow end; the default for UI and anything responding to the user.
- **Ease-in** — slow start; avoided on UI.
- **Ease-in-out** — for things already on screen moving from A to B.
- **Linear** — constant speed; progress, marquee, hold fills only.
- **Cubic-bezier** — a custom curve (the three house curves in motion.md).

## Springs
- **Spring** — physics-driven motion; no fixed duration.
- **Damping ratio** — how much it overshoots (1 = none; 0.8 = a little).
- **Response / duration** — how quickly it approaches the target.
- **Bounce** — the visible overshoot; only after momentum.
- **Perceptual duration** — when a spring *looks* finished, though it micro-settles after.
- **Velocity** — speed and direction at release.
- **Velocity hand-off** — the release velocity fed into the spring; the seam between finger and animation.
- **Momentum projection** — where a flick *would* land, used to choose the target.
- **Interruptible** — can be grabbed and redirected mid-flight from the live value.

## Looping and ambient
- **Loop / alternate (yoyo) / marquee / pulse / float / idle** — motion that runs on its own; rare in product UI, and always off under Reduce Motion.
- **Skeleton / shimmer** — a loading placeholder shaped like the content, with a moving sheen.
- **Number ticker** — digits rolling to a value; needs tabular numerals.

## Performance
- **Frame rate** — 60 fps baseline, 120 on ProMotion (8 ms budget).
- **Jank / dropped frame** — a missed deadline, felt as a hitch.
- **UI thread / JS thread** — where worklets run vs. where React renders; motion belongs on the first.
- **Layout thrash** — animating width/height/margin/top so Yoga re-runs every frame.
- **Recycled row** — a virtualized list item that re-mounts on scroll (no `entering` there).

## Principles
- **Purposeful motion** — orient, give feedback, show relationships; never decorate.
- **Frequency of use** — the more often it's seen, the shorter and subtler (or absent) it is.
- **Spatial consistency** — elements keep identity and position across states; exit the way they entered.
- **Anticipation / follow-through / squash & stretch** — animation-craft terms for wind-up, settle, and weight; use sparingly, in the delight tier.
- **Perceived performance** — the right motion makes the app feel faster than it is.
- **Reduced motion** — fewer and gentler, never zero feedback.
