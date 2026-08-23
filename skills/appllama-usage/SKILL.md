---
name: appllama-usage
description: Use the Appllama MCP (mcp.appllama.io) well — research real top-grossing mobile apps, their screens, flows, and UI elements, then build from what you learn. Load when the Appllama MCP is connected and the task involves building a mobile app or screen, researching app design patterns, studying onboarding/paywall/feature flows, improving an existing screen, or whenever an appllama_* / search_apps / list_app_screens tool is available. Covers the tool map, pagination, expiring media, and the full build-from-research playbooks.
license: MIT
metadata:
  author: Appllama (appllama.io)
  version: 1.1.0
---

# Appllama Usage Skill

Appllama is the design library of top-grossing mobile apps — their real
screens, flows, and UI patterns, with revenue and download context. The MCP
puts that library in an agent's hands: **not just a research tool, a builder's
tool.** You study what already wins, then you build something better.

Pair this skill with **appllama-app-design-skill** for every design/implementation
step — this skill tells you what to study; that one tells you how to build.

## Ground rules (read first)

1. **Start with `get_credits` — it's free.** It tells you the balance,
   limits, and reset date. Pro includes 1,500 credits a month (they reset in
   full on the 1st, UTC); every other call spends 1 credit.
2. **Go deep.** Design language lives in the whole journey, not a sample —
   walk every screen of the apps that matter for the task, images included.
   That is exactly what the library is for. The one thing that's against the
   terms is harvesting: sweeping the catalog to extract the dataset itself
   rather than to answer a real task. That isn't research, and it's detected
   server-side.
3. **Media URLs expire in ~1 hour.** Download/view what you study promptly.
   If links died mid-task, re-request that page for fresh ones — screen ids
   are durable, links are not.
4. **Ignore the watermark.** Every Appllama image and video carries a small
   Appllama watermark in the top-left corner. It is provenance, not part of
   the screen — don't let it skew your read of that corner (status bar,
   back button, title), and never reproduce it in anything you build.
5. **Pagination is sequential.** Every list response carries `next_cursor`;
   pass it back to continue. You cannot jump to page N — and a cursor only
   works for the same query that minted it. If a cursor errors, drop it and
   restart from page one.
6. **If you hit a rate limit, wait it out.** The per-minute and per-day
   limits sit far above real research; on the rare hit, wait the stated
   time — don't retry-hammer.
7. **Errors are instructions.** Tool errors are written to be acted on
   (expired cursor → restart; out of credits → tell the user their credits
   reset on the 1st and they can request more in Settings → Usage).

## Tool map

| Tool | What it gives you | Typical use |
|---|---|---|
| `get_credits` | Balance, limits, reset date. **Free.** | Session start |
| `search_apps` | 10 apps/page: name, revenue, downloads, rating, launch date, screens count, **flow list with screen counts**. Natural-language `query` + filters (revenue/downloads/rating/launch date/price/onboarding steps) + `sort` + `board_id` | Find the top apps for a category or need |
| `get_app` | One app in full: ratings breakdown, category rank, IAP pricing, top countries, flows | Decide if an app deserves a deep study |
| `list_app_screens` | 10 screens/page **in journey order** (welcome → onboarding → paywall → product), each with media URL, flow, UI elements, colors. Filter by `flow` or `section` | Walk an app screen by screen |
| `search_screens` | Screens across the whole library. `mode="keyword"` matches screen names + filters (flow, screen_type, element, app_id); `mode="semantic"` searches by meaning/visual language | Gather design references for one screen type |
| `get_screen` | One screen in full + up to 5 visually similar screens from other apps. Accepts `screen_ref` = `app_id/screen_id` (what appllama.io's "Copy Screen ID" produces) | The user pasted a screen ref; or drill into one reference |
| `list_flows` | The flow taxonomy with screen/app counts | Discover what flows exist for a category |
| `get_flow_apps` | Apps containing a flow, top revenue first | Find the best examples of one flow |
| `list_ui_elements` | ~38 UI-element families with counts (one call) | Vocabulary for element-level research |
| `get_element_screens` | Screens featuring an element family | Study how winners build one component |
| `list_my_boards` | The member's own appllama.io boards (screens / apps / flows) | Find their curation first |
| `get_board` | A board's full contents: screens with media, app profiles, or (app, flow) pairs | When the member curated a board for the task, START from it |

**The screen_ref handshake:** members can click "Copy Screen ID" on any
screen at appllama.io and paste it to you. It looks like
`1393061654/spl_9i075` — feed it straight to
`get_screen(screen_ref=...)` and you're looking at exactly the screen they
mean, plus its closest siblings across the library.

## The playbooks

| Scenario | Reference |
|---|---|
| Build an app from scratch (e.g. "build me a habit tracker") | [references/build-from-scratch.md](references/build-from-scratch.md) |
| Make an existing screen better | [references/improve-a-screen.md](references/improve-a-screen.md) |
| Flow & element research; general research method | [references/research-methods.md](references/research-methods.md) |

Both build playbooks end the same way: **the simulator loop from
appllama-app-design-skill, repeated until you cannot find a flaw.** Research
without that loop is decoration.

## Local reference boards

When you pull screens for study, save them into a local working structure —
links expire in about an hour, but your notes and downloads don't:

```
research/
  <category>/
    apps.md            # the shortlist: metrics, flows, verdicts
    <app-name>/
      screens.md       # per-screen notes: id, name, flow, elements, colors
      img/             # downloaded screens, in journey order
    patterns.md        # cross-app synthesis: the category's design language
```

Download the screens as you study them — synthesis happens with the images
side by side, not from metadata. Notes and screen IDs are durable; re-fetch
a fresh link from the ID if you ever need the pixels again.
