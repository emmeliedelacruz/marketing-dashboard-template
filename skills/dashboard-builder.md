# dashboard-builder

**Changelog**
- 2026-08-22: Initial creation.
- 2026-08-22: Views locked to match hearth-dashboard-v2.html exactly. Onboarding scoped to channels, access/action, and per-timeframe metrics. Dimensions fixed per timeframe. Refresh cadence fixed to automatic daily.
- 2026-08-23: Onboarding asked one question at a time, not as a batch table. Added motion-type question (Product-led vs Sales-led). Clarified: template (pages, tabs, views, layout) is identical across companies; only company name, channels, and KPIs/metrics vary.
- 2026-08-23: Documented reporting granularity per page — Monthly/Weekly = ad set/ad group level, Daily = campaign level, Creatives = ad level. KPIs vary by company/motion type; only granularity level is fixed.
- 2026-08-23: Adopted richer 4-column Kanban spec (Duplicate/Hypothesis/Verdict) as canonical, replacing the simpler 3-column hearth-HTML version. Pause and unpause both fire live MCP calls (symmetric).
- 2026-08-23: Added Product-led metric definitions (parallel to Sales-led set) so the motion-type onboarding question has concrete defaults for both paths.
- 2026-08-23: Added Ecommerce as a third motion type with its own metric definitions and funnel-stage analog. Worked examples now ship for all three motions (`dashboard-example-filled.html` sales-led, `dashboard-example-ecommerce.html`, `dashboard-example-product-led.html`).
- 2026-08-23: Channel count is two **or three** — the funnel and heatmap grids reflow, and every per-channel surface (tabs, panes, platform-changelog panels) repeats per channel. Both three-channel examples use index-based array names (M1/M2/M3 …) rather than channel initials.
- 2026-08-23: Fixed the Monthly/Weekly row-inclusion rule — period tables show every unit that was active at any point in the window, including paused and deleted ones, scoped by delivery rather than current status.
- 2026-08-23: Specified the two navigation behaviours — Monthly/Weekly row click drills into that unit's creatives, and clicking a Changelog card opens it for editing. Note logging moves to a per-row pencil button on Monthly/Weekly.
- 2026-08-23: Added the pre-ship checklist, a `hostPrompt()` guard for host callbacks, and `syncTabCounts()` so label counts are computed from data rather than typed.
- 2026-08-23: Fixed the reporting window per page — Monthly is month-to-date, Weekly is the last 7 days, Daily is the last completed day, and Creatives defaults to MTD with a toggle to last 30 days. Windows are now named in every page header.
- 2026-08-23: Brought the Changelog up to this spec — perf strip now carries Spend/CTR/conversion rate, ad IDs are shown, Test Setup survives into Evaluated, Duplicate writes real versioned ad IDs, Archive archives instead of deleting, the board persists to localStorage, drag has visual feedback, and the platform change-log panels actually render entries.
- 2026-08-23 (latest): Specified where the perf strip's numbers come from — live lookup of each ad ID against the creative data's MTD rows, with the card's stored values as a labelled fallback. Creative rows carry `ad_id`, and a duplicated ad is a real row sharing its parent ad set's spend.

**Name:** `dashboard-builder` — *use when building a custom HTML performance dashboard for a company/product, sourcing from a Google Sheet, direct MCP pull, or both.*

**New vs. update:** New skill.

---

## Purpose / When to use
Build a single-file HTML performance dashboard by cloning the hearth-dashboard-v2.html template exactly — same pages, tabs, views, layout — re-skinned with a company's own channels and KPIs. Triggers: "build me a dashboard," "make a Hearth-style dashboard for [company]," "dashboard from this sheet."

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

## Fixed page specs

**Funnel:** reflowing grid (`repeat(auto-fit,minmax(340px,1fr))`), one funnel card per channel — two or three across (Impressions→Clicks→Submits→SAL→SQL→ICP→BTC→DPC→Sold for sales-led; see Product-led analog below). Below: 2-col heatmap grid, one table per channel, verticals/segments as rows.

