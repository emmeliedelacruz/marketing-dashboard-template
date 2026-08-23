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
   | `CR` | Creatives — Channel 1, ad level |
   | `ADD` / `CUT` | Optimization — Scale Up / Pull Back |
   | `VM` / `VG` | Funnel heatmap segments, per channel |
   | `ADS` | Ad Library gallery |

   All three examples have these populated — copy the row shapes from whichever one matches your motion.

   The two three-channel examples use **index-based names instead**, because channel initials stop scaling once there are more than two: `M1`/`M2`/`M3` (monthly), `W1`/`W2`/`W3` (weekly), `D1`/`D2`/`D3` (daily), `CR1`/`CR2`/`CR3` (creatives), `V1`/`V2`/`V3` (heatmap), with `ADD`/`CUT`/`ADS` unchanged. Same data shapes, clearer numbering — worth copying if you are adding a third channel.
5. Update the KPI labels if your metrics differ. Each table has a matching `*_LABELS` / `*_KEYS` / `*_RIGHTS` triple, all the same length: `PL_MW_*` and `GL_MW_*` (Monthly and Weekly), `PL_D_*` and `GL_D_*` (Daily), `REC_*` (Optimization), `CR_*` (Creatives). `LABELS` sets the header text, `KEYS` names the field sorting reads, `RIGHTS` right-aligns the column.

   Note that the table cells themselves are emitted by the `render*` functions (`renderMW`, `renderGMW`, `renderDaily`, `renderGDaily`, `renderRec`, `renderCr`), which reference row fields by name. To swap a metric end to end (e.g. product-led teams trading SAL/SQL/ICP for Signup/Activation/PQL — see the skill file for full definitions), edit the label array *and* the matching render function.
6. Open in a browser. That's it — no server needed.

## Quick start (with Claude)

1. Create a Claude project (claude.ai) or open Claude Code.
2. Add `skills/dashboard-builder.md` to the project's skill files.
3. Ask Claude: *"build me a dashboard for [my company]"*. It will ask you one question at a time — channels, data access (Google Sheet vs. live MCP connectors vs. both), motion type (sales-led vs. product-led), and which metrics to track — then generate the file for you.

## Data sources

The template ships with hardcoded JS arrays (a snapshot). For live data, wire it up to:
- A **Google Sheet** (read via API, paste in, or use Claude + Google Drive MCP), or
- **Direct API/MCP pulls** (Amplitude, Meta Ads, Google Ads, Stripe, etc.) — see the skill file for guidance on how Claude fetches this at build/refresh time.

## Fixed structure — what doesn't change per company

This template is intentionally opinionated about **layout**, not about **metrics**:

- 7 pages: Funnel, Monthly, Weekly, Daily, Creatives, Optimization, Changelog
- Reporting granularity is fixed: Monthly/Weekly = ad set/ad group level, Daily = campaign level, Creatives = ad level
- Changelog is a 4-column Kanban (To Test → Testing → Evaluated → Completed) with hypothesis/verdict tracking and a live platform-changes feed

What *does* vary per company: channel names, which KPIs populate the tables, and which motion you're tracking — sales-led (SAL/SQL/ICP/BTC/DPC), ecommerce (ATC/Checkout/Orders/ROAS/Refunds), or product-led (Signup/Activation/PQL/Trial-to-Paid/Expansion). All three metric sets are documented in the skill file, and each has a worked example in this repo.

Both three-channel examples also carry one **paused** ad group that still spent during the month, to show the Monthly/Weekly rule: period tables include everything that delivered in the window, not just what is live now. Filtering those rows out silently drops their spend and flatters every remaining row.

## Interactions worth knowing

- **Monthly / Weekly row click** drills into Creatives — it switches to that unit's channel tab and filters the ad table to that ad set or ad group, with a Clear filter button. Names are matched loosely, and the row's campaign is matched too, so search channels reporting creatives at campaign level still resolve.
- **Pencil button** on those rows logs a Changelog note (the row click is taken by the drill-down). On Daily, Creatives and Optimization the row click still opens the note modal.
- **Changelog card click** opens the card in the same modal for editing — note, hypothesis, variable, test setup, verdict, priority and column — and saves back to that card.
- **Pause** buttons appear on every table row, and dragging cards between Changelog columns works as expected.

## Adding a third channel

The blank template ships with two channels. Both three-channel examples show what changes — nothing about the page structure, only how many of each thing there are:

- **CSS** — `.funnel-grid` and `.hm-grid` go from `1fr 1fr` to `repeat(auto-fit,minmax(340px,1fr))` so the cards reflow instead of squashing, and a `.plat.c3` chip colour is added.
- **Markup** — one more funnel card and heatmap card on the Funnel page; one more tab and pane on Monthly, Weekly, Daily and Creatives; one more collapsible panel on the Changelog page.
- **JS** — one more entry per page in the `CONFIGS` registration list, one more `renderFunnel` / `renderHm` call, and the third channel's data arrays.

Use the "ad group" renderers (`renderGMW` / `renderGDaily`, which add Campaign and Status columns) for whichever channel is search-like, and the plain ones for the rest. In the ecommerce example that is Google Shopping (`c2`); in the product-led example it is Google (`c3`).

## Live actions (pause/resume/duplicate ads)

The Settings modal (gear icon, top right) accepts an API key and account ID for live actions via MCP-connected tools. **Never commit real API keys to this repo or any fork of it** — they're meant to be entered locally in your browser session, not stored in code.

## Contributing

This is a community template — PRs welcome for additional channel integrations, metric presets, or bug fixes. Please don't submit PRs containing real company data or credentials.

## License

MIT — see `LICENSE`.
