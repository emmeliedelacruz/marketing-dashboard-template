# dashboard-builder

**Changelog**
- 2026-08-22: Initial creation.
- 2026-08-22: Views locked to match the original source dashboard exactly (that file is not in this repo; `dashboard-template-blank.html` supersedes it). Onboarding scoped to channels, access/action, and per-timeframe metrics. Dimensions fixed per timeframe. Refresh cadence fixed to automatic daily.
- 2026-08-23: Onboarding asked one question at a time, not as a batch table. Added motion-type question (Product-led vs Sales-led). Clarified: template (pages, tabs, views, layout) is identical across companies; only company name, channels, and KPIs/metrics vary.
- 2026-08-23: Documented reporting granularity per page — Monthly/Weekly = ad set/ad group level, Daily = campaign level, Creatives = ad level. KPIs vary by company/motion type; only granularity level is fixed.
- 2026-08-23: Adopted richer 4-column Kanban spec (Duplicate/Hypothesis/Verdict) as canonical, replacing an earlier simpler 3-column version. Pause and unpause both fire live MCP calls (symmetric).
- 2026-08-23: Added Product-led metric definitions (parallel to Sales-led set) so the motion-type onboarding question has concrete defaults for both paths.
- 2026-08-23: Added Ecommerce as a third motion type with its own metric definitions and funnel-stage analog. Worked examples now ship for all three motions (`dashboard-example-filled.html` sales-led, `dashboard-example-ecommerce.html`, `dashboard-example-product-led.html`).
- 2026-08-23: Channel count is two **or three** — the funnel and heatmap grids reflow, and every per-channel surface (tabs, panes, platform-changelog panels) repeats per channel. Both three-channel examples use index-based array names (M1/M2/M3 …) rather than channel initials.
- 2026-08-23: Fixed the Monthly/Weekly row-inclusion rule — period tables show every unit that was active at any point in the window, including paused and deleted ones, scoped by delivery rather than current status.
- 2026-08-23: Specified the two navigation behaviours — Monthly/Weekly row click drills into that unit's creatives, and clicking a Changelog card opens it for editing. Note logging moves to a per-row pencil button on Monthly/Weekly.
- 2026-08-23: Added the pre-ship checklist, a `hostPrompt()` guard for host callbacks, and `syncTabCounts()` so label counts are computed from data rather than typed.
- 2026-08-23: Fixed the reporting window per page — Monthly is month-to-date, Weekly is the last 7 days, Daily is the last completed day, and Creatives defaults to MTD with a toggle to last 30 days. Windows are now named in every page header.
- 2026-08-23: Brought the Changelog up to this spec — perf strip now carries Spend/CTR/conversion rate, ad IDs are shown, Test Setup survives into Evaluated, Duplicate writes real versioned ad IDs, Archive archives instead of deleting, the board persists to localStorage, drag has visual feedback, and the platform change-log panels actually render entries.
- 2026-08-23: Specified where the perf strip's numbers come from — live lookup of each ad ID against the creative data's MTD rows, with the card's stored values as a labelled fallback. Creative rows carry `ad_id`, and a duplicated ad is a real row sharing its parent ad set's spend.
- 2026-08-23: The perf strip auto-populates from the unit's ads once a card is past To Test, so a card moved into Testing by hand is never blank. Explicit ad IDs from Duplicate still take precedence.
- 2026-08-23: Duplicate now actually creates the ad — MCP through the host, else an operator-run relay, else a deep link into the native ads manager. Always PAUSED, always confirmed, and the card reports requested / created / failed.
- 2026-08-23: The Changelog board is shareable. On a local file it stays in `localStorage` — one board per browser, per device — and the docs now say so plainly. Published as an artifact with `capabilities: {artifact: {}}`, the page reads and writes one team board at `data/board.json` behind an explicit "Save to shared board" button.
- 2026-08-23: The Google Sheet is the database. Specified the three legs — MCP populates the sheet on a schedule, the sheet holds every dataset plus the human-authored fields, and the HTML is baked from it rather than fetching at load. Records the write constraint that shapes the whole thing: the Drive connector can create a sheet but cannot update one, so the refresh job needs Sheets API v4 or an Apps Script endpoint.
- 2026-08-23 (latest): Added `docs/railway-setup.md` — the spec and runbook for the server route: what `/healthz` has to report, why both `DATABASE_URL` and the `PG*` variables must work, why a connection string must never be assembled by hand, the store precedence, the two passwords, and the click order. The repo still ships no server code; the doc is the handoff.
- 2026-08-23: Two deployment options, not four — an AI link or a server with a database — chosen by how many people edit the board at once and whether the dashboard should act on ads or only report. Records that the shared board runs on a Claude-specific hook: on any other assistant the page renders but the board is one-per-person, so that must be said rather than assumed.
- 2026-08-23: The board stays editable, always. A sheet-sourced test log would mean a read-only board — a page cannot write cells to Sheets — so that option is struck and the choice is now only whether the board is *also* mirrored out to the `test_log` tab. The mirror is one-way, which has to be said out loud or someone types into the copy and loses it.
- 2026-08-23: One sheet per build, provisioned at onboarding. This is a template repo, so no sheet ID is ever committed and no build reuses another's data. Split provisioning (a one-time `create_file`, which the Drive connector handles fine) from the recurring refresh write (which it cannot do at all). `sheet-template/` pins only `test_log` and `platform_log` — the tabs a human edits; metric columns are derived per build. The examples get no sheet: demo data, nothing to refresh, nothing to persist.
- 2026-08-23 (latest): Fixed the pointers an AI actually follows. The Purpose line told it to clone `hearth-dashboard-v2.html`, which is not in this repo, and the design step sent it to a `frontend-design` skill that does not ship here — both resolved to nothing. Added a file-by-file table of what to read and an explicit rule never to invent names for the data arrays, label triples or renderers, since the drill-downs, sorts and `ad_id` joins all key off the exact names in the examples. Onboarding's three technical questions now ask what a marketer can answer and leave the mechanism to the builder.

**Name:** `dashboard-builder` — *use when building a custom HTML performance dashboard for a company/product, sourcing from a Google Sheet, direct MCP pull, or both.*

**New vs. update:** New skill.

---

## Purpose / When to use
Build a single-file HTML performance dashboard by cloning this repo's template exactly — same pages, tabs, views, layout — re-skinned with a company's own channels and KPIs. Triggers: "build me a dashboard," "make a Hearth-style dashboard for [company]," "dashboard from this sheet."