**Monthly / Weekly:** Channel tabs. Per channel: single sortable table — Name, $/Sold, Won $, RoS, Sold, Spend, $/SAL, SAL, ICP%, BTC%, DPC%, Stuck%, Net EV (+pause column; Google adds Status + Campaign). Row click on either page **drills into Creatives**: switch to the Creatives page, select that unit's channel tab, and filter the ad table to the ads belonging to that ad set / ad group, with a "Showing ads in X" bar carrying a Clear filter button. Match names loosely (lowercase, strip punctuation) — ad set, ad group and campaign names rarely agree on underscores versus spaces — and match on the row's campaign as well as its own name, since search channels report creatives at campaign level. Show a real empty state when nothing matches, never an empty table. Because the row itself is now a link, logging a note moves to a pencil button in the actions column beside Pause.

**Row inclusion — Monthly / Weekly (fixed):** show every ad set / ad group that was **active at any point during the period**, not just the ones live right now. A unit that spent for three weeks and was paused on the 22nd still owns that spend for the month; filtering by current status silently drops its cost, understates blended CAC and overstates efficiency for everything left in the table. Include paused, ended, budget-exhausted and deleted units as long as they delivered in the window, and render them with the Paused chip and 40% opacity rather than hiding them. Scope rows by *delivery during the window*, never by *current state*. Daily is the exception: it reports the most recent day, so only units that delivered that day appear.

Watch out that the plain ad-set renderer (`renderMW`) has no Status column — only the ad-group renderer (`renderGMW`) does. If a non-search channel carries paused-but-spending units, add a Status column to the plain renderer too, or the rows read as live.

**Daily:** Channel tabs (campaign-level). Table: Name, (Status if applicable), CRM Sync%, Submit%, SAL, $/SAL, ICP%, BTC%, DPC%, Stuck%, Net EV (+pause column). No Won$/Sold/RoS — attribution lag makes them misleading daily.

**Creatives:** one tab per channel plus Ad Library, with a page-level window toggle (Month to date / Last 30 days) above the tabs, defaulting to MTD. Ad-level table: Ad Creative, Format, Copy Angle, Impr, CTR, %Submit, ICP%, DPC%, $/SAL, Stuck% + sort bar. Campaign-level tab: same funnel metrics, rank-by bar. Ad Library: filter buttons (All/Video/Image) + card grid (media, title, body, CTA, Ad Library link).

**Optimization:** Net-row banner (Scale-Up Potential / Weekly Bleed to Stop / Net Weekly Opportunity + rule callout). Two tables: Scale Up, Pull Back — Unit, Platform, EV/Spend, $/SAL, Spend, SAL, Net EV $. Row click → Changelog note modal. Unit name link → ad preview modal + "Open in Ads Manager."

