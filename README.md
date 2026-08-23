<p align="center">
  <a href="https://appllama.io">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://public.appllama.io/appllama-logo-dark.png">
      <img src="https://public.appllama.io/appllama-logo-light.png" alt="Appllama" width="360">
    </picture>
  </a>
</p>

<h3 align="center">A builder, not just a researcher.</h3>

<p align="center">
  Agent skills that make AI agents genuinely good at building mobile apps —<br>
  studied against the top-grossing apps, finished to a simulator-verified bar.
</p>

<p align="center">
  <a href="https://skills.sh/appllama/appllama-skills"><img src="https://skills.sh/b/appllama/appllama-skills" alt="skills.sh installs"></a>
  <a href="https://appllama.io"><img src="https://img.shields.io/badge/Appllama-official-1a1a1a" alt="Appllama official"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue" alt="License: MIT"></a>
</p>

<p align="center">
  <a href="https://appllama.io">appllama.io</a> ·
  <a href="https://appllama.io/mcp">MCP</a> ·
  <a href="https://x.com/appllamaio">X</a> ·
  <a href="https://www.linkedin.com/company/appllama">LinkedIn</a> ·
  <a href="https://www.producthunt.com/products/appllama">Product Hunt</a>
</p>

---

[Appllama](https://appllama.io) is the design library of top-grossing mobile
apps — their real screens, flows, and UI patterns, with revenue and download
context. These skills turn that library into an agent's working method:
study every screen of the apps that already win, extract the category's
design language, then build screens that hold up next to them.

## The skills

| Skill | What it does |
|---|---|
| [`appllama-usage`](skills/appllama-usage/SKILL.md) | The research engine: how to use the [Appllama MCP](https://mcp.appllama.io/mcp) like a design director — the full tool map, and the playbooks for building an app from scratch, improving an existing screen, and flow & element research. |
| [`appllama-app-design-skill`](skills/appllama-app-design-skill/SKILL.md) | The build bar: native-feeling Expo / React Native screens — Apple HIG fidelity, semantic colors, native controls, anti-slop discipline, navigation that behaves (push vs replace, modal vs sheet vs overlay, and the one-way doors where back must not exist), a strict motion bar (should it animate at all, exact springs and curves, gestures that carry velocity, haptics on the same frame, nothing on the JS thread), generated image assets, and a full-motion simulator-verified iteration loop (whole flows recorded and scrubbed at 60 fps, not screenshots). |

They are designed as a pair: **usage** decides what to study, **design**
decides how to build, and both insist the loop only ends in a simulator
with a screen you can't fault.

### Inside the design skill

A short set of laws, and a reference library the agent loads only when the
task calls for it:

| Reference | What it settles |
|---|---|
| [`navigation`](skills/appllama-app-design-skill/references/navigation.md) | push vs replace vs `dismissTo` · modal vs form sheet vs overlay · tabs and what covers the tab bar · deep links with a real stack underneath · the one-way doors (sign-in, onboarding done, purchase, finished session) · the back-stack audit |
| [`motion`](skills/appllama-app-design-skill/references/motion.md) | the decision sequence — should it animate at all → purpose → cheapest tool → properties → spring or curve → off the JS thread — with exact values, haptics, reduced motion, and the never-ship list |
| [`motion-recipes`](skills/appllama-app-design-skill/references/motion-recipes.md) | press feedback, drag-to-dismiss sheet, swipe-to-delete, collapsing header, list entrances, keyboard-synced UI, tab indicator, toast, threshold haptics — ready to build |
| [`fluid-interfaces`](skills/appllama-app-design-skill/references/fluid-interfaces.md) | the physics of feel: response, interruptibility, velocity hand-off, momentum projection, rubber-banding — plus materials and depth, multimodal feedback, typography, and the design principles behind all of it |
| [`motion-review`](skills/appllama-app-design-skill/references/motion-review.md) | reviewing a diff's motion, auditing a whole app into plans any agent can execute, and hunting for (and rejecting) places that could animate |
| [`motion-vocabulary`](skills/appllama-app-design-skill/references/motion-vocabulary.md) | the exact words for motion, so a brief that says "bouncy" becomes a spec that says what it means |
| [`variant-lab`](skills/appllama-app-design-skill/references/variant-lab.md) | three genuinely different directions behind a dev-only switcher — for open briefs and hero screens where direction matters more than polish |
| [`native-controls`](skills/appllama-app-design-skill/references/native-controls.md) | the iOS + Android control map, menus, sheets, forms — and the library picks, so nothing solved gets hand-rolled |
| [`performance`](skills/appllama-app-design-skill/references/performance.md) | measure → fix → re-measure, the budgets, and the thread discipline behind 60 fps |
| [`image-assets`](skills/appllama-app-design-skill/references/image-assets.md) | one style system, generated at the highest quality, post-processed and verified in both themes |
| [`simulator-loop`](skills/appllama-app-design-skill/references/simulator-loop.md) | the verification checklist — layout, theming, motion, interaction, navigation and back stack, state — and the device matrix |

## Install

One command, from your project root — works with Claude Code, Cursor,
Codex, and [70+ other agents](https://skills.sh):

```bash
npx skills@latest add appllama/appllama-skills
```

Variations:

```bash
# install for specific agents, no prompts
npx skills@latest add appllama/appllama-skills -a claude-code -a cursor -y

# install user-wide instead of per-project
npx skills@latest add appllama/appllama-skills -g
```

### Only want the app design skill?

`appllama-app-design-skill` stands on its own — the native-quality build
bar, anti-slop discipline, and the full-motion simulator loop work with or
without the Appllama MCP connected:

```bash
npx skills@latest add appllama/appllama-skills --skill appllama-app-design-skill
```

(The same `--skill` flag installs only `appllama-usage` if you want just the
research engine.)

<details>
<summary>Manual install</summary>

Skills are plain directories — copy them into your agent's skills folder
(`.claude/skills/` per project, `~/.claude/skills/` user-wide, or your
harness's equivalent):

```bash
git clone https://github.com/appllama/appllama-skills
cp -r appllama-skills/skills/* ~/.claude/skills/
```

</details>

## Connect the Appllama MCP

The skills assume the Appllama MCP is connected:

```
https://mcp.appllama.io/mcp
```

Add it as a custom connector in Claude, Cursor, Codex, or any MCP client
and approve the connection with your Appllama account. MCP access is part
of [Pro](https://appllama.io/pricing); credits reset in full on the 1st of
each month. Every call spends one credit — `get_credits` is always free.

## Try it

With the MCP connected and the skills installed, ask your agent:

> Build me a habit tracker. Study the top-grossing habit apps first and
> don't stop until every screen survives the simulator comparison.

> Make this screen better. *(paste a screenshot, code, or a "Copy Screen
> ID" ref from appllama.io)*

> How do the best fitness apps structure onboarding — how long, what does
> each step earn, and where does the paywall sit?

> Wire up the checkout flow. Decide which screens push, which present as
> sheets, and make sure nobody can go back into the paywall after paying.

> Review the animations in this app — what should be deleted, what's on the
> wrong thread, what's missing velocity — and give me the plan.

## License

[MIT](LICENSE). The Appllama name, llama, and logo are trademarks of
Antmind Ventures Private Limited — the license does not grant rights to
use them.

---

<p align="center">
  Built by <a href="https://appllama.io">Appllama</a> — the design library of top-grossing apps.<br>
  <a href="https://x.com/appllamaio">X</a> ·
  <a href="https://www.linkedin.com/company/appllama">LinkedIn</a> ·
  <a href="https://www.producthunt.com/products/appllama">Product Hunt</a>
</p>
