---
name: weekly-store-kpi-report
owner: shopmetric
category: Analytics & tracking
description: Writes a one-page Monday KPI report for store owners: backend revenue, orders, AOV, conversion, MER and refunds versus last week and last year, with noise tests and at most two decisions.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/shopmetric/weekly-store-kpi-report
raw: https://ecomdly.com/raw/shopmetric/weekly-store-kpi-report.md
install: npx @ecomdly/cli add shopmetric/weekly-store-kpi-report
---

# Weekly store KPI report

Writes a one-page Monday report for an online store: about ten numbers for the last full week, each with its source, compared with the previous week and the same week last year. It separates real movement from noise and names at most two decisions. The naive weekly report fails in three ways. It mixes revenue definitions across tools, it treats every ±5% wiggle as news, and it sums platform ROAS until the report claims more revenue than the shop took. This skill fixes all three. Revenue comes only from the backend, efficiency is judged on MER, and a change is called a trend only when it clears a stated noise test.

## When to use
- Every Monday, for the previous ISO week (Monday–Sunday).
- The week after a promotion, a price change, a site release or a big budget shift, with that event marked.
- When an owner asks "how did last week go?" and needs one page, not a dashboard.

## When not to use
- The GA4 and backend numbers disagree by more than usual and nobody knows why. Run `revenue-discrepancy-reconciler` first.
- Diagnosing where the checkout loses people. Use `ga4-funnel-analyst`.
- Deciding whether a test won. Use `ab-test-readout`. A weekly report cannot attribute a change to a variant.
- Monthly or quarterly board reporting with cohorts, contribution margin and forecasting. That is a different product. This page stays weekly and operational.

## Inputs
Required, for three windows (this week, previous week, same ISO week last year):
1. **Backend orders**, one row per order: `order_id`, `created_at` (shop time zone), `status`, `revenue_ex_vat` (merchandise after discounts; shipping as its own column), `customer_id` or hashed e-mail, `is_first_order` (or enough history to derive it), `channel` (web, marketplace, phone). Source: order export from Shoptet, Upgates, Shopify, WooCommerce or the ERP.
2. **Backend refunds**: `refund_id`, `order_id`, `refund_date`, `refund_amount_ex_vat`.
3. **GA4 sessions** per day, from Reports > Acquisition > Traffic acquisition or from BigQuery (`COUNT(DISTINCT CONCAT(user_pseudo_id, ga_session_id))`), in the GA4 property time zone.
4. **Ad spend** per platform per day (Google Ads, Meta, Microsoft, Heureka/Zboží.cz CPC, others), in the reporting currency, ex VAT where the invoice charges VAT.

Optional: platform-reported conversion value per platform, top-seller stock-outs (SKU, days out of stock), promotion and holiday calendar, site release log, e-mail campaign sends.

If an input is missing, the cell is blank with the reason ("Meta export not provided"). Never fill it with an estimate or last week's value. If backend orders are missing, the report cannot be produced. Say so and stop, because revenue is never taken from GA4 or ad platforms.

## Best practices

### Definitions (write them once, never change them silently)
1. **Net revenue** = Σ `revenue_ex_vat` of orders created in the week, excluding cancelled and test orders, and excluding shipping unless the owner has decided otherwise. The source is always the backend. It is the only revenue figure that includes every channel and is unaffected by consent and ad blockers.
2. **Orders** = count of those orders. **AOV** = net revenue ÷ orders.
3. **Conversion rate** = backend web orders ÷ GA4 sessions, labelled "backend orders / GA4 sessions". This mixes sources on purpose. The numerator is complete, while the denominator is missing non-consented sessions, which inflates the rate. The level is not comparable with other stores, but the week-over-week change is usable if consent rates are stable. When the CMP or consent mode setup changed between the compared weeks, write "not comparable" instead of a delta.
4. **Ad spend** = Σ spend across all paid platforms. **MER** (marketing efficiency ratio) = net revenue ÷ total ad spend. MER is the efficiency number for the owner because it cannot double-count. **Platform ROAS** = platform conversion value ÷ platform spend, shown per platform for context and never summed or averaged. Each platform claims the same orders under its own attribution window, so their conversion values add up to more than the shop sold.
5. **New-customer share** = orders where `is_first_order` ÷ orders. Say how first orders are identified (customer account vs e-mail). Guest checkouts with new e-mails inflate "new".
6. **Refunds** are cash-basis: refunds processed in the week, whatever the order date. Show the amount and the ratio refunds ÷ net revenue of the same week, and label it as a cash-basis ratio. It is not the true refund rate of this week's orders, which is only known weeks later. If the owner wants the cohort rate, report it monthly for orders at least as old as the return window.
7. **Top 5 products** by net revenue, with units and any days out of stock. A best-seller that was out of stock explains a revenue dip better than any channel metric.