**Changelog (fixed, richer spec — canonical over hearth-HTML's simpler version):**
- Header with "Archive Done" button
- **4-column Kanban:** To Test → Testing → Evaluated → Completed. Drag-and-drop.
- **Card fields:** Note (always) · Hypothesis (always, styled distinctly) · Variable Being Tested (always) · Test Setup (visible on anything past To Test — a card that has been evaluated still needs to show how it was set up) · Control vs Test perf strip, three rows: Spend, CTR and the motion's conversion rate (`PERF_RATE_LABEL` / `PERF_RATE_KEY` — Submit% / ATC% / Signup%), visible once duplicated · Ad IDs, control and test, visible once duplicated · Verdict (visible once Evaluated/Completed)
- **Where the perf strip's numbers come from.** The card stores the two ad IDs; the strip looks each one up in the creative data by `ad_id` at render time and reads its **MTD** row. It always reads MTD, whatever the Creatives page is toggled to — a card's numbers must not change meaning because someone switched that page to L30. Values stored on the card (`controlSpend`, `controlCtr`, `controlRate` and their test counterparts) are a **fallback only**, used when an ad ID does not resolve: a campaign-level channel that has no ad rows, a deleted ad, or an ad that did not run this month. The card labels itself "snapshot" when it falls back, so nobody mistakes a frozen number for a live one. Do not store live metrics on the card as the primary source — that is the hardcoded snapshot this skill tells you not to build.
- **Duplicated ads are real rows.** A test ad is an ad: it appears in the creative array with its own `ad_id` (`<control>_V<n>`), the same `ad_set`, and its share of the parent's spend. The parent ad set's spend is **split** across control and test, so creative spend still rolls up to the channel total. An ad set under test with a single creative row holding the full spend is a sign the test ad was never added to the data.
- **Card actions:** Duplicate (clones the top ad in the unit as `<controlAdId>_V<n>`, auto-incrementing the version, writing both ad IDs onto the card and moving it to Testing — confirm before executing, it creates a live ad) · Complete (→ Completed) · Delete (permanent)
- **Archive Done archives, it does not delete.** Completed cards get an `archived` flag, leave the board and stay in `NOTES`, with a restore link beside the button. A button labelled Archive that silently deletes is the kind of thing people only discover once.
- **The board persists.** It is the one part of the dashboard a person authors by hand, so it is saved to `localStorage` on every change and restored on load — a data refresh replaces the metrics, never the test log. Wrap every storage access in try/catch: private windows and blocked-storage browsers throw on access, and an unguarded read takes the whole page down. Credentials still never go near storage.
- Adding a card: the pencil button on a Monthly/Weekly row, or a row click on Daily, Creatives or Optimization → note modal, which sets note, hypothesis, variable, test setup, verdict, priority (High/Med/Low) and starting column
- **Opening a card: click it.** The card reuses the note modal in edit mode, pre-filled with note, hypothesis, variable, test setup, verdict, priority and column, saving back to the same card rather than creating a new one. Card buttons (Duplicate / Complete / Delete) must stop propagation so they don't also open the editor, and a drag must not count as a click.
- Below board: one collapsible "Platform Changelog" panel per channel, each with an addressable body (`cl-body-<channel>`) rendered from a `PLATFORM_LOG` map — entries carry when / who / what / detail plus a `kind` (budget, status, creative, targeting, bid) that picks the icon and tint. Sync pulls the last ~30 activity entries via MCP and re-renders. A channel with no entries shows the empty state.
- **Pause button** (every row, always visible): live MCP call to pause by name. **Unpause: symmetric live MCP call to resume** — both need confirm-before-execute.
- Settings modal (gear icon): API key + account ID for live actions. Never store credentials in browser storage.

## Onboarding — ask one question at a time, wait for each answer
1. Motion type: Sales-led, Ecommerce, or Product-led? *(Determines relevant KPIs — template stays identical, only metric set changes.)*
2. Channels to track? *(Two or three. Beyond three the tab rows and funnel grid stop being readable — push back and suggest folding the smallest channel into a combined view.)*
3. Per channel — data access: Google Sheet / Direct MCP / Both?
4. Per channel — action layer: Read-only, or read-only + specific actions (name + which MCP)?
5. Metrics per timeframe (Monthly/Weekly/Daily)? Cap 6-8 per timeframe. *(Do not ask about date ranges — the windows are fixed above.)* Offer motion-type-appropriate suggestions (below) if they want defaults instead of listing their own.

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
2. Pull data — Sheet: `Google Drive:read_file_content`/`search_files`. Direct MCP: call connector at build time, never client-side. Both: sheet = manual/strategic fields, MCP = live numbers.
3. Refresh cadence — fixed: auto daily, to support Daily/Weekly/MTD views. No hardcoded snapshot arrays.
4. Design — follow `frontend-design` skill; see design spec below.

## Design — exact UI spec (self-contained, no source file needed)

**Foundation:** Font `-apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif`; base 13px, line-height 1.5, text `#111827`, page bg `#F7F8FA`. Cards: white, `1px solid #E5E7EB`, radius 8px. Max content width 1440px centered, 16px page padding.

**Header (sticky, 52px):** white bg, bottom border `#E5E7EB`. Logo 15px/700 with accent char in `#E07040`. Nav links 12px/500 `#6B7280`; active/hover bg `#F3F4F6`, text `#111827`. Right meta text 11px `#9CA3AF`.

**Page header:** Title 18px/700. Subtitle 12px `#9CA3AF`.

**Summary KPI strip:** Grid `auto-fit minmax(110px,1fr)`, 1px gap on `#E5E7EB` bg (hairline dividers). Cells white, 12/14px padding. Label 9px/600 uppercase, letter-spacing 0.6px, `#9CA3AF`. Value 17px/700 `#111827`. Sub-value 10px `#6B7280`.

**Channel tabs:** Pills, 5/12px padding, radius 6px, 12px/500 `#6B7280`, border `#E5E7EB`, white bg. Active: bg `#111827`, white text. Count badge 60% opacity.

**Tables:** 12px font. Header cells bg `#F9FAFB`, 7/10px padding, 10px/600 `#6B7280`, bottom border `#E5E7EB`, sortable (hover → `#111827`/`#F3F4F6`). Sorted column `#2563EB`. Body cells 7/10px padding, bottom border `#F3F4F6`, row hover bg `#FAFAFA`. Numeric columns right-aligned, monospace (`SF Mono`/`Fira Code`), 11px, tabular-nums. Name column 500 weight, max-width 200px ellipsis, optional 10px `#9CA3AF` subtext.

**Row actions / pause button:** 36px dedicated column, centered. Outlined `#E5E7EB`/`#9CA3AF`; hover → red (`#B91C1C`/`#FEF2F2`); paused state → green (`#16A34A`/`#F0FDF4`). Paused rows: 40% opacity, strikethrough name.

**Status chips/pills** (11px/700, monospace, pill radius): Green bg `#F0FDF4`/text `#15803D`; Yellow bg `#FFFBEB`/text `#A16207`; Red bg `#FEF2F2`/text `#B91C1C`; Paused tag bg `#F9FAFB`/text `#9CA3AF`/border `#E5E7EB`; Platform tags: channel-1 → bg `#EFF6FF`/text `#1D4ED8`, channel-2 → bg `#F0FDF4`/text `#16A34A`; Format tag bg `#F5F3FF`/text `#6D28D9`.

**Funnel (2-col grid):** Stage rows grid `72px 1fr 52px 44px` (label/bar/count/rate). Label 10px/600 uppercase `#6B7280`. Bar track `#F3F4F6`, 12px tall, radius 6px; fill matches stage color (impressions bar 50% opacity). Count 11px/700 right-aligned. Divider rows: centered text flanked by rules in `#D1D5DB`. Summary footer: 2-col grid, label 9px uppercase gray, value 16px/700 `#111827`.

**Heatmap (2-col grid):** Cells tinted by value: green `rgba(22,163,74,.08)`/`#15803D`; yellow `rgba(217,119,6,.09)`/`#A16207`; red `rgba(220,38,38,.07)`/`#B91C1C`. First column left-aligned, no tint.

**Net EV / optimization banner:** Light green row (`#F0FDF4` bg, `#BBF7D0` border, radius 8px), flex label/value pairs. Recommendation header 12px/700 with bottom border.

**Kanban board:** 4-column grid. Columns bg `#F9FAFB`, radius 10px, 12px padding. Header 11px/700 uppercase + colored status dot + count badge. Cards white, bordered, radius 8px, draggable, hover shadow. Priority pills red/yellow/green (high/med/low). Delete icon on card hover only.

**Platform changelog panel:** Collapsible (`<details>`), summary row has dark sync button. Logged items: icon chip + bold title + gray meta line + gray detail line.

**Modals:** Centered overlay `rgba(0,0,0,.35)`, white card, radius 12px. Title 15px/700, subtitle 12px gray. Inputs bordered `#E5E7EB`, radius 6-8px. Primary button dark `#111827`/white text. Secondary button light `#F3F4F6`/gray text.

## Gotchas & learnings
- The reference file hardcodes data in JS arrays — that was a snapshot; this skill requires live daily refresh instead.
- Never store an API key in browser storage (artifacts can't use localStorage anyway).
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
- Every Changelog card with a test resolves both ad IDs against the creative data, or is honestly labelled a snapshot. Switching the Creatives page to L30 must leave the card numbers unchanged.
- After splitting an ad set's spend across a control and test ad, re-check that creative spend still sums to the channel's spend.
- Every period row resolves to at least one creative. If it cannot, the drill-down shows an empty state and the creative data is incomplete.

**Data sourcing**
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
- Run the script through a real parser (`node --check`) rather than eyeballing brace balance, and confirm `{`/`}` balance in the style block and `<div>` balance in the body.
- Counts in labels ("4 ad sets", "6 videos") are computed from the data at render time, never typed. Use `syncTabCounts()`.

## Output format
Single HTML artifact, exact template structure (7 fixed pages), channels/metrics/actions populated per onboarding answers, action buttons (Pause/Unpause/Duplicate/Sync) call back via `sendPrompt()` — always through the `hostPrompt()` guard — for confirmation rather than direct MCP execution. Auto-refreshes daily.
