# Sheet template

Every build gets **its own Google Sheet**. Nothing here is shared between companies, and no sheet ID is committed to this repo — the dashboard is generated from whichever sheet you point the build at.

The examples in this repo have no sheet at all. Their data is seeded demo data that renders on open, so there is nothing to refresh and nothing to persist.

## What's here

Header rows for the two tabs a **human** types into. These have to match the board's data model exactly, so they are pinned here.

| File | Tab | Holds |
| --- | --- | --- |
| `test_log.csv` | `test_log` | The Changelog board — one row per card |
| `platform_log.csv` | `platform_log` | Platform changes, one row per entry |

The metric tabs (`monthly_<channel>`, `weekly_<channel>`, `daily_<channel>`, `creatives_<channel>_mtd` / `_l30`) are **written by the refresh job, not by hand**, so their columns are whatever that build's metric set needs — they follow the motion type and are derived from the dashboard's own data arrays at build time. Pinning them here would only create a second source of truth to drift.

## `test_log` columns → card fields

The header names map 1:1 onto the `NOTES` objects the board renders. Get these wrong and the rows import silently: they land in `NOTES`, the board filters them out, and nothing errors.

| Column | Field | Values |
| --- | --- | --- |
| `id` | `id` | Any unique value |
| `unit` | `unit` | Ad set / ad group name. Must match the creatives data's `ad_set` for stats to auto-populate |
| `ctx` | `ctx` | Short label shown before the unit on the card (the examples use the table key — `MM`, `GM`, `M1`) |
| `date` | `date` | Display string, e.g. `Aug 21` |
| `status` | `status` | **`todo` \| `doing` \| `done` \| `completed`** — the board's four columns (To Test, Testing, Evaluated, Completed). Not the column labels |
| `priority` | `priority` | `high` \| `medium` \| `low` |
| `note` | `note` | What prompted the test |
| `hyp` | `hyp` | The hypothesis |
| `variable` | `variable` | The one thing being changed |
| `test_setup` | `testSetup` | How it was run. Shown from Testing onward |
| `verdict` | `verdict` | **Free text**, shown verbatim. Only rendered once `status` is `done` or `completed` |
| `control_ad_id` | `controlAdId` | Must match an `ad_id` in the creatives data |
| `test_ad_id` | `testAdId` | Same |
| `archived` | `archived` | `TRUE` or blank |

`platform_log` maps to `PLATFORM_LOG`: `channel` is the channel key (`c1`, `c2`, `c3`), and `kind` is one of `budget`, `status`, `creative`, `targeting`, `bid` — it picks the entry's icon and tint.

## Rules that matter

- **`ad_id` is required on every creative row.** It is the join between a Changelog card's control/test ads and their live numbers. Without it the Testing card has nothing to read.
- **`test_log` is yours to edit.** It is the one tab where a human writes rather than a job. The refresh job must never overwrite rows it did not create.
- **`archived` is a flag, not a delete.** "Archive Done" sets it; the row stays.
- **Validate `status` and `priority` on import.** A blank or misspelled `status` makes the card vanish from the board while still sitting in the data — no error, no empty state, nothing. Reject the row loudly instead.
- **Never put credentials in the sheet.** API keys and account IDs belong in the refresh job's environment.