### Comparing weeks
8. **Use ISO weeks**: Monday–Sunday, and week 1 is the week containing the first Thursday of the year. Always write the dates next to the week number, because week numbering near New Year surprises people.
9. **Year-over-year compares the same ISO week number**, not the same calendar dates, so both windows start on Monday. Skip YoY, and say why, when either week contains a promotion, a holiday (for example Czech public holidays, Black Friday week, Christmas shipping cut-off) or a tracking change that the other does not. A misleading YoY is worse than an empty cell.
10. **Time zones**: cut backend orders in the shop time zone and GA4 in the property time zone. If they differ, say so in the footnote.

### Noise vs signal (decision rules, not a fixed percentage)
11. **Order counts**: treat weekly orders as roughly Poisson. A week-over-week change is noise when |orders₁ − orders₀| < 2 × √(orders₁ + orders₀). The same 5% move is noise at 100 orders a week and real at 5,000. A fixed "±5%" rule ignores scale.
12. **Conversion rate**: two-proportion z-test, z = (p₁ − p₀) ÷ √(p̄(1 − p̄)(1/n₁ + 1/n₀)), where p = orders ÷ sessions, n = sessions and p̄ is pooled. Call it movement only if |z| ≥ 2. Sessions are not independent trials, so treat this as a screen, not proof.
13. **Revenue** moves with orders × AOV. Report which of the two drove the change. AOV is sensitive to single large orders, so if one order is more than a few percent of the week's revenue, name it.
14. **Three weeks in the same direction** can be called a trend even when each week alone is noise. State it as such.

### What goes on the page
15. At most two items under "Needs a decision". Each names the metric, the evidence, the decision and who owns it. If nothing clears the noise test and no guardrail broke, write "No action needed". A report that always asks for action trains people to ignore it.
16. Context lines (promotion, stock-out, release, tracking change) come before interpretation. Most "anomalies" are calendar.

## Process
1. Determine the ISO week and the dates of the previous week and the same ISO week last year. Check the promotion, holiday and release calendar for all three.
2. Pull the inputs. For each missing input, record the reason and leave its cells blank.
3. Compute the metrics with the definitions above, for all three windows.
4. Compute the deltas: relative % for revenue, orders, AOV, spend and sessions; percentage points (pt) for rates; absolute difference for MER.
5. Apply the noise tests to orders and conversion rate, and label each delta "noise", "movement" or "trend (3 weeks)".
6. Decide on YoY comparability (best practice 9). If a window is not comparable, replace its cells with "n/a (<reason>)".
7. Check the decision triggers, in this order:
   - MER down with a movement-level change while spend rose → budget question.
   - Spend on one platform up sharply with no change in new-customer share or MER → budget question for that platform.
   - Top seller out of stock ≥ 2 days → stock or ads-pausing question.
   - Cash-basis refund ratio up for 3 weeks → returns question (hand-off: `returns-reason-analyzer`).
   - Conversion rate movement after a release → funnel question (hand-off: `ga4-funnel-analyst`).
   Keep the two with the largest revenue at stake.
8. Write the page and run the quality checklist.

## Pitfalls and edge cases
- **Late orders**: bank-transfer and cash-on-delivery orders can change status after Monday morning. Pull on a fixed schedule and note the pull time. Do not restate last week silently. If it changed, footnote the revision.
- **Marketplace orders** (Heureka Marketplace, Allegro, Amazon) are in backend revenue but have no GA4 session. Either exclude them from the conversion-rate numerator or report web-only CR, and say which.
- **VAT changes or mixed rates**: always use per-order ex-VAT amounts from the backend, never gross ÷ one rate.
- **Currency**: multi-currency stores convert at one stated rate per week, not per tool.
- **Ad spend with invoices incl. VAT** (some local CPC platforms): convert to ex VAT to match revenue.
- **Consent or tag changes** break GA4 comparability for sessions and CR from that date. Mark the week and show "not comparable" for 1 week and for 52 weeks after it.
- **A single B2B order** can move AOV and revenue by double digits on a small shop. Show the result with and without it.
- **Platform ROAS moving opposite to MER** is common, for example when a platform's attribution window captures organic demand. Report both and interpret neither as proof alone.
- **Short weeks** (store closed, site down) are still reported. Note the reason, and do not compare them as normal weeks.

## Rules
- Read-only. No changes to budgets, campaigns, prices or stock. Decisions are proposed to the owner.
- Revenue only from the backend. Every number names its source in the table or the footnotes.
- Never estimate a missing value. Leave the cell blank with the reason.
- Never sum or average platform ROAS or platform-reported revenue across platforms.
- Do not call a change a trend unless it passes the noise test or has persisted for three weeks.
- No customer-level data (names, e-mails) in the report.

