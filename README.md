# Marketing Performance Dashboard — Template

A single-file HTML performance dashboard for paid acquisition teams. Built for sales-led, ecommerce, *or* product-led motions, tracking two or three channels (Meta/Google/LinkedIn/TikTok/etc.) across Monthly, Weekly, Daily, Creative, Optimization, and Changelog views.

No build step, no backend required — it's one HTML file you open in a browser.

## What's in this repo

| File | What it is |
|---|---|
| `dashboard-template-blank.html` | Empty template — two channels, placeholder names, empty data arrays. Start here. |
| `dashboard-example-filled.html` | **Sales-led** example, 2 channels — a fictional roofing supplier ("Acme Roofing") on Meta + Google, tracking SAL/SQL/ICP/BTC/DPC. |
| `dashboard-example-ecommerce.html` | **Ecommerce** example, 3 channels — a fictional DTC home-goods retailer ("Harborline Goods") on Meta + Google Shopping + TikTok, tracking add-to-cart, checkout, orders, ROAS and refunds. |
| `dashboard-example-product-led.html` | **Product-led** example, 3 channels — a fictional PLG SaaS ("Loomstack") on Meta + LinkedIn + Google, tracking signups, activation, PQL, trial-to-paid and churn. |
| `sheet-template/` | Schema for the two hand-edited tabs (`test_log`, `platform_log`) of the Google Sheet that backs a real build. **Create your own** — nothing here points at anyone's data, and the examples use no sheet. |
| `skills/dashboard-builder.md` | A Claude "skill" file — paste this into a Claude project and Claude will walk you through building your own version, onboarding you on your channels, metrics, and motion type. |
| `LICENSE` | MIT |

## Quick start (manual)

1. Open whichever example matches your motion — `dashboard-example-filled.html` (sales-led), `dashboard-example-ecommerce.html`, or `dashboard-example-product-led.html` — to see it working.
2. Copy `dashboard-template-blank.html` to start your own, and keep the closest example open as a reference for row shapes and column definitions.
3. Replace the placeholder names in the markup: `[Company Name]` (page title and logo), `[Reporting Period]`, and the `[Channel 1]` / `[Channel 2]` labels (nav tabs, section headers, table captions) with your own.
4. Fill in the data arrays at the top of the `<script>` block — they all ship empty:

   | Array | Page and grain |
   |---|---|
   | `MM` / `GM` | Monthly — Channel 1 (ad set) / Channel 2 (ad group) |
   | `MW` / `GW` | Weekly — same grain as Monthly |
   | `MD` / `GD` | Daily — campaign level, per channel |
   | `CR` / `GCR` | Creatives — Channel 1 ads, Channel 2 campaigns |
   | `CR_L30` / `GCR_L30` | The same two tables for the Last-30-days window |
   | `ADD` / `CUT` | Optimization — Scale Up / Pull Back |
   | `VM` / `VG` | Funnel heatmap segments, per channel |
   | `ADS` | Ad Library gallery |
   | `NOTES` | Seeded Changelog cards (leave empty for a clean board) |
   | `PLATFORM_LOG` | Platform change-log entries, keyed by channel (`c1`, `c2`, …) |

   **Every creative row needs an `ad_id`.** That is what links a Changelog card's control and test ads to their live numbers; without it the Testing card has nothing to read.

5. Set the five config values just below the arrays:

   | Value | What it does |
   |---|---|
   | `PERF_RATE_LABEL` | Column label for your motion's conversion rate — `Submit%` / `ATC%` / `Signup%` |
   | `PERF_RATE_KEY` | The field that label reads, e.g. `submit_pct` |
   | `AD_PLATFORM` | Per channel: platform name and its ads-manager URL (`{account}` is substituted from Settings) |
   | `DRILL` | Maps each Monthly/Weekly table to its channel tab and creatives table |
   | `DUP_RELAY` | Optional; normally left empty and set from Settings at runtime |

   All three examples have every one of these populated — copy from whichever matches your motion.

   The two three-channel examples use **index-based names instead**, because channel initials stop scaling once there are more than two: `M1`/`M2`/`M3` (monthly), `W1`/`W2`/`W3` (weekly), `D1`/`D2`/`D3` (daily), `CR1`/`CR2`/`CR3` (creatives, each with a `_L30` sibling), `V1`/`V2`/`V3` (heatmap), with `ADD`/`CUT`/`ADS`, `PLATFORM_LOG`, `AD_PLATFORM` and `DRILL` keyed by `c1`/`c2`/`c3`. Same data shapes, clearer numbering — worth copying if you are adding a third channel.