**Read the repo files before writing anything.** This skill specifies *what* the dashboard contains and *why*; the files are the authority on *how* it is wired:

| File | Read it for |
|---|---|
| `dashboard-template-blank.html` | The starting point. Copy this, then fill it in |
| `dashboard-example-filled.html` | Sales-led, 2 channels — the reference for data shapes, `*_LABELS`/`*_KEYS`/`*_RIGHTS` triples, the `render*` functions, `CONFIGS`, `DRILL` |
| `dashboard-example-ecommerce.html` | Ecommerce, 3 channels — and the index-based naming (`M1`/`M2`/`M3`) to copy for a third channel |
| `dashboard-example-product-led.html` | Product-led, 3 channels |
| `README.md` | The array-by-array and config-by-config reference for what to fill in |
| `sheet-template/README.md` | The `test_log` and `platform_log` column-to-field mapping |
| `docs/railway-setup.md` | Only if they chose the server route |

Never invent names for the data arrays, label triples or renderers — take them from the example that matches the motion type. Inventing them produces a dashboard that renders and is wired to nothing: the drill-downs, the sort buttons and the Changelog's `ad_id` joins all key off these exact names.

## Fixed template (identical across every company, never asked)
Funnel → Monthly → Weekly → Daily → Creatives → Optimization → Changelog.

## Fixed reporting granularity per page (not user-configurable)
| Page | Level |
|---|---|
| Monthly | Ad set / ad group |
| Weekly | Ad set / ad group |
| Daily | Campaign |
| Creatives | Ad |

KPIs/metrics shown at each level vary by company and motion type (per onboarding) — only the level of granularity, and the reporting window below, are fixed across every dashboard built with this skill.

## Fixed reporting window per page (not user-configurable)
| Page | Window | Notes |
|---|---|---|
| Funnel | Month to date | Matches Monthly, so the funnel and the monthly table reconcile |
| Monthly | **Month to date** | Not a rolling 30 days, and not last calendar month — MTD, so it moves with the month you are in |
| Weekly | **Last 7 days** | A rolling 7 days, not the current calendar week — a Tuesday check-in on a calendar week is reading two days of data |
| Daily | **Last completed day** | The most recent *finished* day. Today is still filling up and reads as a collapse |
| Creatives | **Month to date by default, toggleable to last 30 days** | MTD is the default so it lines up with Monthly; L30 exists because ad-level volume is thin and a 23-day month often cannot separate two creatives |
| Optimization | Month to date | Same window as the tables its recommendations are drawn from |
| Changelog | n/a | Cards carry their own dates |

Two rules that follow from this:

- **State the window on the page.** Every page header names its window ("Ad set / ad group level · Month to date"). A screenshot of a table with no window on it is unreadable a week later, and it is the fastest way for two people to argue about numbers that were never comparable.
- **Never compare across windows.** Creative spend rolls up to channel spend *within the same window*. When the Creatives page is on L30 and Monthly is on MTD they will not tie, and that is correct — do not "fix" it by scaling one to the other.

**The Creatives toggle:** each window is its own dataset (`CR1` / `CR1_L30`, and so on), registered on the table's config as `windows:{mtd:…, l30:…}` and swapped by `setCrWindow()`. Never derive one window by scaling the other — a modelled number that looks like a measured one is worse than no number. The toggle governs every channel tab on the page at once, and any active drill-down filter survives it.

## Data pipeline — the Google Sheet is the database

The dashboard is a **rendered view of a sheet**, never a place data lives. Three legs, and the middle one is where the available connectors run out — so it is specified here rather than assumed.

**One sheet per build.** Every company gets its own, provisioned at onboarding.

**Examples and the blank template are the exception — they get no sheet at all.** `dashboard-example-*.html` carry seeded demo data whose whole job is to render on open, with nothing to refresh and nothing worth persisting; `dashboard-template-blank.html` is a starting point, not a build. Do not provision a sheet for them, do not declare the `artifact` capability on them, and do not treat their local-only Changelog board as a gap. A demo that quietly writes to a real sheet is worse than one that forgets. Never reuse another build's sheet, never commit a sheet ID to this repo, and never let the template ship pointing at anyone's real data — the file ID belongs in the generated dashboard's config and the refresh job, nowhere else.

### Leg 0 — provisioning the sheet (once, per build)

Create the sheet with the Drive connector — `create_file` with `textContent` as CSV and `contentMimeType: 'text/csv'`, which Drive converts to a native Sheet — one call per tab. This is the one place `create_file` is exactly the right tool: it is a one-time create, so the new-file-ID problem below does not apply. Report the file ID back to the person and hand them the tab list.

`sheet-template/` pins the header rows for `test_log` and `platform_log` only — the two tabs a human types into, whose columns must match the board's data model exactly. **Derive the metric tabs' columns from that build's own data arrays**, not from a stored list: they follow the motion type and the chosen metrics, and a second copy of them in the repo would only drift.

If they already have a sheet, read it and reconcile against the schema before building — a missing `ad_id` column is the failure that shows up much later as a Changelog card with no stats.

### Leg 1 — MCP → Sheet (the refresh job)

Runs on a schedule in an agent session, never in the page. Pull each channel from its ad-platform MCP server, normalise to the tab schema, write the sheet.

**The recurring write is the constrained leg.** The Google Drive connector can *create* a spreadsheet — `create_file` with `textContent` as CSV and `contentMimeType: 'text/csv'`, which Drive converts to a native Sheet — but it **cannot update cells in an existing one**. `update_file` changes title and parent folder only, and there is no Google Sheets connector in the directory. Pick a write path explicitly and record which one this build uses:

| Path | Trade |
|---|---|
| **Sheets API v4 `spreadsheets.values.update`, service-account credential held by the refresh job** *(default)* | Stable file ID, true in-place update, credential stays server-side and never enters the page |
| **Apps Script web app deployed on the sheet, POSTed by the job** | No service account to provision; you own and maintain the endpoint |
| **`create_file` per refresh** | Needs nothing but the Drive connector, but yields a *new file ID every run*, orphaning the sheet the dashboard was built from. Fine for Leg 0, never for a refresh |

### Leg 2 — the Sheet

One tab per dataset, header row = field names, one row per entity. `sheet-template/README.md` pins `test_log` and `platform_log`; the metric tabs follow the build's metric set. The tabs are:

