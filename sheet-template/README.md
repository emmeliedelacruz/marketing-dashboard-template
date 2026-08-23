# Sheet template

Every build gets **its own Google Sheet**. Nothing here is shared between companies, and no sheet ID is baked into this repo — the dashboard HTML is generated from whichever sheet you point the build at.

## Creating yours

Ask Claude to set it up ("create the dashboard sheet for <company>, <motion type>, channels <a/b/c>"), or do it by hand:

1. Create a Google Sheet.
2. Add one tab per row of the table below, per channel where noted.
3. Paste the matching CSV from this folder into row 1 of each tab, then append the motion columns.
4. Give the refresh job write access — a service account, or an Apps Script web app deployed on the sheet. (The Google Drive connector can *create* a sheet but cannot update cells in one, so it cannot be your refresh path.)

## Tabs

| Tab | One per | Grain | Base columns |
| --- | --- | --- | --- |
| `monthly_<channel>` | channel | Ad set / ad group, MTD | `monthly.csv` |
| `weekly_<channel>` | channel | Ad set / ad group, last 7 days | `weekly.csv` |
| `daily_<channel>` | channel | Campaign, last completed day | `daily.csv` |
| `creatives_<channel>_mtd` | channel | Ad, MTD | `creatives.csv` |
| `creatives_<channel>_l30` | channel | Ad, last 30 days | `creatives.csv` |
| `test_log` | sheet | Changelog card | `test_log.csv` |
| `platform_log` | sheet | Platform change | `platform_log.csv` |

The CSVs carry only the columns every motion shares. **Append your motion's metric columns** to `monthly`, `weekly`, `daily` and `creatives`:

| Motion | Append |
| --- | --- |
| Sales-led | `submits,sal,sql,icp,btc,dpc,stuck,crm_sync_pct,submit_pct,ev,net_ev,ros,cost_per_sal,cost_per_sold` |
| Ecommerce | `sessions,atc,checkouts,orders,revenue,aov,roas,repeat_rate,refund_rate,feed_health_pct,session_cvr_pct,cost_per_atc,cost_per_order,profit_per_spend,net_profit` |
| Product-led | `signups,activations,time_to_activation,pqls,trial_to_paid_pct,time_to_paid,expansion_mrr,nrr,feature_adoption_pct,cac_payback` |

Not every metric belongs on every grain — `crm_sync_pct` and `feed_health_pct` are Daily-only by design. Drop the columns a tab does not use rather than filling them with zeros; a zero reads as measured.

## Rules that matter

- **`ad_id` is required on every creative row.** It is the join between a Changelog card's control/test ads and their live numbers. Without it the Testing card has nothing to read.
- **`test_log` is yours to edit.** It is the one tab where a human writes rather than a job — hypotheses, verdicts, setup notes. The refresh job must never overwrite rows it did not create.
- **`archived` is a flag, not a delete.** "Archive Done" sets it; the row stays.
- **`column` is one of** `totest`, `testing`, `evaluated`, `completed`.
- **`verdict` is one of** `win`, `loss`, `flat`, or empty while a test is still running.
- **`kind` on `platform_log` is one of** `budget`, `status`, `creative`, `targeting`, `bid` — it picks the icon and tint.
- **Never put credentials in the sheet.** API keys and account IDs belong in the refresh job's environment.
