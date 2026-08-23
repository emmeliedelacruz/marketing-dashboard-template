# Marketing Performance Dashboard — Template

A single-file HTML performance dashboard for paid acquisition teams. Built for sales-led *or* product-led motions, tracking any two channels (Meta/Google/TikTok/etc.) across Monthly, Weekly, Daily, Creative, Optimization, and Changelog views.

No build step, no backend required — it's one HTML file you open in a browser.

## What's in this repo

| File | What it is |
|---|---|
| `dashboard-template-blank.html` | Empty template — placeholder channel names, empty data arrays. Start here. |
| `dashboard-example-filled.html` | Same template with **dummy data** for a fictional company ("Acme Roofing"), so you can see it working before you plug in your own numbers. |
| `skills/dashboard-builder.md` | A Claude "skill" file — paste this into a Claude project and Claude will walk you through building your own version, onboarding you on your channels, metrics, and motion type. |
| `LICENSE` | MIT |

## Quick start (manual)

1. Open `dashboard-example-filled.html` in a browser to see it working.
2. Copy `dashboard-template-blank.html` to start your own.
3. Find the `<script>` block near the bottom — replace the placeholder `CHANNELS`, `MONTHLY`, `WEEKLY`, `DAILY`, `CREATIVES`, `ADD`/`CUT` (optimization) arrays with your own data.
4. Update the KPI labels in the `PL_LABELS` / `PL_KEYS` arrays if your metrics differ (e.g. product-led teams: swap SAL/SQL/ICP for Signup/Activation/PQL — see the skill file for full definitions).
5. Open in a browser. That's it — no server needed.

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

What *does* vary per company: channel names, which KPIs populate the tables, and whether you're tracking a sales-led funnel (SAL/SQL/ICP/BTC/DPC) or a product-led one (Signup/Activation/PQL/Trial-to-Paid/Expansion). Both metric sets are documented in the skill file.

## Live actions (pause/resume/duplicate ads)

The Settings modal (gear icon, top right) accepts an API key and account ID for live actions via MCP-connected tools. **Never commit real API keys to this repo or any fork of it** — they're meant to be entered locally in your browser session, not stored in code.

## Contributing

This is a community template — PRs welcome for additional channel integrations, metric presets, or bug fixes. Please don't submit PRs containing real company data or credentials.

## License

MIT — see `LICENSE`.