| Tab | Grain | Key columns |
|---|---|---|
| `monthly_<channel>` | Ad set / ad group, MTD | `unit, campaign, status, spend, impressions, clicks, ctr, <motion metrics>` |
| `weekly_<channel>` | Ad set / ad group, last 7d | same |
| `daily_<channel>` | Campaign, last completed day | same + `date` |
| `creatives_<channel>_mtd` / `_l30` | Ad | `ad_id, name, ad_set, spend, impressions, clicks, ctr, <motion metrics>` |
| `test_log` | Changelog card | `id, column, unit, context, priority, hypothesis, variable, setup, verdict, control_ad_id, test_ad_id, archived` |
| `platform_log` | Platform change | `channel, when, who, what, detail, kind` |

**Every creative row needs `ad_id`** — that is the join the Changelog perf strip depends on.

The sheet is the audit layer *and* the human-editable one. A corrected spend figure, a re-labelled ad set, a hypothesis someone typed on their phone — all live here and survive every rebuild. **Anything a person authors belongs in the sheet, not in the HTML.** That is the actual argument for this architecture, more than freshness.

### Leg 3 — Sheet → HTML

**This follows the deployment choice above.**

**On an AI link, baking is the only option.** The refresh job reads the sheet, regenerates the data arrays, republishes. A published artifact's CSP blocks *every* external host but Google Fonts, so `fetch()` to `docs.google.com` does not fail gracefully — it does not run at all. Two things make this less of a loss than it sounds: "publish to web" would put a spend-and-CAC table on an unauthenticated URL, and nothing on any page is intraday anyway — every window is MTD, last-7, or last-completed-day, so a live read buys nothing a daily bake doesn't already give.

**On a server, read the database.** The refresh job writes to it; the page reads from it. The sheet becomes the mirror people read and edit outside the app, not the thing the page queries at load.

One fetch route worth knowing, for the case that genuinely wants a live sheet read:

- **Published CSV / `gviz`** — `.../pub?output=csv` or `/gviz/tq?tqx=out:json`. Both send CORS headers, so a plain `fetch()` works from a server. Blocked on an AI link. The cost is that publishing to web makes the sheet readable by anyone holding the URL — on a server, query the database instead.
- **The artifact `mcp` capability** — the one way to read live data on a Claude link: declare `capabilities: {mcp: {servers: [{server: 'Google Drive', tools: ['search_files','read_file_content']}]}}` and read through `watchTool`. Runs on the *viewer's* credentials, so no key in the page and no CSP problem. Costs: every viewer needs the Drive connector and access to the sheet, and a page declaring `mcp` cannot be shared publicly. Claude only.

**A caveat that decides the route where cells matter:** `read_file_content` on a Sheet returns a *markdown table*, not structured cells — it escapes characters, and its own tool docs say the representation will change over time. An agent parsing that at build time is fine, because it re-reads and adapts. A *page* regex-parsing it at runtime is building on a format with no contract. Where you need real cells, use the CSV export, not the natural-language read.

### Where the dashboard lives — ask, don't assume

Two options. Ask before building, because this decides what the dashboard can do more than anything else in onboarding.

| | Cost | Shared board | Reads the sheet live | Can act on ads | Setup |
|---|---|---|---|---|---|
| **An AI link** | free | On Claude yes, with a Save button. Elsewhere no | No — baked in at each refresh | Hands the job to the assistant, or opens the ads manager | Publish it |
| **A server with a database** | monthly bill | Yes — live, no Save button | Yes | Yes, directly | A real build |

**An AI link** — publish it as an artifact and share the URL. No infrastructure, no cost, works on a phone.

- The page cannot reach outside itself on any of these hosts, so the numbers are **baked in at each refresh**, not read live.
- **The shared board is Claude-specific.** It works through `claude.use('artifact')`, which exists in Claude and has no equivalent in other assistants' canvas/preview surfaces. On Claude, declare `capabilities: {artifact: {}}` and everyone with edit access shares one board, last Save winning. On any other host the page still renders correctly but the board falls back to **one per person, per browser** — so if they name a non-Claude assistant, say that plainly rather than promising a shared board. Do not guess what a given host allows; if it matters to them, verify it on that host before shipping.
- Ad actions route through the assistant, or fall back to opening the native ads manager.

**A server with a database** (Railway, Render, Fly) — everything works properly: several people editing the board at once with no Save button and no conflicts, the sheet or database read live, credentials held server-side so ad actions run for real, and the refresh job on a schedule beside it. The board moves to Postgres and the `NOTES` array becomes the table's **seed**, not the live board.

**The repo ships no server code** — `docs/railway-setup.md` is the spec and the runbook, not an implementation. Say that before they commit to the route. Follow that doc; these five are the ones that actually decide whether it works, and each came from a real failure:

1. **`/healthz` must separate "no variable arrived" from "connection failed."** `dbConfigured` false means platform config; `dbConfigured` true with a `dbError` means credentials, host or TLS. Collapse them into one "db: down" and both look identical from outside while needing different fixes.
2. **Accept `DATABASE_URL` *and* the five `PG*` variables.** Some Railway Postgres services publish only `PG*`, and a reference to a variable the target service never publishes resolves to an **empty string with no error**.
3. **Never assemble a connection string from the parts.** Any password containing `@ : / # ? %` produces `Invalid URL` and a silent fall back to the memory store. Let `pg` read `PG*` natively instead.
4. **Store precedence Postgres → file → memory, and never ship on file.** Railway rebuilds the container filesystem on every deploy, so file writes reset silently — correct locally, wrong in production.
5. **Two passwords, view and edit.** While both are unset the deploy is public, spend figures included. Report whether auth is on, so the page never implies protection it does not have.

**Attaching a database does not migrate anything.** It seeds an empty table from the code's seed array; whatever was written to the file or memory store is stranded on a dead container disk. Export the board first (`GET /api/cards`), bake it into the seed, then switch — and warn them before, not after.

Steer by two questions, not by preference: **how many people edit the board at once**, and **should the dashboard act on ads or just report on them**. A few people who mostly read → an AI link. A daily-driver for a team, several editors, or anything where a wrong click spends money → a server, so the credentials are never in the page.

Opening the HTML file directly is how someone *looks* at a build — a demo, a client deliverable, a check before publishing. It is not one of the two options above, because nothing is shared and nothing refreshes.

### Where the Changelog board lives — ask, don't assume

The metrics always come from the sheet. The **test log is different**: it is the one thing people write rather than read, so the board stays editable in the dashboard either way. The question is only whether it is also mirrored out to the sheet.