6. Update the KPI labels if your metrics differ. Each table has a matching `*_LABELS` / `*_KEYS` / `*_RIGHTS` triple, all the same length: `PL_MW_*` and `GL_MW_*` (Monthly and Weekly), `PL_D_*` and `GL_D_*` (Daily), `REC_*` (Optimization), `CR_*` (Creatives). `LABELS` sets the header text, `KEYS` names the field sorting reads, `RIGHTS` right-aligns the column.

   Note that the table cells themselves are emitted by the `render*` functions (`renderMW`, `renderGMW`, `renderDaily`, `renderGDaily`, `renderRec`, `renderCr`), which reference row fields by name. To swap a metric end to end (e.g. product-led teams trading SAL/SQL/ICP for Signup/Activation/PQL — see the skill file for full definitions), edit the label array *and* the matching render function.
7. Open in a browser. That's it — no server needed.

## Quick start (with Claude)

1. Create a Claude project (claude.ai) or open Claude Code.
2. Add `skills/dashboard-builder.md` to the project's skill files.
3. Ask Claude: *"build me a dashboard for [my company]"*. It will ask you one question at a time — motion type (sales-led, ecommerce or product-led) first, since that decides the metric set, then channels, data access (Google Sheet vs. live MCP connectors vs. both), the action layer per channel, and which metrics to track — then generate the file for you.

## Data sources

The template ships with hardcoded JS arrays (a snapshot). The intended architecture is **a Google Sheet as the database**, in three legs:

0. **Your own sheet.** Every real build gets one, created from `sheet-template/`. The three examples in this repo deliberately have none — their data is seeded demo data that renders on open, so there is nothing to refresh and nothing to persist. Their Changelog board is local to your browser by design; that is not a missing feature. No sheet ID is committed to this repo and no build reuses another's — a template shipping with someone's real spend data would leak it silently, because the dashboard renders perfectly either way.
1. **MCP → Sheet.** A scheduled agent session pulls each channel (Meta Ads, Google Ads, TikTok, LinkedIn, Amplitude, Stripe) and writes the sheet.
2. **The Sheet.** One tab per dataset, plus a `test_log` tab for the Changelog and a `platform_log` tab for platform changes. Everything a *person* authors — a corrected figure, a hypothesis, a re-labelled ad set — lives here and survives every rebuild.
3. **Sheet → HTML.** The refresh job regenerates the data arrays and republishes. It bakes rather than fetches, because a published artifact's CSP blocks external hosts outright, "publish to web" would put spend data on an unauthenticated URL, and no page on this dashboard shows an intraday number anyway.

One constraint worth knowing before you build the refresh job: the Google Drive connector can **create** a spreadsheet but **cannot update cells** in an existing one (`update_file` changes title and parent folder only, and there is no Sheets connector). So it is fine for step 0 — a one-time create — and unusable for step 1. Recurring writes need the Sheets API v4 with a service account held by the job, or an Apps Script web app you deploy on the sheet. `sheet-template/README.md` has the schema for the two tabs a human edits; the metric tabs are written by the refresh job and take their columns from that build's metric set. `skills/dashboard-builder.md` has the trade-offs.

## Fixed structure — what doesn't change per company

This template is intentionally opinionated about **layout**, not about **metrics**:

- 7 pages: Funnel, Monthly, Weekly, Daily, Creatives, Optimization, Changelog
- Reporting granularity is fixed: Monthly/Weekly = ad set/ad group level, Daily = campaign level, Creatives = ad level
- Changelog is a 4-column Kanban (To Test → Testing → Evaluated → Completed) with hypothesis/verdict tracking and a live platform-changes feed

What *does* vary per company: channel names, which KPIs populate the tables, and which motion you're tracking — sales-led (SAL/SQL/ICP/BTC/DPC), ecommerce (ATC/Checkout/Orders/ROAS/Refunds), or product-led (Signup/Activation/PQL/Trial-to-Paid/Expansion). All three metric sets are documented in the skill file, and each has a worked example in this repo.

Both three-channel examples also carry one **paused** ad group that still spent during the month, to show the Monthly/Weekly rule: period tables include everything that delivered in the window, not just what is live now. Filtering those rows out silently drops their spend and flatters every remaining row.

## Reporting windows

Fixed per page, and named in each page header so a screenshot is unambiguous:

| Page | Window |
|---|---|
| Funnel, Monthly, Optimization | Month to date |
| Weekly | Last 7 days (rolling, not the calendar week) |
| Daily | Last completed day |
| Creatives | Month to date by default, toggleable to last 30 days |

The Creatives toggle swaps a genuine second dataset (`CR1` / `CR1_L30`), not a scaled estimate, and drives every channel tab at once. Creative spend rolls up to channel spend within the same window — on L30 it will not tie to MTD Monthly, which is expected.

## Interactions worth knowing

