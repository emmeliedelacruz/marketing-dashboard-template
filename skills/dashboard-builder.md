# dashboard-builder

**Changelog**
- 2026-08-22: Initial creation.
- 2026-08-22: Views locked to match hearth-dashboard-v2.html exactly. Onboarding scoped to channels, access/action, and per-timeframe metrics. Dimensions fixed per timeframe. Refresh cadence fixed to automatic daily.
- 2026-08-23: Onboarding asked one question at a time, not as a batch table. Added motion-type question (Product-led vs Sales-led). Clarified: template (pages, tabs, views, layout) is identical across companies; only company name, channels, and KPIs/metrics vary.
- 2026-08-23: Documented reporting granularity per page — Monthly/Weekly = ad set/ad group level, Daily = campaign level, Creatives = ad level. KPIs vary by company/motion type; only granularity level is fixed.
- 2026-08-23: Adopted richer 4-column Kanban spec (Duplicate/Hypothesis/Verdict) as canonical, replacing the simpler 3-column hearth-HTML version. Pause and unpause both fire live MCP calls (symmetric).
- 2026-08-23 (latest): Added Product-led metric definitions (parallel to Sales-led set) so the motion-type onboarding question has concrete defaults for both paths.

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

KPIs/metrics shown at each level vary by company and motion type (per onboarding) — only the level of granularity is fixed across every dashboard built with this skill.

## Fixed page specs

**Funnel:** 2-col grid, one funnel card per channel (Impressions→Clicks→Submits→SAL→SQL→ICP→BTC→DPC→Sold for sales-led; see Product-led analog below). Below: 2-col heatmap grid, one table per channel, verticals/segments as rows.

**Monthly / Weekly:** Channel tabs. Per channel: single sortable table — Name, $/Sold, Won $, RoS, Sold, Spend, $/SAL, SAL, ICP%, BTC%, DPC%, Stuck%, Net EV (+pause column; Google adds Status + Campaign). Weekly row-click opens 2-button modal (View Creatives / Log Optimization).

**Daily:** Channel tabs (campaign-level). Table: Name, (Status if applicable), CRM Sync%, Submit%, SAL, $/SAL, ICP%, BTC%, DPC%, Stuck%, Net EV (+pause column). No Won$/Sold/RoS — attribution lag makes them misleading daily.

**Creatives:** 3 tabs: channel-1 ads, channel-2 campaigns, Ad Library. Ad-level table: Ad Creative, Format, Copy Angle, Impr, CTR, %Submit, ICP%, DPC%, $/SAL, Stuck% + sort bar. Campaign-level tab: same funnel metrics, rank-by bar. Ad Library: filter buttons (All/Video/Image) + card grid (media, title, body, CTA, Ad Library link).

**Optimization:** Net-row banner (Scale-Up Potential / Weekly Bleed to Stop / Net Weekly Opportunity + rule callout). Two tables: Scale Up, Pull Back — Unit, Platform, EV/Spend, $/SAL, Spend, SAL, Net EV $. Row click → Changelog note modal. Unit name link → ad preview modal + "Open in Ads Manager."

**Changelog (fixed, richer spec — canonical over hearth-HTML's simpler version):**
- Header with "Archive Done" button
- **4-column Kanban:** To Test → Testing → Evaluated → Completed. Drag-and-drop.
- **Card fields:** Note (always) · Hypothesis (always, styled distinctly) · Variable Being Tested (always) · Test Setup (visible once Testing/Completed) · Control vs Test perf strip — Spend/CTR/Submit% (visible once duplicated) · Ad IDs, control+test (visible once duplicated) · Verdict (visible once Evaluated/Completed)
- **Card actions:** Duplicate (calls MCP to clone top ad in the ad set with version suffix V2/V3…, auto-increments, logs old/new ad IDs, moves card to Testing — confirm before executing, it creates a live ad) · Complete (→ Completed) · Delete
- Adding a card: click any table row → note modal → priority (High/Med/Low) + starting column
- Below board: one collapsible "Platform Changelog" panel per channel — sync button pulls last ~30 activity log entries via MCP; empty-state if not connected
- **Pause button** (every row, always visible): live MCP call to pause by name. **Unpause: symmetric live MCP call to resume** — both need confirm-before-execute.
- Settings modal (gear icon): API key + account ID for live actions. Never store credentials in browser storage.

## Onboarding — ask one question at a time, wait for each answer
1. Company name?
2. Channels to track?
3. Per channel — data access: Google Sheet / Direct MCP / Both?
4. Per channel — action layer: Read-only, or read-only + specific actions (name + which MCP)?
5. Motion type: Product-led or Sales-led? *(Determines relevant KPIs — template stays identical, only metric set changes.)*
6. Metrics per timeframe (Monthly/Weekly/Daily)? Cap 6-8 per timeframe. Offer motion-type-appropriate suggestions (below) if they want defaults instead of listing their own.

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

**Funnel stage analog:** Sales-led = `Clicks→Submits→SAL→SQL→ICP→BTC→DPC→Sold`. Product-led = `Clicks→Signups→Activation→PQL→Trial-to-Paid→Expansion`. Same visual funnel component, different stage labels/thresholds — granularity table and page structure unchanged regardless of motion type.

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
- Don't let dimensions-per-timeframe or granularity-per-page be re-litigated — fixed for consistency across every build.
- Product-led vs sales-led changes *what* is measured (metrics, funnel stage labels), never *how* it's laid out — resist adding/removing pages or restructuring views per company.
- "CRM Sync%" is a generic label — rename per company's actual CRM stack, but the underlying check (lead-submit-to-CRM sync health) stays fixed as a Daily-page metric.
- Cap KPI strip at 6-8 metrics per timeframe or it becomes noise.

## Output format
Single HTML artifact, exact template structure (7 fixed pages), channels/metrics/actions populated per onboarding answers, action buttons (Pause/Unpause/Duplicate/Sync) call back via `sendPrompt()` for confirmation rather than direct MCP execution. Auto-refreshes daily.