**Never make the sheet the source for the test log.** A page cannot write cells into a Google Sheet — the Drive connector updates a file's title and folder only, and an artifact's CSP blocks calling the Sheets API directly. So a sheet-sourced test log means a read-only board: no drag-and-drop, no card editor, every change made in Sheets. That trades away the interaction the board exists for, in exchange for durability you can get from Option B instead. Do not offer it.

**Option A — the board is the log.** Cards live in the published dashboard at `data/board.json`.
- Drag-and-drop and click-to-edit, which is the point of a board.
- Everyone with edit access on the link sees the same cards.
- No Google account or sheet access needed.
- Only works on the published dashboard — a downloaded copy falls back to local-only.
- Last Save wins if two people edit at once. Fine for a small team, wrong for a busy one.
- Nothing outside the dashboard can read the log.

**Option B — the board is the log, mirrored to the sheet.** Same editing, plus the refresh job copies the board into the `test_log` tab on each run.
- Everything in Option A, plus a searchable, backed-up, versioned history.
- People who never open the dashboard can read the log.
- The mirror is **one-way**. The sheet is a copy: edits made there are overwritten on the next run. Say this plainly, or someone will type into it and lose the work.
- Costs one more write in the refresh job.

Default to **Option B** when the build has a sheet and a refresh job, since the mirror is nearly free and the durability is real. Option A is right for a build with no sheet at all.

**Both options require `capabilities: {artifact: {}}` at publish time.** Without that declaration `claude.use('artifact')` returns `null`, the Save button never appears, and the board silently falls back to local-only — the failure is indistinguishable from the feature not existing.

**Examples and demo builds** → `localStorage`, no capability declared. One board per browser, per device, shared with nobody. Say that plainly; never call it "persistent" without saying to whom.

## Fixed page specs

**Funnel:** reflowing grid (`repeat(auto-fit,minmax(340px,1fr))`), one funnel card per channel — two or three across (Impressions→Clicks→Submits→SAL→SQL→ICP→BTC→DPC→Sold for sales-led; see Product-led analog below). Below: 2-col heatmap grid, one table per channel, verticals/segments as rows.

**Monthly / Weekly:** Channel tabs. Per channel: single sortable table — Name, $/Sold, Won $, RoS, Sold, Spend, $/SAL, SAL, ICP%, BTC%, DPC%, Stuck%, Net EV (+pause column; Google adds Status + Campaign). Row click on either page **drills into Creatives**: switch to the Creatives page, select that unit's channel tab, and filter the ad table to the ads belonging to that ad set / ad group, with a "Showing ads in X" bar carrying a Clear filter button. Match names loosely (lowercase, strip punctuation) — ad set, ad group and campaign names rarely agree on underscores versus spaces — and match on the row's campaign as well as its own name, since search channels report creatives at campaign level. Show a real empty state when nothing matches, never an empty table. Because the row itself is now a link, logging a note moves to a pencil button in the actions column beside Pause.

**Row inclusion — Monthly / Weekly (fixed):** show every ad set / ad group that was **active at any point during the period**, not just the ones live right now. A unit that spent for three weeks and was paused on the 22nd still owns that spend for the month; filtering by current status silently drops its cost, understates blended CAC and overstates efficiency for everything left in the table. Include paused, ended, budget-exhausted and deleted units as long as they delivered in the window, and render them with the Paused chip and 40% opacity rather than hiding them. Scope rows by *delivery during the window*, never by *current state*. Daily is the exception: it reports the most recent day, so only units that delivered that day appear.

Watch out that the plain ad-set renderer (`renderMW`) has no Status column — only the ad-group renderer (`renderGMW`) does. If a non-search channel carries paused-but-spending units, add a Status column to the plain renderer too, or the rows read as live.

**Daily:** Channel tabs (campaign-level). Table: Name, (Status if applicable), CRM Sync%, Submit%, SAL, $/SAL, ICP%, BTC%, DPC%, Stuck%, Net EV (+pause column). No Won$/Sold/RoS — attribution lag makes them misleading daily.

**Creatives:** one tab per channel plus Ad Library, with a page-level window toggle (Month to date / Last 30 days) above the tabs, defaulting to MTD. Ad-level table: Ad Creative, Format, Copy Angle, Impr, CTR, %Submit, ICP%, DPC%, $/SAL, Stuck% + sort bar. Campaign-level tab: same funnel metrics, rank-by bar. Ad Library: filter buttons (All/Video/Image) + card grid (media, title, body, CTA, Ad Library link).

**Optimization:** Net-row banner (Scale-Up Potential / Weekly Bleed to Stop / Net Weekly Opportunity + rule callout). Two tables: Scale Up, Pull Back — Unit, Platform, EV/Spend, $/SAL, Spend, SAL, Net EV $. Row click → Changelog note modal. Unit name link → ad preview modal + "Open in Ads Manager."

