# Variant lab — explore genuinely different directions before committing

Some briefs have one right answer ("add a Done button"). Others are open —
"make the paywall feel premium", "the home screen feels flat", a hero
screen (welcome, onboarding step one, paywall, home) where the *direction*
matters more than the polish. For those, building one version and iterating
on it anchors you to your first idea. The variant lab is the alternative:
build several genuinely different answers, put them behind a switcher in
the running app, flip through them at full size, and let the user pick.

The value is **divergence**. Three tints of one idea waste the lab — the
user learns nothing flipping between them. Each variant is a direction you
could defend shipping on its own.

## Hard rules

1. **Production code is untouched during exploration.** Everything lives in
   a dev-only route (`app/__lab/<slug>.tsx`, rendered only when `__DEV__`).
   Integration happens once, for the winner.
2. **Variants diverge on a named axis** — layout, density, personality,
   motion story, interaction model, information hierarchy. Before building,
   state each variant's axis in a phrase. Sharing the app's tokens is not
   convergence; variants *should* look native to the product.
3. **Every variant fully works** — real interactions, real motion per
   motion.md, realistic content (product-shaped copy, plausible numbers,
   real images from the asset pipeline). No lorem ipsum, no dead buttons.
4. **Each variant still clears the bar** — semantic colors both themes,
   native controls, anti-slop counts, the motion standards. A sloppy variant
   doesn't widen the exploration; it loses on execution and teaches nothing.
5. **The switcher is chrome, not a contestant.** One neutral floating pill,
   identical every time, never styled with the app's tokens. Switching is
   instant — a 100+/session action gets no animation.
6. **Clean up after the choice.** Promote the winner, delete the lab route.

## Method

1. **Scope** — one piece per run. "The dashboard" becomes "the home header
   and its first module"; say what you narrowed to and why.
2. **Recon** — tokens, personality, neighbours, what this screen sits next to
   in the flow, and the Appllama references already on the board for this
   screen type. Divergent variants are often *anchored to different
   references*: the dense, data-forward direction one winner uses vs. the
   calm, single-number direction another uses.
3. **Choose directions** — default 3, up to 5. Names describe the direction
   ("Quiet", "Editorial", "Dense", "Playful"), never "A/B/C". Two directions
   that differ only in accent or copy are one; replace one with a real
   alternative.
4. **Build the lab** — the route renders **one variant at a time, full size,
   in realistic surrounding context** (a sheet needs the screen behind it; a
   card needs siblings; a row needs the list). Side-by-side thumbnails lie
   about spacing and scale.
5. **Verify and present** — run it in the simulator, flip through every
   variant in light and dark, screenshot each, then stop: the choice belongs
   to the user.

   | # | Variant | Axis | When it's the right call | Its cost |
   |---|---|---|---|---|
   | 1 | Quiet | minimal motion, borders over shadows | a daily-use tool | least memorable |
   | 2 | Editorial | large type, generous whitespace | the moment deserves weight | eats vertical space |

   Never pre-pick a favourite in the table; asked, answer from personality
   and frequency of use, not aesthetics. If two variants converged while
   you built them, cut one and say so.
6. **Promote** — integrate the chosen variant where it belongs, following
   the project's conventions, then delete the lab. Or run another round,
   diverging *around* the direction the user gravitated to.

## The switcher

```tsx
// app/__lab/paywall.tsx — dev only
import { useLocalSearchParams, router } from 'expo-router';
const VARIANTS = [
  { name: 'Quiet',     Component: PaywallQuiet },
  { name: 'Editorial', Component: PaywallEditorial },
  { name: 'Dense',     Component: PaywallDense },
];

export default function Lab() {
  if (!__DEV__) return null;
  const { v = '1' } = useLocalSearchParams<{ v?: string }>();
  const i = Math.min(Math.max(Number(v) - 1, 0), VARIANTS.length - 1);
  const { Component } = VARIANTS[i];
  return (
    <View style={{ flex: 1 }}>
      <Component key={i} />                                  {/* re-mount so entrances replay */}
      <View pointerEvents="box-none" style={pill.wrap}>
        <View style={pill.bar}>
          {VARIANTS.map((x, j) => (
            <Pressable key={x.name} onPress={() => router.setParams({ v: String(j + 1) })} hitSlop={8}>
              <Text style={[pill.item, j === i && pill.active]}>{x.name}</Text>
            </Pressable>
          ))}
        </View>
      </View>
    </View>
  );
}
// pill: absolute, bottom: insets.bottom + 16, centered; dark glass (rgba(10,10,10,0.82), radius 999,
// 13pt system font, white 55% → 100% when active). Not theme-aware, not tokenized — it's harness chrome.
```

`setParams` keeps the variant in the URL, so a deep link
(`yourapp://__lab/paywall?v=2`) opens straight to one variant for a teammate
or a Maestro flow, and re-mounting on change replays the entrance motion you
want to judge.