- **Monthly / Weekly row click** drills into Creatives — it switches to that unit's channel tab and filters the ad table to that ad set or ad group, with a Clear filter button. Names are matched loosely, and the row's campaign is matched too, so search channels reporting creatives at campaign level still resolve.
- **Pencil button** on those rows logs a Changelog note (the row click is taken by the drill-down). On Daily, Creatives and Optimization the row click still opens the note modal.
- **Changelog card click** opens the card in the same modal for editing — note, hypothesis, variable, test setup, verdict, priority and column — and saves back to that card.
- **The Changelog board persists** to `localStorage`, so cards you add or move survive a reload. The saved copy is stamped with a fingerprint of the seeded cards it came from: rebuild the file with different seeds and the current seed wins, keeping only the cards you added yourself. A data refresh replaces the metrics, not the test log. "Archive Done" archives completed cards (with a restore link) rather than deleting them.
- **The board can be shared.** `localStorage` is per-browser and per-device, so on a file you open locally the board is yours alone — two people, or the same person on a laptop and a phone, each see their own. Published as a Claude Artifact with the `artifact` capability declared, the page instead reads a shared board from `data/board.json` next to itself, and a **Save to shared board** button beside "Archive Done" writes it back for everyone with edit access. Details in [Sharing the board](#sharing-the-board).
- **Duplicate** writes versioned ad IDs (`<control>_V2`) onto the card and moves it to Testing, and the card then shows a Control vs Test strip with spend, CTR and the motion's conversion rate.
- **That strip auto-populates.** Move a card into Testing and it finds the unit's ads in the creative data by itself — no ad IDs to type. Explicit IDs written by Duplicate take precedence; where no test ad exists yet the test column shows an em dash.
- It **reads the live ad rows**, looked up by `ad_id`, always on the MTD window so a card's numbers don't shift when the Creatives page is toggled to L30. The values stored on the card are a fallback for ad IDs that don't resolve — a campaign-level channel, a deleted ad — and the card labels itself "snapshot" when it uses them.
- Duplicated ads are **real creative rows** sharing their parent ad set's spend, so creative spend still rolls up to the channel total.
- **Platform change-log panels** under the board render from a `PLATFORM_LOG` map, one per channel, and fall back to an empty state for channels with no entries.
- **Pause** buttons appear on every table row, and dragging cards between Changelog columns works as expected.

## Adding a third channel

The blank template ships with two channels. Both three-channel examples show what changes — nothing about the page structure, only how many of each thing there are:

- **CSS** — `.funnel-grid` and `.hm-grid` go from `1fr 1fr` to `repeat(auto-fit,minmax(340px,1fr))` so the cards reflow instead of squashing, and a `.plat.c3` chip colour is added.
- **Markup** — one more funnel card and heatmap card on the Funnel page; one more tab and pane on Monthly, Weekly, Daily and Creatives; one more collapsible panel on the Changelog page.
- **JS** — one more entry per page in the `CONFIGS` registration list, one more `renderFunnel` / `renderHm` call, and the third channel's data arrays.

Use the "ad group" renderers (`renderGMW` / `renderGDaily`, which add Campaign and Status columns) for whichever channel is search-like, and the plain ones for the rest. In the ecommerce example that is Google Shopping (`c2`); in the product-led example it is Google (`c3`).

## Duplicate: creating the ad for real

A single HTML file cannot call Meta, Google, TikTok or LinkedIn directly — those APIs send no CORS headers, and an access token in the page would be a leaked credential. So Duplicate takes the best route available:

1. **MCP** — if the dashboard is open inside a host (Claude, or the Visualizer widget), it hands the job to the agent, which calls the connected ad-platform MCP server and reports the real ad ID back.
2. **Relay** — if you set a *Duplication relay URL* in Settings, it POSTs the request there. That endpoint is yours and holds the platform credentials server-side. A published artifact's CSP blocks outbound fetches, so this route works from a standalone file or your own host.
3. **Deep link** — always works, no credentials: it opens the native ads manager for that channel (from `AD_PLATFORM`, with your ad account ID substituted) and shows the exact payload to replicate.

Every new ad is requested **PAUSED**, and every route confirms first. The card then shows whether the test ad is `requested`, `created` or `failed`, and says "awaiting next data refresh" until an ad with that ID actually appears in the creative data — so a requested ad never reads as a measured one.

The ads-manager URLs in `AD_PLATFORM` are starting points. **Verify them against your own accounts** — account-ID parameters differ per platform and the URL shapes change.

## Where to run it

Two options. This decides what the dashboard can do more than anything else you pick.

| | Cost | Shared board | Reads the sheet live | Can act on ads | Setup |
|---|---|---|---|---|---|
| **An AI link** | free | On Claude yes, with a Save button. Elsewhere no | No — baked in at each refresh | Hands the job to the assistant, or opens the ads manager | Publish it |
| **A server with a database** | monthly bill | Yes — live, no Save button | Yes | Yes, directly | A real build |

**An AI link** — publish it and share the URL. No infrastructure, no cost, opens on a phone.

The page can't reach outside itself on these hosts, so the numbers are baked in at each refresh rather than read live. And the shared board is **Claude-specific**: it works through a hook (`claude.use('artifact')`) that Claude provides and other assistants don't. On Claude, everyone with edit access shares one board and the last Save wins. On another assistant the dashboard still renders correctly, but the board goes back to one per person, per browser. Worth knowing before you promise a teammate a shared board.

**A server with a database** (Railway, Render, Fly) — everything works properly: several people editing the board at once with no Save button and no conflicts, live data, credentials held on the server so ad actions run for real, and the refresh job on a schedule beside it. **This repo doesn't ship that** — there's no server or database code here, so this route means building that part.

Two questions settle it: **how many people edit the board at once**, and **should the dashboard act on ads or just report on them**. A few people who mostly read → an AI link. A daily-driver for a team, several editors, or anything where a wrong click spends money → a server, so credentials are never sitting in the page.

Opening the HTML file directly is how you *look* at a build — a demo, a client deliverable, a check before publishing. It isn't one of the two options above, because nothing is shared and nothing refreshes.

## Sharing the board

The Changelog is the one part of the dashboard people write to, so the board stays editable in the dashboard — drag-and-drop, click-to-edit. The only choice is whether it is *also* copied out to your sheet.

**Board only.** Cards live in the published dashboard. Everyone with edit access on the link sees the same cards, nobody needs a Google account, and the last person to press Save wins if two people edit at once. Nothing outside the dashboard can read the log.

**Board, mirrored to the sheet.** Same editing, plus the refresh job copies the board into the `test_log` tab each run — so you get a searchable, backed-up history that people can read without opening the dashboard. The mirror is **one-way**: the sheet is a copy, and anything typed into it is overwritten on the next run.

Mirroring is the better default when you have a sheet and a refresh job. Either way the board needs `capabilities: {artifact: {}}` declared at publish time — without it the Save button never appears and the board silently falls back to local-only, which looks identical to the feature not existing.

**This all describes the AI-link route.** On a server the board lives in the database instead: it saves as you drag, several people can edit at once, and there is no Save button and no conflict to resolve. The rest of this section — the Save button, last-Save-wins, the capability declaration — applies only to a published artifact.

Why not make the sheet the source and skip the board? Because a web page cannot write cells into a Google Sheet — the Drive connector only changes a file's title and folder, and a published page is blocked from calling the Sheets API directly. A sheet-sourced log means a **read-only** board: every change made in Sheets, no dragging. That trades away the reason a board exists.

**Local file — `localStorage`, one board per browser.** Open the HTML from disk, a static host, or a `file://` path and each viewer gets their own board. Nothing is transmitted. Two people looking at the same URL do not see each other's cards, and the same person on a laptop and a phone has two boards. That is a property of `localStorage` (per-origin, per-browser, per-device), not a bug — but it means a local file is not a team board.

**Published Artifact — one board for everyone with edit access.** Publish the page as a Claude Artifact with the `artifact` capability declared:

```
capabilities: {artifact: {}}
```

On load the page calls `claude.use('artifact')`. If it resolves, it fetches `data/board.json` alongside itself and adopts the team's board; anything that viewer changed and hasn't saved is layered back on top so an in-progress edit is never lost to someone else's save. A **Save to shared board** button appears next to "Archive Done", enabled only when the local board differs from the published one, and writing goes through the *files* form of `publish`:

```js
artifact.publish({'data/board.json': {content: JSON.stringify(doc), contentType: 'application/json'}})
```

The files form names only that one path, so the page itself is carried over untouched and the saving view keeps running — the `html` form would rebuild and reload the whole document. Saving is **never automatic**: no publish per keystroke, per drag, or per edit. A person presses the button.

The state line beside the button says which path is in effect:

| Line | Meaning |
| --- | --- |
| `local to this browser` | No artifact host. Your board, your device. |
| `shared board · up to date` | Reading and writing the team board; nothing unsaved. |
| `unsaved changes` | Your edits aren't on the shared board yet. |
| `read-only — your board stays in this browser` | You can view the artifact but not write to it (`not_granted` / `not_writer`). Edits still persist locally. |
| `someone else saved first — reload to see their board` | A `conflict`. Reload, then reapply. |

Two caveats worth knowing before you rely on it. Saves are last-writer-wins per save, not per card: two people editing at once, both pressing Save, means the second overwrites the first — the conflict state catches the common case but the window is a real one, so treat the board like a shared doc, not a live-sync app. And a published artifact is only shared with the people you share it with; the board inherits exactly those permissions, no more.

## Live actions (pause/resume/duplicate ads)

The Settings modal (gear icon, top right) accepts an API key, an ad account ID, and an optional duplication relay URL. **Never commit real API keys to this repo or any fork of it**, and note that nothing typed into Settings is persisted — not to `localStorage`, not anywhere. It lives in the page for that session only, which is verified by a test that types a sentinel into both credential fields, saves, and asserts it never reaches storage.

## Testing a build

`skills/dashboard-builder.md` carries a pre-ship checklist covering data completeness, sourcing, interactions, responsive behaviour and pre-ship greps. The short version: page-level horizontal overflow must be zero at 375px, every clickable thing needs a handler, label counts must be computed from the data rather than typed, and one aggregate should be spot-checked against its line items (creative spend rolls up to channel spend in all three examples).

## Contributing

This is a community template — PRs welcome for additional channel integrations, metric presets, or bug fixes. Please don't submit PRs containing real company data or credentials.

## License

MIT — see `LICENSE`.