**Changelog (fixed, richer spec):**
- Header with "Archive Done" button
- **4-column Kanban:** To Test → Testing → Evaluated → Completed. Drag-and-drop.
- **Card fields:** Note (always) · Hypothesis (always, styled distinctly) · Variable Being Tested (always) · Test Setup (visible on anything past To Test — a card that has been evaluated still needs to show how it was set up) · Control vs Test perf strip, three rows: Spend, CTR and the motion's conversion rate (`PERF_RATE_LABEL` / `PERF_RATE_KEY` — Submit% / ATC% / Signup%), **auto-populated on any card past To Test** · Ad IDs, control and test, shown with it · Verdict (visible once Evaluated/Completed)
- **The strip populates itself.** A card put into Testing by hand carries no ad IDs, and a blank strip on a card that says it is testing something reads as broken. Once a card is past To Test, `resolveUnitAds()` finds the unit's ads in the creative data — matching the card's unit against `ad_set` (or ad `name`), taking the highest-spending ad with no `_V<n>` suffix as the control and the highest `_V<n>` as the test. Explicit ad IDs written by Duplicate always win over this. Where no test ad exists yet the test column shows an em dash, which is the honest reading: nothing has been duplicated. Auto-matched cards label themselves "matched from ad set", so an inferred pair is never mistaken for a recorded one.
- **Where the numbers come from.** However the ad IDs were resolved, the strip looks each one up in the creative data by `ad_id` at render time and reads its **MTD** row. It always reads MTD, whatever the Creatives page is toggled to — a card's numbers must not change meaning because someone switched that page to L30. Values stored on the card (`controlSpend`, `controlCtr`, `controlRate` and their test counterparts) are a **fallback only**, used when an ad ID does not resolve: a campaign-level channel that has no ad rows, a deleted ad, or an ad that did not run this month. The card labels itself "snapshot" when it falls back, so nobody mistakes a frozen number for a live one. Do not store live metrics on the card as the primary source — that is the hardcoded snapshot this skill tells you not to build.
- **Duplicated ads are real rows.** A test ad is an ad: it appears in the creative array with its own `ad_id` (`<control>_V<n>`), the same `ad_set`, and its share of the parent's spend. The parent ad set's spend is **split** across control and test, so creative spend still rolls up to the channel total. An ad set under test with a single creative row holding the full spend is a sign the test ad was never added to the data.
- **Card actions:** Duplicate (creates the ad on the platform — see below) · Complete (→ Completed) · Delete (permanent)
- **Duplicate actually creates the ad.** A browser page cannot call an ad platform API directly: the platforms send no CORS headers, and an access token pasted into the page is a leaked credential. So Duplicate routes through the best channel available, in this order, and records which one it used:
  1. **MCP** — inside a host, `sendPrompt()` hands the job to the agent, which calls the connected ad-platform MCP server. The prompt must name the platform, ad account, control ad ID, new ad ID and name, say **PAUSED**, and ask for the real platform ad ID back.
  2. **Relay** — POST the request to an endpoint the operator runs, which holds the credentials server-side. Configured in Settings, and it is *their* endpoint, never a platform URL. Note that a published artifact's CSP blocks this; it works in a standalone file or a host that permits the origin.
  3. **Deep link** — always available and needs no credentials: open the native ads manager (`AD_PLATFORM[channel].manager`, with `{account}` substituted from Settings) so a human finishes it. The request payload is shown so it can be replicated exactly.
- **New ads are created PAUSED.** A dashboard button that starts live spend without anyone reviewing the creative is not a feature. Confirm before executing, always.
- **The card tells you the truth about state.** Duplicate writes the intended `<control>_V<n>` and marks the test ad *requested*, *created* or *failed*. Until an ad with that ID actually appears in the creative data the card says so — created ads read "awaiting next data refresh", not as though the numbers were real. The marker keys on the **test** ad: a control that resolves says nothing about whether the duplicate exists.
- **Archive Done archives, it does not delete.** Completed cards get an `archived` flag, leave the board and stay in `NOTES`, with a restore link beside the button. A button labelled Archive that silently deletes is the kind of thing people only discover once.
- **The board persists.** It is the one part of the dashboard a person authors by hand, so it is saved to `localStorage` on every change and restored on load — a data refresh replaces the metrics, never the test log. Wrap every storage access in try/catch: private windows and blocked-storage browsers throw on access, and an unguarded read takes the whole page down. Credentials still never go near storage.
- **Make the board shareable when the host allows it.** `localStorage` is per-browser and per-device, so on a local file the board is one person's, and calling that "persistent" without saying "to you, on this machine" oversells it. When the page is published as a Claude Artifact with `capabilities: {artifact: {}}`, call `claude.use('artifact')` on load (branch on `null` — absence is the normal case), fetch `data/board.json` beside the page, adopt the team's board, and layer the viewer's unsaved local edits back on top so a save elsewhere never eats work in progress. Write with the **files** form — `publish({'data/board.json': {content, contentType:'application/json'}})` — which names one path and leaves the running view alone; the `html` form rebuilds and reloads the whole document. Saving is an explicit button ("Save to shared board"), never automatic: no publish per keystroke, per drag, per edit. Handle each error code as its own state — `not_granted`/`not_writer` → read-only, keep saving locally; `conflict` → tell them someone saved first and to reload; `rate_limited` → back off. Surface which path is live in a state line next to the button, and derive "unsaved changes" by comparing fingerprints rather than setting a flag inside the render function, which fires on load and makes a fresh viewer look dirty.
- **Stamp the saved board with a fingerprint of the seed it came from** (`seedFingerprint()` over `NOTES`, stored as `{v, notes}`). Restore a saved board wholesale only when that fingerprint matches; otherwise take the current seed and carry over only the cards the viewer added, matched by id. Skip this and anyone who opened an earlier build keeps seeing that build's cards forever, never picking up a single correction — and it is invisible in testing, because a fresh browser has nothing stored.
- Adding a card: the pencil button on a Monthly/Weekly row, or a row click on Daily, Creatives or Optimization → note modal, which sets note, hypothesis, variable, test setup, verdict, priority (High/Med/Low) and starting column
- **Opening a card: click it.** The card reuses the note modal in edit mode, pre-filled with note, hypothesis, variable, test setup, verdict, priority and column, saving back to the same card rather than creating a new one. Card buttons (Duplicate / Complete / Delete) must stop propagation so they don't also open the editor, and a drag must not count as a click.
- Below board: one collapsible "Platform Changelog" panel per channel, each with an addressable body (`cl-body-<channel>`) rendered from a `PLATFORM_LOG` map — entries carry when / who / what / detail plus a `kind` (budget, status, creative, targeting, bid) that picks the icon and tint. Sync pulls the last ~30 activity entries via MCP and re-renders. A channel with no entries shows the empty state.
- **Pause button** (every row, always visible): live MCP call to pause by name. **Unpause: symmetric live MCP call to resume** — both need confirm-before-execute.
- Settings modal (gear icon): API key + account ID for live actions. Never store credentials in browser storage.