## Output format
```
## Week <NN> (<Mon d> – <Sun d> <yyyy>) · <shop> · <currency>, ex VAT
Context: <promotions / holidays / releases / stock-outs, or "none">

| metric | this week | vs prev week | vs same week LY | source |
|---|---|---|---|---|
| Net revenue | x | +x% | +x% | backend |
| Orders | n | +x% (<noise/movement>) | +x% | backend |
| AOV | x | +x% | +x% | backend |
| Sessions | n | +x% | | GA4 |
| Conversion rate | x.xx% | +x.xx pt (<noise/movement>) | | backend orders / GA4 sessions |
| New-customer share | x% | +x pt | | backend (<identification>) |
| Ad spend | x | +x% | +x% | platforms |
| MER | x.xx | ±x.xx | ±x.xx | backend / platforms |
| Refunds processed | x (x.x% of net rev.) | +x pt | | backend, cash basis |

Platform ROAS (context, not additive): Google x.xx · Meta x.xx · ...
Top 5 products: <name — revenue — units — stock note> ...

Needs a decision:
1. <metric + evidence + decision + owner>
(or: No action needed.)

Footnotes: <time zones, pull time, missing inputs, comparability notes>
```

## Worked example
Illustrative numbers. ISO week 38 of 2026 runs from Monday 14 to Sunday 20 September 2026. The same ISO week in 2025 was 15–21 September 2025. There were no promotions or holidays in any of the three weeks. A consent banner change on 2 March 2026 makes YoY sessions and CR not comparable. Amounts are in CZK, ex VAT.

Inputs: this week has 612 orders, 1,224,000 net revenue and 26,500 sessions. The previous week had 589 orders, 1,153,000 net revenue and 26,900 sessions. Last year had 560 orders and 1,072,700 net revenue. Spend this week was Google Ads 148,000 and Meta 98,000, for a total of 246,000. The previous week was 146,800 and 74,800, for a total of 221,600. Last year's total was 207,600. New customers were 233 of 612 this week, against 224 of 589 last week. Refunds processed were 88,100 this week and 78,400 last week.

Noise tests: the orders change of +23 is below 2 × √(612 + 589) = 69.3, so it is noise. For conversion rate, 2.31% vs 2.19% gives z = 0.93, so it is also noise.

```
## Week 38 (14 – 20 Sep 2026) · CZK, ex VAT
Context: none

| metric | this week | vs prev week | vs same week LY | source |
|---|---|---|---|---|
| Net revenue | 1 224 000 | +6.2% | +14.1% | backend |
| Orders | 612 | +3.9% (noise) | +9.3% | backend |
| AOV | 2 000 | +2.2% | +4.4% | backend |
| Sessions | 26 500 | −1.5% | n/a (consent change 2 Mar 2026) | GA4 |
| Conversion rate | 2.31% | +0.12 pt (noise) | n/a (consent change) | backend orders / GA4 sessions |
| New-customer share | 38.1% | +0.0 pt | | backend (customer account + e-mail) |
| Ad spend | 246 000 | +11.0% | +18.5% | platforms |
| MER | 4.98 | −0.22 | −0.19 | backend / platforms |
| Refunds processed | 88 100 (7.2% of net rev.) | +0.4 pt | | backend, cash basis |

Needs a decision:
1. Meta spend +31.0% (74 800 → 98 000) while new-customer share stayed flat (38.0% → 38.1%) and MER fell 5.20 → 4.98. Decide whether to return Meta budget to the previous level. Owner: marketing.
```
Revenue rose because of both more orders and a higher AOV, but neither change clears the noise test on its own, so the page does not call it growth.

## Quality checklist
- The ISO week number and dates are correct. The LY window is the same ISO week and starts on a Monday.
- Every row has a source. Revenue comes from the backend only.
- The deltas were recomputed from the raw numbers (AOV = revenue ÷ orders, MER = revenue ÷ spend), and rates use pt.
- The noise test was applied and labelled for orders and conversion rate.
- YoY and sessions comparability was checked against the calendar and the tracking-change log.
- Platform ROAS is not summed anywhere.
- Missing inputs appear as blank cells with a reason. There are at most two decision items, or "No action needed".

## Sources
- ISO 8601 week date (ISO week definition): https://www.iso.org/iso-8601-date-and-time-format.html
- [GA4] About Analytics sessions: https://support.google.com/analytics/answer/9191807
- [GA4] Behavioral modeling for consent mode (why GA4 sessions miss non-consented traffic): https://support.google.com/analytics/answer/11161109
- [GA4] Select attribution settings (platform-specific attribution): https://support.google.com/analytics/answer/10597962
- Google Ads, About conversion windows: https://support.google.com/google-ads/answer/3123169

## License
MIT