## Onboarding — ask one question at a time, wait for each answer
1. Motion type: Sales-led, Ecommerce, or Product-led? *(Determines relevant KPIs — template stays identical, only metric set changes.)*
2. Channels to track? *(Two or three. Beyond three the tab rows and funnel grid stop being readable — push back and suggest folding the smallest channel into a combined view.)*
3. "Do you want me to make a Google Sheet to hold the numbers, or do you already have one?" *(Every build gets its own — never reuse another company's, never hardcode a sheet ID. If creating, provision it and report the file ID back.)* Then: **"Once it's set up, would you rather paste the numbers in yourself, or have them update automatically each day?"** Automatic needs a one-time developer setup (Sheets API with a service account, or an Apps Script endpoint) — say that plainly and offer manual as the perfectly good starting point. Do not ask them to choose between those two mechanisms by name.
4. Where will this run — an AI link, or a server with a database? *(See **Where the dashboard lives** below. It decides whether the board can be shared, whether the numbers can be read live, and whether ad actions can run for real. If they name a non-Claude assistant, tell them the shared board will not work there. Ask before building, not after.)*
5. Should the test log be mirrored to the sheet, or live only in the dashboard? *(The board is editable either way — see **Where the Changelog board lives** below. Default to mirroring when there is a sheet and a refresh job. Either way, declare `capabilities: {artifact: {}}` at publish and verify the shared-board state line.)*
6. Per channel — **"Where should this channel's numbers come from: the sheet, or straight from the ad platform?"** *(Sheet = they maintain it; straight from the platform = a connected ad-platform MCP server pulls it. Both is fine — sheet for the fields a person writes, platform for the live numbers. Decide the mechanism yourself; do not ask them to name a connector.)*
7. **"Should the dashboard be able to change things in your ad accounts — pause an ad set, create a test — or only report on them?"** *(Reporting only is the safe default; say so. If they want actions, work out which MCP server or platform each one needs yourself, and confirm what is actually connected rather than assuming.)*
8. Metrics per timeframe (Monthly/Weekly/Daily)? Cap 6-8 per timeframe. *(Do not ask about date ranges — the windows are fixed above.)* Offer motion-type-appropriate suggestions (below) if they want defaults instead of listing their own.

## Motion-type default metrics

**Sales-led:** SAL, SQL, ICP, BTC, DPC, Stuck, CRM Sync%, Submit%, EV/Spend, Net EV, RoS, $/SAL, $/Sold.
| Metric | Definition | What it tells you |
|---|---|---|
| SAL | Sales Accepted Lead — entered CRM pipeline | Total lead volume from paid |
| SQL | Sales Qualified Lead — passed initial qualification | How much SAL volume is worth pursuing |
| ICP | Ideal Customer Profile match | Creative/targeting quality |
| BTC | Book the Call | Sales team's ability to convert qualified leads to meetings |
| DPC | Demo/Pitch Call held | Show rate / lead responsiveness |
| Stuck | Leads stopped progressing | Rep capacity, routing, or disqualification issues |
| CRM Sync% | Leads appearing in CRM ÷ leads submitted | Data pipeline health (generic label — rename per company's CRM stack, function stays fixed as a Daily metric) |
| Submit% | Form fills ÷ Clicks | Landing page conversion quality |
| EV/Spend | Expected Value ÷ Spend | Blended ROI accounting for pipeline lag |
| Net EV | EV − Spend | Absolute expected profit/loss |
| RoS | Won Revenue ÷ Spend | Realized return (lags by sales cycle) |
| $/SAL | Spend ÷ SAL | Lead acquisition efficiency |
| $/Sold | Spend ÷ Closed Deals | Full-funnel cost per acquisition |

**Product-led:** Signup, Activation, Time-to-Activation, PQL, Trial-to-Paid %, Time-to-Paid, Expansion MRR, NRR, Feature Adoption %, CAC Payback.
| Metric | Definition | What it tells you |
|---|---|---|
| Signup | Account/user created | Top-of-funnel volume from paid |
| Activation | User hit the "aha moment" | Whether paid traffic finds real value |
| Time-to-Activation | Signup → activation event | Onboarding friction |
| PQL | Product Qualified Lead — usage crossed intent threshold | Which signups warrant sales/CS touch |
| Trial-to-Paid % | Paid conversions ÷ trial starts | Core monetization efficiency |
| Time-to-Paid | Signup → first payment | Sales-cycle equivalent for PLG |
| Expansion MRR | Net new MRR from existing accounts | Post-conversion value growth |
| NRR | Net Revenue Retention | Whether the cohort is worth the CAC over time |
| Feature Adoption % | % of activated users reaching a deeper feature | Engagement depth, churn-risk proxy |
| CAC Payback (PLG) | Months to recover CAC from converted user's revenue | Efficiency counterpart to RoS |

**Ecommerce:** ATC, Checkout, Orders, AOV, ROAS, Repeat Rate, Refund Rate, Feed Health%, Session CVR%, $/ATC, $/Order, Profit/Spend, Net Profit.
| Metric | Definition | What it tells you |
|---|---|---|
| ATC | Add to cart | Mid-funnel intent volume from paid |
| Checkout | Checkout started | How much cart intent survives to the payment step |
| Orders | Completed purchases | Realized conversion volume |
| AOV | Revenue ÷ Orders | Basket size — the lever that moves breakeven ROAS |
| ROAS | Revenue ÷ Spend | Headline return; compare against the margin-derived breakeven, not against 1.0 |
| Repeat Rate | Repeat orders ÷ Orders | Whether paid acquisition is buying one-time or returning customers |
| Refund Rate | Refunds ÷ Orders | Quality of the demand a creative or term is buying (inverted — lower is better) |
| Feed Health% | Approved SKUs ÷ submitted SKUs | Catalog/feed pipeline health (the ecommerce analog of CRM Sync%; fixed as a Daily metric) |
| Session CVR% | Sessions reaching ATC ÷ Sessions | Landing page and PDP conversion quality |
| $/ATC | Spend ÷ ATC | Mid-funnel efficiency, readable before purchases land |
| $/Order | Spend ÷ Orders | Full-funnel CAC |
| Profit/Spend | Contribution profit ÷ Spend | Margin-aware ROI; the scale/cut decision metric |
| Net Profit | (Revenue × gross margin) − Spend | Absolute contribution after COGS and ad spend |

*Breakeven ROAS is 1 ÷ gross margin — at a 55% margin that is 1.82×, so a 1.7× ad set is losing money even though it looks positive. Always set the ROAS chip thresholds off the client's real margin.*

**Funnel stage analog:** Sales-led = `Clicks→Submits→SAL→SQL→ICP→BTC→DPC→Sold`. Ecommerce = `Clicks→Sessions→Product Views→Add to Cart→Checkouts→Purchases→Repeat Orders`. Product-led = `Clicks→Signups→Activation→PQL→Trial-to-Paid→Expansion`. Same visual funnel component, different stage labels/thresholds — granularity table and page structure unchanged regardless of motion type.

## Steps
1. Run onboarding (above).
2. Pull data — see **Data pipeline** above. The sheet is the database; MCP populates it on a schedule; the HTML is baked from it. Never call a connector client-side except through the artifact `mcp` capability.
3. Refresh cadence — fixed: auto daily, to support Daily/Weekly/MTD views. No hardcoded snapshot arrays.
4. Design — follow the design spec below. It is complete on its own; do not go looking for a separate design skill.

## Design — exact UI spec

Complete on its own for *visual* decisions — colours, type, spacing, components. It is **not** a substitute for reading the example files, which remain the authority on the data layer and wiring (see **Purpose** above).

**Foundation:** Font `-apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif`; base 13px, line-height 1.5, text `#111827`, page bg `#F7F8FA`. The dashboard commits to a single light theme, so declare it: `:root{color-scheme:light}`, and give every `input`, `textarea` and `select` an explicit `background`. Without both, a dark-mode host darkens the user-agent form controls while the explicit dark text colour stays put, and every field in both modals becomes dark-on-dark. A control that sets a text colour must set its background from the same palette. Cards: white, `1px solid #E5E7EB`, radius 8px. Max content width 1440px centered, 16px page padding.

**Header (sticky, 52px):** white bg, bottom border `#E5E7EB`. Logo 15px/700 with accent char in `#E07040`. Nav links 12px/500 `#6B7280`; active/hover bg `#F3F4F6`, text `#111827`. Right meta text 11px `#9CA3AF`.

**Page header:** Title 18px/700. Subtitle 12px `#9CA3AF`.

**Summary KPI strip:** Grid `auto-fit minmax(110px,1fr)`, 1px gap on `#E5E7EB` bg (hairline dividers). Cells white, 12/14px padding. Label 9px/600 uppercase, letter-spacing 0.6px, `#9CA3AF`. Value 17px/700 `#111827`. Sub-value 10px `#6B7280`.

**Channel tabs:** Pills, 5/12px padding, radius 6px, 12px/500 `#6B7280`, border `#E5E7EB`, white bg. Active: bg `#111827`, white text. Count badge 60% opacity.

**Tables:** 12px font. Header cells bg `#F9FAFB`, 7/10px padding, 10px/600 `#6B7280`, bottom border `#E5E7EB`, sortable (hover → `#111827`/`#F3F4F6`). Sorted column `#2563EB`. Body cells 7/10px padding, bottom border `#F3F4F6`, row hover bg `#FAFAFA`. Numeric columns right-aligned, monospace (`SF Mono`/`Fira Code`), 11px, tabular-nums. Name column 500 weight, max-width 200px ellipsis, optional 10px `#9CA3AF` subtext.

**Row actions / pause button:** 64px dedicated column, centered — it carries the pause button and, on Monthly/Weekly, the pencil that logs a note. Outlined `#E5E7EB`/`#9CA3AF`; hover → red (`#B91C1C`/`#FEF2F2`); paused state → green (`#16A34A`/`#F0FDF4`). Paused rows: 40% opacity, strikethrough name.

**Status chips/pills** (11px/700, monospace, pill radius): Green bg `#F0FDF4`/text `#15803D`; Yellow bg `#FFFBEB`/text `#A16207`; Red bg `#FEF2F2`/text `#B91C1C`; Paused tag bg `#F9FAFB`/text `#9CA3AF`/border `#E5E7EB`; Platform tags: channel-1 → bg `#EFF6FF`/text `#1D4ED8`, channel-2 → bg `#F0FDF4`/text `#16A34A`; Format tag bg `#F5F3FF`/text `#6D28D9`.

**Funnel (2-col grid):** Stage rows grid `72px 1fr 52px 44px` (label/bar/count/rate). Label 10px/600 uppercase `#6B7280`. Bar track `#F3F4F6`, 12px tall, radius 6px; fill matches stage color (impressions bar 50% opacity). Count 11px/700 right-aligned. Divider rows: centered text flanked by rules in `#D1D5DB`. Summary footer: 2-col grid, label 9px uppercase gray, value 16px/700 `#111827`.

**Heatmap (2-col grid):** Cells tinted by value: green `rgba(22,163,74,.08)`/`#15803D`; yellow `rgba(217,119,6,.09)`/`#A16207`; red `rgba(220,38,38,.07)`/`#B91C1C`. First column left-aligned, no tint.

**Net EV / optimization banner:** Light green row (`#F0FDF4` bg, `#BBF7D0` border, radius 8px), flex label/value pairs. Recommendation header 12px/700 with bottom border.

**Kanban board:** 4 columns on desktop via `repeat(auto-fit,minmax(230px,1fr))`; under 900px it becomes a horizontally scrolling flex row so the page itself never scrolls sideways, and under 480px it collapses to a single column. Columns bg `#F9FAFB`, radius 10px, 12px padding. Header 11px/700 uppercase + colored status dot + count badge. Cards white, bordered, radius 8px, draggable, hover shadow. Priority pills red/yellow/green (high/med/low). Delete icon on card hover only.

**Platform changelog panel:** Collapsible (`<details>`), summary row has dark sync button. Logged items: icon chip + bold title + gray meta line + gray detail line.

**Modals:** Centered overlay `rgba(0,0,0,.35)`, white card, radius 12px. Title 15px/700, subtitle 12px gray. Inputs bordered `#E5E7EB`, radius 6-8px. Primary button dark `#111827`/white text. Secondary button light `#F3F4F6`/gray text.

## Gotchas & learnings
- The reference file hardcodes data in JS arrays — that was a snapshot; this skill requires live daily refresh instead.
- One sheet per build, always. A template that ships pointing at a real sheet leaks one company's spend data to everyone who clones it — and the leak is silent, because the dashboard renders perfectly.
- Never store an API key in browser storage. (Artifacts *can* use `localStorage` — each is served from its own origin — which is exactly why the rule has to be stated rather than assumed away.)
- Duplicate, Pause, and Unpause all fire live MCP calls that create/change real state — confirm with the user before executing each, never wire buttons straight to the call.
- Don't let dimensions-per-timeframe, granularity-per-page or the reporting windows be re-litigated — fixed for consistency across every build. The one exception is the Creatives MTD/L30 toggle, which is part of the spec rather than a per-company option.
- Motion type (sales-led / ecommerce / product-led) changes *what* is measured (metrics, funnel stage labels, chip thresholds), never *how* it's laid out — resist adding/removing pages or restructuring views per company.
- Changing a metric means editing three things in step: the `*_LABELS` array (header), the matching `render*` function (the cell), and the chip helper that colors it. Retune the chip thresholds to the client's economics — the defaults are per-motion, not universal.
- "CRM Sync%" is a generic label — rename per company's actual CRM stack, but the underlying check (lead-submit-to-CRM sync health) stays fixed as a Daily-page metric.
- Cap KPI strip at 6-8 metrics per timeframe or it becomes noise.
- Anything that looks clickable must do something, and anything that does something must look clickable. A Kanban card that ignores a click, or a table row that only sorts, reads as broken — wire the drill-down and the card editor before adding new metrics.
- Never scope Monthly/Weekly rows to currently-active units. Period tables report what happened in the period, so anything that delivered in the window belongs in the table even if it is paused or deleted now — otherwise spend goes missing and the totals stop reconciling with the ad platform.
- Channel count drives repetition, not structure: adding a third channel means one more funnel card, heatmap card, tab, pane, `CONFIGS` entry and changelog panel — never a new page or a different layout. Use the ad-group renderers (`renderGMW`/`renderGDaily`, which add Campaign and Status) for the search-like channel and the plain ones for the rest.

## Pre-ship checklist (run before calling a build done)

Every item here came from a real bug. Run it as a test pass, not a self-assessment — open the file in a browser and check.

**Data completeness**
- For any table filtered to specific creatives, confirm the underlying ad set / campaign actually has no more members than shown. Pull the full list filtered by `spend > 0`, not a fixed top-N, unless the person asked for a leaderboard.
- Every metric implied by the page title is present. An "ad-level performance" page needs impressions, clicks, spend *and* a cost-per metric — not just a cost ratio.
- Every entity active during the window appears, paused ones included (see the row-inclusion rule above).
- Each page states its window in the header, and the data actually matches it — Monthly is MTD, Weekly is a rolling 7 days, Daily is the last *completed* day.
- The Creatives window toggle swaps a real second dataset, and both windows are internally consistent. Roll-up checks compare like windows only.
- Every Changelog card with a test resolves both ad IDs against the creative data, or is honestly labelled a snapshot. Move a card into Testing by hand and confirm its stats appear with no ad ID typed.
- Exercise Duplicate down all three routes — host present, relay configured, neither — and confirm each confirms first, creates PAUSED, and leaves the card honestly labelled. Verify the ads-manager deep link against a real account before shipping: account-ID parameters and URL shapes differ per platform and change. Switching the Creatives page to L30 must leave the card numbers unchanged.
- After splitting an ad set's spend across a control and test ad, re-check that creative spend still sums to the channel's spend.
- Every period row resolves to at least one creative. If it cannot, the drill-down shows an empty state and the creative data is incomplete.

**Data sourcing**
- On a server: `/healthz` returns `db:"postgres"` — not `file`, not `memory`; the URL prompts for a password before showing any spend figure; and a card added, reloaded, then survived a redeploy. Follow the checklist in `docs/railway-setup.md`.
- The shared board actually works: open the published link and confirm the state line says "shared board" rather than "local to this browser". An undeclared capability fails silently and looks identical to the feature not existing.
- If the log is mirrored, confirm the mirror runs one-way and that nobody has been told they can edit the sheet copy.
- No sheet ID, file ID or account ID from a real build appears anywhere in the template repo. Grep for them before every commit.
- The sheet, not the HTML, is the source of truth. Grep the build for any number that exists only in the file — if a person could want to correct it and there is no sheet cell behind it, it is in the wrong place.
- Confirm the refresh job can actually *write* the sheet it reads. `create_file` returns a new file ID; if the build assumed in-place updates, every refresh silently orphans the sheet the dashboard was built from.
- If two sources exist for one metric (platform-reported results vs. analytics conversions), confirm which is the source of truth and apply it to *every* table, not just the one that was mentioned.
- Any channel or connector unavailable this session is flagged in the UI, never silently zeroed or omitted.
- Spot-check one aggregate against its line items — creative spend should roll up to the channel's spend, and the funnel footer should equal the sum of the period table. Grouping by a leading ID inside a compound string is where rows quietly vanish or double-count.

**Interactions — click every clickable thing**
- Anything with a pointer or grab cursor has a working handler; anything with a handler looks clickable. Draggable things use `grab`/`grabbing`, not `pointer`.
- Grep the stylesheet for classes it styles but nothing ever applies. An orphaned rule is usually a feature that was specified, styled and never wired — that is how the platform change-log panels sat empty behind six unused styles.
- Every modal exists on every page it belongs on, not just the one it was first wired to.
- Buttons that claim to scope ("View creatives" from an ad set) actually filter to that thing rather than navigating to a generic page.
- Every callback into the host (`sendPrompt()` and friends) is wrapped in a guard. Those functions do not exist when the file is opened standalone, and an unguarded call throws and silently kills the rest of the handler. Use the `hostPrompt()` wrapper.
- After adding a click handler to something already draggable or containing buttons (Kanban cards, rows with an inline pause button), confirm the inner controls still work — `event.stopPropagation()` everywhere it is needed.

**Responsive**
- Check at desktop (~1440px), tablet (~768–900px) and phone (~375–430px).
- Page-level horizontal overflow must be **zero** at every width. Wide content scrolls inside its own container instead.
- Tables, nav and filter/sort bars scroll rather than break. On phones, hide the header's reporting-period text so the nav keeps its room.
- Multi-column grids — funnel cards, heatmaps, Kanban, ad gallery — collapse to one column at phone width.
- Modals stay inside the viewport (`max-width:calc(100vw - 32px)`).

**Before shipping**
- Grep for references to renamed or removed fields; a rename in one place silently breaks a sort button or renderer elsewhere.
- Open both modals with the OS in dark mode, and again with `color-scheme:dark` forced on the root. Every field must stay light-on-white; a field with a text colour but no background is the classic dark-on-dark bug.
- Run the script through a real parser (`node --check`) rather than eyeballing brace balance, and confirm `{`/`}` balance in the style block and `<div>` balance in the body.
- Counts in labels ("4 ad sets", "6 videos") are computed from the data at render time, never typed. Use `syncTabCounts()`.
- Anything described as "persistent" is checked from a **second** browser profile, not a reload. `localStorage` survives a refresh and still shares nothing; if the board is meant to be a team board, open the published link as two different people and confirm one sees the other's card. State in the reply which of those two you actually verified.

## Output format
Single HTML artifact, exact template structure (7 fixed pages), channels/metrics/actions populated per onboarding answers, action buttons (Pause/Unpause/Duplicate/Sync) call back via `sendPrompt()` — always through the `hostPrompt()` guard — for confirmation rather than direct MCP execution. Auto-refreshes daily.
